---
title: "Bellabeat Analysis"
author: "Emily"
date: "2023-01-27"
output: html_document
---
## Introduction
Welcome to my data analysis for Bellabeat, a high-tech manufacturer that creates health-focused products for women. This project followed six steps in order to answer key business questions; ask, prepare, process, analyze, share, and act.

In this study I focused on Fitbit data to gain insight on how consumers use their smart devices, to make business decisions for Bellabeat’s products.

### Bellabeat
Bellabeat is a high-tech producer of health-focused products for women, which includes smart devices like an app, a wellness tracker, a wellness watch, and an interactive water bottle.

## Ask: To better understand the data and the business task.
1.	What are some trends in smart device usage? 
2.	How could these trends apply to Bellabeat customers? 
3.	How could these trends help influence Bellabeat's marketing strategy? 
4.	Who are the key stakeholders?

*Business Task: How do Bellabeat customers use their smart devices and what potential opportunities does the marketing team have based on trends in smart device usage.*

## Prepare: Prepare data before manipulation.
-	Data was downloaded from Kaggle and stored in the case study’s folder.
-	The data has .CSV files with long and wide formats.
-	Bias: The data is from FitBit users, not from Bellabeat users. This means tha there is a gender bias since the data used includes men and women, but Bellabeat's products are for women.
-	Integrity and licensing are adressed bellow.
-	The data was sorted and filtered to my needs.
- The data has information about 33 different individuals, however, only 30 have complete information.

### Data set description
Public data from FitBit Fitness Tracker Data was used for the analysis (CC0: Public Domain, dataset made available through Mobius). This Kaggle data set contains 18 CSV files about personal fitness tracker information. Thirty Fitbit users consented to the submission of data including physical activity, heart rate, and sleep monitoring. 

The dataset was generated by respondents to a distributed survey via Amazon Mechanical Turk between 03.12.2016 - 05.12.2016. 

## Process: R studio was chosen for this case study. 

#### Load packages

```{r, echo=FALSE}
library(tidyverse)
library(lubridate)
library(dplyr)
library(ggplot2)
library(tidyr)
library(here)
library(skimr)
library(janitor)
```
#### Set working directory

```{r}
getwd()
setwd("~/Desktop/personal stuff/Analytics/CASE STUDY/Fitabase Data 4.12.16-5.12.16")
```

### Importing data sets

```{r, echo=FALSE}
Activity <- read.csv("dailyActivity_merged.csv")
head(Activity)
Calories <- read.csv("dailyCalories_merged.csv")
head(Calories)
Intensities <- read.csv("dailyIntensities_merged.csv")
head(Intensities)
Heartrate <- read.csv("heartrate_seconds_merged.csv")
head(Heartrate)
Sleep <- read.csv("sleepDay_merged.csv")
head(Sleep)
Weight <- read.csv("weightLogInfo_merged.csv")
head(Weight)
```

### Review data

```{r}
glimpse(Activity)
glimpse(Calories)
glimpse(Intensities)
glimpse(Heartrate)
glimpse(Sleep)
glimpse(Weight)
```

### Cleaning the data sets

-	Three duplicates were removed from the *sleep data*.
```{r}
dim(Sleep)
sum(is.na(Sleep))
sum(duplicated(Sleep))
Sleep <- Sleep[!duplicated(Sleep), ]
```
-	 Time stamp data formatted.
```{r}
Activity$ActivityDate=as.POSIXct(Activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone())
Activity$date <- format(Activity$ActivityDate, format = "%m/%d/%y")
Activity$ActivityDate=as.Date(Activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone())
Activity$date=as.Date(Activity$date, format="%m/%d/%Y")

Intensities$ActivityDay=as.Date(Intensities$ActivityDay, format="%m/%d/%Y", tz=Sys.timezone())

Sleep$SleepDay=as.POSIXct(Sleep$SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
Sleep$date <- format(Sleep$SleepDay, format = "%m/%d/%y")
Sleep$date=as.Date(Sleep$date, "% m/% d/% y")
```

## Analyze: Putting the data to work.
The tables have a different amount of individuals. Since Weight and Heartrate have less than half of the individuals these tables were not taken into consideration. The analysis is focused on Calories, Intensities, Activity, and Sleep.

```{r}
Activity %>%
  summarise(Activity_participants = n_distinct(Activity$Id))

n_distinct(Calories$Id)
n_distinct(Intensities$Id)
n_distinct(Heartrate$Id)
n_distinct(Sleep$Id)
n_distinct(Weight$Id)
```

### Statistics of the 4 dataframes considered for analysis.
```{r}
Activity %>%  
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes, Calories) %>%
  summary()
```

```{r}
Intensities %>%
  select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes) %>%
  summary()
```

```{r}
Calories %>%
  select(Calories) %>%
  summary()
```

```{r}
Sleep %>%
  select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>%
  summary()
```

#### Merging data
```{r}
Combined_data_inner <- merge(Sleep, Activity, by="Id")
n_distinct(Combined_data_inner$Id)
```

```{r}
Combined_data_outer <- merge(Sleep, Activity, by="Id", all = TRUE)
n_distinct(Combined_data_outer$Id)
```

### Key Findings
- Sleep is 7 hours average.
- The average number of steps taken in a day is 8.329.
- The average calories burned in a day is 2.362.
- The average time asleep is 419 min (7 hr).
- The average sedentary time in a day is 497 min (8hr).

## Share: Data visualization.

### Total Steps

#### Total Steps vs. Sedentary Minutes

```{r, echo = TRUE}
ggplot(data=Activity, aes(x=TotalSteps, y=SedentaryMinutes)) + 
  geom_point() + geom_smooth() + labs(title="Total Steps vs. Sedentary Minutes")
```


#### Total Steps vs. Calories
```{r}
ggplot(data=Activity, aes(x=TotalSteps, y=Calories)) + 
  geom_point() + geom_smooth() + labs(title="Total Steps vs. Calories")
```

### Intensities

#### Intensities vs. Time
```{r}
Intensities$ActiveIntensity <- (Intensities$VeryActiveMinutes)/60

Combined_data <- merge(Weight, Intensities, by="Id", all=TRUE)
Combined_data$time <- format(Combined_data$Date, format = "%H:%M:%S")

ggplot(data=Combined_data, aes(x=time, y=ActiveIntensity)) + geom_histogram(stat = "identity", fill='purple3') +
  theme(axis.text.x = element_text(angle = 90)) +
  labs(title="Total very Active Intensity vs. Time ")
```
### Summary Analysis
- Positive correlation between Total Steps and Calories, the more steps done the more calories burned.
- The average sedentary time is very high; participants are lightly active.
- The more active the user, more calories burned.
- Most users are active before and after work hours.
- Users have only 23 very active minutes in the day.

## Act: Conclusions and marketing recommendations

*Final Conclusion: Most smart device users are sedentary which is not ideal for their health, sleep, and calories burned. This should be changed.*

*Target Audience: Bellabeat tracking devices are ideal for sedentary people like those who have full-time jobs. These devices could help them create healthy habits involving staying fit, frequent movement, and exercise*

Marketing Recommendations to the Bellabeat Marketing team
- Update notifications to remind users to stay active and follow a sleep schedule.
  - Increase in notifications before and after work hours so the users can actually work out.
- Offer an education platform so users are informed of the benefits of staying active and sleeping well.
- App must be updated to apply these changes.

Additional Data
- It is worth looking into other fitness tracker data of just women since this analysis used a very small sample of both genders.
- Data about needed exercise, calorie intake, and daily sleep for women should be looked into.
