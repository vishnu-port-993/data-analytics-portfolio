# Project Approach: Bellabeat Case Study

This document outlines the structured approach used in the Bellabeat case study, following the Google Data Analytics process phases: **Ask, Prepare, Process, Analyze, Share,** and **Act**.

## Business Context: 
Bellabeat is a company that specializes in wellness technology, designing products to help users understand and improve their health. As Bellabeat prepares to launch its new product, IVY+, a wellness tracker, there is a strong interest in understanding users' physical activity, sleep habits, and overall wellness trends. This knowledge will not only help inform product development but also enhance Bellabeat's marketing strategies and customer engagement by addressing key health areas important to its target audience.

# 1. Ask Phase - Defining the Business Problem
- **Business Problem**: How can Bellabeat use data to provide meaningful insights into user health patterns and wellness goals, while also identifying areas for improvement in line with recognized health standards?
- **Goal**: Identify actionable insights that Bellabeat can use to enhance wellness features and engage users effectively.
- **Key Questions**:
  - How active are users on a daily and weekly basis?
  - How does users' sleep duration compare to recommended baselines?
  - Are there patterns in users' heart rates that indicate stress or health issues?

While Fitbit data provides valuable insights into user behaviours, it has some limitations. The data reflects only those users who have consented to share it, and it lacks broader population-wide health benchmarks. This limits Bellabeat's ability to see how users compare to national and global wellness standards. 

To address this, we have integrated data from the **2016 National Health Interview Survey (NHIS)**, which provides health information about the general U.S. adult population, and **World Health Organization (WHO)** recommendations. 

By comparing Fitbit user data against NHIS survey data and WHO guidelines, we can:
1.	Identify patterns and gaps in user behaviour.
2.	Benchmark user activity and wellness metrics against broader health standards.
3.	Provide actionable insights for users and product development recommendations for Bellabeat.

**Objectives of the Analysis:**
-	Compare Fitbit users' physical activity, sleep, and heart rate metrics to **NHIS** data and **WHO guidelines**.
-	Identify Gaps where Fitbit users may fall short of recommended wellness standards, highlighting areas for potential product features or user guidance.
-	Deliver Insights that are relevant, measurable, and aligned with Bellabeat's wellness-focused mission, supporting both the company’s product strategy and its goal of promoting healthier lifestyles.

# 2. Prepare Phase
## Data Sources:
To address the business problem, we’ve selected two primary data sources that together offer both user-specific insights and population-level benchmarks:
### 1. Fitbit Data (03.12.2016 - 05.12.2016):
This dataset includes detailed activity and health metrics for users who consented to share their data. The dataset includes:
-	**Daily Activity**: Total steps, distance, and time spent in different activity levels (sedentary, lightly active, fairly active, and very active).
-	**Sleep Metrics**: Sleep duration, time in bed, and various stages of sleep, providing insights into users’ sleep patterns.
-	**Heart Rate Data**: Detailed heart rate information that helps understand users' cardiovascular health.

