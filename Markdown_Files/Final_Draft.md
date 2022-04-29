---
title: 'US Accidents (2016-2021) or: What the hell is going on in California?'
author: "Seth Buesing, Jason Whitelaw, Kai Yamanishi"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---




```r
library(tidyverse)
library(lubridate)
library(openintro)
library(maps)
library(ggmap)
library(gplots)
library(RColorBrewer)
library(sf)
library(leaflet)
library(ggthemes)
library(gganimate)
library(gifski)
library(transformr)
library(osmdata)
library(reticulate)
library(knitr)
theme_set(theme_minimal())
```


```r
accidents_data <- read_csv("US_Accidents_Dec21_updated.csv")
states_map <- map_data("state")
```


```r
accidents_clean <- accidents_data %>%
  mutate(state = state.name[match(State, state.abb)]) %>%
  mutate(state = str_to_lower(state)) %>%
  filter(is.na(state)==FALSE) %>%
  mutate(year = substring(Start_Time, 0, 4))
```


```r
accidents_ca <- accidents_clean %>%
  filter(State == "CA")
```

#Introduction
  In our search for an interesting dataset to analyze we came across a wonderfully organized table detailing traffic accidents in the contiguous United States over the last 5 years. What caught our interest in specific was looking at the evolution over time of crash statistics in specific states- most notably California. As the graph below shows, the number of accidents in California has been doubling each year, and we wanted to figure out just what was happening. The dataset we're working with is extraordinarily large, containing over 2.8 million accidents taken from automated traffic sensors and logged reports. Each accident has 47 columns of information.


```r
  accidents_by_state <- accidents_clean %>%
  group_by(state, year) %>%
  mutate(num_accidents = n()) %>%
  select(state, year, num_accidents) %>%
  unique() %>%
  arrange(year) %>%
  ungroup() %>%
  add_row(year = "2017", state = "north dakota", num_accidents = 0) %>%
  ggplot() +
  geom_map(map = states_map, aes(map_id = state, fill = num_accidents, group = year)) +
  expand_limits(x = states_map$long, y = states_map$lat) + 
  theme_map() +
  labs(title = "Number of Car Accidents by State, 2016-2021", subtitle = "Year: {closest_state}", fill = "Number of Accidents") +
  transition_states(year)
animate(accidents_by_state, fps = 15)
```


```r
anim_save("accidents.gif")
```

```
## Error: The animation object does not specify a save_animation method
```

```r
knitr::include_graphics("accidents.gif")
```

![](accidents.gif)<!-- -->



