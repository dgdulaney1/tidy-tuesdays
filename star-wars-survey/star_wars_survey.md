Analyzing Tidy Tuesday Star Wars Survey Dataset
================

  - [Setup](#setup)
  - [Character-specific questions](#character-specific-questions)
      - [Who are the most and least popular
        characters?](#who-are-the-most-and-least-popular-characters)
      - [How does character favorability differ by age and
        gender?](#how-does-character-favorability-differ-by-age-and-gender)
      - [Which characters do the Expanded Universe fans
        like?](#which-characters-do-the-expanded-universe-fans-like)
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

    ## -- Attaching packages ------------------------------------------------------------------------------------------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 3.2.1     v purrr   0.3.3
    ## v tibble  2.1.3     v dplyr   0.8.3
    ## v tidyr   1.0.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.4.0

    ## -- Conflicts ---------------------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
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
    age = Age,
    income = `Household Income`,
    education = Education,
    region = `Location (Census Region)`
    ) %>% 
  slice(2:nrow(.)) 
```

``` r
film_questions <- star_wars_survey %>% 
  select(-contains("fav"))
```

``` r
character_questions <- star_wars_survey %>% 
  select(-contains("ep")) %>% 
  pivot_longer(
    cols = han_fav:yoda_fav,
    names_to = "character",
    values_to = "favorability"
    ) %>% 
  mutate(character = str_remove(character, "_fav"))
```

# Character-specific questions

## Who are the most and least popular characters?

## How does character favorability differ by age and gender?

## Which characters do the Expanded Universe fans like?

## Do those who like Anakin also like Vader? Or is there some divide there?

# Film-specific questions

## What are the most common “films seen” histories? (i.e. all 6, all but \_\_, only one, etc.)

## What is the distribution of film rank for each movie?

## How do film rankings look across age groups?

# Are there any interesting character-film relationships? For ex: Do those who like Anakin like prequels more, and Luke sequels more?

# Are there trends among those who aren’t fans?