#### ROCCC Check:
-	**Reliable**: This data is sourced from Kaggle, a reputable platform for dataset sharing, but it includes a small sample of 30 users, which may not fully represent the broader population.
-	**Original**: The dataset is original, and submitted with user consent, ensuring authenticity.
-	**Comprehensive**: Although it offers minute-level tracking of activity, heart rate, and sleep, the small sample size limits its comprehensiveness in representing diverse demographics or activity levels.
-	**Current**: Collected from 30 eligible Fitbit users between 03.12.2016 and 05.12.2016 through Amazon Mechanical Turk, this data is relatively current for analyzing individual behaviour patterns, even though it is now a few years old.
-	**Cited**: Publicly available under a CC0 license, ensuring transparency and privacy compliance. The dataset is cited as follows:
    -	Owners: Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894
    -	Privacy considerations are detailed in Furberg et al. (2017) [https://link.springer.com/article/10.1007/s00779-017-1068-3].

#### 1.1 Daily Activity Data Setup and Import (including a bit of initial time frame cleaning):
In this step, I created the `daily_activity` table to store daily activity data from the *dailyActivity_merged.csv* files. These files were available in both **Fitabase Data 3.12.16-4.11.16** and **Fitabase Data 4.12.16-5.12.16** folders, containing user activity metrics like steps, distance, and calories for each day.

The `daily_activity` table was structured with columns for user ID, activity date, total steps, distance metrics, various activity intensities, active minutes, sedentary minutes, and calories burned. I used `STR_TO_DATE` to format the date consistently during import, ensuring accurate date-based queries.

##### **SQL QUERIES:**
Create and import data set to table `daily_activity` :
```sql
CREATE TABLE daily_activity (
    Id BIGINT,
    ActivityDate DATE,
    TotalSteps INT,
    TotalDistance FLOAT,
    TrackerDistance FLOAT,
    LoggedActivitiesDistance FLOAT,
    VeryActiveDistance FLOAT,
    ModeratelyActiveDistance FLOAT,
    LightActiveDistance FLOAT,
    SedentaryActiveDistance FLOAT,
    VeryActiveMinutes INT,
    FairlyActiveMinutes INT,
    LightlyActiveMinutes INT,
    SedentaryMinutes INT,
    Calories INT
);
```
```sql
-- Import dailyActivity_merged.csv for time frame 3.12.16-4.11.16 into table "daily_activity"
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 3.12.16-4.11.16/dailyActivity_merged.csv'
INTO TABLE daily_activity
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityDate, TotalSteps, TotalDistance, TrackerDistance, LoggedActivitiesDistance, VeryActiveDistance, ModeratelyActiveDistance, LightActiveDistance, SedentaryActiveDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes, Calories)
SET ActivityDate = STR_TO_DATE(@ActivityDate, '%m/%d/%Y');
```
```sql
-- Ensuring that all records for this import fall within the observation time frame (2016-03-12 to 2016-04-11)
DELETE FROM daily_activity
WHERE ActivityDate NOT BETWEEN '2016-03-12' AND '2016-04-11';
```

```sql
-- Import dailyActivity_merged.csv for timeframe, but for the time frame 4.12.16-5.12.16 into the same table "daily_activity" which will merge both the CSV files into one table.
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv'
INTO TABLE daily_activity
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityDate, TotalSteps, TotalDistance, TrackerDistance, LoggedActivitiesDistance, VeryActiveDistance, ModeratelyActiveDistance, LightActiveDistance, SedentaryActiveDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes, Calories)
SET ActivityDate = STR_TO_DATE(@ActivityDate, '%m/%d/%Y');
```

```sql
-- Ensuring that all records for this import fall within the overall observation time frame (2016-03-12 to 2016-05-12)
DELETE FROM daily_activity
WHERE ActivityDate NOT BETWEEN '2016-03-12' AND '2016-05-12';
```

#### 1.2 Heart Rate Data Setup and Import (including a bit of initial time frame cleaning):
In this step, I created the `heartrate_seconds` table to store high-resolution, second-level heart rate data from the *heartrate_seconds_merged.csv* files. These files were available in both **Fitabase Data 3.12.16-4.11.16** and **Fitabase Data 4.12.16-5.12.16** folders, providing detailed insights into users' heart rate patterns across the project’s timeframe.

The table includes fields for ID, Time, and Value, representing the user ID, timestamp, and heart rate value respectively. Consistent with Step 1.1, I formatted the Time field to ensure uniformity and applied a timeframe filter to retain data only within the observation period of 2016-03-12 to 2016-05-12.

##### **SQL QUERIES:**
Create and import data set to table `heartrate_seconds`:
```sql
CREATE TABLE heartrate_seconds (
    Id BIGINT,
    Time TIMESTAMP,
    Value INT
);
```
```sql
-- Import heartrate_seconds_merged.csv for time frame 3.12.16-4.11.16 into table "heartrate_seconds"
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 3.12.16-4.11.16/heartrate_seconds_merged.csv'
INTO TABLE heartrate_seconds
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @Time, Value)
SET Time = STR_TO_DATE(@Time, '%m/%d/%Y %r');
```
```sql
-- Ensuring that all records for this import fall within the observation time frame (2016-03-12 to 2016-04-11)
DELETE FROM heartrate_seconds
WHERE Time NOT BETWEEN '2016-03-12 00:00:00' AND '2016-04-11 23:59:59';
```sql
-- Import heartrate_seconds_merged.csv for time frame, but for the time frame 4.12.16-5.12.16 into the same table "heartrate_seconds" which will merge both the CSV files into one table.
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/heartrate_seconds_merged.csv'
INTO TABLE heartrate_seconds
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @Time, Value)
SET Time = STR_TO_DATE(@Time, '%m/%d/%Y %r');
```
```sql
-- Ensuring that all records for this import fall within the overall observation time frame (2016-03-12 to 2016-05-12)
DELETE FROM heartrate_seconds
WHERE Time NOT BETWEEN '2016-03-12 00:00:00' AND '2016-05-12 23:59:59';
```


#### 1.3 Sleep Data Setup and Import (including a bit of initial time frame cleaning):
In this step, I created the `sleep_day` table to store daily sleep data from the *sleepDay_merged.csv* file. This file was available only in the **Fitabase Data 4.12.16-5.12.16** folder, indicating that sleep data was not collected for the earlier timeframe (March 12 - April 11, 2016).

The table includes essential metrics on user sleep patterns, such as total minutes asleep and total time in bed, which are crucial for understanding correlations between sleep and daily activity levels.

The table was structured with columns for user ID, sleep date, total sleep records, total minutes asleep, and total time in bed. Consistent with previous steps, I formatted the timestamp for `SleepDay` to maintain uniformity across tables, ensuring that only data within the timeframe between 2016-04-12 and 2016-05-12 was included.

##### **SQL QUERIES:**
Create and import data set to table `sleep_day`:

```sql
CREATE TABLE sleep_day (
    Id BIGINT,
    SleepDay DATETIME,
    TotalSleepRecords INT,
    TotalMinutesAsleep INT,
    TotalTimeInBed INT
);
```
```sql
-- Import sleepDay_merged.csv for time frame 4.12.16 - 5.12.16 into table "sleep_day" 
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv'
INTO TABLE sleep_day
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed)
SET SleepDay = STR_TO_DATE(@SleepDay, '%m/%d/%Y %r');
```
```sql
-- Ensuring that all records for this import fall within the observation time frame (BETWEEN '2016-03-12 00:00:00' AND '2016-05-12 23:59:59')
DELETE FROM sleep_day
WHERE SleepDay NOT BETWEEN '2016-04-12 00:00:00' AND '2016-05-12 23:59:59';
```

#### 1.4 Hourly Activity Data Setup and Import (including a bit of initial time frame cleaning):
In this step, we set up and cleaned two hourly-level activity tables to capture detailed insights on users’ physical activity: `hourly_steps` and `hourly_calories`. Both tables pull from respective CSV files across two date ranges (3.12.16 - 5.12.16), allowing for fine-grained hourly analysis. Key actions included setting up the tables, importing data, and filtering based on the observation timeframe.

##### **SQL QUERIES:**
Create and import data set to table `hourly_steps` and `hourly_calories`:
```sql
CREATE TABLE hourly_steps (
    Id BIGINT,
    ActivityHour TIMESTAMP,
    StepTotal INT
);

