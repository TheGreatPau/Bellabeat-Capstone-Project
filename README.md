# **Bellabeat Capstone Project**
##### Author: **Paulo Escalante**
##### Date: **11 June 2022**
##### *This is my capstone project to complete [Google Data Analytics](https://www.coursera.org/professional-certificates/google-data-analytics#courses) course. Your feedback would be highly appreciated. :)*

### **Introduction**

Bellabeat is a high-tech manufacturer of health-focused products for women. Bellabeat is a successful small company, but has the potential to become a larger player in the global smart device market. Urška Sršen, co-founder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. The team have been asked to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The insights that will be discovered will then help guide marketing strategy for the company. 

### **Ask**

The main objective of this project is to focus on a Bellabeat product and analyze smart device usage data in order to gain insight into how people are already using their smart devices. Then, using this information, the insights gained will be used to produce high-level recommendations for how these trends can inform Bellabeat marketing strategy. Bellabeat is currently a small company but has the potential to become a larger player in the global smart device market and hoping this project may help progress to that level.

### **Prepare**

* The data was stored via Google’s BigQuery. Pre-cleaning was done through Microsoft Excel.
* Data source is from [Kaggle](https://www.kaggle.com/datasets/arashnic/fitbit). Refer to this [link](https://www.fitabase.com/media/1930/fitabasedatadictionary102320.pdf) for the documentation of the dataset.
* Dataset is not that reliable as this was conducted by a third party.
* Not all critical information is included. Demographic information such as gender was not included, which is critical to this project as Bellabeat’s product line caters for women. It also only has 33 participants, which is too few to make a comprehensive analysis.
* Data is outdated for 6 years now as the study was conducted in 2016.

### **Process**

The tools that were used in the process phase of this project are Google’s BigQuery and Microsoft Excel. Pre-cleaning of the data was done through Microsoft Excel before the data were stored to BigQuery. 

* An error with regards to the date time columns was appearing when uploading the dataset to BigQuery. The following were done via Microsoft Excel so BigQuery could automatically detect datetime values:
* Converted datasets into table and loaded into Microsoft Excel’s built-in Power Query.
* Splitted datetime column into two columns by space delimiter to segregate date and time.
* Splitted date column into 3 columns - day, month, and year - by slash delimiter.
* Merged day, month, and year into one column rearranged into dd/mm/yyyy format.
* Merged the time column into the date column to create a new datetime column with dd/mm/yyyy hh:mm format.
* Loaded cleaned dataset into a new sheet, saved as CSV, and uploaded into BigQuery.

After uploading the cleaned dataset to BigQuery, the following were done to ensure that the data is clean before proceeding to the Analyze phase:

``` sql
-- View and examine tables

-- 940 records from Daily Activity table
SELECT *
FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity

-- Only 413 records from Sleep table
SELECT *
FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay

-- 1,325,580 records from Sleep table
SELECT *
FROM `gda-course-4-332812.Capstone.MinuteMETs` AS MinuteMETS

-- Hourly intensities, calories, and steps each have 22,099 rows
SELECT *
FROM `gda-course-4-332812.Capstone.HourlyIntensities` AS HourlyIntensities

SELECT *
FROM `gda-course-4-332812.Capstone.HourlyCalories` AS HourlyCalories

SELECT *
FROM `gda-course-4-332812.Capstone.HourlySteps` AS HourlySteps

-- Check for the number of unique participants

-- There are 33 unique participants in Daily Activities table
SELECT COUNT(DISTINCT Id) AS UserCount 
FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity

-- There are only 24 unique participants with sleep data
SELECT COUNT(DISTINCT Id) AS UserCount
FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay

-- 33 unique participants from METs table.
SELECT COUNT(DISTINCT Id) AS UserCount
FROM `gda-course-4-332812.Capstone.MinuteMETs` AS MinuteMETS

-- Hourly tables each have 33 participants, matching with Daily Activity table.
SELECT COUNT(DISTINCT Id) AS UserCount 
FROM `gda-course-4-332812.Capstone.HourlyIntensities` AS HourlyIntensities

SELECT COUNT(DISTINCT Id) AS UserCount 
FROM `gda-course-4-332812.Capstone.HourlyCalories` AS HourlyCalories

SELECT COUNT(DISTINCT Id) AS UserCount 
FROM `gda-course-4-332812.Capstone.HourlySteps` AS HourlySteps

-- HeartRate table only has 7 participants while WeightLogInfo has 8 participants, out of 33. These tables will not be used for this analysis due to this limitation.
-- Check for duplicates.

-- No duplicates found on DailyActivities table
SELECT Id, 
  ActivityDate, 
  TotalSteps, 
  TotalDistance, 
  TrackerDistance, 
  LoggedActivitiesDistance,	
  VeryActiveDistance,	
  ModeratelyActiveDistance,	
  LightActiveDistance,	
  SedentaryActiveDistance,	
  VeryActiveMinutes,	
  FairlyActiveMinutes,	
  LightlyActiveMinutes,	
  SedentaryMinutes,	
  Calories,
  COUNT(*) AS Dupes
FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity
GROUP BY Id, 
  ActivityDate, 
  TotalSteps, 
  TotalDistance, 
  TrackerDistance, 
  LoggedActivitiesDistance,	
  VeryActiveDistance,	
  ModeratelyActiveDistance,	
  LightActiveDistance,	
  SedentaryActiveDistance,	
  VeryActiveMinutes,	
  FairlyActiveMinutes,	
  LightlyActiveMinutes,	
  SedentaryMinutes,	
  Calories
HAVING Dupes > 1

-- Found 3 duplicates from SleepDay table, removed those duplicates.
SELECT
  Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed,
  COUNT(*) AS Dupes
FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay1
GROUP BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed
HAVING Dupes = 1
ORDER BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed

-- Created user no table to clearly identify unique participant
-- Merged Daily Activity, Sleep Day, and User No tables.
-- Nulls have been set to 0.
SELECT 
  DailyActivity.Id,
  UserNumberTable.UserNo,
  ActivityDate, 
  SUM(TotalSteps) AS Total_Steps, 
  SUM(TotalDistance) AS Total_Distance, 
  SUM(TrackerDistance) AS Total_Tracker_Distance, 
  SUM(LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,	
  SUM(VeryActiveDistance) AS Total_VeryActiveDistance,	
  SUM(ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,	
  SUM(LightActiveDistance) AS Total_LightActiveDistance,	
  SUM(SedentaryActiveDistance) AS Total_SedentaryActiveDistance,	
  SUM(VeryActiveMinutes) AS Total_VeryActiveMinutes,
  SUM(FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
  SUM(LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
  SUM(SedentaryMinutes) AS Total_SedentaryMinutes,
  SUM(Calories) AS Total_Calories,
  IFNULL(SUM(TotalSleepRecords),0) AS Total_SleepRecords,
  IFNULL(SUM(TotalMinutesAsleep),0) AS Total_MinutesAsleep,
  IFNULL(SUM(TotalTimeInBed),0) AS Total_TimeInBed,
FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity
LEFT JOIN 
(SELECT
  Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed,
  COUNT(*) AS Dupes
FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay1
GROUP BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed
HAVING Dupes = 1
ORDER BY Id,
  SleepDay,
  TotalSleepRecords,
  TotalMinutesAsleep,
  TotalTimeInBed) AS SleepDay1
ON DailyActivity.Id = SleepDay1.Id
AND DailyActivity.ActivityDate = SleepDay1.SleepDay
LEFT JOIN `gda-course-4-332812.Capstone.UserNumberTable` AS UserNumberTable
ON DailyActivity.Id = UserNumberTable.Id
GROUP BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate
ORDER BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate

--Hourly intensity data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT HourlyIntensities.Id, HourlyIntensities.ActivityHour, SUM(TotalIntensity) AS Total_Intensity, AVG(AverageIntensity) AS Avg_Intensity, 
    EXTRACT(YEAR FROM HourlyIntensities.ActivityHour) AS Year,
    EXTRACT(MONTH FROM HourlyIntensities.ActivityHour) AS Month,
    EXTRACT(DAY FROM HourlyIntensities.ActivityHour) AS Day,
    EXTRACT(DAYOFWEEK FROM HourlyIntensities.ActivityHour) AS DayName,
    EXTRACT(Hour FROM HourlyIntensities.ActivityHour) AS Hour,
    COUNT(*) AS Duplicates
FROM `gda-course-4-332812.Capstone.HourlyIntensities` AS HourlyIntensities
GROUP BY HourlyIntensities.Id, HourlyIntensities.ActivityHour
ORDER BY HourlyIntensities.Id, HourlyIntensities.ActivityHour

--Hourly calories data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT Id, ActivityHour, SUM(Calories) AS Total_Calories,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
FROM `gda-course-4-332812.Capstone.HourlyCalories` AS Hourly_Calories
GROUP BY Id, ActivityHour
ORDER BY Id, ActivityHour

--Hourly steps data, extracted year, month, day, day name, hour
-- No duplicates found
SELECT Id, ActivityHour, SUM(StepTotal) AS Total_Steps,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
FROM `gda-course-4-332812.Capstone.HourlySteps` AS HourlySteps
GROUP BY Id, ActivityHour
ORDER BY Id, ActivityHour

--Merged 3 hourly tables into one
SELECT HourlyIntensities.Id, HourlyIntensities.ActivityHour, SUM(TotalIntensity) AS Total_Intensity, AVG(AverageIntensity) AS Avg_Intensity, 
  SUM(HourlyCalories.Total_Calories) AS TotalCalories,
  SUM(HourlySteps.Total_Steps) AS TotalSteps,
  EXTRACT(DATE FROM HourlyIntensities.ActivityHour) AS Date,
  EXTRACT(YEAR FROM HourlyIntensities.ActivityHour) AS Year,
  EXTRACT(MONTH FROM HourlyIntensities.ActivityHour) AS Month,
  EXTRACT(DAY FROM HourlyIntensities.ActivityHour) AS Day,
  FORMAT_DATE('%A', DATE(HourlyIntensities.ActivityHour)) AS DayName,
  CAST(EXTRACT(TIME FROM HourlyIntensities.ActivityHour) AS TIME) AS Time,
  EXTRACT(HOUR FROM HourlyIntensities.ActivityHour) AS Hour
FROM `gda-course-4-332812.Capstone.HourlyIntensities` AS HourlyIntensities
LEFT JOIN 
  (SELECT Id, ActivityHour, SUM(Calories) AS Total_Calories,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAYOFWEEK FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  FROM `gda-course-4-332812.Capstone.HourlyCalories` AS Hourly_Calories
  GROUP BY Id, ActivityHour) AS HourlyCalories
ON HourlyIntensities.Id = HourlyCalories.Id
AND HourlyIntensities.ActivityHour = HourlyCalories.ActivityHour
LEFT JOIN 
  (SELECT Id, ActivityHour, SUM(StepTotal) AS Total_Steps,
  EXTRACT(YEAR FROM ActivityHour) AS Year,
  EXTRACT(MONTH FROM ActivityHour) AS Month,
  EXTRACT(DAY FROM ActivityHour) AS Day,
  EXTRACT(DAY FROM ActivityHour) AS DayName,
  EXTRACT(Hour FROM ActivityHour) AS Hour,
  COUNT(*) AS Duplicates
  FROM `gda-course-4-332812.Capstone.HourlySteps` AS Hourly_Steps
  GROUP BY Id, ActivityHour
  ORDER BY Id, ActivityHour) AS HourlySteps
ON HourlyIntensities.Id = HourlySteps.Id
AND HourlyIntensities.ActivityHour = HourlySteps.ActivityHour
GROUP BY HourlyIntensities.Id, HourlyIntensities.ActivityHour
ORDER BY HourlyIntensities.Id, HourlyIntensities.ActivityHour

--METs per minute, extracted year, month, day, day name, hour 
-- No duplicates found 
-- Aggregated into METs per day
WITH Minute_METs AS (SELECT Id, SUM(METs) AS METs,
EXTRACT(DATE FROM ActivityMinute) AS Date,
  EXTRACT(YEAR FROM ActivityMinute) AS Year,
  EXTRACT(MONTH FROM ActivityMinute) AS Month,
  EXTRACT(DAY FROM ActivityMinute) AS Day,
  FORMAT_DATE('%A', DATE(ActivityMinute)) AS DayName,
  CAST(EXTRACT(TIME FROM ActivityMinute) AS TIME) AS Time,
  EXTRACT(HOUR FROM ActivityMinute) AS Hour,
  COUNT(*) AS Duplicates
FROM `gda-course-4-332812.Capstone.MinuteMETs` AS MinuteMETS
GROUP BY Id, ActivityMinute
ORDER BY Id, ActivityMinute)

SELECT Id, Date, SUM(METs) AS Total_METs, ROUND(AVG(METs), 2) AS Avg_METs,
  EXTRACT(YEAR FROM Date) AS Year,
  EXTRACT(MONTH FROM Date) AS Month,
  EXTRACT(DAY FROM Date) AS Day,
  FORMAT_DATE('%A', DATE(Date)) AS DayName
FROM Minute_METs
GROUP BY Id, Date
```

To summarize:
* Tables **daily intensities, daily calories,** and **daily steps** were not used in this project as data points from these tables were already included in the Daily Activities table.
* A total of **33** participants were found among all tables except the SleepDay table, which only had 24 participants.
* **HeartRate** and **WeightLogInfo** would’ve been useful in this analysis, but were also not utilized due to the lack of participants.
* No duplicates found among all tables except the SleepDay table. Duplicates were successfully removed.
* DailyActivity and SleepDay tables were combined into one table.
* Hourly tables - Hourly Intensities, Hourly Calories, and Hourly Steps - were combined into one table.
* HourlyMETs table was aggregated into daily METs.

### **Analyze**

First, I analyzed the tables by creating summary statistics.

``` sql
-- Analyze Phase. Created summary stats

WITH DailyActivity AS (
  SELECT 
    DailyActivity.Id,
    UserNumberTable.UserNo,
    ActivityDate, 
    SUM(TotalSteps) AS Total_Steps, 
    SUM(TotalDistance) AS Total_Distance, 
    SUM(TrackerDistance) AS Total_Tracker_Distance, 
    SUM(LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,	
    SUM(VeryActiveDistance) AS Total_VeryActiveDistance,	
    SUM(ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,	
    SUM(LightActiveDistance) AS Total_LightActiveDistance,	
    SUM(SedentaryActiveDistance) AS Total_SedentaryActiveDistance,	
    SUM(VeryActiveMinutes) AS Total_VeryActiveMinutes,
    SUM(FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
    SUM(LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
    SUM(SedentaryMinutes) AS Total_SedentaryMinutes,
    SUM(Calories) AS Total_Calories,
    IFNULL(SUM(TotalSleepRecords),0) AS Total_SleepRecords,
    IFNULL(SUM(TotalMinutesAsleep),0) AS Total_MinutesAsleep,
    IFNULL(SUM(TotalTimeInBed),0) AS Total_TimeInBed,
  FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity
  LEFT JOIN 
    (SELECT
      Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed,
      COUNT(*) AS Dupes
    FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay1
    GROUP BY Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed
    HAVING Dupes = 1
    ORDER BY Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed) AS SleepDay1
  ON DailyActivity.Id = SleepDay1.Id
  AND DailyActivity.ActivityDate = SleepDay1.SleepDay
  LEFT JOIN `gda-course-4-332812.Capstone.UserNumberTable` AS UserNumberTable
  ON DailyActivity.Id = UserNumberTable.Id
  GROUP BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate
  ORDER BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate
)
SELECT 
  AVG(Total_Steps) AS AvgSteps,
  AVG(Total_Distance) AS AvgDistance,
  AVG(Total_VeryActiveMinutes) AS AvgVeryActiveMins,
  SUM(Total_VeryActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS VeryActivePerc,
  AVG(Total_FairlyActiveMinutes) AS AvgFairlyActiveMins,
  SUM(Total_FairlyActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS FairlyPerc,
  AVG(Total_LightlyActiveMinutes) AS AvgLightlyActiveMins,
  SUM(Total_LightlyActiveMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS LightlyActivePerc,
  AVG(Total_SedentaryMinutes) AS AvgSedentaryMins,
  SUM(Total_SedentaryMinutes) / SUM(Total_VeryActiveMinutes + Total_FairlyActiveMinutes + Total_LightlyActiveMinutes + Total_SedentaryMinutes) AS SedentaryPerc,
  AVG(Total_Calories) AS AvgCalories,
  SUM(Total_MinutesAsleep) / SUM(CASE WHEN Total_MinutesAsleep > 0 THEN 1 END) AS AvgMinsAsleep,
  SUM(Total_TimeInBed) / SUM(CASE WHEN Total_TimeInBed > 0 THEN 1 END) AS AvgBedTime
FROM DailyActivity
```

Key Insights:
* The average daily steps by all the product users in this study is **7,637 steps per day**. As per [UC Davis Health](https://health.ucdavis.edu/internalmedicine/cardio/pdf/WomensHeartHealth/2015/10000-steps-a-day.pdf), the recommended average steps is 10,000 steps per day. That’s a number said to help reduce certain health conditions, such as high blood pressure and heart disease.
* The average daily distance is **5.49 kilometers**, shorter than the recommended distance of 8 kilometers.
* The average sleep time by all users is **418 minutes, or 7 hrs per day**. [National Sleep Foundation](https://www.sleepfoundation.org/how-sleep-works/how-much-sleep-do-we-really-need#:~:text=National%20Sleep%20Foundation%20guidelines1,to%208%20hours%20per%20night.) advise that healthy adults need between 7 and 9 hours of sleep per night. The average time in bed is 458 minutes, or 7.63 hrs, which means it takes 40 minutes for the users to fall asleep. According to the [Sleep Foundation](https://www.sleepfoundation.org/sleep-hygiene/how-is-sleep-quality-calculated#:~:text=For%20quality%20sleep%2C%20it%20should,minutes%20to%20fall%20asleep%20again.), regularly taking longer than 30 minutes to fall asleep is an indicator of poor sleep.
* Users spend the majority of their time in a sedentary state, which comprises **81%, or 16 hours on average in a day**. Users are only active on about 19%, or 4 hrs in a day. As per Mayo Clinic, a general goal should be to aim for at least 30 minutes of moderate physical activity every day.

Second, I used [Pearson Correlation](https://towardsdatascience.com/eveything-you-need-to-know-about-interpreting-correlations-2c485841c0b8) to find trends and measure how strong a relationship is between two variables. The results returned values between -1 and 1, where:
* 1 indicates a strong positive relationship.
* -1 indicates a strong negative relationship.
* A result of zero indicates no relationship at all.

``` sql
-- Correlations
WITH DailyActivity AS (
  SELECT 
    DailyActivity.Id,
    UserNumberTable.UserNo,
    ActivityDate, 
    SUM(TotalSteps) AS Total_Steps, 
    SUM(TotalDistance) AS Total_Distance, 
    SUM(TrackerDistance) AS Total_Tracker_Distance, 
    SUM(LoggedActivitiesDistance) AS Total_LoggedActivitiesDistance,	
    SUM(VeryActiveDistance) AS Total_VeryActiveDistance,	
    SUM(ModeratelyActiveDistance) AS Total_ModeratelyActiveDistance,	
    SUM(LightActiveDistance) AS Total_LightActiveDistance,	
    SUM(SedentaryActiveDistance) AS Total_SedentaryActiveDistance,	
    SUM(VeryActiveMinutes) AS Total_VeryActiveMinutes,
    SUM(FairlyActiveMinutes) AS Total_FairlyActiveMinutes,
    SUM(LightlyActiveMinutes) AS Total_LightlyActiveMinutes,
    SUM(SedentaryMinutes) AS Total_SedentaryMinutes,
    SUM(Calories) AS Total_Calories,
    IFNULL(SUM(TotalSleepRecords),0) AS Total_SleepRecords,
    IFNULL(SUM(TotalMinutesAsleep),0) AS Total_MinutesAsleep,
    IFNULL(SUM(TotalTimeInBed),0) AS Total_TimeInBed,
  FROM `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity
  LEFT JOIN 
    (SELECT
      Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed,
      COUNT(*) AS Dupes
    FROM `gda-course-4-332812.Capstone.SleepDay` AS SleepDay1
    GROUP BY Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed
    HAVING Dupes = 1
    ORDER BY Id,
      SleepDay,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed) AS SleepDay1
  ON DailyActivity.Id = SleepDay1.Id
  AND DailyActivity.ActivityDate = SleepDay1.SleepDay
  LEFT JOIN `gda-course-4-332812.Capstone.UserNumberTable` AS UserNumberTable
  ON DailyActivity.Id = UserNumberTable.Id
  GROUP BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate
  ORDER BY DailyActivity.Id, UserNumberTable.UserNo, DailyActivity.ActivityDate
)
SELECT CORR(Total_Calories, Total_Steps) AS CorrCaloriesSteps,
  CORR(Total_Calories, Total_Distance) AS CorrCaloriesDistance,
  CORR(Total_Calories, Total_VeryActiveMinutes) AS CorrCaloriesVeryActive,
  CORR(Total_Calories, Total_FairlyActiveMinutes) AS CorrCaloriesFairlyActive,
  CORR(Total_Calories, Total_LightlyActiveMinutes) AS CorrCaloriesLightlyActive,
  CORR(Total_Calories, Total_SedentaryMinutes) AS CorrCaloriesSedentary
FROM DailyActivity
-- high positive correlation between daily calories and steps
-- high positive correlation between daily calories and distance
-- high positive correlation between daily calories and very active minutes
-- low positive correlation between daily calories and fairly active minutes
-- low positive correlation between daily calories and lightly active minutes
-- low negative correlation between daily calories and sedentary minutes

SELECT CORR(Calories, Total_METs) AS CorrCaloriesMETs
FROM (WITH Minute_METs AS (SELECT Id, SUM(METs) AS METs,
EXTRACT(DATE FROM ActivityMinute) AS Date,
  EXTRACT(YEAR FROM ActivityMinute) AS Year,
  EXTRACT(MONTH FROM ActivityMinute) AS Month,
  EXTRACT(DAY FROM ActivityMinute) AS Day,
  FORMAT_DATE('%A', DATE(ActivityMinute)) AS DayName,
  CAST(EXTRACT(TIME FROM ActivityMinute) AS TIME) AS Time,
  EXTRACT(HOUR FROM ActivityMinute) AS Hour,
  COUNT(*) AS Duplicates
FROM `gda-course-4-332812.Capstone.MinuteMETs` AS MinuteMETS
GROUP BY Id, ActivityMinute
ORDER BY Id, ActivityMinute)

SELECT Id, Date, SUM(METs) AS Total_METs, ROUND(AVG(METs), 2) AS Avg_METs,
  EXTRACT(YEAR FROM Date) AS Year,
  EXTRACT(MONTH FROM Date) AS Month,
  EXTRACT(DAY FROM Date) AS Day,
  FORMAT_DATE('%A', DATE(Date)) AS DayName
FROM Minute_METs
GROUP BY Id, Date
) AS METS
LEFT JOIN `gda-course-4-332812.Capstone.DailyActivity` AS DailyActivity
ON METS.Id = DailyActivity.Id
AND METS.Date = DailyActivity.ActivityDate
-- High positive correlation between calories and METs
```

Key insights from correlation analyses:
* At 0.59, there is a moderately high positive correlation between calories and steps.
* At 0.64, there is a moderately high positive correlation between calories and distance.
* At 0.62, there is a moderately high positive correlation between calories and very active minutes.
* At 0.30, there is a low positive correlation between calories and fairly active minutes.
* At 0.29, there is a negligible positive correlation between calories and lightly active minutes.
* At -0.11, there is a negligible negative correlation between calories and lightly active minutes.
* At 0.71, there is a high positive correlation between calories and distance.

### **Share**

Tableau Public was used to visualize the analyses made on this study. Refer to this [link](https://public.tableau.com/app/profile/paulo8563/viz/BellabeatCapstoneProject_16543389242590/Correlations#1) for a full comprehensive view.

**Correlations**

![Correlations](https://user-images.githubusercontent.com/72072808/173175891-8a69039b-8c96-4427-b263-0dede3192e83.png)

* **Calories vs. Distance** - This figure shows a moderately high positive correlation between calories and distance, which means the higher the distance tracked, the higher the calories burnt.
* **Calories vs. Steps** - This figure shows a moderately high positive correlation between calories and steps, which means the higher the steps taken, the higher the calories burnt.
* **Calories vs. METs** - A MET is a ratio of your working metabolic rate relative to your resting metabolic rate. Metabolic rate is the rate of energy expended per unit of time. It’s one way to describe the intensity of an exercise or activity. This figure depicts an upward trajectory on its trendline, indicating that the total METs is highly correlated with calories burned.
* **Calories vs. Very Active Minutes** - It shows that the higher the time allocated in doing vigorous activities and exercises you do, more calories will be slashed.

**Daily**

![Daily](https://user-images.githubusercontent.com/72072808/173175885-45b93882-937e-41cc-b997-dbc73ba0514b.png)

* **Average Calories by Day** - As we can see in this figure, users on average burn the most calories on a Tuesday and lowest on a Thursday.
* **Average METs by Day** - This figure shows that users have highest METs on a Tuesday and lowest on a Thursday.
* **Average Very Active Minutes by Day** - Users do most of their very active activities on a Tuesday and the lowest on a Thursday.
* **Average Steps by Day** - Users walk the most on a Tuesday and lowest on a Friday.

**Hourly**

![Hourly](https://user-images.githubusercontent.com/72072808/173175863-cfdb99dd-e857-4d58-a61f-1797551f98bf.png)
* **Average Calories by Hour** - On average, users burn most of their calories from 5PM-7PM.
* **Average Intensity by Hour** - Users recorded their highest intensity levels from 5-7 PM.
* **Average Steps by Hour** - Users walk the most by 5PM-7PM.

### **Act**


Here are my top recommendations to help guide marketing strategy for Bellabeat mostly focusing on its notification system:
* The dataset of this study is already 6 years old, which means Bellabeat will need to make a much updated and comprehensive version of this dataset. Key points for improvement would be an increase in the number of participants and capture of demographic data such as gender, age, location, height, weight, and so on.
* Heart Rate and Weight Log Info tables would’ve been useful in this study had there been a large number of participants with records on these tables and is also an integral part of tracking one’s fitness. One way to improve capture of these data points is better integration of heart rate and weight into Bellabeat’s notification system.
* We saw from the insights of this study that calories burned are positively correlated into steps, distance, METs, and very active minutes. Bellabeat can use these insights to market its product line showing how these variables impact calories burnt and the relevant studies that back it. Bellabeat can also use these insights to create a predictive analytics feature into the product, showing users forecasted calories burnt in accordance with the variables mentioned. This can help motivate users to stay active and achieve their fitness goals, as one insight in this study showed that 81% of the users’ day are spent in a sedentary state.
* On average, users take 40 minutes to fall asleep when they go to bed, which is higher than most studies recommend. Bellabeat should also integrate this into the notification system in order for the users to be reminded to get sufficient sleep.
* As the majority of the users spend most of their day in a sedentary state, Bellabeat can notify the user to do some light to moderate activity 1 hour after being in a sedentary state.
* Users are mostly active from 5-7 PM. In order to ensure that users maintain a healthy lifestyle, Bellabeat can fire up a notification during this time to remind the user to do moderate to intense activities such as walking, running, cycling, etc.
* Bellabeat should create a reward system in order to encourage users to stay active and motivated, rewarding the users on their milestones and the ability to share these on social media.
