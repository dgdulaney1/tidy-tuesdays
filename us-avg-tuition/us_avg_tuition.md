Analyzing Tidy Tuesday US Avg Tuition Dataset
================

Load packages, read in data

``` r
library(tidyverse)
library(here)
library(scales)
library(readxl)
library(mapdata)
library(maptools)
library(kableExtra)
```

``` r
theme_set(theme_light())
```

Read in data, change from wide to long.

``` r
tuition <- readxl::read_xlsx(here("us-avg-tuition", "us_avg_tuition.xlsx")) %>% 
  rename(state = State) %>% 
  pivot_longer(cols = -state, names_to = "year", values_to = "avg_tuition")
```

-----

How does the average tuition look for each state over the entire period
of time?

``` r
tuition_all_years <- tuition %>% 
  group_by(state) %>% 
  summarise(avg_tuition = mean(avg_tuition))
```

``` r
tuition_all_years %>% 
  mutate(state = fct_reorder(state, avg_tuition)) %>% 
  ggplot(aes(state, avg_tuition)) +
  geom_col(aes(fill = state)) +
  scale_y_continuous(breaks = c(2500, 5000, 7500, 10000, 12500), labels = dollar_format()) +
  coord_flip() +
  labs(title = "Average tuition from 2004-2016",
       x = "State",
       y = "Average tuition") +
  theme(legend.position = "none")
```

![](us_avg_tuition_files/figure-gfm/tuition_all_years-1.png)<!-- -->

-----

How does the average tuition look in different regions of the country?

``` r
us_map <- map_data("state")
```

``` r
us_map %>% 
  mutate(region = str_to_title(region)) %>% 
  left_join(tuition_all_years, by = c("region" = "state")) %>% 
  ggplot(aes(long, lat)) +
  geom_polygon(aes(group = group, fill = avg_tuition), color = "white") +
  scale_fill_gradient2(low = "blue", mid = "lightgrey", high = "red", midpoint = 7500, name = "Average tuition") +
  theme(legend.position = "none",
        legend.title = element_text("Average tuition")) +
  theme_void()
```

![](us_avg_tuition_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

As the bar chart suggested, the highest tuition schools are in the
northeast and the lowest tuition schools are out west and in the south.

-----

Which states have seen the largest change in tuition from 2004-2015?

``` r
tuition_04_15 <- tuition %>% 
  filter(year %in% c("2004-05", "2015-16")) %>% 
  pivot_wider(names_from = year, values_from = avg_tuition) %>% 
  rename(tuition_04 = `2004-05`,
         tuition_15 = `2015-16`) %>% 
  mutate(pct_change = (tuition_15 - tuition_04) / (tuition_04)) %>% 
  arrange(desc(pct_change)) %>% 
  head(10) %>% 
  kable() %>% 
  kable_styling()
```