CREATE TABLE hourly_calories (
    Id BIGINT,
    ActivityHour TIMESTAMP,
    Calories INT
);
```

```sql
-- Import hourlySteps_merged.csv and hourlyCalories_merged.csv for time frame 3.12.16-4.11.16 into table "hourly_steps" and "hourly_calories" respectively.
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 3.12.16-4.11.16/hourlySteps_merged.csv'
INTO TABLE hourly_steps
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityHour, StepTotal)
SET ActivityHour = STR_TO_DATE(@ActivityHour, '%m/%d/%Y %h:%i:%s %p');

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 3.12.16-4.11.16/hourlyCalories_merged.csv'
INTO TABLE hourly_calories
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityHour, Calories)
SET ActivityHour = STR_TO_DATE(@ActivityHour, '%m/%d/%Y %h:%i:%s %p');
```
```sql
-- Ensuring that all records for this import fall within the observation time frame (2016-03-12 to 2016-04-11)
DELETE FROM hourly_steps
WHERE ActivityHour NOT BETWEEN '2016-03-12 00:00:00' AND '2016-04-11 23:59:59';

DELETE FROM hourly_calories
WHERE ActivityHour NOT BETWEEN '2016-03-12 00:00:00' AND '2016-04-11 23:59:59';
```

```sql
-- Import hourlySteps_merged.csv and hourlyCalories_merged.csv for timeframe, but for the time frame 4.12.16-5.12.16 into the same table "hourly_steps" and "hourly_calories" respectively which will merge both the CSV files into one table each.
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/hourlySteps_merged.csv'
INTO TABLE hourly_steps
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityHour, StepTotal)
SET ActivityHour = STR_TO_DATE(@ActivityHour, '%m/%d/%Y %h:%i:%s %p');

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/hourlyCalories_merged.csv'
INTO TABLE hourly_calories
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @ActivityHour, Calories)
SET ActivityHour = STR_TO_DATE(@ActivityHour, '%m/%d/%Y %h:%i:%s %p');
```



```sql
-- Ensuring that all records for this import fall within the overall observation time frame (2016-03-12 to 2016-05-12)
DELETE FROM hourly_steps
WHERE ActivityHour NOT BETWEEN '2016-03-12 00:00:00' AND '2016-05-12 23:59:59';

