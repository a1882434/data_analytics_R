---
title: "Does poverty affect children out of school?"
author: "Pui Lam Paulina Suen, Maliny Sophal, Ummay Kulsum Samiha"
date: "2023-06-12"
output:
  rmdformats::readthedown:     
    lightbox: true
    gallery: true
    toc_depth: 2
    fig_width: 9
    fig_height: 9
    highlight: espresso
    
---

#   1   Introduction
In the past three decades, the world economy grew four times larger, but millions of children around the world are still suffering from poverty every day. The United Nations' Sustainable Development Goals prioritize "No Poverty" as the first goal, recognizing that poverty is a major problem that must be addressed as it brings far-reaching impacts, especially for children. Poverty not only affects necessities but also limits access to development, as well as education opportunity (United Nations report, 2021). This report aims to determine whether poverty affects children drop out of school. 

According to Reay (2019), there is an indistinguishable linkage between poverty and education opportunities, poor children are always unable to achieve much educational qualifications. Rueckert (2019) also pointed out that poverty is a barrier for children in poverty to access education, due to numerous factors such as expenses of education. Even though The Universal Declaration of Human Rights protects all children over the world to have free education rights, some developing countries cannot provide sufficient resources on children education, and to fulfil the free education rights, parents are forced to pay informal fees on schooling item such as uniform and exam fee to support the government’s education spending, and poor families can never afford those expenses. Facing the high cost of sending children to school, families in poverty instead force children to work. As a result, poverty becomes a leading factor of child labour. Research has found the correlation between poverty and child labour, and addressed improvement in poverty and reduce child labour can increase education opportunities (Edmonds & Turk, 2002). Another research also found negative correlation between child labour and overall school attendance (Guarcello, et,al., 2006), suggesting children in employment is related to children out of school.  
 
Despite many studies focus on the impacts of poverty on education, they mostly studied the influence of poverty on education inequality or education performance, limited research focus on the relationship of poverty and children’s dropouts. High percentage of children out of school is still an unsolved problem to many countries, not only developing countries, but also developed countries like the UK and the USA. Poverty and education are not a single way relationship, they affect each other. While poverty affects children out of school, education has been proved to be a significant key to break the cycle of poverty. Studies from UNESCO suggested that people with higher levels of education have the higher probability to be under employment and earn higher income, and less households with better education are being poor (UNESCO & International Academy of Education, 2008). The problem of high percentage of children dropouts cannot be completely solved without reducing the number of poverty population, while poverty rate is difficult to go down when children lack of education opportunity. Paradoxically, as a result of poverty, when children in poor family are forced to abandon their education early to join the workforce, they often find themselves trapped in occupations that restrict their opportunities for upward mobility and escaping the cycle of poverty (Globalpartnership, 2016).

This report targets to present the seriousness of the impact of poverty on children out of school, provide important implications to policymakers and advocates who help children achieve education to lift them and their families out of poverty. Using data from Gapminder.org, the influence of poverty on children’s dropouts around the world will be examined. Representations of each dataset with graphs and maps provide a clear vision of the spread of data in each country and regions for comparison. A multiple regression model is built in this report to perform regression analysis, with the result of data analysis, to identify significant relationships between poverty and children drop out of school, estimate how poverty headcount ratio and percentage of children in employment affect the percentage of primary school age children out of school. 


```{r setup, include = FALSE}
knitr::opts_chunk$set(warning = FALSE, message = FALSE, echo = TRUE) 

#Loading packages
library(tidyverse)
library(tidyr)
library(tidymodels)
library(gapminder)
library(lubridate)
library(readr)
library(countrycode)
library(tibble)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
library(ggrepel)
library(RColorBrewer)
library(sjPlot)
library(sjmisc)
library(sjlabelled)

```


#   2   Data Wrangling
Data related to poverty and children out of school are sourced and used for analysis in this project. To avoid raw data affects the performance and result of analysis, data wrangling is performed in this section to convert data to become reliable and complete.

##    2.1   Date Source
Three sets of data are sourced in this project: 

1) Children out of school (% of primary school age)
2) Poverty headcount ratio at $1.90 a day (2011 PPP) (% of population)
3) Children in employment, work only (% of children in employment, ages 7-14)

