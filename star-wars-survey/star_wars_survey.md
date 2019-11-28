Analyzing Tidy Tuesday Star Wars Survey Dataset
================

  - [Setup](#setup)
  - [Character-specific questions](#character-specific-questions)
      - [Who are the most and least popular
        characters?](#who-are-the-most-and-least-popular-characters)
      - [Which characters see the largest favorability differences with
        different age
        groups?](#which-characters-see-the-largest-favorability-differences-with-different-age-groups)
      - [Do fans familiar with the Expanded Universe feel differently
        about any
        characters?](#do-fans-familiar-with-the-expanded-universe-feel-differently-about-any-characters)
      - [Do those who like Anakin also like Vader? Or is there some
        divide
        there?](#do-those-who-like-anakin-also-like-vader-or-is-there-some-divide-there)
  - [Film-specific questions](#film-specific-questions)
      - [What are the most common “films seen” histories? (i.e. all 6,
        all but \_\_, only one,
        etc.)](#what-are-the-most-common-films-seen-histories-i.e.-all-6-all-but-__-only-one-etc.)
      - [What is the distribution of film rank for each
        movie?](#what-is-the-distribution-of-film-rank-for-each-movie)
      - [How do film rankings look across age
        groups?](#how-do-film-rankings-look-across-age-groups)
  - [Are there any interesting character-film relationships? For ex: Do
    those who like Anakin like prequels more, and Luke sequels
    more?](#are-there-any-interesting-character-film-relationships-for-ex-do-those-who-like-anakin-like-prequels-more-and-luke-sequels-more)
  - [Are there trends among those who aren’t
    fans?](#are-there-trends-among-those-who-arent-fans)

# Setup

``` r
library(tidyverse)
```

    ## -- Attaching packages ----------------------------------------------------------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 3.2.1     v purrr   0.3.3
    ## v tibble  2.1.3     v dplyr   0.8.3
    ## v tidyr   1.0.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.4.0

    ## -- Conflicts -------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
theme_set(theme_light())

knitr::opts_chunk$set(
  cache = TRUE,
  message = FALSE, 
  warning = FALSE, 
  fig.width = 10, 
  fig.height = 7
  )
```

``` r
star_wars_survey_raw <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018/2018-05-14/week7_starwars.csv")

star_wars_survey <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018/2018-05-14/week7_starwars.csv") %>%
  select(-32) %>% 
  rename(
    seen_any_star_wars = `Have you seen any of the 6 films in the Star Wars franchise?`,
    fan_of_star_wars = `Do you consider yourself to be a fan of the Star Wars film franchise?`,
    seen_ep_I = `Which of the following Star Wars films have you seen? Please select all that apply.`,
    seen_ep_II = X5,
    seen_ep_III = X6,
    seen_ep_IV = X7,
    seen_ep_V = X8,
    seen_ep_VI = X9,
    ep_I_rank = `Please rank the Star Wars films in order of preference with 1 being your favorite film in the franchise and 6 being your least favorite film.`,
    ep_II_rank = X11,
    ep_III_rank = X12,
    ep_IV_rank = X13,
    ep_V_rank = X14,
    ep_VI_rank = X15,
    han_fav = `Please state whether you view the following characters favorably, unfavorably, or are unfamiliar with him/her.`,
    luke_fav = X17,
    leia_fav = X18,
    anakin_fav = X19,
    obi_won_fav = X20,
    palpy_fav = X21,
    vader_fav = X22,
    lando_fav = X23,
    boba_fett_fav = X24,
    c3po_fav = X25,
    r2d2_fav = X26,
    jar_jar_fav = X27,
    padme_fav = X28,
    yoda_fav = X29,
    which_char_shot_first = `Which character shot first?`,
    familiar_with_expanded = `Are you familiar with the Expanded Universe?`,
    #fan_of_expanded = `Do you consider yourself to be a fan of the Expanded Universe?\u008c\xe6`,
    star_trek_fan = `Do you consider yourself to be a fan of the Star Trek franchise?`,
    gender = Gender,
    age_group = Age,
    income = `Household Income`,
    education = Education,
    region = `Location (Census Region)`
    ) %>% 
  slice(2:nrow(.)) %>% 
  mutate(age_group = fct_relevel(age_group, c(NA, "18-29", "30-44", "45-60", "> 60")))

star_wars_survey %>% head(5)
```

    ## # A tibble: 5 x 37
    ##   RespondentID seen_any_star_w~ fan_of_star_wars seen_ep_I seen_ep_II
    ##          <dbl> <chr>            <chr>            <chr>     <chr>     
    ## 1   3292879998 Yes              Yes              Star War~ Star Wars~
    ## 2   3292879538 No               <NA>             <NA>      <NA>      
    ## 3   3292765271 Yes              No               Star War~ Star Wars~
    ## 4   3292763116 Yes              Yes              Star War~ Star Wars~
    ## 5   3292731220 Yes              Yes              Star War~ Star Wars~
    ## # ... with 32 more variables: seen_ep_III <chr>, seen_ep_IV <chr>,
    ## #   seen_ep_V <chr>, seen_ep_VI <chr>, ep_I_rank <chr>, ep_II_rank <chr>,
    ## #   ep_III_rank <chr>, ep_IV_rank <chr>, ep_V_rank <chr>, ep_VI_rank <chr>,
    ## #   han_fav <chr>, luke_fav <chr>, leia_fav <chr>, anakin_fav <chr>,
    ## #   obi_won_fav <chr>, palpy_fav <chr>, vader_fav <chr>, lando_fav <chr>,
    ## #   boba_fett_fav <chr>, c3po_fav <chr>, r2d2_fav <chr>, jar_jar_fav <chr>,
    ## #   padme_fav <chr>, yoda_fav <chr>, which_char_shot_first <chr>,
    ## #   familiar_with_expanded <chr>, star_trek_fan <chr>, gender <chr>,
    ## #   age_group <fct>, income <chr>, education <chr>, region <chr>

``` r
character_questions <- star_wars_survey %>%  
  select(-contains("ep")) %>% 
  pivot_longer(
    cols = contains("fav"),
    names_to = "character", 
    values_to = "favorability"
    ) %>%
  drop_na(favorability) %>% 
  mutate(favorability = ifelse(favorability == "Neither favorably nor unfavorably (neutral)", "Neutral", favorability)) %>% 
  mutate(
    character = str_remove(character, "_fav"),
    favorability = fct_relevel(favorability, c("Unfamiliar (N/A)", 
                                               "Very unfavorably", 
                                               "Somewhat unfavorably", 
                                               "Neutral", 
                                               "Somewhat favorably",
                                               "Very favorably")
                               ),
    favorability_score = case_when(favorability == "Unfamiliar (N/A)" ~ 0,
                                   favorability == "Very unfavorably" ~ -3,
                                   favorability == "Somewhat unfavorably" ~ -2,
                                   favorability == "Neutral" ~ -1,
                                   favorability == "Somewhat favorably" ~ 2,
                                   favorability == "Very favorably" ~ 4,
                                   TRUE ~ -9999
                                   )
    )

character_questions %>% head(5)
```

    ## # A tibble: 5 x 14
    ##   RespondentID seen_any_star_w~ fan_of_star_wars which_char_shot~
    ##          <dbl> <chr>            <chr>            <chr>           
    ## 1   3292879998 Yes              Yes              I don't underst~
    ## 2   3292879998 Yes              Yes              I don't underst~
    ## 3   3292879998 Yes              Yes              I don't underst~
    ## 4   3292879998 Yes              Yes              I don't underst~
    ## 5   3292879998 Yes              Yes              I don't underst~
    ## # ... with 10 more variables: familiar_with_expanded <chr>,
    ## #   star_trek_fan <chr>, gender <chr>, age_group <fct>, income <chr>,
    ## #   education <chr>, region <chr>, character <chr>, favorability <fct>,
    ## #   favorability_score <dbl>

-----

# Character-specific questions

## Who are the most and least popular characters?

``` r
character_questions %>% 
  mutate(character = fct_reorder(character, favorability_score, .desc = TRUE)) %>% 
  ggplot(aes(favorability, fill = favorability)) +
  geom_bar() +
  facet_wrap(~ character) +
  scale_fill_manual(values = c("black", "red", "red4", "lightgrey", "green4", "green")) +
  coord_flip() +
  labs(
    title = "Favorability of Star Wars Characters",
    x = "Favorability",
    y = "Count"
    ) +
  theme(legend.position = "none")
```

![](star_wars_survey_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

This is a great demonstration of what vague survey questions can lead to
:)

People love Vader but gave him bad “favorability” because he’s an
asshole, whereas the good guys from the originals all grade out well.
The “good guys” from the prequels (Padme and Anakin) have middling
reviews as well, probably because so many people hate those movies.

-----

## Which characters see the largest favorability differences with different age groups?

``` r
fav_by_age_group <- character_questions %>% 
  group_by(character, age_group) %>% 
  summarise(n = n(), fav_score = mean(favorability_score, na.rm = TRUE)) %>% 
  drop_na()

fav_by_age_group %>% head(10)
```

    ## # A tibble: 10 x 4
    ## # Groups:   character [3]
    ##    character age_group     n fav_score
    ##    <chr>     <fct>     <int>     <dbl>
    ##  1 anakin    18-29       178    1.07  
    ##  2 anakin    30-44       205    0.829 
    ##  3 anakin    45-60       239    1.62  
    ##  4 anakin    > 60        185    1.72  
    ##  5 boba_fett 18-29       177    0.768 
    ##  6 boba_fett 30-44       204    0.779 
    ##  7 boba_fett 45-60       233   -0.0815
    ##  8 boba_fett > 60        183   -0.0383
    ##  9 c3po      18-29       179    2.17  
    ## 10 c3po      30-44       204    2.60

``` r
fav_by_age_group %>%
  ggplot(aes(age_group, fav_score, fill = age_group)) +
  geom_col() +
  facet_wrap(~ character) +
  coord_flip()
```

![](star_wars_survey_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

-----

## Do fans familiar with the Expanded Universe feel differently about any characters?

``` r
character_questions %>% 
  group_by(character, familiar_with_expanded) %>% 
  summarise(fav_score = mean(favorability_score)) %>% 
  drop_na() %>% 
  pivot_wider(id_cols = "character", names_from = "familiar_with_expanded", values_from = "fav_score") %>% 
  mutate(expanded_fav_diff = Yes - No) %>% 
  arrange(expanded_fav_diff) %>% 
  View()
```

## Do those who like Anakin also like Vader? Or is there some divide there?

# Film-specific questions

## What are the most common “films seen” histories? (i.e. all 6, all but \_\_, only one, etc.)

## What is the distribution of film rank for each movie?

## How do film rankings look across age groups?

# Are there any interesting character-film relationships? For ex: Do those who like Anakin like prequels more, and Luke sequels more?

# Are there trends among those who aren’t fans?