DELETE FROM hourly_calories
WHERE ActivityHour NOT BETWEEN '2016-03-12 00:00:00' AND '2016-05-12 23:59:59';
```


#### 1.5 Hourly Activity Data Setup and Import (including a bit of initial time frame cleaning):
In this step, I created the `minute_sleep` table to store detailed minute-by-minute sleep data from the *minuteSleep_merged.csv* files. These files, located in both the **Fitabase Data 3.12.16-4.11.16** and **Fitabase Data 4.12.16-5.12.16** folders, provide a high-resolution view of users’ sleep patterns, logging minute-level sleep statuses across the observation timeframe.

The table was structured with columns for user ID, timestamp, sleep value (indicating sleep state), and a unique log identifier. Additionally, for analysis convenience, the timestamp was split into separate date and time columns.


##### **SQL QUERIES:**
Create and import data set to table `minute_sleep`:
```sql
CREATE TABLE minute_sleep (
    Id BIGINT,
    Date TIMESTAMP,
    Value INT,
    LogId BIGINT
);
```

```sql
-- Import minuteSleep_merged.csv for time frame 3.12.16-4.11.16 into table "minute_sleep".
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 3.12.16-4.11.16/minuteSleep_merged.csv'
INTO TABLE minute_sleep
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @Date, Value, LogId)
SET Date = STR_TO_DATE(@Date, '%m/%d/%Y %h:%i:%s %p');
```

```sql
-- Ensuring that all records for this import fall within the observation time frame (2016-03-12 to 2016-04-11)
DELETE FROM minute_sleep
WHERE Date NOT BETWEEN '2016-03-12 00:00:00' AND '2016-04-11 23:59:59';
```

```sql
-- Import minuteSleep_merged.csv for timeframe, but for the time frame 4.12.16-5.12.16 into the same table "minute_sleep" which will merge both the CSV files into one table each.
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Fitabase Data 4.12.16-5.12.16/minuteSleep_merged.csv'
INTO TABLE minute_sleep
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Id, @Date, Value, LogId)
SET Date = STR_TO_DATE(@Date, '%m/%d/%Y %h:%i:%s %p');
```

```sql
-- Ensuring that all records for this import fall within the overall observation time frame (2016-03-12 to 2016-05-12)
DELETE FROM minute_sleep
WHERE Date NOT BETWEEN '2016-03-12 00:00:00' AND '2016-05-12 23:59:59';
```

```sql
-- Splitting the Date column into Date and Time for better hold over the data set.
ALTER TABLE minute_sleep 
ADD COLUMN date_only DATE, 
ADD COLUMN time_only TIME;

