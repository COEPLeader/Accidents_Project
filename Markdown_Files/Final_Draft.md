---
title: 'US Accidents (2016-2021), or what the hell is going on in California?'
author: "Seth Buesing, Jason Whitelaw, Kai Yamanishi"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---











# Introduction
  In our search for an interesting dataset to analyze we came across a wonderfully organized table detailing traffic accidents in the contiguous United States over the last 5 years. What caught our interest in specific was looking at the evolution over time of crash statistics in specific states- most notably California. As the graph below shows, the number of accidents in California has been doubling each year, and we wanted to figure out just what was happening. The dataset we're working with is extraordinarily large, containing over 2.8 million accidents taken from automated traffic sensors and logged reports. Each accident has 47 columns of information.





<img src="accidents.gif" title="Animation of US map showing number of car accidents by state from 2016 to 2019." alt="Animation of US map showing number of car accidents by state from 2016 to 2019."  />

# Data Collection
  Our Data Collection process was pretty simple- we found the data set on [https://www.kaggle.com/datasets/sobhanmoosavi/us-accidents]() and downloaded the csv file. We did run into some issues using this csv, however due to its sheer size (1.1 GB). The fun thing about this data set is that it contains so many details about each crash, which allows us to look for correlations without them being spoon-fed to us. Additionally, the types of data provided allowed us to explore quite a few different types of visualizations: the time was given for each crash, as well as location plus details on the cause of the crash.

# Analysis

  After deciding to focus in on California, we wanted to take a closer look at the distribution of accidents within the state. Unsurprisingly, this showed that the vast majority of accidents were clustered in Los Angeles, and to a lesser extent the south of the state in general. The second map plots all of the accidents recorded in the data set over a map of Los Angeles.This shows the clustering of accidents in the busier parts of the city and freeways.



![](Final_Draft_files/figure-html/unnamed-chunk-8-1.png)<!-- -->







<img src="images/LA_bigstreets.png" title="A map of traffic accidents in Los Angeles, every red dot is one accident." alt="A map of traffic accidents in Los Angeles, every red dot is one accident." width="2100" />

So, what could be causing this pattern of exponential growth? The dataset we found had columns for factors such as whether the accident was in the day or the night, what the temperature and weather were like, and the type of accident.Time of day is a big factor in how often accidents occur, even though that effect is mitigated by the fact that fewer people are driving later in the day. We first decided to look at the proportion of day to night accidents, still not expecting too much of a correlation.

<img src="Final_Draft_files/figure-html/unnamed-chunk-13-1.png" title="Percent of day/night accidents in California by year." alt="Percent of day/night accidents in California by year."  />

Oddly enough, we do see an increase in the proportion of night accidents over the last 3 years, though with a dip back down in 2021. This isn't sufficient to say anything meaningful on its own though, so we decided to look a bit further into detail. This next graph shows the number of accidents broken down by hour of the day, and colored by the year in which they occurred. It revealed that while the binary proportions shown in the last graph have been shifting, in reality this has only meant things shifting  few minutes later in the day. The two visible spikes correlate well with morning and evening rush hours.

<img src="Final_Draft_files/figure-html/unnamed-chunk-14-1.png" title="Graph of the number of accidents occuring each hour of the day, coloured by the year in which they occured." alt="Graph of the number of accidents occuring each hour of the day, coloured by the year in which they occured."  />

Next we decided to look at possible effects of changing weather on accidents. While there is an increase in the percentage of clear weather, there's no indication this is correlated with the increase in accidents.

<img src="Final_Draft_files/figure-html/unnamed-chunk-15-1.png" title="Types of weather during accidents in California by year." alt="Types of weather during accidents in California by year."  />

With environmental factors not showing anything interesting, we next chose to look at the types of accidents that are occuring.



<img src="causes_bar_graph.gif" title="Bar graph showing the top 5 associated cause tags provided for each accident, and the proportion of total accidents each made up. The graph is animated over the 6 year span, and we see that the accidents at Junctions were the largest proportion recorded, followed increasingly closely by Traffic Signals as time went on." alt="Bar graph showing the top 5 associated cause tags provided for each accident, and the proportion of total accidents each made up. The graph is animated over the 6 year span, and we see that the accidents at Junctions were the largest proportion recorded, followed increasingly closely by Traffic Signals as time went on."  />

We see that the spread of causes for accidents grows quite substantially as time goes on, as Traffic Signal accidents started to rival Junction accidents. This could be interesting, but looking at the total proportion of all reported accidents that fall under at least one of these categories (for all of the causes available) and adding them up we get substantially less than 1; if anything the total should be greater than 1 since there are some accidents that fall under two categories. What we can conclude from this is that the accident reporting that this data set is built on is incomplete, and there are a lot of unclassified accidents in this data set. Unfortunately, this isn't the answer we were looking for, but it still gives us some information about the changes occurring in types of accidents over time, as at least among reported accidents the relative frequency of Junction accidents has fallen substantially.

# Conclusions

So where does all of this leave us? At least in all of the data we have, there are no real clues to the reasons behind the continued growth of the number of car accidents occurring in California. This is remarkable given the sustained and exponential nature of the growth even during the relatively slow period of the Covid-19 pandemic. While we have failed to pull any explanations from this data set, it does give us enough information to note that if this trend continues, within just a few years the average Californian will have to get into multiple accidents a day just to keep up appearances. 
