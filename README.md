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
![Data recorded per weekday](https://github.com/user-attachments/assets/7b5332b6-1ec2-445c-a8b7-f7de477f2009)

Based on this plot, we notice that data recordings are significantly higher from Tuesdays to Thursdays, while the rest of the week keep a consistent number of recordings. This brings up the question of the reason behind this. However, given the data's third-party origin, it will be hard to find proof to make our speculations any more than that. Therefore, the results in this analysis should not be taken as absolute and definitive. This discovery further confirms the need to make a future analysis with more recent and reliable data. 

Other relevant weekly data:

```r
#Average steps per weekday
daily_data %>%
  group_by(Weekday) %>%
  summarise(Mean_Steps=mean(TotalSteps)) %>%
  ggplot(aes(x=Weekday,
             y=Mean_Steps))+
  geom_col(fill="lightblue")+
  labs(title="Average steps taken per weekday",
       x=NULL,
       y="Average Steps")+
  geom_text(aes(label=ceiling(Mean_Steps)),
            size=3,
            hjust=0.5,
            position=position_stack(vjust=1.05))
```
![Average steps taken per weekday](https://github.com/user-attachments/assets/c88b914d-dcc5-4cbe-8aa3-476f9cf70aca)

Tuesday and Saturdays are the days when the users walked the most steps, while Monday had a significantly lower number of steps taken. 

![Average time asleep per weekday](https://github.com/user-attachments/assets/9ab89768-34e0-4971-889b-eca90ef9b7e0)

Comparing the average time asleep and the steps taken, Sunday coincide with having the most time asleep and the least number of steps taken. Aditionally, on Tuesdays people also took a high number of steps while sleeping less time. However, this is not enough evidence to suggest a direct relationship between the variables.

```r
#Activity level per weekday
ggplot(Intensity_mean_long, aes(x=Weekday,
                                y=Mean_Minutes,
                                fill=Activity_Level))+
  geom_col()+
  labs(title="Activity Level per Weekday",
       x=NULL,
       y="Minutes")+
  guides(fill=guide_legend(title="Activity Level"))+
  scale_fill_discrete(labels=c("Very Active",
                               "Fairly Active",
                               "Lightly Active",
                               "Sedentary"))+
  geom_text(aes(label=ifelse(Mean_Minutes>100, Daily_Percent, '')), 
            size=3,
            hjust=0.5,
            position=position_stack(vjust=0.5))+
  geom_text(aes(label=after_stat(y), group=Weekday),
            stat="summary", fun=sum, vjust=1)
```
![Activity Level per Weekday](https://github.com/user-attachments/assets/cf495c41-6deb-405d-9c1e-543c56d754c1) 

Correspondingly with the weekly average step and sleep distribution, Sunday, which recorded the least steps taken and most time in bed, had the highest percentage of time doing sedentary activities (82.6%). On the other side, Saturdays, the first day of the conventional weekend, had the lowest percentage of sedentary activities (79.8%). Other interesting findings is that Mondays had the highest number of minutes recorded, but the second highest percentage of sedentary activities (81.8%). Even though Thursdays had the third highest number of overall  data recorded, it had the lowest minutes of activities recorded. Given that the number of users recorded is the same, it leads to conclude that 1) the overall data difference in overall recordings is due to other variables, like time asleep, and 2) the minutes of activity are dependant on the device's ability to identify a change in the users' heartbeat and blood pressure. 

Now, let's review some variables by hour.

*4.3 Review and plot hourly data*

```r
#Average Hourly Steps
hourly_steps %>%
  group_by(Time) %>%
  summarise(Average_Steps=mean(StepTotal)) %>%
  ggplot(aes(x=Time,
             y=Average_Steps,
             fill=Average_Steps))+
  geom_col()+
  labs(title="Average Step Distribution per Hour",
       x=NULL,
       y="Average Steps",
       fill=NULL)+
  theme(axis.text.x=element_text(angle=90))+
  scale_fill_viridis_c()
```
![Average Step Distribution per Hour](https://github.com/user-attachments/assets/c8df7818-1627-428a-97db-424d6f6c2f60) 

During the 12 hour "usual" work window, between 7 am and 7 pm, the number of hourly steps stay above 300, with 2 notable high windows: noon to 2 pm, and 5 to 7 pm. As expected, past 7 pm, as the night advances, the number of steps decreases. A significant dropoff exists between midnight and 4 am. Most people wake up between 5 and 7 am. 

![Average Calorie Distribution per Hour](https://github.com/user-attachments/assets/46cff603-42a3-431d-8da7-35a1ae05ac84) 

Similarly to the step distribution, there is a 12 hour window, from 8 am to 8 pm, where users burnt over 100 calories per hour, with notable high windows between noon to 2 pm and 5 to 7 pm. From 8 pm to midnight, there is a progressive decrease in calories burnt, until reaching constant values from midnight to 4 am. These constant values are consistent with the calories burnt during the normal sleeping process.

Next, let's review individual data. 

*4.4 Review and plot individual data*

```r
#Calories vs Total steps
ggplot(daily_data, aes(x=TotalSteps,
                       y=Calories,
                       color=VeryActiveMinutes))+
  geom_point()+
  geom_smooth(method="loess")+
  scale_color_gradient(low="green", high="red")+
  labs(title="Calories vs Total Steps per Day",
       x="Total Steps",
       caption="Based on daily data")
```
![Calories vs Total Steps](https://github.com/user-attachments/assets/c007a76e-4e67-4c28-a00a-736afe8dd37f) 

As predicted, there is a direct positive relationship between calories and steps taken. However, above the tendency line we observe cases where users had a similar amount of burnt calories but a varying number of steps taken. Notably, some users had a very high calories-steps ratio, meaning they burnt many calories while walking few steps. This could be due to either a non-registered high-intensity activity, a difference in the duration and intensity of physical activities (running vs walking 1 mile) or, in less degree, particular physiological conditions. 

There are some outlying cases where users had an exceptionally high number of steps taken but lower calories burned, which could suggest a long but low-intensity activity (like walking at a slow pace); others burned a high amount of calories while taking few steps, which could be associated with stationary but high-energy activities (like a gym workout). 
Overall, more data is needed into the type of physical activities or sports the users practice, to further identify preffered activities and calories/steps tendencies. 

```r
#Activity Minutes vs Total steps for each activity level
ggplot(daily_data)+
  #Very Active Minutes
  geom_point(aes(x=TotalSteps,
                 y=VeryActiveMinutes),
             color="navy")+ 
  geom_smooth(method="loess", aes(x=TotalSteps,
                                  y=VeryActiveMinutes),
              color="navy")+
#The full code can be found in the Markdown file
```
![Activity Minutes vs Daily Steps](https://github.com/user-attachments/assets/70b5e8fe-30e1-4a8b-8497-210f7e0ac332) 

Most users took less than 15,000 steps per day, regardless of however many minutes they spent at each intensity level. Nevertheless, in accordance with the Activity Level distribution plot, most time was spent in sedentary and lightly active activities. As noticed in the plot, people spent the less time doing fairly active activities. The outlying points refer to user who had few steps but high-intensity activities, or many steps with a lot of time spent in high-intensity activities.

![Activity Minutes vs Calories](https://github.com/user-attachments/assets/2ab762c1-9572-41d1-bc89-750eae0d48f5) 

Similarly to the previous plot, most users burned between 1,500 and 3,500 calories, regardless of however many miutes they spent at each intensity level.

```r
#Time Asleep vs Sedentary Minutes
ggplot(daily_data)+
  geom_point(aes(x=TotalMinutesAsleep,
                 y=SedentaryMinutes))+
  geom_smooth(method="loess", aes(x=TotalMinutesAsleep,
                                  y=SedentaryMinutes))+
  labs(title="Sleep Time vs Sedentary Minutes",
       x="Daily Total Minutes Asleep",
       y="Sedentary Minutes",
       caption="Based on reported data. NA values not considered")+
  annotate("text", x=750,
           y=350,
           label="italic(R)^2==0.3597",
           parse=TRUE,
           size=3)
```
![Sleep Time vs Sedentary Minutes](https://github.com/user-attachments/assets/4cb49a23-d983-4901-a401-c5657ef93fd8) 

It seems logical that a relationship could be possible between sleep time and sedentary minutes, as they relate to each other. Therefore, by plotting both variables, we look to find evidence to this assumption. Just based on the plot, there seems to be a downward tendency of minutes asleep and sedentary activities. However, confirmation is needed via a linear regression analysis. 

```r
#Linear Regression analysis for Time Asleep vs Sedentary Minutes
Asleep_vs_Sedentary <- lm(TotalMinutesAsleep ~ SedentaryMinutes, data=daily_data)

summary(Asleep_vs_Sedentary)
```

According to the analysis, the R squared value (0.3597) is too low to indicate a correlation between the variables. Therefore, and contrary to the original assumption, there is no more minutes asleep do not translate to more sedentary activities, despite the general tendency shown in the plot.

Next, let's review the types of users in the data

*4.5 Review user distribution based on activity levels*

Based on a research study by Catrine Tudor-Locke and David Bassett Jr (insert link), who propose an index for classification of physical activity based on pedometer measurements, we can, for analysis purposes, establish a classification that fits the four activity levels available in our data.

 less than 5,000 steps/day – Sedentary Lifestyle

 5,000 - 7,499 steps/day – Lightly Active Lifestyle

 7,500 - 10,000 steps/day – Fairly Active Lifestyle

 more than 10,000 steps/day – Very Active Lifestyle

Guided by this classification, how many users are in each class? 

```r
#Creating a data frame with average individual data
average_individual_data <- daily_data %>%
  group_by(Id) %>%
  summarise(av_steps=mean(TotalSteps),
            av_calories=mean(Calories),
            av_sleep=mean(TotalMinutesAsleep, na.rm=TRUE))

#Classifying users by level of intensity based on average daily steps
average_individual_data <- average_individual_data %>%
  mutate(user_class=case_when(
    av_steps < 5000 ~ "Sedentary",
    av_steps >= 5000 & av_steps < 7500 ~ "Lightly Active",
    av_steps >= 7500 & av_steps < 10000 ~ "Fairly Active",
    av_steps >= 10000 ~ "Very Active"))

#Checking the percentage of users in each class 
user_activity_class <- average_individual_data %>%
  group_by(user_class) %>%
  summarise(user_count=n()) %>%
  mutate(user_percentage=user_count/sum(user_count)*100)

user_activity_class$user_percentage=paste0(sprintf("%2.1f", user_activity_class$user_percentage), "%")

#Plotting activity class percentage
ggplot(user_activity_class, aes(x="",
                                y=user_percentage,
                                fill=user_class))+
  geom_col()+
  coord_polar("y", start=0)+
  geom_text(aes(label=user_percentage),
            position=position_stack(vjust=0.5),
            size=3)+
  guides(fill=guide_legend(title="User Class"))+
  scale_fill_discrete(breaks=c("Lightly Active",
                               "Fairly Active",
                               "Sedentary",
                               "Very Active"))+
  theme_void()
```
![User Distribution based on Activity Level](https://github.com/user-attachments/assets/e2f64542-d6a4-4694-8d31-2371797b0a9d) 

We notice that most users are lightly active or fairly active, meaning they take between 5,000 and 10,000 daily steps. The rest of the users are either sedentary or very active. The mostly equal distribution of users is due to the low number of unique users (30). 

*4.6 Data Frames Summary*

Finally, to better keep track of the data used in this analysis, next is presented a summary of the original data frames and the ones created throughout the analysis. 

```r
project_data_frames <- data.frame(Initial_data_frames=c("daily_activity", "daily_calories", "daily_intensities", "daily_steps", "hourly_calories", "hourly_steps", "sleep", "weight","","",""), Final_data_frames=c("daily_activity", "daily_calories", "daily_intensities", "daily_steps", "hourly_calories", "hourly_steps", "sleep", "average_individual_data", "daily_data", "Intensity_mean", "Intensity_mean_long"))
library(knitr)

kable(project_data_frames, caption="Data frames summary")
```

| Initial data frames | Final data frames |
|:---:|:---:|
| daily_activity | daily_activity |
| daily_calories | daily_calories |
| daily_intensities | daily_intensities |
| daily_steps | daily_steps | 
| hourly_steps | hourly_steps | 
| sleep | sleep |
| weight | average_individual_data |
| | daily_data |
| | Intensity_mean | 
| | Intensity_mean_long |

## Stage 5. Share 

The dashboard for this project was made using Tableau Public and is available as [Bellabeat Case Study (Google Capstone)](https://public.tableau.com/views/BellabeatCaseStudyGoogleCapstone/BellabeatCaseStudyGoogleCapstone?:language=es-ES&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

<div class='tableauPlaceholder' id='viz1737055379430' style='position: relative'><noscript><a href='#'><img alt='Bellabeat Case Study (Google Capstone) ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Be&#47;BellabeatCaseStudyGoogleCapstone&#47;BellabeatCaseStudyGoogleCapstone&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='BellabeatCaseStudyGoogleCapstone&#47;BellabeatCaseStudyGoogleCapstone' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Be&#47;BellabeatCaseStudyGoogleCapstone&#47;BellabeatCaseStudyGoogleCapstone&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='es-ES' /></object></div>              

## Stage 6. Act

Analyze the smart device usage data of non-Bellabeat users to gain insight into the current usage trends. Using this data, create high-level recommendations applied to one Bellabeat product that will guide the company's marketing strategy. 

Based on the analysis performed with the available data, the following insights into the current usage trends were found, and in consequence, some final high-level recommendations for Bellabeat´s marketing strategy are made. 


**What are the current usage trends?**
- Most users are lightly active or fairly active, meaning they take bewtween 5k and 10k daily steps.
- Saturdays are the most active day, with the least percentage of sedentary time, while Sundays have the highest percentage of sedentary time recorded by users.
- People sleep the most on Sundays and Wednesdays.
- Majority of people wake up after 6 am, and go to bed after 11 pm.
- People are most active (measured by steps taken and calories burnt) between 5 and 7 pm, and between noon and 2 pm.
- Before noon and after 3 pm, people spend long sedentary times, with a constant number of steps taken per hour. 

**Final Recommendations**

1. The data used for this analysis may not accurately reflect the current trends. Given that Bellabeat's products are thought for and designed for women, I recommend obtaining more data that is recent, and which considers socioeconomical factors, such as gender, location and age.

2. Simplify the weight tracking features in the "Leaf" tracker, as not enough people manually enter their data.

3. Promote a special campaign that encourages women to do lightly active or higher activities on Sundays, such as point-based reward system or weekly streaks rewards.

4. Include a dietary program to sync with people's workout regimes. During weekdays, people are most active around lunch and before dinner.

5. The "Leaf" tracker can include a reminder function that encourages people to walk for a few minutes after long periods of inactivity, and to go to bed if active after 11 pm.