UPDATE minute_sleep
SET 
    date_only = DATE(Date),
    time_only = TIME(Date);
    
ALTER TABLE minute_sleep DROP COLUMN Date;
```

### 2. CDC’s 2016 National Health Interview Survey (NHIS):
This dataset provides national health data for the U.S. adult population, collected and standardized by the CDC. The NHIS dataset includes:
-	**Physical Activity**: Self-reported duration of moderate and vigorous activity, as well as indicators of whether individuals meet aerobic and strength guidelines.
-	**Sleep Patterns**: Data on average sleep duration, quality, and issues related to sleep consistency and restfulness.
-	**Wellness and Health Recommendations Compliance**: Information on whether individuals meet recommended health guidelines for physical activity and sleep, serving as a benchmark for analyzing Fitbit users' data.

#### ROCCC Check:
-	**Reliable**: This data is sourced from the **CDC’s National Center for Health Statistics (NCHS)**, a well-established and reliable source of public health data in the United States. The NHIS is conducted annually, following stringent data collection and validation protocols to ensure accuracy.
-	**Original**: The dataset is an original and direct collection of health data gathered from U.S. residents by the NCHS. It is released for statistical and analytical use only, with de-identified information to protect participant privacy.
-	**Comprehensive**: The NHIS 2016 survey provides extensive health information across a large, nationally representative sample. It includes variables on physical activity, sleep, health conditions, and other demographic factors, making it comprehensive for public health analysis. However, as survey data, it relies on self-reported information, which may introduce response biases.
-	**Current**: Although collected in 2016, NHIS data remains relevant for longitudinal analysis of health behaviours and for establishing baseline health statistics. The 2016 data provides useful historical context, even if it may not capture the latest trends.
-	**Cited**: The NHIS dataset is publicly available through the CDC and is provided under strict data use guidelines to maintain privacy and confidentiality. The dataset can be cited as follows: 
    - CDC/NCHS (2016). National Health Interview Survey, 2016 [Public Use Data Release]. National Center for Health Statistics. Available from: https://www.cdc.gov/nchs/nhis/nhis_2016_data_release.htm
    - *Data Disclaimer: This project uses data from the 2016 National Health Interview Survey (NHIS), which is made available by the National Center for Health Statistics (NCHS) for statistical reporting and analysis only. All analyses are conducted in compliance with the data use restrictions provided by NCHS, with no attempt to identify individuals or link data to any identifiable information.*

#### 2.1 NHIS Data Setup:
In this step, I filtered and did an initial cleaning of the original dataset, i.e. *samadult.csv* which was derived from **CDC’s 2016 National Health Interview Survey (NHIS)** archives, as the original dataset had several columns that were not needed for this project and loading these irrelevant columns would slow down the data processing. Hence, this new filtered dataset was created with only relevant columns and I called it  *nhis_2016_data_cleaned.csv*. 

I then created the `nhis_2016_data` table to store data from the *nhis_2016_data_cleaned.csv* file. The purpose of this data is to serve as a benchmark for wellness metrics, allowing a comparison of Fitbit users’ activity, sleep, and health behaviours with broader national averages. This table includes variables related to physical activity, sleep patterns, and wellness indicators, essential for our comparative analysis with Fitbit data.

##### **SQL QUERIES:**
Create and import data set to table `nhis_2016_data`:
```sql
CREATE TABLE nhis_2016_data (
    SEX INT,
    AGE_P INT,
    SRVY_YR INT,
    INTV_MON INT,
    DBHVPAY INT,
    DBHVCLY INT,
    DBHVPAN INT,
    DBHVCLN INT,
    VIGNO INT,
    VIGTP INT,
    VIGFREQW INT,
    VIGLNGNO INT,
    VIGLNGTP INT,
    VIGMIN INT,
    MODNO INT,
    MODTP INT,
    MODFREQW INT,
    MODLNGNO INT,
    MODLNGTP INT,
    MODMIN INT,
    STRNGNO INT,
    STRNGTP INT,
    STRFREQW INT,
    ASISLEEP INT,
    ASISLPFL INT,
    ASISLPST INT,
    ASISLPMD INT,
    ASIREST INT
);
```
```sql
-- Import nhis_2016_data_cleaned.csv into table "nhis_2016_data".
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/nhis_2016_data_cleaned.csv'
INTO TABLE nhis_2016_data
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### 3. WHO Health Guidelines:
- We incorporate WHO’s recommendations for sleep, physical activity, and heart health as baseline standards. These recommendations help contextualize the analysis by providing universal benchmarks for healthy behaviour.

