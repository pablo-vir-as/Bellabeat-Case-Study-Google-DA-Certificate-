# Bellabeat Case Study (Google Data Analytics Certification)

## Introduction - The scenario

Bellabeat is a high-tech manufacturer of health-focused products for women. Founded in 2013 as a joint venture by an artist and a mathematician, Bellabeat´s goal is to empower women with their own health and habits through the use of smart health devices.

In 2016, Bellabeat had invested in traditional advertising media (radio, print, TV) but heavily focused on digital marketing. The company invested in Google Search, engaged with customers in social media, runned video ads on Youtube and displayed ads on the Google Display Network.

In order to further expand in the global smart device market, the company's executive team asked the marketing analytics team to look into the consumer's usage data and come up with high-level recommendations for Bellabeat's marketing strategy.

## Stage 1 - Ask

**Business Task**

Analyze the smart device usage data of non-Bellabeat users to gain insight into the current usage trends. Using this data, create high-level recommendations applied to one Bellabeat product that will guide the company's marketing strategy. 
Key stakeholders: 
- Urška Sršen - Bellabeat's cofounder and CCO
- Sando Mur: Bellabeat's cofounder and member of the executive team.
- Bellabeat marketing analytics team 

## Stage 2 - Prepare

The data available for the project is the Fitbit Fitness Tracker Data, made available through Mobius. The dataset consists of personal fitness tracker from 30 Fitbit users, who answered via a survey distributed by Amazon Mechanical Turk from March to May 2016. The data included is minute-level, and organized in wide format.

The dataset is comprised of 18 CSV files. A ROCCC analysis of the data is possible: 

**R** - Reliability: **LOW**. The data suffers from a sampling bias, as it only has data from 30 different users. Even though the sample size is just enough to perform an analysis, the results will not represent the non-Bellabeat smart device user population.

**O** -  Originality: **MED**. The data is from a third party, but its origin is clearly indicated. The data is from 30 Fitbit users who consented to the submission of personal tracker data, and was collected via a survey by Amazon Mechanical Turk and made available by Mobius.

**C** - Comprehensiveness: **LOW**. The data contains minute-level information on health habits, but is not available for all days of the week not it includes socioeconomical or geographical information. This will limit the number and type of insights achievable with this data.

**C** - Current: **LOW**. The data is from 8 years ago, and has not been updated since. The results derived from the analysis will not reflect the current trends.

**C** - Cited: **MED**. The data's origin is clearly stated, but as a public, third-party provided dataset, the possibility of uncited changes is to be taken into account.

## Stage 3 - Process

Due to the amount of data available, the data cleaning and analysis will be performed using RStudio.

### Step 1.
Install the packages.
```r
library(tidyverse) #system of packages
library(readr) #data import
library(tidyr) #data cleaning
library(janitor) #data manipulation
library(plyr) #data manipulation
library(lubridate) #date-time manipulation
library(dplyr) #data manipulation
library(ggplot2) #data viz
library(cowplot) #data viz features
```
### Step 2.
Import the data.
```r
#Setting a working directory
setwd("C:/Amryt/Data Analytics/Case study/BellaBeat case study/Fitabase Data 4.12.16-5.12.16") 

#Importing each table
daily_activity <- read.csv("dailyActivity_merged.csv") 
hourly_calories <- read.csv("hourlyCalories_merged.csv")
hourly_steps <- read.csv("hourlySteps_merged.csv")
sleep <- read.csv("sleepDay_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
```
### Step 3.

*3.1 Checking for NA values and duplicates*
```r
#Checking for NA values
sum(is.na(daily_activity))
sum(is.na(hourly_calories))
sum(is.na(hourly_steps))
sum(is.na(sleep))
sum(is.na(weight))

#Checking for duplicates
sum(duplicated(daily_activity))
sum(duplicated(hourly_calories))
sum(duplicated(hourly_steps))
sum(duplicated(sleep))
sum(duplicated(weight))
```
From this analysis, one can notice that the *weight* data frame has 65 NA values, that correspond to the "Fat" column; the *sleep* data frame has 3 duplicate rows. Given that the *weight* data frame do have BMI data for all users, the lack of Fat data has minor impact on this analysis. However, the duplicate *sleep* rows have to be eliminated.

```r
#Dropping the duplicate values found in table "sleep"
sleep <- unique(sleep)

#Verifying duplicate values were dropped
sum(duplicated(sleep))
```
Next, one must verify that the data from unique users in each table is enought to make relevant insights from it.

