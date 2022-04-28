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






```
## Error: 'US_Accidents_Dec21_updated.csv' does not exist in current working directory ('C:/Users/kaiya/Documents/Data Science/Accidents_Project/Markdown_Files').
```


```
## Error in mutate(., state = state.name[match(State, state.abb)]): object 'accidents_data' not found
```


```
## Error in filter(., State == "CA"): object 'accidents_clean' not found
```

# Introduction
  In our search for an interesting dataset to analyze we came across a wonderfully organized table detailing traffic accidents in the contiguous United States over the last 5 years. What caught our interest in specific was looking at the evolution over time of crash statistics in specific states- most notably California. As the graph below shows, the number of accidents in California has been doubling each year, and we wanted to figure out just what was happening. The dataset we're working with is extraordinarily large, containing over 2.8 million accidents taken from automated traffic sensors and logged reports. Each accident has 47 columns of information.




```
## Error: The animation object does not specify a save_animation method
```

![](accidents.gif)<!-- -->

# Data Collection
  Our Data Collection process was pretty simple- we found the data set on [https://www.kaggle.com/datasets/sobhanmoosavi/us-accidents]() and downloaded the csv file. We did run into some issues using this csv, however due to its sheer size (1.1 GB).

#Analysis

So, what would cause this pattern? The dataset we found had columns for factors such as whether the accident was in the day or the night, what the temperature and weather were like, and the type of accident. Time of day is a big factor in how often accidents occur, even though that effect is mitigated by the fact that fewer people are driving later in the day. We decided to look at the proportion of day to night accidents, still not expecting too much of a correlation.


```
## Error in filter(., is.na(Sunrise_Sunset) == FALSE): object 'accidents_ca' not found
```

Oddly enough, we do see an increase in the proportion of night accidents over the last 3 years. 


```
## Error in filter(., is.na(Weather_Condition) == FALSE): object 'accidents_ca' not found
```

While there is an increase in proportion of Clear weather accidents over the past 3 years, there's no indication that this is correlated with the increase in accidents.
  

```r
big_streets <- opq(bbox = c(-118.965,33.6818,-117.3367,34.4395), timeout = 100)%>%
  add_osm_feature(key = "highway", 
                  value = c("motorway", "primary", "motorway_link", "primary_link")) %>%
  osmdata_sf()
```


```r
  accidents_data_LA <- accidents_data %>% 
    filter((Start_Lng >= -118.9655) & (Start_Lng <= -117.3367) & (Start_Lat >= 33.6818) & (Start_Lat <= 34.4395))
```

```
## Error in filter(., (Start_Lng >= -118.9655) & (Start_Lng <= -117.3367) & : object 'accidents_data' not found
```

```r
  accidents_data_LA_points <- accidents_data_LA %>% 
    select(Start_Lat:End_Lng)
```

```
## Error in select(., Start_Lat:End_Lng): object 'accidents_data_LA' not found
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
       y = NULL)
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


```r
  knitr::include_graphics("images/LA_bigstreets.png")
```

<img src="images/LA_bigstreets.png" width="2100" />