All data are from [Gapminder official website](https://www.gapminder.org/data/), the CSV format files are downloaded and used in this project.


###   Dataset 1: Children out of school (% of primary school age) 
```{r}
#Import data
Primary_school_age_children_out_of_school <- read.csv("children_out_of_school_primary.csv")
```
```{r, message=FALSE, echo=FALSE}
#Preview of data
head(Primary_school_age_children_out_of_school, n = c(5,10))
```
This dataset recorded the percentage of children out of school, the children are in primary school age. The data source is from UNESCO Institute for Statistics and the data is last updated in September 2021. This dataset includes data from 1970 to 2021 for 201 countries.


###   Dataset 2: Poverty headcount ratio at $1.90 a day (2011 PPP) (% of population)
```{r}
#Import data
Poverty_headcount_ratio <- read.csv("extreme_poverty_percent_people_below_190_a_day.csv")
```
```{r, message=FALSE, echo=FALSE}
#Preview of data
head(Poverty_headcount_ratio, n = c(5,10))
```
The poverty headcount ratio dataset specifically provides the percentage of people living on less than $1.90 a day. The data source is from the World Bank, poverty and inequality platform. The data are based on primary household survey data obtained from government statistical agencies and World Bank country departments. This dataset contains 168 countries' data tracking from 1967 to 2021.
 
 
###   Dataset 3: Children in employment, work only (% of children in employment, ages 7-14)
```{r}
#Import data
Children_in_employment <- read.csv("children_in_employment_work_only.csv")
```
```{r, message=FALSE, echo=FALSE}
head(Children_in_employment, n = c(5,10))
```
This dataset includes the percentage of children in employment, children are in ages 7 to 14 who work only. The data is based on children’s work projects by ILO, UNICEF, and the World Bank. This dataset includes data in 100 countries from 1994 to 2016.


##    2.2   From wide format to long format
All datasets are transformed from a wide format to a long format for better reading, visualizing and performing functions in R.
```{r}
#Transform from wide format to long format
Primary_school_age_children_out_of_school <- pivot_longer(Primary_school_age_children_out_of_school,
                                                          -c(country),
                                                          names_to = "Year",
                                                          values_to = "children_out_of_school",
                                                          values_transform = list(children_out_of_school = as.numeric)
                                                          )

Poverty_headcount_ratio <- pivot_longer(Poverty_headcount_ratio,
                                        -c(country),
                                        names_to = "Year",
                                        values_to = "poverty_ratio",
                                        values_transform = list(poverty_ratio = as.numeric)
                                        )

Children_in_employment <- pivot_longer(Children_in_employment,
                                       -c(country),
                                       names_to = "Year",
                                       values_to = "percentage_of_children_in_employment",
                                       values_transform = list(percentage_of_children_in_employment = as.numeric)
                                       )
```


##    2.3   Add new columns
To increase the comprehensiveness, two new columns are created and added to each dataset. 


###   Create continent column 
The "countrycode" package is used to add the continent column. The continent column is used in the summary section to provide more detailed explanation of the data. 
```{r}
#Add a new column with countrycode package
Primary_school_age_children_out_of_school <- Primary_school_age_children_out_of_school%>%
  mutate(continent = countrycode(country, 
                                 origin = "country.name", 
                                 destination = "continent")
         )

Poverty_headcount_ratio <- Poverty_headcount_ratio %>% 
  mutate(continent = countrycode(country, 
                                 origin = "country.name", 
                                 destination = "continent")
         )


Children_in_employment <- Children_in_employment %>% 
  mutate(continent = countrycode(country, 
                                 origin = "country.name", 
                                 destination = "continent")
         )
```

 
###   Create average percentage column
Since the data includes long periods of data and some data is missing, by calculating the average percentage, the analysis can be performed with more reliable data, and the result will be more accurate.

Based on the nature of the data, it is assuming that all 0 values mean incomplete data, to avoid influence on the result of analysis, 0 value is assigned as NA value. NA value is dropped out before calculating the average percentage.
```{r}
#assign all 0 value as NA
Primary_school_age_children_out_of_school[Primary_school_age_children_out_of_school == 0] <-NA
Poverty_headcount_ratio[Poverty_headcount_ratio == 0] <- NA
Children_in_employment[Children_in_employment == 0] <- NA

#drop out the NA values
Primary_school_age_children_out_of_school <- Primary_school_age_children_out_of_school %>% drop_na()
Poverty_headcount_ratio <- Poverty_headcount_ratio %>% drop_na()
Children_in_employment <- Children_in_employment %>% drop_na()

#Compute the average value and add as a new column
Primary_school_age_children_out_of_school <- Primary_school_age_children_out_of_school %>%
  group_by(country) %>%
  mutate(average_dropout = mean(children_out_of_school)) %>%
  ungroup()

Poverty_headcount_ratio <- Poverty_headcount_ratio %>%
  group_by(country) %>%
  mutate(average_poverty_ratio = mean(poverty_ratio)) %>%
  ungroup()

Children_in_employment <- Children_in_employment %>%
  group_by(country) %>%
  mutate(average_children_in_employment = mean(percentage_of_children_in_employment)) %>%
  ungroup()
```
 
 
All datasets are re-organized and only necessary data is selected, including country, continent and the average percentage. Below is the preview of each dataset after data wrangling:
 
 
<font size="5"> Dataset of Children out of school </font> 
```{r, message=FALSE, echo=FALSE}
Primary_school_age_children_out_of_school <- Primary_school_age_children_out_of_school %>%
  group_by(country) %>%
  #Since the average percentage for each year is the same, only one data for each country is selected
  filter(!duplicated(country)) %>%
  select(country, continent, average_dropout)

#Preview data 
head(Primary_school_age_children_out_of_school, n = 5)
```
 
 
<font size="5"> Dataset of Poverty headcount ratio </font> 
```{r, message=FALSE, echo=FALSE}
Poverty_headcount_ratio <- Poverty_headcount_ratio %>% 
  group_by(country) %>% 
  #Since the average percentage for each year is the same, only one data for each country is selected
  filter(!duplicated(country)) %>% 
  select(country, continent, average_poverty_ratio)

#Preview data 
head(Poverty_headcount_ratio, n = 5)
```
 
 
<font size="5"> Dataset of Children in employment </font> 
```{r, message=FALSE, echo=FALSE}
Children_in_employment <- Children_in_employment %>% 
  group_by(country) %>% 
  #Since the average percentage for each year is the same, only one data for each country is selected
  filter(!duplicated(country)) %>% 
  select(country, continent, average_children_in_employment)

#Preview data 
head(Children_in_employment, n = 5)
```


##    2.4   Merge dataset
To perform summary and multiple regression, datasets are merged. Dataset of children out of school and poverty headcount ratio are merged with the left_join() function by variables "country" and "continent” to summarise the relationship between these two variables. Dataset of children out of school and children in employment are also performed the same merging. All three datasets are also merged into a new dataset with left_join() function by variable “country” to perform multiple regression. 

The reason of choosing left_join() function to perform all merging is due to the number of observations. Children out of school dataset has the largest number of observations of 201. Left_join() function can keep the observations of children out of school, and by the time performing summary or regression analysis, the NA value will be dropped out automatically.

```{r}
#Merge data with inner_join() function
Relation_of_chlidren_out_of_school_and_poverty <- Primary_school_age_children_out_of_school %>%
  left_join(Poverty_headcount_ratio,
             by = c("country",
                    "continent")
             )

```
```{r message=FALSE, echo=FALSE}
#Preview data of the new dataset
head(Relation_of_chlidren_out_of_school_and_poverty, n = 5)
```

```{r}
#Merge data with inner_join() function
Relation_of_chlidren_out_of_school_and_child_employment <- Primary_school_age_children_out_of_school %>%
  left_join(Children_in_employment,
             by = c("country",
                    "continent")
             )
```
```{r message=FALSE, echo=FALSE}
#Preview data of the new dataset
head(Relation_of_chlidren_out_of_school_and_child_employment, n = 5)
```

```{r}
#Join all data into one dataset to perform multiple regression analysis
multiple_data <- Primary_school_age_children_out_of_school %>%
  select(-c(continent)) %>%
  left_join(Poverty_headcount_ratio %>%
              select(-c(continent)),
            by = "country") %>%
  left_join(Children_in_employment %>%
              select(-c(continent)),
            by = "country")
```
```{r message=FALSE, echo=FALSE}
#the column name is renamed to show data clearly in section 5
colnames(multiple_data)[2] = "Children_out_of_school"
colnames(multiple_data)[3] = "Poverty_headcount_ratio"
colnames(multiple_data)[4] = "Children_in_employment"
#Preview data of the new dataset
head(multiple_data, n=5)
```


##    2.5   Preparation for map visualization
To present the summary of data, a world map will be used to visualize the data for each dataset in the summary section, and some prior coding are prepared in this section for the map visualization.

First, a world map object is created, all datasets will use the same map. Then, a colour palette is defined to increase the comprehensiveness of data value. A new dataset will be created for each dataset, it includes data in the original dataset, and also the iso code that corresponding to the country name. It aims to match the specific location of each country on the map.  
```{r}
#Create a world map
world_map <- ne_countries(scale = "medium", 
                          returnclass = "sf")

map_colour <- brewer.pal(9, 'YlOrRd')
  
```
 
  
<font size="5"> Dataset of Children out of school </font> 
```{r}
#create a new dataset and add column for country name to correspond to the iso code
map_for_dropout <- Primary_school_age_children_out_of_school %>%
  mutate(Iso3 = countrycode::countrycode
         (sourcevar = country,
           origin = "country.name",
           destination = "iso3c")
         )

#join data to the map with iso code
map_for_dropout_all <- world_map %>%
  select(geometry,
         name,
         iso_a3) %>%
  inner_join(map_for_dropout, by = c("iso_a3" = "Iso3")) %>% 
  drop_na() #two countries cannot find in the map
```
```{r message=FALSE, echo=FALSE}
#Preview data that prepare for map visualization
head(map_for_dropout_all, n = 5)
```
 
 
<font size="5"> Dataset of Poverty headcount ratio </font> 
```{r}
#create a new dataset and add column for country name to correspond to the iso code
map_for_poverty <- Poverty_headcount_ratio %>%
  mutate(Iso3 = countrycode::countrycode
         (sourcevar = country,
           origin = "country.name",
           destination = "iso3c")
         )

#join data to the map with iso code
map_for_poverty_all <- world_map %>%
  select(geometry,
         name,
         iso_a3) %>%
  inner_join(map_for_poverty, by = c("iso_a3" = "Iso3")) %>%
  drop_na() #one country cannot find in the map
```
```{r message=FALSE, echo=FALSE}
#Preview data that prepare for map visualization
head(map_for_poverty_all, n = 5)
```
 
 
<font size="5"> Dataset of Children in employment </font> 
```{r}
#create a new dataset and add column for country name to correspond to the iso code
map_for_employment <- Children_in_employment %>%
  mutate(Iso3 = countrycode::countrycode
         (sourcevar = country,
           origin = "country.name",
           destination = "iso3c")
         )

#join data to the map with iso code
map_for_employment_all <- world_map %>%
  select(geometry,
         name,
         iso_a3) %>%
  inner_join(map_for_employment, by = c("iso_a3" = "Iso3"))
```
```{r message=FALSE, echo=FALSE}
#Preview data that prepare for map visualization
head(map_for_employment_all, n = 5)
```
 

#   3   Data summary
This section includes summary of each data, it is visualized in box plots, maps, and scatter plots. It aims to provide a brief understanding of the data and determine the distribution of data and the outliers.

##    3.1   Summary of children out of school
```{r}
#summarise the percentage of children out of school
summary(Primary_school_age_children_out_of_school$average_dropout)
```
```{r}
#Visualize the summary of children out of school in box plot
par(family.main = "serif", 
    cex.main = 1,
    family.lab = "sans",
    cex.lab = 1)
boxplot(Primary_school_age_children_out_of_school$average_dropout, 
        horizontal = T, 
        col = "#8EFAD6")
title(main = "Boxplot of children out of school",
        xlab = "Average percentage of children out of school",
        col.main = "#004C99",
        col.lab = "#317AC4")
```
The box plot visualizes the data of children out of school. It consists of a box represents the interquartile range (IQR) of the data, with a vertical line inside indicating the median. The left and right whiskers extend from the box, representing the minimum and maximum values within a certain range. Any points outside the whiskers are considered outliers.

The average percentage of children out of school for all observation ranges from a minimum of 0.087% to a maximum of 83.92%. The first quartile indicates approximately 25% of data falls below 3.00%, while the third quartile suggests that about 75% of data is below 18.78%. The median, which represents the midpoint of the data, is 7.61%. The mean, or average, is calculated to be around 13.51%. These statistics provide an overview of the distribution of the average dropout rates among primary school-age children, and they are visualized in the box plot. It is observed that there are several outliers in this dataset, and they may be removed when performing the regression analysis in the following section, if necessary.
```{r}
#Visualize data of children out of school in a world map
world_map %>%
  filter(admin != "Antarctica") %>%
  ggplot() +
  #generate a map
  geom_sf(fill = "grey", 
          colour = "black") +
  #add data into map
  geom_sf(data = map_for_dropout_all,
          aes(fill = average_dropout),
          colour = "black") +
  #add colour features
  scale_fill_gradientn(colours = map_colour,
                       limits=c(0,85), 
                       name = "average percentage of\nchildren out of school") + 
  theme_minimal() +
  #specify all titles
  labs(title = "Map visualization of children out of school") +
  scale_y_continuous(breaks = seq(0,7)) +
  #add features to show text clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_blank(),
        axis.text = element_blank(),
        panel.border = element_blank(),
        panel.grid=element_blank(),
        axis.ticks = element_blank()
        )
```
The world map shows data of children out of school of all observations. Dark red colour represents countries that have the highest average percentage in dropouts. As the red gets lighter the percentage gets lower. Location in grey colour means they are not observations. 

The map clearly shows that African countries are facing serious children out of school. It is no doubt that within all continent, Africa has the highest percentage of children out of school in average, most data are higher than 40%. A country in middle east also has a high average percentage of children out of school, but overall other continents did much better than Africa, with percentage lower than 20, and European countries generally have the lowest dropouts.
 
 
##    3.2   Summary of Poverty headcount ratio
```{r}
#summarise the poverty headcount ratio
summary(Poverty_headcount_ratio$average_poverty_ratio)
```
```{r}
#Visualize the summary of poverty headcount ratio in box plot
par(family.main = "serif", 
    cex.main = 1,
    family.lab = "sans",
    cex.lab = 1)
boxplot(Poverty_headcount_ratio$average_poverty_ratio, 
        horizontal = T, 
        col = "#8EFAD6")
title(main = "Boxplot of poverty headcount ratio",
        xlab = "Average poverty headcount ratio",
        col.main = "#004C99",
        col.lab = "#317AC4")
```
Above statistics and box plot offer insights into the range and distribution of poverty levels based on the given data. The lowest poverty headcount ratio is 0.10 and 80.6 is the highest. The median of 7.85 serves as the midpoint, where the vertical line inside the box of the box plot. The mean of 17.81 provides an average value of all observation. 

There are some observations fall significantly outside the overall data range, which are outliers. All those observations in this dataset have a value higher than the third quartile. To avoid performing a bad regression analysis, they may not be included in the regression model if necessary.
```{r}
#Visualize data of poverty headcount ratio in a world map
world_map %>%
  filter(admin != "Antarctica") %>%
  ggplot() +
  #generate a map
  geom_sf(fill = "grey", 
          colour = "black") +
  #add data into map
  geom_sf(data = map_for_poverty_all, 
          aes(fill = average_poverty_ratio),
          colour = "black") +
  #add colour features
  scale_fill_gradientn(colours = map_colour,
                       limits=c(0,85), 
                       name = "average poverty headcount ratio") + 
  theme_minimal() +
  #specify all titles 
  labs(title = "Map visualization of poverty headcount ratio") +
  scale_y_continuous(breaks = seq(0,7)) +
  #add features to show text clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_blank(),
        axis.text = element_blank(),
        panel.border = element_blank(),
        panel.grid=element_blank(),
        axis.ticks = element_blank()
        )
```
The world map shows the average poverty headcount ratio of all observations. Colour dark red represents countries that have the highest poverty headcount ratio. The percentage of poverty gets lower if the red gets lighter, and grey colour represents not being observed. 

From this map it is clearly visible that Africa has the highest average poverty headcount ratio among all continents. Most African countries are in red colour, and some are even in dark red suggesting a high number of population are in poverty. Countries in Europe and North America generally did well with very light colour meaning low poverty headcount ratio.

  
###    Scatter plot of poverty headcount ratio and children out of school
A scatter plot is created to present the situation of children out of school and poverty in each country.
```{r}
#Visualize the average percentage in poverty and children out of school with scatter plot
Relation_of_chlidren_out_of_school_and_poverty %>%
  ggplot(aes(y = average_dropout, 
             x = average_poverty_ratio)
         ) + 
  #generate point plot
  geom_point(aes(colour = continent)) + 
  scale_x_log10() + 
  scale_y_log10() + 
  #generate dot chart with linear model
  geom_smooth(method="lm",
              colour = "yellow") + 
  #specify all titles of the graph
  labs(title = "Scatter plot of poverty headcount ratio and children out of school",
       y = "Average percentage of children out of school",
       x = "Average poverty headcount ratio") +
  #add features to show the text in the graph clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_text(family = "sans",
                                  size = 12,
                                  colour = "#317AC4")
        )
```
In the above scatter plot, x-axis represents average poverty headcount ratio, y-axis represents average percentage of children out of school. The position of each dot indicates the value of each observation in both variables. The points are grouped by continents, plots that represents African countries generally located at the top right corner, suggesting they have a higher percentage in both children out of school and poverty headcount ratio, which is identical to the observation in above summary. There is relatively less points position in the bottom left corner meaning only a few countries have low percentage in both, and they are mostly from Europe. 

A trend line is added to the graph, showing the pattern of two variables. Points located exactly at the trend line or in the shadow of the line means those observations have a strong relationship in children out of school and poverty. Plots far from the trend line usually are outlier. There is a positive trend between children out of school and poverty since the trend line in the graph goes from the bottom left to the top right, implying children out of school in each country generally will increase when its poverty headcount ratio increases. This trend will be proved and estimated by multiple regression model in the following sections. 


##    3.3   Summary of children in employment
```{r}
#summarise the percentage of children in employment
summary(Children_in_employment$average_children_in_employment)
```
```{r}
#Visualize the summary of children in employment in box plot
par(family.main = "serif", 
    cex.main = 1,
    family.lab = "sans",
    cex.lab = 1)
boxplot(Children_in_employment$average_children_in_employment, 
        horizontal = T, 
        col = "#8EFAD6")
title(main = "Boxplot of children in employment",
        xlab = "Average pecentage of children in employment",
        col.main = "#004C99",
        col.lab = "#317AC4")
```
Summary statistics of children in employment is provided and visualized in box plot, to allow a quick understanding of the distribution of children in employment and help identify any extreme values.

The minimum value of 0.70% indicates that there are observations have a very small proportion of children are engaged in employment. The median value of 21.96% is presented as the line in the box of box plot, which represents the typical percentage of children in employment, with 50% of the data falling below and 50% falling above this value. The mean value of 27.37% provides an overall average of the average percentage of children in employment across the dataset. The maximum value of 88.85% represents the highest observed average percentage of children in employment in the dataset. The box plot shows there is no outlier in this dataset.
```{r}
#Visualize data of children in employment in a world map
world_map %>%
  filter(admin != "Antarctica") %>%
  ggplot() +
  #generate a map with colour features
  geom_sf(fill = "grey", 
          colour = "black") +
  #add data into map
  geom_sf(data = map_for_employment_all, 
          aes(fill = average_children_in_employment),
          colour = "black") +
  #add colour features
  scale_fill_gradientn(colours = map_colour,
                       limits=c(0,85), 
                       name = "average percentage of\nchildren in employment") + 
  theme_minimal() +
  #specify all titles 
  labs(title = "Map visualization of children in employment") +
  scale_y_continuous(breaks = seq(0,7)) +
  #add features to show text clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_blank(),
        axis.text = element_blank(),
        panel.border = element_blank(),
        panel.grid=element_blank(),
        axis.ticks = element_blank()
        )
```
Data of children in employment of all observations are visualized in the world map. Dark red colour represents the highest average percentage of children in employment, and lighter colour means lower value, in a gradual way. Location in grey colour means no data for those countries. 

It is clearly visible that Africa has the highest percentage of children in employment in average. Number of African countries are in darker colours representing high number of child labour, with value in 40% or above. South and Southeast Asia also find a few countries with dark colour, indicating under aged children employment is a critical issue, over 60% of children are in employment in average. Overall, South America has the lowest value within all continents. Some countries in Europe also perform well with an average percentage of nearly 0 in children in employment.


###   Scatter plot of children in employment and children out of school
```{r}
#Visualize the average percentage in children in employment and children out of school in scatter plot
Relation_of_chlidren_out_of_school_and_child_employment %>% 
  ggplot(aes(y = average_dropout, 
             x = average_children_in_employment)
         ) + 
  #generate point plot
  geom_point(aes(colour = continent)) + 
  scale_x_log10() + 
  scale_y_log10() + 
  #generate dot chart with linear model
  geom_smooth(method="lm",
              colour = "yellow") + 
  scale_colour_discrete(name = "continent") + 
  #specify all titles of the graph
  labs(title = "Scatter plot of children in employment and children out of school",
       y = "Average percentage of children out of school",
       x = "Average percentage of children in employment") +
  #add features to show the text in the graph clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_text(family = "sans",
                                  size = 12,
                                  colour = "#317AC4")
        )
```
The data of children dropouts and child labour of each observation are showing together in the scatter plot. Data of average percentage of children in employment is presented in x-axis, and average percentage of children out of school is shown in y-axis. Points are in different colours, representing specific continents. The scatter plot delivers the worse situation in Africa that most countries have a relatively high value in both dropouts and child labour. Countries in Asia and Americas have a visible different performance, some countries located at the top right side meaning a high average percentage in both, while some are in the middle or even the bottom left side which have a related low value in both.

The graph also shows the pattern of two variables with the trend line. Some points are located exactly at the trend line or in the shadow of the line, those observations have a strong relationship in children out of school and children in employment, and they are mostly Asian and African countries. The trend line goes from the bottom left to the top right, suggesting a positive trend between children out of school and children in employment. In other word, it indicated that children out of school in each country generally will increase when the percentage of child in employment increases. A multiple regression analysis will be performed with a good fit model in the following sections to prove and estimate this relation. 

 
#   4   Methodology
In this report, a multiple regression analysis is performed to estimate the effect of poverty headcount ratio and children in employment (predictor variables) on the children out of school (response variable). Several regression models are built to find the best fit regression model by comparing the adjusted R-squared.

##    First regression model
In the first regression model, all predictor variables are included in the model, and no observation, including outlier, is removed from the dataset. 
```{r}
#first regression model
Regression_multiple_1 <- lm(Children_out_of_school ~ Poverty_headcount_ratio + Children_in_employment, data = multiple_data)
summary(Regression_multiple_1)
```
The result of the regression model will be discussed in the next section, yet, to determine the best fit of regression model, some parts of the result of regression model is discussed in this section. 

From the result of the first regression model, the p-value of both independent variables suggested they are statistically significant, hence no variable should be removed from the model. Besides, the adjusted R-squared value is 0.6199, this value will be different when changing the model, and the value will go up only when the model improve. In other words, higher adjusted R-square value suggests a better fit regression model. In the following, comparison of adjusted R-square value of new models will be performed to find the best fit regression model.

Before building other new regression model, a scatter plot is generated to find the outlier(s) that may need to be removed from the dataset.
```{r}
#Observe outlier from scatter plot to find the specific observation
#Scatter plot of children out of school
multiple_data %>%
  mutate(outlier = case_when(Children_out_of_school > 80 ~ 4,
                             T ~ 1)
         ) %>%
  ggplot(aes(x = country, 
             y = Children_out_of_school, 
             colour = factor(outlier)
             )
         )+
  geom_point() +
  geom_text_repel(aes(label = ifelse(Children_out_of_school >80, 
                                     as.character(country), 
                                     "")
                      )
                  ) +
  scale_colour_identity() + 
    #specify all titles of the graph
  labs(title = "Scatter plot of average percentage of children out of school",
       x = "Country",
       y = "Average percentage of children out of school") +
  #add features to show the text in the graph clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_text(family = "sans",
                                    size = 12,
                                    colour = "#317AC4"),
        axis.text.x = element_blank()
        )
#Scatter plot of poverty headcount ratio
multiple_data %>%
  mutate(outlier = case_when(Poverty_headcount_ratio > 80 ~ 4,
                             T ~ 1)
         ) %>%
  ggplot(aes(x = country, 
             y = Poverty_headcount_ratio, 
             colour = factor(outlier)
             )
         )+
  geom_point() +
  geom_text_repel(aes(label = ifelse(Poverty_headcount_ratio >80, 
                                     as.character(country), 
                                     "")
                      )
                  ) +
  scale_colour_identity() + 
    #specify all titles of the graph
  labs(title = "Scatter plot of average poverty headcount ratio",
       x = "Country",
       y = "Average poverty headcount ratio") +
  #add features to show the text in the graph clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_text(family = "sans",
                                    size = 12,
                                    colour = "#317AC4"),
        axis.text.x = element_blank()
        )

#Scatter plot of children in employment
multiple_data %>%
  mutate(outlier = case_when(Children_in_employment > 80 ~ 4,
                             T ~ 1)
         ) %>%
  ggplot(aes(x = country, 
             y = Children_in_employment, 
             colour = factor(outlier)
             )
         )+
  geom_point() +
  geom_text_repel(aes(label = ifelse(Children_in_employment >80, 
                                     as.character(country), 
                                     "")
                      )
                  ) +
  scale_colour_identity() + 
    #specify all titles of the graph
  labs(title = "Scatter plot of average percentage of children in employment",
       x = "Country",
       y = "Average percentage of children in employment") +
  #add features to show the text in the graph clearly
  theme(plot.title = element_text(family = "serif",
                                  size = 15,
                                  face = "bold",
                                  colour = "#004C99"),
        axis.title = element_text(family = "sans",
                                    size = 12,
                                    colour = "#317AC4"),
        axis.text.x = element_blank()
        )
```
The scatter plots indicate that "Somalia" in Children out of school dataset has a great distance with other observation, which define it as an extreme outlier. "Congo, Dem. Rep." in poverty headcount ratio and "Morocco" in children in employment are also far from other observations and being defined as extreme outlier.

##    Second regression model
In the second regression model, one outlier will be removed from the dataset to observe whether this will improve the model. Since "Somalia" in Children out of school dataset has the highest percentage, it will be the first observation to be removed in the new regression model.
```{r}
#remove Somalia from the dataset
multiple_data_r2 <- multiple_data %>%
  filter(country != "Somalia")

#second regression model
Regression_multiple_2 <- lm(Children_out_of_school ~ Poverty_headcount_ratio + Children_in_employment, 
                            data = multiple_data_r2)

summary(Regression_multiple_2)
```
From the result of the second regression model, the adjusted R-squared value is 0.6208 which is higher than the previous model, implying it is a better regression model than the first model. Since adjusted R-squared value is going up, new model will be built until the value did not go up. 


##    Third regression model
The third regression model is built with additionally removing the second largest extreme outlier.
```{r}
#remove Morocco from the dataset
multiple_data_r3 <- multiple_data_r2 %>%
  filter(country != "Morocco")

#third regression model
Regression_multiple_3 <- lm(Children_out_of_school ~ Poverty_headcount_ratio + Children_in_employment, 
                            data = multiple_data_r3)

summary(Regression_multiple_3)
```
The adjusted R-squared value for the third regression model is 0.6235 which is higher than the previous models, suggesting it is a better regression model than the first two models. 


##    Fourth regression model
The fourth regression model is built with removing the third largest extreme outlier.
```{r}
#remove Congo, Dem. Rep. from the dataset
multiple_data_r4 <- multiple_data_r3 %>%
  filter(country != "Congo, Dem. Rep.")

#fourth regression model
Regression_multiple_4 <- lm(Children_out_of_school ~ Poverty_headcount_ratio + Children_in_employment, 
                            data = multiple_data_r4)

summary(Regression_multiple_4)
```
The adjusted R-squared value for the fourth regression model is 0.6053 which is lower than the third regression model, meaning it is not a good fit regression model. This result indicate that removing more outliers will not improve the regression model and it can be concluded that the third regression model is the best, which will be used in this report, and its result will be discussed in the next section.


#   5   Result and analysis
This section discusses the result of the regression model which proves that poverty affects children out of school.

```{r, message=FALSE, echo=FALSE}
tab_model(Regression_multiple_3)
```

The results of the regression model indicate that poverty ratio and children in employment are statistically significant predictors of children out of school by observing the p-value of less than 0.1%. This means that the percentage of children out of school increases as the poverty ratio and the percentage of the children in employment increases. 

The estimated coefficients suggest that every one percentage increase in the poverty ratio is associated with a 28.4% increase in the percentage of children out of school, and every one percentage increase in the percentage of children in employment is associated with a 42.3% increase in the percentage of children out of school. The regression equation for children out of school equal to 0.53 + 0.28 (poverty headcount ratio) + 0.42 (percentage of children in employment). 

These results suggest that poverty headcount ratio and children in employment have amplifying effects on children out of school. In other words, the effects of poverty and child labour interact in a way that increases the likelihood that children will be out of school. This result also explains the observation in the summary section that Africa is always recorded the highest percentage among all continents in each dataset, since three variables have a positive correlationship. African countries are mostly developing countries, with higher poverty headcount ratio and percentage of child labour, as a result also have higher percentage in children out of school. In contrast, other continents with more developed countries with a lower poverty headcount ratio and children in employment, have a lower proportion of children out of school.


#   6   Conclusion
In conclusion, it is proved that poverty affects children out of school. The result of the multiple regression model indicates that poverty headcount ratio and percentage of children in employment both have significant positive relationship with the percentage of children out of school. The number of children  drop out of school will be reduced when the number of poverty population and child labour decline. 

Meanwhile, it is important to note that the result in this report is based on a cross-sectional analysis, meaning it cannot establish a causal relationship between poverty, children in employment, and children out of school. It is possible that other factors, such as the quality of education or the availability of jobs, may also play a role in explaining the relationship between these variables. Additionally, the data used in this study was collected for only 100-200 countries from around 1967 to 2021, the data for children in employment even just collected between 1994 and 2016, which is not comprehensive enough to present and estimate the years and countries that are not included in the dataset. 

Despite that, the result of the regression model is still an important implication to policymakers that interventions addressing both poverty and children in employment are more effective at reducing the number of children out of school than addressing either issue in isolation.
 
 
#   7   Reference
Guarcello, L., Lyon, S., & Rosati, F., (2006). Child Labour and Education For All: An Issue Paper. SSRN Electronic Journal. Available online:
https://www.researchgate.net/publication/272300404_Child_Labour_and_Education_For_All_An_Issue_Paper. Accessed 11 June 2023.

Percentage of aged 7-14 children in employment (work only).Available online:https://www.gapminder.org/data/. Accessed 11 June 2023.

Percentage of population who live with $1.90 a day (2011 PPP). Available online:https://www.gapminder.org/data/ & http://pip.worldbank.org. Accessed 11 June 2023.

Percentage of primary school age children who are out of school. Available online:https://www.gapminder.org/data/ & http://unis.unesco.org/. Accessed 11 June 2023.

Reay, D. (2019). Poverty and education. Available online:http://www.researchgate.net/publication/334398213_Poverty_and_Education. Accessed 11 June 2023.

Rueckert P. (2019). 10 Barriers to Education That Children Living in Poverty Face. Available online:https://www.globalcitizen.org/en/content/10-barriers-to-education-around-the-world-2/. Accessed 11 June 2023.

The Global Partnership for Education Secretariat. (2016). Child labour hinders children’s education. Available online: https://www.globalpartnership.org/blog/child-labour-hinders-childrens-education. Accessed 11 June 2023.

Turk, C., & Edmonds, E. (2002). Child labour in transition in Vietnam. Policy Research Working Papers. Available online:https://doi.org/10.1596/1813-9450-2774. Accessed 3 June 2023.

UNESCO, International Academy of Education. (2008) Poverty and Education. Available online:https://unesdoc.unesco.org/ark:/48223/pf0000181754. Accessed 11 June 2023.

United Nations. (2021) The Sustainable Development Goals Report. Available online: https://unstats.un.org/sdgs/report/2021/The-Sustainable-Development-Goals-Report-2021.pdf. Accessed 11 June 2023.