#Data Collection
  Our Data Collection process was pretty simple- we found the data set on [https://www.kaggle.com/datasets/sobhanmoosavi/us-accidents]() and downloaded the csv file. We did run into some issues using this csv, however due to its sheer size (1.1 GB). The fun thing about this data set is that it contains so many details about each crash, which allows us to look for correlations without them being spoon-fed to us. Additionally, the types of data provided allowed us to explore quite a few different types of visualizations: the time was given for each crash, as well as location plus details on the cause of the crash.

#Analysis


```r
CA_map <- map_data("county", region = "california") %>%
  rename(state = region, region = subregion) %>%
  mutate(region = str_to_title(region))
```


```r
accidents_clean %>%
  filter(State == "CA") %>%
  group_by(County) %>%
  summarise(total_accidents = n()) %>%
  ungroup() %>%
  ggplot() +
  geom_map(map = CA_map, aes(map_id = County, fill = total_accidents)) +
  expand_limits(x = CA_map$long, y = CA_map$lat) +
  labs(x = "", y = "", fill = "Total Accidents", title = "Accidents by County 2016-2021") +
  theme_map()
```

![](Final_Draft_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

<img src="Final_Draft_files/figure-html/unnamed-chunk-8-1.png" title="Total car accidents in each Califronia county between 2016 and 2021." alt="Total car accidents in each Califronia county between 2016 and 2021."  />

So, what would cause this pattern? The dataset we found had columns for factors such as whether the accident was in the day or the night, what the temperature and weather were like, and the type of accident. Time of day is a big factor in how often accidents occur, even though that effect is mitigated by the fact that fewer people are driving later in the day. We decided to look at the proportion of day to night accidents, still not expecting too much of a correlation.


```r
accidents_ca %>%
  filter(is.na(Sunrise_Sunset)==FALSE) %>%
  group_by(year) %>%
  mutate(num_accidents = n()) %>%
  ungroup() %>%
  group_by(year, Sunrise_Sunset) %>%
  mutate(percent_day_night = n()/num_accidents) %>%
  ungroup() %>%
  select(percent_day_night, year, Sunrise_Sunset) %>%
  unique() %>%
  ggplot() +
  geom_col(aes(x = year, y = percent_day_night, fill = Sunrise_Sunset)) +
  labs(title = "Percent of Day/Night Accidents in California by Year", fill = "Time of Day", y = "Year")
```

![](Final_Draft_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Oddly enough, we do see an increase in the proportion of night accidents over the last 3 years, though with a dip back down in 2021. Still no sign of what might be contributing to the exponential growth.


```r
accidents_ca %>%
  filter(is.na(Weather_Condition)==FALSE) %>%
  group_by(year) %>%
  mutate(num_accidents = n()) %>%
  ungroup() %>%
  group_by(year, Weather_Condition) %>%
  mutate(percent_weather = n()/num_accidents) %>%
  filter(percent_weather>.01) %>%
  ungroup() %>%
  select(percent_weather, year, Weather_Condition) %>%
  unique() %>%
  group_by(year) %>%
  mutate(other = 1-sum(percent_weather)) %>%
  ungroup() %>%
  add_row(year = "2016", Weather_Condition = "Other", percent_weather = 0.02103897) %>%
  add_row(year = "2017", Weather_Condition = "Other", percent_weather = 0.02290385) %>%
  add_row(year = "2018", Weather_Condition = "Other", percent_weather = 0.02343750) %>%
  add_row(year = "2019", Weather_Condition = "Other", percent_weather = 0.03828895) %>%
  add_row(year = "2020", Weather_Condition = "Other", percent_weather = 0.03021382) %>%
  add_row(year = "2021", Weather_Condition = "Other", percent_weather = 0.03736665) %>%
  ggplot() +
  geom_col(aes(x = "", y = percent_weather, fill = Weather_Condition)) +
  coord_polar("y", start = 0) +
  facet_wrap(vars(year)) +
  labs(title = "Types of Weather During Accidents in California by Year", fill = "Weather") +
  theme_void()
```

![](Final_Draft_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

We also can choose to look at the types of accidents that are occuring 


```r
  accidents_by_type <- accidents_data_LA %>% 
    mutate(Year = year(Start_Time)) %>% 
    group_by(Year) %>% 
    mutate(total = n()) %>%
    mutate(across(Amenity:Turning_Loop,sum)) %>% 
    select(Amenity:Turning_Loop, Year, total) %>% 
    distinct() %>%
    pivot_longer(Amenity:Turning_Loop, values_to = "count", names_to = "type") %>% 
    group_by(Year, type) %>% 
    summarize(prop = round(count/total, digits = 3)) %>% 
    slice_max(prop, n = 5) %>% 
    arrange(Year) %>% 
    mutate(rank = 1:n()) %>% 
  ggplot(aes(x = factor(rank), y = prop, group = type)) +
    geom_col(aes(fill = type)) +
    geom_text(aes(label = sub("_", " ", type)), vjust = -.2) +
    labs(x = NULL) +
    scale_y_continuous(n.breaks = 10) +
    labs(y = "Proportion of all accidents",
         title = "Top 5 types of accidents by proportion of total accidents",
         subtitle = "{closest_state}") + 
    theme(axis.text.x = element_blank(),
          legend.position = "none") +
    transition_states(Year)
```

```
## Error in mutate(., Year = year(Start_Time)): object 'accidents_data_LA' not found
```

```r
    animate(accidents_by_type, fps = 30, duration = 10)
```

```
## Error in animate(accidents_by_type, fps = 30, duration = 10): object 'accidents_by_type' not found
```

```r
    anim_save("causes_bar_graph.gif")
```

```
## Error: The animation object does not specify a save_animation method
```


```r
knitr:include_graphics("causes_bar_graph.gif")
```

```
## Error in eval(expr, envir, enclos): object 'knitr' not found
```

##
We see that the spread of causes for accidents grows quite substantially as time goes on, since Traffic Signal accidents start to rival Junction accidents. Looking at the total proportion of all reported accidents that fall under at least one of these categories (for all of the causes available) we get less than 1; if anything the total should be greater than 1 since there are some accidents that fall under two categories. Basically, what we can conclude from this is that the accident reporting that this dataset is built on is incomplete, and there are a lot of unclassified accidents in this dataset. Unfortunately, this isn't the answer we were looking for, but it still gives us some information about the changes occurring in types of accidents over time. 


```r
  accidents_data_LA <- accidents_data %>% 
    filter((Start_Lng >= -118.9655) & (Start_Lng <= -117.3367) & (Start_Lat >= 33.6818) & (Start_Lat <= 34.4395))

  accidents_data_LA_points <- accidents_data_LA %>% 
    select(Start_Lat:End_Lng)
```


```r
  LA_map <- get_stamenmap(
     bbox = c(left = -118.9655 , bottom = 33.6818, right = -117.3367, top = 34.4395), 
     maptype = "toner-lite",
     zoom = 9)
```


```r
  LA_bigstreets <- ggmap(LA_map) +
    geom_point(data = accidents_data_LA_points, 
               aes(x = Start_Lng,
                   y = Start_Lat),
               size = .1,
               alpha = .1,
               color = "red") +
  labs(x = NULL,
       y = NULL) +
  geom_sf(data = big_streets$osm_lines,
          inherit.aes = FALSE,
          color = "green",
          size = .1,
          alpha = .9) +
    theme_map() +
    theme(axis.text.x = "none",
          axis.text.y = "none")
  
  ggsave(filename = "images/LA_bigstreets.png", LA_bigstreets)
```

# ```{r}
# LA_bigstreets <- ggmap(LA_map) +
#     geom_point(data = accidents_data_LA, 
#                aes(x = Start_Lng,
#                    y = Start_Lat,
#                    shape = factor(year(Start_Time))),
#                size = .1,
#                alpha = .1) +
#   labs(x = NULL,
#        y = NULL) +
#           theme_map() +
#   scale_color_viridis_d()
#   LA_bigstreets
# ```


```r
  knitr::include_graphics("images/LA_bigstreets.png")
```

<img src="images/LA_bigstreets.png" width="2100" />
