---
layout: post
title: Football History Time Lapse
subtitle: using gganimate
---

How the beautiful game of football propogated throughout the world?

I am a bit of 'Sleepless in Seattle' on Saturday morning and really excited for the world cup which is going to kick-off next week on June 14th.

To get in the mood for the upcoming month, I found myself a dataset with all the international football matches ever played since the 1872 friendly between England vs Scotland which was held in Glasgow.

In the below geo plot, count of matches corresponds to size of circle which becomes darker as years go by giving us a sense of how the game of football became popular across the world!

<a href="https://imgur.com/RK2tH5B"><img src="https://i.imgur.com/RK2tH5B.gif" title="source: imgur.com" /></a>

Football has its origins in England from where it got spread to rest of Europe, then South America followed by Africa and finally Asia
Interestingly, India did not played its first match till 1938 friendly away against Australia where it lost 5-3 while Indian players were playing bare feet. Yeps, the legend is true!

<iframe width="700" height="315" src="https://www.youtube.com/embed/Iuzkq-VDJrE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>



PS - Bear in mind above animation only accounts for count of matches and not the quality or FIFA ranking





HERE IS THE CODE FOR CREATING ABOVE ANIMATION


Importing required libraries
```r
library(ggplot2)
library(dplyr)
library(DT)
library(maps)
library(ggthemes)
library(plotly)
library(tibble)
library(lubridate)
library(gganimate)
library(readr)
library(ggmap)
library(animation)
```

Glancing at the dataset
```r
head(df)
```

```
|date|home_team|away_team|home_score|away_score|tournament|city|country|neutral|
| -------------| -------------| -------------| -------------| -------------| -------------| -------------| -------------| -------------|
|1872-11-30|Scotland|England|0|0|Friendly|Glasgow|Scotland|FALSE|
|1873-03-08|England|Scotland|4|2|Friendly|London|England|FALSE|
```

Some data wrangling to get a data frame with counts of matches for all countries and years
```r
df_home = df %>% select(date, home_team) %>% rename(country = home_team)
df_away = df %>% select(date, away_team) %>% rename(country = away_team)

df_all = rbind(df_home,df_away)
df_all$date = year(df_all$date)
df_all = df_all %>% group_by(date, country) %>% summarise(matches = n()) %>% rename(year = date)
```

Getting the map coordinates
```r
unique_countries = unique(df_all$country)

map_data <- data.frame(
                      country=unique_countries,
                      lat=1:length(unique_countries),
                      lon=1:length(unique_countries),
                      stringsAsFactors=FALSE
                      )


for (i in 1:nrow(map_data)){
  call <- geocode(as.character(map_data$country[i]))
  map_data$lat[i] <- call$lat
  map_data$lon[i] <- call$lon
}

head(map_data)
```

Static map
```r
world <- ggplot() +
  borders("world", colour = "gray85", fill = "gray80") +
  theme_map() 

map <- world +
  geom_point(aes(x = lon, y = lat, size = matches),
             data = map_data_all, 
             colour = 'purple', alpha = .5) +
  scale_size_continuous(range = c(1, 8), 
                        breaks = c(250, 500, 750, 1000)) +
  labs(size = 'Followers')
map

ggplotly(map)
```

Introducing ghost points to see the final result for a bit longer
```r
ghost_points_ini <- tibble(
  year = '1872',
  matches = 0, lon = 0, lat = 0)

ghost_points_fin <- tibble(
  year = rep('2017', 15),
  matches = 0, lon = 0, lat = 0)
```

Animate the map
```r
map <- world +
  geom_point(aes(x = lon, y = lat, size = matches, 
                 frame = year,
                 cumulative = TRUE),
             data = map_data_all %>% filter(year > 2000), colour = 'purple', alpha = .1) +
  geom_point(aes(x = lon, y = lat, size = matches, # this is the init transparent frame
                 frame = year,
                 cumulative = TRUE),
             data = ghost_points_ini, alpha = 0) +
  geom_point(aes(x = lon, y = lat, size = matches, # this is the final transparent frames
                 frame = year,
                 cumulative = TRUE),
             data = ghost_points_fin, alpha = 0) +
  scale_size_continuous(range = c(1, 8), breaks = c(250, 500, 750, 1000)) +
  labs(size = 'Count of Matches')  +
    labs(title = 'Year =') +
    theme(plot.title = element_text(size = 20))

library(gganimate)
ani.options(interval = 0.2)
gganimate(map, ani.width = 1000, ani.height = 750, interval=0.2)

```