```r
#Checking the number of unique users in the table
n_distinct(daily_activity$Id) 
n_distinct(hourly_calories$Id)
n_distinct(hourly_steps$Id)
n_distinct(sleep$Id)
n_distinct(weight$Id)
```
The analysis reveals that all but the sleep and weight tables have 33 unique users. The former has 24 unique users, while the latter only has 8. Given the small amount of unique users in the weight table, its information is not reliable and thus will be dropped from the analysis.

*3.2 Checking data content and format*

The next step is to review in detail the data and its format.

```r
#Reviewing each table
head(daily_activity) 
head(hourly_calories)
head(hourly_steps)
head(sleep)

#Checking that each table is properly formatted
str(daily_activity) 
str(hourly_calories)
str(hourly_steps)
str(sleep)
```
After reviewing all data, various details come up. 

a. Date column names differ accross all tables. For clarity, all relevant columns must be renamed.

b. The time section of the SleepDay column in the Sleep table is irrelevant to the analysis, as all data was obtained at the same time (12:00:00 AM). By deleting this info, it will convert from a datetime class to a date.

c. The dates and datetimes in all tables are formatted as characters. They must be converted to "date" and "POSIXct" formats, correspondingly.

d. For future ease when analyzing the data, the datetimes columns should be split into time and date columns.

e. The IDs are saved as numbers, instead of characters.

```r
#a. Renaming date columns
daily_activity <- rename(daily_activity, Date = ActivityDate)
hourly_calories <- rename(hourly_calories, Datetime= ActivityHour)
hourly_steps <- rename(hourly_steps, Datetime= ActivityHour)
sleep <- rename(sleep, Date = SleepDay)

#b. Deleting the time section in the Sleep table
sleep$Date <- gsub("12:00:00 AM", "", sleep$Date)

#c. Converting character format to date format
daily_activity$Date <- as_date(daily_activity$Date, format="%m/%d/%Y")
sleep$Date <- mdy(sleep$Date)

#c. Converting character format to POSIXct format
hourly_calories$Datetime <- convert_to_datetime(hourly_calories$Datetime, tz="UTC", character_fun = lubridate::mdy_hms)
hourly_steps$Datetime <- convert_to_datetime(hourly_steps$Datetime, tz="UTC", character_fun = lubridate::mdy_hms)

#d. Splitting the datetime columns
hourly_calories$Time <- format(as.POSIXct(hourly_calories$Datetime), format="%H:%M:%S")
hourly_calories$Date <- as.Date(hourly_calories$Datetime)
hourly_steps$Time <- format(as.POSIXct(hourly_steps$Datetime), format="%H:%M:%S")
hourly_steps$Date <- as.Date(hourly_steps$Datetime)

#e. Converting number format to character format
daily_activity$Id <- as.character(daily_activity$Id)
sleep$Id <- as.character(sleep$Id)
hourly_calories$Id <- as.character(hourly_calories$Id)
hourly_steps$Id <- as.character(hourly_steps$Id)
```
Now that the data is clean, some tables can be merged to facilitate the data manipulation during the next stage, and a new "Weekday" column can be added to the merged data frame.

*3.3 Merge the data frames and add a new Weekday column*

```r
#Merging the daily data
daily_data <- merge(daily_activity, sleep, by=c("Id", "Date"), all=TRUE)

#Creating a new Weekday column in the daily_data data frame
daily_data <- daily_data %>%
  mutate(Weekday=wday(daily_data$Date, label=TRUE, abbr=FALSE, locale="UTC"))
```

## Stage 4 - Analyze

Firstly, one can calculate basic statistics, such as mean, min and max values of the daily data.

*4.1 Basic statistics*

```r
#Calculating basic statistics
summary(daily_data)
summary(sleep)
```

From this, one find that the average total daily steps per person is 7,638 while traveling an average distance of 5.49 km. On average, people spent most of their time (16h 31 min) doing sedentary activities, followed by lightly active (3h 13 min), very active (21 minutes) and fairly active activities (14 minutes). From this activity distribution, each person burnt an average of 2,304 calories per day. Regarding their rest, people slept an average of almost 7h per day but spent an additional 38 minutes in bed (most likely just laying down).

To gain insight into the use of these devices, it will be useful to see how they are used by day of the week.

*4.2 Review and plot weekly data*

```r
#Number of data entries per weekday
ggplot(daily_data, aes(x=Weekday))+
  geom_bar(fill="orangered1")+
  labs(title="Data recorded per weekday",
       x=NULL,
       y="Number of data entries")+
  geom_text(aes(label=after_stat(count)),
            stat="count",
            size=3,
            position=position_stack(vjust=1.05))
```