## Data Limitations and Mitigation:
1. **Sample-Specific Data in Fitbit**: The Fitbit data reflects only users who consented to share their data and may not represent the general population. However, by comparing it to NHIS data, we can gain perspective on where Fitbit users align with or diverge from broader health norms.
2. **Self-Reported Data in NHIS**: NHIS relies on self-reported data, which may introduce some reporting bias. We mitigate this by using NHIS data primarily as a benchmark rather than a direct comparison.
3. **NHIS Data Filtering**: The NHIS dataset was extensive and initially included variables beyond the scope of this analysis. To streamline it, we filtered the data to retain only fields relevant to physical activity, sleep, and compliance with health guidelines. This filtering process resulted in a more targeted dataset, stored as *"nhis_2016_data_cleaned.csv"*


# 2. Process Phase: Data Validation, Cleaning, and Preparation
The PROCESS phase in this project involved validating, cleaning, and structuring the selected datasets to ensure accurate, high-quality analysis. This phase is critical because clean, validated data forms the foundation of reliable insights and sound decision-making. Given the scope of this capstone project, the process phase is conducted with careful attention to Data Validation, Data Cleaning and Data Transformation.

## Data Cleaning Overview:
To ensure data integrity and quality, each table was subjected to the following cleaning procedures:
1.	**Duplicate Identification**: Records with identical ID and date/timestamp entries were reviewed to ensure data uniqueness per user and time period.
2.	**Trimming Spaces**: All text columns were trimmed to remove leading or trailing spaces.
3.	**Missing or Null Values Check**: Null or missing values were addressed or removed, particularly in key columns.
4.	**Date Format Standardization**: Dates were standardized across tables to ensure consistency.
5.	**Validation of Numeric Columns**: Each numeric column was verified to contain only valid numbers.
6.	**Date Range Verification**: Dates in each table were checked to confirm they fall within the project’s specified timeframe (March 12 to May 12, 2016).
7.	**Null Values in Key Columns**: Essential columns were checked for null values, and missing entries were handled as needed.
8.	**Descriptive Statistics Review**: Basic statistics were calculated for numeric columns to identify any anomalies or data inconsistencies.
9.	**Overall Data Integrity Check**: A general check was conducted to ensure logical coherence across values.

## 2.1 Processing and cleaning daily_activity table:
Following the import, I conducted several data cleaning steps as per the Data Cleaning Overview, including handling duplicates, standardizing date formats, ensuring valid numeric data, and removing null values.

### 2.1.1 Key SQL queries used for cleaning:
1.	**Identify Duplicates**: I checked for duplicate records based on Id and ActivityDate to avoid redundancy.
```sql
SELECT
    Id, 
    ActivityDate,
    COUNT(*)
FROM daily_activity
GROUP BY 
    Id, ActivityDate
HAVING COUNT(*) > 1;
```
