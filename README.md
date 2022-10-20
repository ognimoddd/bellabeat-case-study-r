![BellabeatLogo](https://user-images.githubusercontent.com/77591203/196562658-bfe5df3b-4e68-4c4e-97b8-d9c057d28dec.jpg)


## Introduction 

Bellabeat is a high-tech manufacturer of health-focused products for women. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits.
Although Bellabeat is a successful small company, they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company.

## Step 1: Ask ❓

## Business Task 
To identify potential opportunities for growth and provide recommendations for the Bellabeat marketing strategy improvement based on trends in smart device usage.

**Key Stakeholders:** 

* Urška Sršen: Bellabeat's cofounder and Chief Creative Officer
* Sando Mur: Mathematician and Bellabeat’s co-founder

**Questions to explore for the analysis:**

1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

## Step 2: Prepare 💻

The data being used in this case study can be found here: [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) CC0: Public Domain, dataset made available through [Mobius](https://www.kaggle.com/arashnic)

The data is stored and uploaded in R Studio. This Kaggle data set contains personal fitness tracker from thirty fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

The data set contains 18 CSV files organized in long format. Below is a breakdown of the data using the ROCCC approach:

* **Reliability - LOW:** The data comes from 30 fitbit users who consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.
* **Original - LOW:** Third party data collect using Amazon Mechanical Turk.
* **Comprehensive - MED:** The dataset contains multiple fields on daily activity intensity, calories used, daily steps taken, daily sleep time and weight record.
* **Current - LOW:** This data is from March 2016 through May 2016. The data is not current, meaning that user habits may have changed over the years.
* **Cited - LOW:** Data was collected from a third party, therefore unknown.

### Loading packages

```
library(tidyverse)
library(lubridate) 
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
```

## Step 3: Process 🛠

### Importing the datasets

```
# Read the dataframes
activity <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
calories <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyCalories_merged.csv")
intensities <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/hourlyIntensities_merged.csv")
sleep <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
weight <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

Now that I have imported all of the dataframes I will be using, I am ready to start cleaning the data. I will take a look at each data frame to familiarize myself with the data and to check for errors.

```
head(activity)
```

![HeadActivity](https://user-images.githubusercontent.com/77591203/196822184-bdaf0ad6-e22b-4000-85ab-43b647e28727.png)
![HeadActivity2](https://user-images.githubusercontent.com/77591203/196822994-6942b851-0a5a-4cbb-bdcf-9282fede5721.png)
![HeadActivity3](https://user-images.githubusercontent.com/77591203/196823320-d93655c5-131a-4899-beb9-b545c8679d0e.png)

```
colnames(activity)
```


'Id''ActivityDate''TotalSteps''TotalDistance''TrackerDistance''LoggedActivitiesDistance''VeryActiveDistance''ModeratelyActiveDistance''LightActiveDistance''SedentaryActiveDistance''VeryActiveMinutes''FairlyActiveMinutes''LightlyActiveMinutes''SedentaryMinutes''Calories'


```
head(weight)
```

![HeadWeight](https://user-images.githubusercontent.com/77591203/196824510-8dc1df1e-c227-4f09-bd42-fc92af669643.png)

```
colnames(weight)
```


'Id''Date''WeightKg''WeightPounds''Fat''BMI''IsManualReport''LogId'
<br>
<br>
Upon looking a the timestamp data, I noticed some problems. To solve this, I will convert it to date time format.

```
# intensities
intensities$ActivityHour=as.POSIXct(intensities$ActivityHour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
intensities$time <- format(intensities$ActivityHour, format = "%H:%M:%S")
intensities$date <- format(intensities$ActivityHour, format = "%m/%d/%y")
# activity
activity$ActivityDate=as.POSIXct(activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone())
activity$date <- format(activity$ActivityDate, format = "%m/%d/%y")
# sleep
sleep$SleepDay=as.POSIXct(sleep$SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
sleep$date <- format(sleep$SleepDay, format = "%m/%d/%y")
```

Now that the dates are properly formatted and all dataframes have been checked for errors, I can start investigating the data.
<br>
<br>
## Step 4: Analyze 🔬


To beging the analysis phase, I will first see how many participants there are in each category.

```
# Finding number of participants in each category
n_distinct(activity$Id)  
n_distinct(calories$Id)   
n_distinct(intensities$Id)
n_distinct(sleep$Id)
n_distinct(weight$Id)
```

33 <br>
33 <br>
33 <br>
24 <br>
8 <br>

To summarize the above data, there are 33 participants in the activity, calories, and intensities datasets, 24 in the sleep dataset, and only 8 in the weight dataset. The fact that there are only 8 participants in the weight dataset means that more data would be needed to make a strong reccomendation or conclusion.

```
# checking for significant change in weight
weight%>%
group_by(Id)%>%
summarise(min(WeightKg),max(WeightKg))
```

![ChangeInWeight](https://user-images.githubusercontent.com/77591203/196825550-56373a3c-13d9-4338-9806-19c3436273b8.png)

There are no significant changes in weight coming from these 8 participants. Due to the combination of low sample size and lack of variable change, I decided to not use this dataset further.

Let's have a look at the summaries for the rest of the datasets:

```
# activity
activity %>%  
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes, Calories) %>%
  summary()

# active minutes per category
activity %>%
  select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes) %>%
  summary()

# calories
calories %>%
  select(Calories) %>%
  summary()
# sleep
sleep %>%
  select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>%
  summary()
# weight
weight %>%
  select(WeightKg, BMI) %>%
  summary()
```

![Summaries](https://user-images.githubusercontent.com/77591203/196825950-399f2048-ea32-4560-b4f4-91a395d079b9.png)

### Observations made from the above summaries:

* Sedetary minutes on average is 16.5 hours 
* The average number of steps per day is 7638. The CDC recommends people take 10,000 steps daily.
* The majority of the participants are lightly active.
* The average participant burns 97 calories per hour
* On average, participants sleep for 7 hours.

### Merging Data

Before visualizing the data, two of the datasets will be merged. I will inner join the activity and sleep datasets on columns Id and date.

```
merged_data <- merge(sleep, activity, by = c('Id', 'date'))
head(merged_data) 
```
![MergedData](https://user-images.githubusercontent.com/77591203/196826489-0dc398f1-a3b5-4d35-9821-d98e10cfc279.png)
![MergedData2](https://user-images.githubusercontent.com/77591203/196826615-bcfe1e97-c012-4925-be30-511315abeaae.png)
![MergedData3](https://user-images.githubusercontent.com/77591203/196826760-53a97d4c-f1f2-444a-94a7-e60c66415120.png)
![MergedData4](https://user-images.githubusercontent.com/77591203/196826888-ae1195f0-a406-48b6-8237-6890cdc1ea67.png)
