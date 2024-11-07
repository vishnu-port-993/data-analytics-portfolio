# Project Approach: Bellabeat Case Study

This document outlines the structured approach used in the Bellabeat case study, following the Google Data Analytics process phases: **Ask, Prepare, Process, Analyze, Share,** and **Act**.

## Business Context: 
Bellabeat is a company that specializes in wellness technology, designing products to help users understand and improve their health. As Bellabeat prepares to launch its new product, IVY+, a wellness tracker, there is a strong interest in understanding users' physical activity, sleep habits, and overall wellness trends. This knowledge will not only help inform product development but also enhance Bellabeat's marketing strategies and customer engagement by addressing key health areas important to its target audience.

# 1. ASK PHASE - Defining the Business Problem
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

# 2. PREPARE PHASE
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


# 3. PROCESS PHASE: Data Validation, Cleaning, and Preparation
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

## 3.1 Processing and cleaning `daily_activity` table:
Following the import, I conducted several data cleaning steps as per the Data Cleaning Overview, including handling duplicates, standardizing date formats, ensuring valid numeric data, and removing null values.

### 3.1.1 Key SQL queries used for cleaning:
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
2.	**Remove Duplicates**: For cases where duplicates existed, I retained the record with the higher calorie count as it likely reflects a more complete activity log.
```sql
DELETE t1
FROM daily_activity t1
INNER JOIN daily_activity t2 
ON t1.Id = t2.Id 
AND t1.ActivityDate = t2.ActivityDate
WHERE t1.TotalSteps = 0
   OR t1.Calories < t2.Calories;
```
3.	**Data Integrity Checks**: I conducted a series of checks to ensure data integrity, including verifying the earliest and latest dates, total records, and unique users, and generating basic descriptive statistics.
```sql
SELECT 
    MIN(ActivityDate) AS earliest_date,
    MAX(ActivityDate) AS latest_date,
    COUNT(*) AS total_records,
    COUNT(DISTINCT Id) AS unique_users
FROM daily_activity;
```
4. **Sample Output Verification**: Below is a sample output from the `daily_activity` table to confirm the imported data and ensure it falls within the specified timeframe.
```sql
+------------+--------------+------------+---------------+-----------------+--------------------------+--------------------+--------------------------+---------------------+-------------------------+-------------------+---------------------+----------------------+------------------+----------+
| Id         | ActivityDate | TotalSteps | TotalDistance | TrackerDistance | LoggedActivitiesDistance | VeryActiveDistance | ModeratelyActiveDistance | LightActiveDistance | SedentaryActiveDistance | VeryActiveMinutes | FairlyActiveMinutes | LightlyActiveMinutes | SedentaryMinutes | Calories |
+------------+--------------+------------+---------------+-----------------+--------------------------+--------------------+--------------------------+---------------------+-------------------------+-------------------+---------------------+----------------------+------------------+----------+
| 6775888955 | 2016-04-01   |       7225 |          5.18 |            5.18 |                        0 |               1.73 |                     1.27 |                2.18 |                       0 |                25 |                  50 |                  163 |             1189 |     3065 |
| 4020332650 | 2016-03-22   |       6662 |          4.78 |            4.78 |                        0 |                  0 |                        0 |                   0 |                       0 |                 0 |                   0 |                    0 |             1440 |     3162 |
| 4558609924 | 2016-04-08   |       4195 |          2.77 |            2.77 |                        0 |                  0 |                        0 |                2.77 |                       0 |                 0 |                   0 |                  241 |             1199 |     1778 |
| 8792009665 | 2016-04-21   |        144 |          0.09 |            0.09 |                        0 |                  0 |                        0 |                0.09 |                       0 |                 0 |                   0 |                    9 |             1431 |     1720 |
| 1927972279 | 2016-04-02   |       5662 |          3.92 |            3.92 |                        0 |                  0 |                        0 |                3.92 |                       0 |                 0 |                   0 |                  267 |              858 |     2783 |
| 3372868164 | 2016-04-24   |       6731 |          4.59 |            4.59 |                        0 |               0.89 |                     0.19 |                3.49 |                    0.02 |                14 |                   7 |                  292 |             1127 |     1921 |
| 2873212765 | 2016-04-21   |       8859 |          5.98 |            5.98 |                        0 |               0.13 |                     0.37 |                5.47 |                    0.01 |                 2 |                  10 |                  371 |             1057 |     1970 |
| 6117666160 | 2016-04-28   |       3403 |           2.6 |             2.6 |                        0 |                  0 |                        0 |                 2.6 |                       0 |                 0 |                   0 |                  141 |              758 |     1879 |
| 4319703577 | 2016-04-28   |      10817 |          7.28 |            7.28 |                        0 |               1.01 |                     0.33 |                5.94 |                       0 |                13 |                   8 |                  359 |              552 |     2367 |
| 6962181067 | 2016-05-11   |       6722 |          4.44 |            4.44 |                        0 |               1.49 |                     0.31 |                2.65 |                       0 |                24 |                   7 |                  199 |              709 |     1855 |
+------------+--------------+------------+---------------+-----------------+--------------------------+--------------------+--------------------------+---------------------+-------------------------+-------------------+---------------------+----------------------+------------------+----------+
```

## 3.2 Processing and cleaning `heartrate_seconds` table:
After importing the data, I conducted multiple cleaning and validation steps to ensure the integrity of the heart rate data.

### 3.2.1 Key SQL queries used for cleaning:
1.	**Identify and Remove Duplicates**: Due to overlapping dates across the two folders, there were potential duplicates for the same Id and Time. I retained the record with the higher Value in cases where duplicates were found.
```sql
SELECT Id, Time, COUNT(*)
FROM heartrate_seconds
GROUP BY Id, Time
HAVING COUNT(*) > 1;

DELETE t1
FROM heartrate_seconds t1
INNER JOIN heartrate_seconds t2 
ON t1.Id = t2.Id 
AND t1.Time = t2.Time
WHERE t1.Value < t2.Value;
```
2.	**Check for Missing or Invalid Values**: I validated that the Value column had no null or invalid entries (e.g., values less than or equal to zero).
```sql
SELECT * FROM heartrate_seconds
WHERE Value IS NULL OR Value <= 0;
```
3.	**Data Integrity Check**: I conducted integrity checks by viewing a random sample of records, checking the earliest and latest timestamps, total records, and unique users, and verifying the presence of null values in key columns.
```sql
SELECT 
    MIN(Time) AS earliest_time,
    MAX(Time) AS latest_time,
    COUNT(*) AS total_records,
    COUNT(DISTINCT Id) AS unique_users
FROM heartrate_seconds;
```
4. **Sample Output Verification**: The following sample output from `heartrate_seconds` shows random records to verify the data import and timeframe filtering.
```sql
+------------+---------------------+-------+
| Id         | Time                | Value |
+------------+---------------------+-------+
| 7007744171 | 2016-04-18 21:32:05 |    91 |
| 4388161847 | 2016-04-15 00:08:40 |    54 |
| 8877689391 | 2016-04-25 20:51:23 |    76 |
| 6775888955 | 2016-04-06 04:05:00 |    81 |
| 4020332650 | 2016-04-06 20:52:22 |   105 |
| 2347167796 | 2016-04-18 16:39:20 |    73 |
| 6962181067 | 2016-04-06 07:09:30 |    75 |
| 8877689391 | 2016-05-02 08:39:47 |    60 |
| 4558609924 | 2016-04-10 18:16:55 |    83 |
| 4020332650 | 2016-05-05 22:11:55 |    95 |
+------------+---------------------+-------+
```
## 3.3 Processing and cleaning `sleep_day` table:

### 3.3.1 Key SQL queries used for cleaning:
1.	**Identify and Remove Duplicates**: I identified duplicate records where the same Id and SleepDay occurred more than once. For simplicity, I retained only the earliest entry by creating a temporary temp_id column, which I used to differentiate and delete duplicate rows.
```sql
SELECT Id, SleepDay, COUNT(*)
FROM sleep_day
GROUP BY Id, SleepDay
HAVING COUNT(*) > 1;

ALTER TABLE sleep_day ADD COLUMN temp_id INT AUTO_INCREMENT PRIMARY KEY;
DELETE FROM sleep_day
WHERE temp_id NOT IN (
    SELECT MIN(temp_id)
    FROM (SELECT * FROM sleep_day) AS subquery
    GROUP BY Id, SleepDay
);

ALTER TABLE sleep_day DROP COLUMN temp_id;
```
2.	**Check for Null Values in Key Columns**: I validated that all key columns (`TotalSleepRecords`, `TotalMinutesAsleep`, `TotalTimeInBed`) contained no null values to ensure data completeness.
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(TotalSleepRecords) AS non_null_records,
    COUNT(TotalMinutesAsleep) AS non_null_minutes_asleep,
    COUNT(TotalTimeInBed) AS non_null_time_in_bed
FROM sleep_day;
```

3.	**Data Integrity Checks**: I conducted additional checks to verify the data's consistency, including confirming the date range, checking the number of unique users, and calculating basic descriptive statistics for sleep metrics.
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(TotalSleepRecords) AS non_null_records,
    COUNT(TotalMinutesAsleep) AS non_null_minutes_asleep,
    COUNT(TotalTimeInBed) AS non_null_time_in_bed
FROM sleep_day;

SELECT 
    MIN(TotalMinutesAsleep) AS min_minutes_asleep,
    MAX(TotalMinutesAsleep) AS max_minutes_asleep,
    AVG(TotalMinutesAsleep) AS avg_minutes_asleep,
    MIN(TotalTimeInBed) AS min_time_in_bed,
    MAX(TotalTimeInBed) AS max_time_in_bed,
    AVG(TotalTimeInBed) AS avg_time_in_bed
FROM sleep_day;
```

4.	**Sample Output Verification**: Below is a sample output from the `sleep_day` table to confirm the imported data and ensure it falls within the specified timeframe.
```sql
+------------+---------------------+-------------------+--------------------+----------------+
| Id         | SleepDay            | TotalSleepRecords | TotalMinutesAsleep | TotalTimeInBed |
+------------+---------------------+-------------------+--------------------+----------------+
| 2026352035 | 2016-04-23 00:00:00 |                 1 |                522 |            554 |
| 4702921684 | 2016-04-27 00:00:00 |                 1 |                432 |            449 |
| 1503960366 | 2016-04-16 00:00:00 |                 2 |                340 |            367 |
| 4445114986 | 2016-04-13 00:00:00 |                 2 |                370 |            406 |
| 4319703577 | 2016-05-10 00:00:00 |                 1 |                487 |            517 |
| 5577150313 | 2016-04-20 00:00:00 |                 1 |                447 |            480 |
| 4445114986 | 2016-05-06 00:00:00 |                 2 |                374 |            402 |
| 4020332650 | 2016-04-12 00:00:00 |                 1 |                501 |            541 |
| 1503960366 | 2016-05-05 00:00:00 |                 1 |                247 |            264 |
| 6962181067 | 2016-05-09 00:00:00 |                 1 |                489 |            497 |
+------------+---------------------+-------------------+--------------------+----------------+
```


## 3.4 Processing and cleaning `hourly_steps` and `hourly_calories` tables:
### 3.4.1 Key SQL queries used for cleaning:
1.	**Check for Null Values**: Queried each table for nulls in critical columns (`ActivityHour`, `StepTotal` in `hourly_steps`, and `Calories` in `hourly_calories`).
```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(ActivityHour) AS non_null_activity_hour,
    COUNT(StepTotal) AS non_null_steps
FROM hourly_steps;

SELECT
    COUNT(*) AS total_rows,
    COUNT(ActivityHour) AS non_null_activity_hour,
    COUNT(Calories) AS non_null_calories
FROM hourly_calories;
```
2.	**Date Range Verification**: Confirmed each table's earliest and latest timestamps to ensure they matched the observation period.
```sql
SELECT
    MIN(ActivityHour) AS earliest_date,
    MAX(ActivityHour) AS latest_date
FROM hourly_steps;

SELECT
    MIN(ActivityHour) AS earliest_date,
    MAX(ActivityHour) AS latest_date
FROM hourly_calories;
```
3. **Sample Output Verification**: The following random samples verify that the data in the `hourly_steps` and `hourly_calories` tables are correctly formatted and fall within the specified timeframe.
- `hourly_steps`
```sql
+------------+---------------------+-----------+
| Id         | ActivityHour        | StepTotal |
+------------+---------------------+-----------+
| 2022484408 | 2016-04-06 09:00:00 |      4630 |
| 1844505072 | 2016-04-18 04:00:00 |         0 |
| 2026352035 | 2016-05-02 23:00:00 |         0 |
| 1927972279 | 2016-04-05 07:00:00 |       158 |
| 4558609924 | 2016-03-12 08:00:00 |         0 |
| 5553957443 | 2016-04-17 12:00:00 |         0 |
| 8253242879 | 2016-04-05 15:00:00 |         0 |
| 6962181067 | 2016-04-29 02:00:00 |         0 |
| 6962181067 | 2016-05-10 06:00:00 |        36 |
| 4020332650 | 2016-05-02 12:00:00 |         0 |
+------------+---------------------+-----------+
```

- `hourly_calories`
```sql
+------------+---------------------+----------+
| Id         | ActivityHour        | Calories |
+------------+---------------------+----------+
| 1644430081 | 2016-04-21 23:00:00 |       86 |
| 2026352035 | 2016-04-16 03:00:00 |       47 |
| 8792009665 | 2016-03-12 18:00:00 |       70 |
| 1624580081 | 2016-04-17 16:00:00 |       50 |
| 4020332650 | 2016-05-11 02:00:00 |       83 |
| 8583815059 | 2016-04-30 13:00:00 |      248 |
| 5577150313 | 2016-03-24 20:00:00 |      147 |
| 1927972279 | 2016-03-23 14:00:00 |      104 |
| 8877689391 | 2016-05-10 23:00:00 |       73 |
| 8792009665 | 2016-03-24 23:00:00 |       70 |
+------------+---------------------+----------+
```
## 3.5 Processing and cleaning `minute_sleep` table:
The following steps ensured data quality and integrity within the minute_sleep table:
### 3.5.1 Key SQL queries used for cleaning:
1.	Splitting Date and Time Columns: After importing the data, the Date column was divided into `date_only` and `time_only` columns to support flexible time-based analyses.

```sql
ALTER TABLE minute_sleep 
ADD COLUMN date_only DATE, 
ADD COLUMN time_only TIME;

UPDATE minute_sleep
SET 
    date_only = DATE(Date),
    time_only = TIME(Date);
    
ALTER TABLE minute_sleep DROP COLUMN Date;
```

2.	Duplicate Removal:
•	Identified and removed duplicate records based on unique combinations of `Id`, `date_only`, `time_only`, `Value`, and `LogId`. 
•	To ensure selective removal, a temporary `temp_id` column was added, and duplicates were eliminated while retaining only the first occurrence of each unique entry.
```sql
ALTER TABLE minute_sleep ADD COLUMN temp_id INT AUTO_INCREMENT PRIMARY KEY;

CREATE TEMPORARY TABLE temp_minute_sleep AS
SELECT MIN(temp_id) AS temp_id
FROM minute_sleep
GROUP BY Id, date_only, time_only, Value, LogId;

DELETE FROM minute_sleep
WHERE temp_id NOT IN (
    SELECT temp_id FROM temp_minute_sleep
);

DROP TEMPORARY TABLE temp_minute_sleep;
```
3.	Null and Integrity Checks:
•	Verified that essential columns (`Id`, `date_only`, `time_only`, `Value`, and `LogId`) contained no null values.
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(Id) AS non_null_id,
    COUNT(date_only) AS non_null_date,
    COUNT(time_only) AS non_null_time,
    COUNT(Value) AS non_null_value,
    COUNT(LogId) AS non_null_logid
FROM minute_sleep;
```
•	Checked the date range, confirming that entries fell within the defined timeframe.
```sql
SELECT MIN(date_only) AS earliest_date, MAX(date_only) AS latest_date
FROM minute_sleep;
```
•	Conducted descriptive statistics on Value to assess data distribution and consistency across sleep states.
```sql
SELECT 
    MIN(Value) AS min_value,
    MAX(Value) AS max_value,
    AVG(Value) AS avg_value
FROM minute_sleep;
```
4. **Sample Output Verification**: The following random samples verify that the data in the `minute_sleep` table is correctly formatted and falls within the specified timeframe.
```sql
+------------+-------+-------------+------------+-----------+
| Id         | Value | LogId       | date_only  | time_only |
+------------+-------+-------------+------------+-----------+
| 5553957443 |     1 | 11534854778 | 2016-05-01 | 00:03:00  |
| 7086361926 |     3 | 11581172715 | 2016-05-08 | 00:33:00  |
| 2347167796 |     1 | 11213899920 | 2016-03-25 | 05:41:00  |
| 8378563200 |     1 | 11231487569 | 2016-03-27 | 04:49:30  |
| 7086361926 |     1 | 11341334566 | 2016-04-08 | 02:46:00  |
| 5553957443 |     1 | 11234106086 | 2016-03-24 | 02:53:30  |
| 8378563200 |     1 | 11471818605 | 2016-04-24 | 06:48:30  |
| 4445114986 |     2 | 11572172742 | 2016-05-07 | 06:06:30  |
| 8792009665 |     1 | 11304567345 | 2016-04-03 | 06:44:00  |
| 6962181067 |     1 | 11321297696 | 2016-04-06 | 01:08:00  |
+------------+-------+-------------+------------+-----------+
```

## 3.6 Processing and cleaning `minute_sleep` table:
### 3.6.1 Key SQL queries used for cleaning:
1.	**Duplicate Records Check**: Verified that there were no duplicate records in the table.
```sql
SELECT SEX, AGE_P, SRVY_YR, INTV_MON, DBHVPAY, DBHVCLY, DBHVPAN, DBHVCLN,
       VIGNO, VIGTP, VIGFREQW, VIGLNGNO, VIGLNGTP, VIGMIN,
       MODNO, MODTP, MODFREQW, MODLNGNO, MODLNGTP, MODMIN,
       STRNGNO, STRNGTP, STRFREQW, ASISLEEP, ASISLPFL, ASISLPST, ASISLPMD, ASIREST,
       COUNT(*) AS count
FROM nhis_2016_data
GROUP BY SEX, AGE_P, SRVY_YR, INTV_MON, DBHVPAY, DBHVCLY, DBHVPAN, DBHVCLN,
         VIGNO, VIGTP, VIGFREQW, VIGLNGNO, VIGLNGTP, VIGMIN,
         MODNO, MODTP, MODFREQW, MODLNGNO, MODLNGTP, MODMIN,
         STRNGNO, STRNGTP, STRFREQW, ASISLEEP, ASISLPFL, ASISLPST, ASISLPMD, ASIREST
HAVING COUNT(*) > 1;
```
2.	**Missing Values Check**: Ensured that all columns had non-null values, verifying completeness.
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(SEX) AS non_null_sex,
    COUNT(AGE_P) AS non_null_age,
    COUNT(SRVY_YR) AS non_null_survey_year,
    COUNT(INTV_MON) AS non_null_interview_month,
    COUNT(DBHVPAY) AS non_null_dbhvpay,
    COUNT(DBHVCLY) AS non_null_dbhvcly,
    COUNT(DBHVPAN) AS non_null_dbhvpan,
    COUNT(DBHVCLN) AS non_null_dbhvcln,
    COUNT(VIGNO) AS non_null_vigno,
    COUNT(VIGTP) AS non_null_vigtp,
    COUNT(VIGFREQW) AS non_null_vigfreqw,
    COUNT(VIGLNGNO) AS non_null_viglngno,
    COUNT(VIGLNGTP) AS non_null_viglngtp,
    COUNT(VIGMIN) AS non_null_vigmin,
    COUNT(MODNO) AS non_null_modno,
    COUNT(MODTP) AS non_null_modtp,
    COUNT(MODFREQW) AS non_null_modfreqw,
    COUNT(MODLNGNO) AS non_null_modlngno,
    COUNT(MODLNGTP) AS non_null_modlngtp,
    COUNT(MODMIN) AS non_null_modmin,
    COUNT(STRNGNO) AS non_null_strngno,
    COUNT(STRNGTP) AS non_null_strngtp,
    COUNT(STRFREQW) AS non_null_strfreqw,
    COUNT(ASISLEEP) AS non_null_asisleep,
    COUNT(ASISLPFL) AS non_null_asislpfl,
    COUNT(ASISLPST) AS non_null_asislpst,
    COUNT(ASISLPMD) AS non_null_asislpmd,
    COUNT(ASIREST) AS non_null_asirest
FROM nhis_2016_data;
```
3.	**Range Validation**: Checked that each column’s values fell within the expected ranges based on NHIS variable definitions mentioned on the dataset manual:
**Expected Ranges**:
- `AGE_P`: 18 to 85
- `DBHVPAY`, `DBHVCLY`, `DBHVPAN`, `DBHVCLN`: 1 (Yes) or 2 (No)
- `VIGNO`, `MODNO`, `STRNGNO`: 0 (never) or 1-995 (frequency counts)
- `VIGTP`, `MODTP`, `STRNGTP`: 0 (never) or 1 (per day), 2 (per week), 3 (per month), 4 (per year)
- `VIGLNGNO`, `MODLNGNO`: 1 to 995
- `VIGLNGTP`, `MODLNGTP`: 1 (minutes) or 2 (hours)
- `VIGFREQW`, `MODFREQW`, `STRFREQW`: 0 (less than once per week) to 28 (times per week), or 95 (never)
- `VIGMIN`, `MODMIN`: 10 to 720 (minutes)
- `ASISLEEP`: 1 to 24 (hours)
- `ASISLPFL`, `ASISLPST`: 0 (no issues) to 7 (7 or more times)
- `ASISLPMD`: 0 (no medication) to 7 (7 or more times)
- `ASIREST`: 0 (never rested) to 7 (days felt rested)

```sql
SELECT
    MIN(AGE_P) AS min_age, MAX(AGE_P) AS max_age,
    MIN(DBHVPAY) AS min_dbhvpay, MAX(DBHVPAY) AS max_dbhvpay,
    MIN(DBHVCLY) AS min_dbhvcly, MAX(DBHVCLY) AS max_dbhvcly,
    MIN(DBHVPAN) AS min_dbhvpan, MAX(DBHVPAN) AS max_dbhvpan,
    MIN(DBHVCLN) AS min_dbhvcln, MAX(DBHVCLN) AS max_dbhvcln,
    MIN(VIGNO) AS min_vigno, MAX(VIGNO) AS max_vigno,
    MIN(VIGTP) AS min_vigtp, MAX(VIGTP) AS max_vigtp,
    MIN(VIGFREQW) AS min_vigfreqw, MAX(VIGFREQW) AS max_vigfreqw,
    MIN(VIGLNGNO) AS min_viglngno, MAX(VIGLNGNO) AS max_viglngno,
    MIN(VIGLNGTP) AS min_viglngtp, MAX(VIGLNGTP) AS max_viglngtp,
    MIN(VIGMIN) AS min_vigmin, MAX(VIGMIN) AS max_vigmin,
    MIN(MODNO) AS min_modno, MAX(MODNO) AS max_modno,
    MIN(MODTP) AS min_modtp, MAX(MODTP) AS max_modtp,
    MIN(MODFREQW) AS min_modfreqw, MAX(MODFREQW) AS max_modfreqw,
    MIN(MODLNGNO) AS min_modlngno, MAX(MODLNGNO) AS max_modlngno,
    MIN(MODLNGTP) AS min_modlngtp, MAX(MODLNGTP) AS max_modlngtp,
    MIN(MODMIN) AS min_modmin, MAX(MODMIN) AS max_modmin,
    MIN(STRNGNO) AS min_strngno, MAX(STRNGNO) AS max_strngno,
    MIN(STRNGTP) AS min_strngtp, MAX(STRNGTP) AS max_strngtp,
    MIN(STRFREQW) AS min_strfreqw, MAX(STRFREQW) AS max_strfreqw,
    MIN(ASISLEEP) AS min_asisleep, MAX(ASISLEEP) AS max_asisleep,
    MIN(ASISLPFL) AS min_asislpfl, MAX(ASISLPFL) AS max_asislpfl,
    MIN(ASISLPST) AS min_asislpst, MAX(ASISLPST) AS max_asislpst,
    MIN(ASISLPMD) AS min_asislpmd, MAX(ASISLPMD) AS max_asislpmd,
    MIN(ASIREST) AS min_asirest, MAX(ASIREST) AS max_asirest
FROM nhis_2016_data;
```
Result:
```sql
+---------+---------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-------------+-------------+-------------+-------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+-------------+-------------+
| min_age | max_age | min_dbhvpay | max_dbhvpay | min_dbhvcly | max_dbhvcly | min_dbhvpan | max_dbhvpan | min_dbhvcln | max_dbhvcln | min_vigno | max_vigno | min_vigtp | max_vigtp | min_vigfreqw | max_vigfreqw | min_viglngno | max_viglngno | min_viglngtp | max_viglngtp | min_vigmin | max_vigmin | min_modno | max_modno | min_modtp | max_modtp | min_modfreqw | max_modfreqw | min_modlngno | max_modlngno | min_modlngtp | max_modlngtp | min_modmin | max_modmin | min_strngno | max_strngno | min_strngtp | max_strngtp | min_strfreqw | max_strfreqw | min_asisleep | max_asisleep | min_asislpfl | max_asislpfl | min_asislpst | max_asislpst | min_asislpmd | max_asislpmd | min_asirest | max_asirest |
+---------+---------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-------------+-------------+-------------+-------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+-------------+-------------+
|      18 |      85 |           1 |           2 |           1 |           2 |           1 |           2 |           1 |           2 |         1 |       500 |         1 |         4 |            0 |           28 |            1 |          210 |            1 |            2 |         10 |        600 |         1 |       150 |         1 |         4 |            0 |           28 |            1 |          309 |            1 |            2 |         10 |        720 |           0 |         100 |           0 |           4 |            0 |           95 |            2 |           18 |            0 |            7 |            0 |            7 |            0 |            7 |           0 |           7 |
+---------+---------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-----------+-----------+-----------+-----------+--------------+--------------+--------------+--------------+--------------+--------------+------------+------------+-------------+-------------+-------------+-------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+--------------+-------------+-------------+
```
4. **Sample Output Verification**: The following random samples verify that the data in the `nhis_2016_data` table is correctly formatted:
```sql
+-----+-------+---------+----------+---------+---------+---------+---------+-------+-------+----------+----------+----------+--------+-------+-------+----------+----------+----------+--------+---------+---------+----------+----------+----------+----------+----------+---------+
| SEX | AGE_P | SRVY_YR | INTV_MON | DBHVPAY | DBHVCLY | DBHVPAN | DBHVCLN | VIGNO | VIGTP | VIGFREQW | VIGLNGNO | VIGLNGTP | VIGMIN | MODNO | MODTP | MODFREQW | MODLNGNO | MODLNGTP | MODMIN | STRNGNO | STRNGTP | STRFREQW | ASISLEEP | ASISLPFL | ASISLPST | ASISLPMD | ASIREST |
+-----+-------+---------+----------+---------+---------+---------+---------+-------+-------+----------+----------+----------+--------+-------+-------+----------+----------+----------+--------+---------+---------+----------+----------+----------+----------+----------+---------+
|   2 |    55 |    2016 |        3 |       1 |       2 |       1 |       1 |     2 |     2 |        2 |        2 |        2 |    120 |     4 |     2 |        4 |        3 |        2 |    180 |       0 |       0 |       95 |        6 |        7 |        4 |        1 |       1 |
|   1 |    29 |    2016 |        5 |       2 |       2 |       1 |       1 |     5 |     2 |        5 |       90 |        1 |     90 |     2 |     2 |        2 |       60 |        1 |     60 |       1 |       1 |        7 |        6 |        0 |        0 |        0 |       2 |
|   2 |    55 |    2016 |        4 |       2 |       2 |       2 |       2 |     4 |     2 |        4 |        1 |        2 |     60 |     1 |     1 |        7 |        1 |        2 |     60 |       4 |       2 |        4 |        7 |        2 |        2 |        0 |       7 |
|   1 |    33 |    2016 |        4 |       2 |       2 |       2 |       2 |     1 |     2 |        1 |       20 |        1 |     20 |     3 |     2 |        3 |       30 |        1 |     30 |       0 |       0 |       95 |        7 |        0 |        0 |        0 |       7 |
|   1 |    68 |    2016 |        4 |       2 |       2 |       1 |       1 |     3 |     2 |        3 |       60 |        1 |     60 |     1 |     2 |        1 |        3 |        2 |    180 |       2 |       2 |        2 |        6 |        0 |        0 |        0 |       7 |
|   2 |    30 |    2016 |        4 |       2 |       2 |       2 |       2 |     2 |     2 |        2 |       30 |        1 |     30 |     3 |     2 |        3 |       30 |        1 |     30 |       0 |       0 |       95 |        8 |        4 |        0 |        0 |       5 |
|   2 |    23 |    2016 |        4 |       2 |       2 |       2 |       2 |     3 |     2 |        3 |        3 |        2 |    180 |     4 |     2 |        4 |        2 |        2 |    120 |       1 |       2 |        1 |        8 |        7 |        0 |        0 |       7 |
|   1 |    30 |    2016 |        4 |       2 |       2 |       2 |       1 |     7 |     2 |        7 |        1 |        2 |     60 |     7 |     2 |        7 |        1 |        2 |     60 |       5 |       2 |        5 |        7 |        0 |        0 |        0 |       3 |
|   1 |    18 |    2016 |        4 |       2 |       2 |       1 |       2 |     1 |     2 |        1 |       90 |        1 |     90 |     1 |     1 |        7 |       90 |        1 |     90 |       5 |       2 |        5 |        8 |        3 |        2 |        0 |       0 |
|   2 |    23 |    2016 |        4 |       2 |       2 |       2 |       2 |     4 |     2 |        4 |       10 |        1 |     10 |     4 |     2 |        4 |       10 |        1 |     10 |       4 |       2 |        4 |        9 |        2 |        0 |        0 |       6 |
+-----+-------+---------+----------+---------+---------+---------+---------+-------+-------+----------+----------+----------+--------+-------+-------+----------+----------+----------+--------+---------+---------+----------+----------+----------+----------+----------+---------+
```
## 3.7 Creating Summary Tables:
The purpose of this step is to prepare simplified, aggregated tables that focus on key metrics. These summary tables will make it easier to compare Fitbit data with NHIS benchmarks and analyze insights effectively.

### 3.7.1: Weekly Physical Activity Summary Table
In this sub-step, I created a `weekly_activity_summary` table of key physical activity metrics from the `daily_activity` dataset. The goal was to aggregate moderate and vigorous activity minutes, steps, and calories for each Fitbit user on a weekly basis. This summary will facilitate comparisons with **NHIS data** and **WHO** recommendations on physical activity.

**Initial Actions for Table Setup and Data Aggregation**:
1.	**Source Data**: This summary table is derived from the `daily_activity` dataset, where individual daily records are aggregated by user ID and week.
2.	Create the `weekly_activity_summary` Table with Key Metrics:
- **Total Steps**: Sum of daily steps for each week.
- **Total Calories**: Sum of calories burned each week.
- **Total Vigorous Minutes**: Sum of very active minutes each week.
- **Total Moderate Minutes**: Sum of fairly active minutes each week.

#### Key SQL queries used:
1. Create the `weekly_activity_summary` Table:
```sql
CREATE TABLE weekly_activity_summary (
    Id BIGINT,
    year INT,
    week INT,
    total_weekly_steps INT,
    total_weekly_calories INT,
    total_weekly_vigorous_minutes INT,
    total_weekly_moderate_minutes INT,
    PRIMARY KEY (Id, year, week)
);), WEEK(ActivityDate);
```
2.	Data Aggregation and Insertion:
Insert the weekly activity data directly into this permanent table, using the same aggregation logic as before.
```sql
INSERT INTO weekly_activity_summary (Id, year, week, total_weekly_steps, total_weekly_calories, total_weekly_vigorous_minutes, total_weekly_moderate_minutes)
SELECT 
    Id,
    YEAR(ActivityDate) AS year,
    WEEK(ActivityDate) AS week,
    SUM(TotalSteps) AS total_weekly_steps,
    SUM(Calories) AS total_weekly_calories,
    SUM(VeryActiveMinutes) AS total_weekly_vigorous_minutes,
    SUM(FairlyActiveMinutes) AS total_weekly_moderate_minutes
FROM 
    daily_activity
GROUP BY 
    Id, YEAR(ActivityDate), WEEK(ActivityDate);
```

3. Verification of Data in weekly_activity_summary
```sql
DESCRIBE weekly_activity_summary;
```
Output: 
```sql
-------------------------------+--------+------+-----+---------+-------+
| Field                         | Type   | Null | Key | Default | Extra |
+-------------------------------+--------+------+-----+---------+-------+
| Id                            | bigint | NO   | PRI | null    |       |
| year                          | int    | NO   | PRI | null    |       |
| week                          | int    | NO   | PRI | null    |       |
| total_weekly_steps            | int    | YES  |     | null    |       |
| total_weekly_calories         | int    | YES  |     | null    |       |
| total_weekly_vigorous_minutes | int    | YES  |     | null    |       |
| total_weekly_moderate_minutes | int    | YES  |     | null    |       |
+-------------------------------+--------+------+-----+---------+-------+
```

4. **Sample Output Verification**: The following random samples verify that the data in the `weekly_activity_summary` table is correctly formatted:
```sql
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+
| Id         | year | week | total_weekly_steps | total_weekly_calories | total_weekly_vigorous_minutes | total_weekly_moderate_minutes |
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+
| 2026352035 | 2016 |   16 |              31548 |                 10341 |                             0 |                             0 |
| 2026352035 | 2016 |   15 |              27531 |                 10148 |                             3 |                             8 |
| 2022484408 | 2016 |   19 |              51858 |                 11751 |                           166 |                           116 |
| 2320127002 | 2016 |   17 |              19904 |                 11154 |                             3 |                             5 |
| 7007744171 | 2016 |   18 |              59620 |                 13329 |                           219 |                            81 |
| 4702921684 | 2016 |   15 |              57230 |                 20954 |                             5 |                            57 |
| 8877689391 | 2016 |   18 |              92845 |                 22478 |                           389 |                            83 |
| 4319703577 | 2016 |   18 |              61708 |                 15334 |                            52 |                           123 |
| 5577150313 | 2016 |   17 |              64415 |                 25773 |                           775 |                           283 |
| 6775888955 | 2016 |   15 |              15229 |                  9259 |                            67 |                            60 |
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+
```

### 3.7.2: Weekly Sleep Duration Summary
In this step, we analyzed users’ sleep patterns by aggregating weekly sleep metrics from the `sleep_day` table. This allows us to observe trends in users' sleep behaviours, such as total weekly time in bed, total weekly minutes asleep, and average daily sleep duration.

#### Key SQL queries used:
1.	**Create the `weekly_sleep_summary` Table:** We'll create a table named `weekly_sleep_summary` with columns for `Id`, `year`, `week`, `total_weekly_time_in_bed`, `total_weekly_minutes_asleep`, and `average_daily_minutes_asleep`.
```sql
CREATE TABLE weekly_sleep_summary (
    Id BIGINT,
    year INT,
    week INT,
    total_weekly_time_in_bed INT,
    total_weekly_minutes_asleep INT,
    average_daily_minutes_asleep DECIMAL(10,2),
    PRIMARY KEY (Id, year, week)
);
```
2.	**Data Aggregation and Insertion**: Now, we will populate the `weekly_sleep_summary` table by aggregating data from `sleep_day`:
```sql
INSERT INTO weekly_sleep_summary (Id, year, week, total_weekly_time_in_bed, total_weekly_minutes_asleep, average_daily_minutes_asleep)
SELECT 
    Id,
    YEAR(SleepDay) AS year,
    WEEK(SleepDay) AS week,
    SUM(TotalTimeInBed) AS total_weekly_time_in_bed,
    SUM(TotalMinutesAsleep) AS total_weekly_minutes_asleep,
    AVG(TotalMinutesAsleep) AS average_daily_minutes_asleep
FROM 
    sleep_day
GROUP BY 
    Id, YEAR(SleepDay), WEEK(SleepDay);
```
3. **Sample Output Verification**: The following random samples verify that the data in the `weekly_sleep_summary` table is correctly formatted:
```sql
+------------+------+------+--------------------------+-----------------------------+------------------------------+
| Id         | year | week | total_weekly_time_in_bed | total_weekly_minutes_asleep | average_daily_minutes_asleep |
+------------+------+------+--------------------------+-----------------------------+------------------------------+
| 5577150313 | 2016 |   16 |                     3286 |                        3108 |                       444.00 |
| 4388161847 | 2016 |   18 |                     2378 |                        2248 |                       449.60 |
| 2320127002 | 2016 |   16 |                       69 |                          61 |                        61.00 |
| 4445114986 | 2016 |   15 |                     2233 |                        2039 |                       407.80 |
| 2026352035 | 2016 |   15 |                     2819 |                        2626 |                       525.20 |
| 1503960366 | 2016 |   15 |                     1562 |                        1463 |                       365.75 |
| 4319703577 | 2016 |   16 |                     2929 |                        2783 |                       463.83 |
| 7086361926 | 2016 |   18 |                     2644 |                        2556 |                       426.00 |
| 2026352035 | 2016 |   19 |                     2542 |                        2408 |                       481.60 |
| 4020332650 | 2016 |   18 |                     1524 |                        1411 |                       352.75 |
+------------+------+------+--------------------------+-----------------------------+------------------------------+
```

### 3.7.3: Weekly Heart Rate Summary
The purpose of this step is to create a weekly summary of users’ heart rate data, focusing on their average weekly heart rates. This information will be instrumental in understanding heart rate trends over time and providing insights into user health and wellness. We calculate this summary from the `heartrate_seconds` table by averaging each user's heart rate values within each week. This dataset will later be compared to WHO and NHIS benchmarks to provide a comprehensive view of the users' cardiovascular health.
#### Key SQL queries used:
1.	**Create the `weekly_heart_rate_summary` Table:**
```sql
CREATE TABLE weekly_heart_rate_summary (
    Id BIGINT,
    year INT,
    week INT,
    average_weekly_heart_rate DECIMAL(6,3)
);
```

2.	**Data Aggregation and Insertion**: Using the `heartrate_seconds` table, we calculated the weekly average heart rate for each user and inserted these aggregated values into the `weekly_heart_rate_summary` table. To accommodate possible high values, we set `average_weekly_heart_rate` to `DECIMAL(6,3)`.
```sql
INSERT INTO weekly_heart_rate_summary (Id, year, week, average_weekly_heart_rate)
SELECT
    Id,
    YEAR(Time) AS year,
    WEEK(Time) AS week,
    ROUND(AVG(Value), 4) AS average_weekly_heart_rate
FROM
    heartrate_seconds
GROUP BY
    Id, YEAR(Time), WEEK(Time)
ORDER BY
    Id, year, week;
```

3.	**Duplicate Check**: Ensured there were no duplicate entries in the `weekly_heart_rate_summary` table by verifying that each user had only one average weekly heart rate per week. 
```sql
SELECT Id, year, week, COUNT(*)
FROM weekly_heart_rate_summary
GROUP BY Id, year, week
HAVING COUNT(*) > 1;
```

4.	**Null or Missing Values Check**: Verified that there were no null values in the `Id`, `year`, `week`, or `average_weekly_heart_rate` columns.
```sql
SELECT 
    COUNT(Id) AS non_null_id,
    COUNT(year) AS non_null_year,
    COUNT(week) AS non_null_week,
    COUNT(average_weekly_heart_rate) AS non_null_avg_heart_rate
FROM weekly_heart_rate_summary;
```
5.	**Range Validation**: Ensured that the values for average_weekly_heart_rate fell within expected physiological ranges (generally between 40 and 200 bpm for resting heart rate averages).
```sql
SELECT *
FROM weekly_heart_rate_summary
WHERE average_weekly_heart_rate < 40 OR average_weekly_heart_rate > 200;
```

6. **Sample Output Verification**: The following random samples verify that the data in the `weekly_heart_rate_summary` table is correctly formatted:
```sql
+------------+------+------+---------------------------+
| Id         | year | week | average_weekly_heart_rate |
+------------+------+------+---------------------------+
| 4388161847 | 2016 |   15 |                    69.705 |
| 8792009665 | 2016 |   18 |                    70.516 |
| 6775888955 | 2016 |   18 |                    99.686 |
| 2022484408 | 2016 |   16 |                    81.804 |
| 6962181067 | 2016 |   15 |                    79.571 |
| 6117666160 | 2016 |   16 |                    82.838 |
| 6962181067 | 2016 |   16 |                    79.191 |
| 5553957443 | 2016 |   17 |                    69.499 |
| 2347167796 | 2016 |   17 |                    73.763 |
| 4020332650 | 2016 |   16 |                    98.826 |
+------------+------+------+---------------------------+
```

## 3.8 Additional Consistency Checks:
The purpose of this step is to perform a final round of consistency checks across all weekly summary tables (`weekly_activity_summary`, `weekly_sleep_summary`, `weekly_heart_rate_summary`, and `final_weekly_summary`). By identifying and addressing any remaining discrepancies, we ensure the reliability of the dataset used in the analysis phase.

### 3.8.1: 
1. **Check Consistent Date Ranges** - To confirm that all dates are within our defined observation period (e.g., March 12, 2016, to May 12, 2016).
#### Key SQL queries used:

- For `weekly_activity_summary`:
```sql
SELECT MIN(week), MAX(week), MIN(year), MAX(year)
FROM weekly_activity_summary;
```
**Output**:
```sql
+-----------+-----------+-----------+-----------+
| MIN(week) | MAX(week) | MIN(year) | MAX(year) |
+-----------+-----------+-----------+-----------+
|        10 |        19 |      2016 |      2016 |
+-----------+-----------+-----------+-----------+
```
- For `weekly_sleep_summary`:
```sql
SELECT MIN(week), MAX(week), MIN(year), MAX(year)
FROM weekly_sleep_summary;
```
**Output**:
```sql
+-----------+-----------+-----------+-----------+
| MIN(week) | MAX(week) | MIN(year) | MAX(year) |
+-----------+-----------+-----------+-----------+
|        15 |        19 |      2016 |      2016 |
+-----------+-----------+-----------+-----------+
```
For `weekly_heart_rate_summary`:
```sql
SELECT MIN(week), MAX(week), MIN(year), MAX(year)
FROM weekly_heart_rate_summary;
```
**Output**:
```sql
+-----------+-----------+-----------+-----------+
| MIN(week) | MAX(week) | MIN(year) | MAX(year) |
+-----------+-----------+-----------+-----------+
|        13 |        19 |      2016 |      2016 |
+-----------+-----------+-----------+-----------+
```

*Note: The date ranges for each table appear to vary slightly, with `weekly_activity_summary` starting from week 10, `weekly_heart_rate_summary` starting from week 13, and `weekly_sleep_summary` starting from week 15. This discrepancy might be due to missing or incomplete data for certain weeks in the source tables. Here’s how we’ll address this by filtering the tables to a common observation window (Weeks 15-19, 2016)*
```sql
DELETE FROM weekly_activity_summary
WHERE week < 15 OR week > 19;

DELETE FROM weekly_sleep_summary
WHERE week < 15 OR week > 19;

DELETE FROM weekly_heart_rate_summary
WHERE week < 15 OR week > 19;
```
2. **Verify Consistency of User IDs Across Tables:** - Identify any user IDs that might be missing from one or more of the tables within the specified weeks.
(*Note: This step will filter down the Fitbit user sample size from 30 users to 12 users. However, this is an intended approach to ensure that comparative study between Fitbit, NHIS and WHO data are cohesive*)
#### Key SQL queries used:
- Find User IDs Missing in `weekly_activity_summary`, `weekly_sleep_summary` and `weekly_heart_rate_summary`:
```sql
SELECT DISTINCT ws.Id
FROM weekly_sleep_summary ws
LEFT JOIN weekly_activity_summary wa ON ws.Id = wa.Id AND ws.week = wa.week
WHERE wa.Id IS NULL;
```
**Output**:
```sql
0 records retrieved
```

```sql
SELECT DISTINCT wa.Id
FROM weekly_activity_summary wa
LEFT JOIN weekly_sleep_summary ws ON wa.Id = ws.Id AND wa.week = ws.week
WHERE ws.Id IS NULL;
```
**Output**:
```sql
+------------+
| Id         |
+------------+
| 1624580081 |
| 1644430081 |
| 1844505072 |
| 1927972279 |
| 2022484408 |
| 2320127002 |
| 2873212765 |
| 3372868164 |
| 4020332650 |
| 4057192912 |
| 4558609924 |
| 6290855005 |
| 6775888955 |
| 7007744171 |
| 8053475328 |
| 8253242879 |
| 8583815059 |
| 8877689391 |
+------------+
```

```sql
SELECT DISTINCT wa.Id
FROM weekly_activity_summary wa
LEFT JOIN weekly_heart_rate_summary wh ON wa.Id = wh.Id AND wa.week = wh.week
WHERE wh.Id IS NULL;
```
**Output**:
```sql
+------------+
| Id         |
+------------+
| 1503960366 |
| 1624580081 |
| 1644430081 |
| 1844505072 |
| 1927972279 |
| 2026352035 |
| 2320127002 |
| 2873212765 |
| 3372868164 |
| 3977333714 |
| 4057192912 |
| 4319703577 |
| 4445114986 |
| 4702921684 |
| 6290855005 |
| 7086361926 |
| 8053475328 |
| 8253242879 |
| 8378563200 |
| 8583815059 |
+------------+
```
**Inference**:
This shows that there are several user IDs present in the `weekly_activity_summary` table that are missing from the other two tables:
- Missing from `weekly_sleep_summary`: **18** user IDs
- Missing from `weekly_heart_rate_summary`: **20** user IDs

This inconsistency will affect our analysis because we need a full dataset for each user across all metrics. Here’s the plan to handle this:

1. **Filter Out Inconsistent User IDs:**
We’ll create a final consolidated table that only includes user IDs present in all three summary tables for weeks 15 to 19. For this, first, we’ll find user IDs that are consistently represented across all three tables within weeks 15 to 19. 
```sql
SELECT DISTINCT wa.Id
FROM weekly_activity_summary wa
JOIN weekly_sleep_summary ws ON wa.Id = ws.Id AND wa.week = ws.week
JOIN weekly_heart_rate_summary wh ON wa.Id = wh.Id AND wa.week = wh.week
WHERE wa.week BETWEEN 15 AND 19;
```
**Output**: We now have a list of 12 user IDs with complete data across all three summary tables for weeks 15 to 19.
```sql
+------------+
| Id         |
+------------+
| 2026352035 |
| 2347167796 |
| 4020332650 |
| 4388161847 |
| 4558609924 |
| 5553957443 |
| 5577150313 |
| 6117666160 |
| 6775888955 |
| 6962181067 |
| 7007744171 |
| 8792009665 |
+------------+
```

2. **Create a Unified View or Final Summary Table:**
We’ll generate a new table called `final_weekly_summary` that will consolidate the relevant weekly metrics for these user IDs. This table will include:
- `Id`: User ID
- `year`: Year (2016)
- `week`: Week number (15 to 19)
- `total_weekly_steps`: Total steps for the week
- `total_weekly_calories`: Total calories for the week
- `total_weekly_vigorous_minutes`: Total vigorous activity minutes for the week
- `total_weekly_moderate_minutes`: Total moderate activity minutes for the week
- `total_weekly_time_in_bed`: Total time in bed for the week
- `total_weekly_minutes_asleep`: Total minutes asleep for the week
- `average_daily_minutes_asleep`: Average daily minutes asleep for the week
- `average_weekly_heart_rate`: Average heart rate for the week

#### Key SQL queries used for this:
- **Create the `final_weekly_summary ` Table**
```sql
CREATE TABLE final_weekly_summary (
    Id BIGINT,
    year INT,
    week INT,
    total_weekly_steps INT,
    total_weekly_calories INT,
    total_weekly_vigorous_minutes INT,
    total_weekly_moderate_minutes INT,
    total_weekly_time_in_bed INT,
    total_weekly_minutes_asleep INT,
    average_daily_minutes_asleep FLOAT,
    average_weekly_heart_rate FLOAT
);
```
- **Insert Data into the Table** - Insert data for the 12 users across weeks 15 to 19
```sql
INSERT INTO final_weekly_summary (Id, year, week, total_weekly_steps, total_weekly_calories, total_weekly_vigorous_minutes, total_weekly_moderate_minutes, total_weekly_time_in_bed, total_weekly_minutes_asleep, average_daily_minutes_asleep, average_weekly_heart_rate)
SELECT 
    wa.Id,
    wa.year,
    wa.week,
    wa.total_weekly_steps,
    wa.total_weekly_calories,
    wa.total_weekly_vigorous_minutes,
    wa.total_weekly_moderate_minutes,
    ws.total_weekly_time_in_bed,
    ws.total_weekly_minutes_asleep,
    ws.average_daily_minutes_asleep,
    wh.average_weekly_heart_rate
FROM 
    weekly_activity_summary wa
JOIN 
    weekly_sleep_summary ws ON wa.Id = ws.Id AND wa.week = ws.week
JOIN 
    weekly_heart_rate_summary wh ON wa.Id = wh.Id AND wa.week = wh.week
WHERE 
    wa.Id IN (2026352035, 2347167796, 4020332650, 4388161847, 4558609924, 5553957443, 5577150313, 6117666160, 6775888955, 6962181067, 7007744171, 8792009665)
    AND wa.week BETWEEN 15 AND 19;
```

- **Verification of `final_weekly_summary` Table** -
```sql
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+--------------------------+-----------------------------+------------------------------+---------------------------+
| Id         | year | week | total_weekly_steps | total_weekly_calories | total_weekly_vigorous_minutes | total_weekly_moderate_minutes | total_weekly_time_in_bed | total_weekly_minutes_asleep | average_daily_minutes_asleep | average_weekly_heart_rate |
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+--------------------------+-----------------------------+------------------------------+---------------------------+
| 2026352035 | 2016 |   16 |              31548 |                 10341 |                             0 |                             0 |                     3161 |                        2915 |                       485.83 |                    68.656 |
| 2026352035 | 2016 |   17 |              40236 |                 11012 |                             0 |                             0 |                     3329 |                        3145 |                       524.17 |                    99.506 |
| 2026352035 | 2016 |   18 |              47741 |                 11552 |                             0 |                             0 |                     3203 |                        3079 |                       513.17 |                    84.135 |
| 2026352035 | 2016 |   19 |              33938 |                  7627 |                             0 |                             0 |                     2542 |                        2408 |                        481.6 |                    98.234 |
| 2347167796 | 2016 |   15 |              83382 |                 15368 |                           132 |                           244 |                     1524 |                        1364 |                       454.67 |                    79.442 |
| 2347167796 | 2016 |   16 |              66214 |                 14993 |                            87 |                           161 |                     3004 |                        2760 |                          460 |                    76.278 |
| 2347167796 | 2016 |   17 |              41837 |                 10594 |                            41 |                            20 |                     2842 |                        2578 |                       429.67 |                    73.763 |
| 4020332650 | 2016 |   15 |              20633 |                 16991 |                           110 |                            61 |                      618 |                         578 |                          289 |                     84.56 |
| 4020332650 | 2016 |   18 |              37452 |                 18022 |                            51 |                            99 |                     1524 |                        1411 |                       352.75 |                    81.893 |
| 4020332650 | 2016 |   19 |              20243 |                 12565 |                             5 |                            13 |                      896 |                         806 |                          403 |                    79.221 |
| 4388161847 | 2016 |   15 |              45316 |                 15184 |                            17 |                            58 |                      974 |                         925 |                        462.5 |                    69.705 |
| 4388161847 | 2016 |   16 |              71833 |                 21863 |                           208 |                           108 |                     2578 |                        2470 |                       352.86 |                    68.658 |
| 4388161847 | 2016 |   17 |              75697 |                 21407 |                            96 |                           178 |                     2329 |                        2147 |                        429.4 |                    63.738 |
| 4388161847 | 2016 |   18 |              91001 |                 22871 |                           263 |                           198 |                     2378 |                        2248 |                        449.6 |                    64.499 |
| 4388161847 | 2016 |   19 |              51385 |                 14585 |                           134 |                            89 |                     1475 |                        1414 |                        353.5 |                    65.296 |
| 4558609924 | 2016 |   16 |              59012 |                 14562 |                            53 |                            94 |                      137 |                         126 |                          126 |                    82.787 |
| 4558609924 | 2016 |   17 |              62966 |                 14927 |                            80 |                           134 |                      300 |                         274 |                          137 |                    83.879 |
| 4558609924 | 2016 |   18 |              44421 |                 13510 |                            58 |                            34 |                      129 |                         115 |                          115 |                    77.473 |
| 4558609924 | 2016 |   19 |              39844 |                 10361 |                            87 |                           114 |                      134 |                         123 |                          123 |                     81.79 |
| 5553957443 | 2016 |   15 |              73277 |                 13930 |                           174 |                           144 |                     2465 |                        2281 |                        456.2 |                    67.122 |
| 5553957443 | 2016 |   16 |              50717 |                 12881 |                           141 |                            86 |                     3540 |                        3237 |                       462.43 |                    68.335 |
| 5553957443 | 2016 |   17 |              60536 |                 13186 |                           202 |                            73 |                     3747 |                        3391 |                       484.43 |                    69.499 |
| 5553957443 | 2016 |   18 |              57861 |                 13123 |                           114 |                            76 |                     3424 |                        3119 |                       445.57 |                    67.355 |
| 5553957443 | 2016 |   19 |              42099 |                  8803 |                           131 |                            76 |                     2506 |                        2340 |                          468 |                    71.293 |
| 5577150313 | 2016 |   15 |              60802 |                 23146 |                           637 |                           219 |                     2251 |                        2126 |                        425.2 |                    69.553 |
| 5577150313 | 2016 |   16 |              70668 |                 25369 |                           705 |                           236 |                     3286 |                        3108 |                          444 |                    68.889 |
| 5577150313 | 2016 |   17 |              64415 |                 25773 |                           775 |                           283 |                     3195 |                        2974 |                       424.86 |                    68.586 |
| 5577150313 | 2016 |   18 |              49558 |                 19757 |                           439 |                           146 |                     2206 |                        2089 |                        417.8 |                    68.963 |
| 5577150313 | 2016 |   19 |              16328 |                  7995 |                           183 |                            63 |                     1038 |                         935 |                        467.5 |                    71.864 |
| 6117666160 | 2016 |   15 |              28469 |                  5693 |                             7 |                            21 |                      398 |                         380 |                          380 |                    90.954 |
| 6117666160 | 2016 |   16 |              73130 |                 19345 |                            37 |                            29 |                     3489 |                        3248 |                          464 |                    82.838 |
| 6117666160 | 2016 |   17 |              46559 |                 14045 |                             0 |                             7 |                     1976 |                        1888 |                          472 |                    83.875 |
| 6117666160 | 2016 |   18 |              37345 |                 13250 |                             0 |                             0 |                     2195 |                        2055 |                       513.75 |                    83.588 |
| 6117666160 | 2016 |   19 |              11805 |                  3498 |                             0 |                             0 |                     1125 |                        1047 |                        523.5 |                    79.502 |
| 6775888955 | 2016 |   15 |              15229 |                  9259 |                            67 |                            60 |                     1107 |                        1049 |                       349.67 |                    81.814 |
| 6962181067 | 2016 |   15 |              58692 |                 13395 |                           148 |                            86 |                     2353 |                        2231 |                        446.2 |                    79.571 |
| 6962181067 | 2016 |   16 |              88810 |                 15122 |                           242 |                           201 |                     3301 |                        3167 |                       452.43 |                    79.191 |
| 6962181067 | 2016 |   17 |              72491 |                 14383 |                           160 |                           115 |                     3166 |                        3102 |                       443.14 |                    76.619 |
| 6962181067 | 2016 |   18 |              63869 |                 13923 |                            68 |                           133 |                     3068 |                        2921 |                       417.29 |                    76.327 |
| 6962181067 | 2016 |   19 |              42287 |                  8908 |                           129 |                            84 |                     2562 |                        2467 |                        493.4 |                    78.527 |
| 7007744171 | 2016 |   15 |              70431 |                 18092 |                           162 |                            75 |                       82 |                          79 |                           79 |                    90.374 |
| 7007744171 | 2016 |   18 |              59620 |                 13329 |                           219 |                            81 |                       61 |                          58 |                           58 |                    89.844 |
| 8792009665 | 2016 |   15 |              12613 |                 13649 |                             0 |                             0 |                     1925 |                        1838 |                        459.5 |                    70.845 |
| 8792009665 | 2016 |   16 |              12604 |                  9075 |                             9 |                            75 |                     1314 |                        1258 |                       419.33 |                    72.842 |
| 8792009665 | 2016 |   17 |              25170 |                 14571 |                            19 |                            42 |                     1610 |                        1566 |                        391.5 |                    76.147 |
| 8792009665 | 2016 |   18 |               8154 |                  8341 |                             0 |                             0 |                     1958 |                        1873 |                       468.25 |                    70.516 |
+------------+------+------+--------------------+-----------------------+-------------------------------+-------------------------------+--------------------------+-----------------------------+------------------------------+---------------------------+
```

## 3.9 Integrate WHO Baseline Recommendations:
The aim of this step is to incorporate WHO-recommended baseline metrics into our dataset for meaningful comparison. This will allow us to evaluate users' weekly activity and sleep data against globally recognized health standards. Specifically, we’ll focus on recommended activity and sleep levels, enabling a direct assessment of each user's adherence to these standards.

**WHO Baseline Metrics**
Based on WHO guidelines, we’re establishing the following baselines for each key metric:
•	P**hysical Activity**: Adults should engage in at least **150 minutes** of moderate activity or **75 minutes** of vigorous activity per week or an equivalent combination.
•	**Sleep Duration**: Adults should aim for **7-8 hours** of sleep per night, translating to a total of **49-56 hours** (or **2940-3360 minutes**) per week.
These baselines will serve as reference points in the Analyze phase, where we will assess whether users are meeting or falling short of these recommended thresholds.

#### Key SQL queries used:
**Add WHO Baseline Fields**: Added two new columns to the `final_weekly_summary` table to hold the **WHO** baselines:
- `who_baseline_activity_minutes` with a value of **150** for moderate or **75** for vigorous activity minutes.
- `who_baseline_sleep_minutes` with a value range of **2940–3360** minutes per week.
```sql
ALTER TABLE final_weekly_summary 
ADD COLUMN who_baseline_activity_minutes INT DEFAULT 150, 
ADD COLUMN who_baseline_sleep_minutes INT DEFAULT 2940;
```

## 3.10 Integrate NHIS Baseline Averages as Benchmarks:
Use the NHIS averages (vigorous minutes, moderate minutes, and sleep duration) as benchmarks to compare against the Fitbit weekly summaries for activity and sleep. This will allow us to assess how Fitbit users’ activity and sleep metrics align with or differ from national averages.
```sql
CREATE VIEW nhis_2016_summary AS
SELECT 
   AVG(VIGMIN * VIGFREQW) AS avg_nhis_vigorous_minutes,
   AVG(MODMIN * MODFREQW) AS avg_nhis_moderate_minutes,
   AVG(ASISLEEP * 60) AS avg_nhis_sleep_duration_minutes  
FROM 
   nhis_2016_data;
```
```sql
+---------------------------+---------------------------+---------------------------------+
| avg_nhis_vigorous_minutes | avg_nhis_moderate_minutes | avg_nhis_sleep_duration_minutes |
+---------------------------+---------------------------+---------------------------------+
|                  201.0707 |                  235.7518 |                        418.8351 |
+---------------------------+---------------------------+---------------------------------+
```
**Reasoning for this:**
1. 	The NHIS benchmarks provide a practical and reliable reference for analyzing user fitness and wellness metrics, as they represent national averages for similar metrics.
2. 	WHO guidelines are often broader, whereas NHIS provides actual observed data that aligns well with the specifics of the dataset we’re analyzing.

# 4. ANALYZE PHASE
In the Analysis phase, our goal is to interpret user behaviour by comparing activity and sleep data against benchmarks established by the **NHIS** dataset. We aimed to assess how well users are meeting recommended levels of physical activity and sleep, which can provide insights for **Bellabeat's IVY+ wellness tracker**. These findings will inform recommendations on promoting healthier habits among users, ultimately aligning with **Bellabeat’s** mission to support wellness and fitness goals.
Approach:
We structured our analysis to evaluate user behaviour in three key areas:
1.	**Vigorous Activity Minutes**
2.	**Moderate Activity Minutes**
3.	**Sleep Duration**
For each metric, we calculated weekly totals and then compared them with **NHIS** benchmark averages. The comparative analysis focused on percentage differences to indicate whether users are meeting, exceeding, or falling short of recommended activity and sleep levels.

## 4.1 Analysis Steps and Key SQL Queries:
1.	**Calculating Average, Minimum, and Maximum Weekly Metrics:** To understand the range of user activity and sleep, we calculated the average, minimum, and maximum values for steps, calories burned, vigorous and moderate activity minutes, time in bed, minutes asleep, and heart rate. These basic aggregations allowed us to understand general patterns across users before making specific comparisons.
**Query 1: Descriptive Statistics for `total_weekly_steps`**
```sql
SELECT 
    AVG(total_weekly_steps) AS avg_weekly_steps,
    MIN(total_weekly_steps) AS min_weekly_steps,
    MAX(total_weekly_steps) AS max_weekly_steps
FROM final_weekly_summary;
```
**Output**:
```sql
+------------------+------------------+------------------+
| avg_weekly_steps | min_weekly_steps | max_weekly_steps |
+------------------+------------------+------------------+
|       48657.3478 |             8154 |            91001 |
+------------------+------------------+------------------+
```
**Query 2: Descriptive Statistics for `total_weekly_calories`**
```sql
SELECT 
    AVG(total_weekly_calories) AS avg_weekly_calories,
    MIN(total_weekly_calories) AS min_weekly_calories,
    MAX(total_weekly_calories) AS max_weekly_calories
FROM final_weekly_summary;
```
**Output**:
```sql
+---------------------+---------------------+---------------------+
| avg_weekly_calories | min_weekly_calories | max_weekly_calories |
+---------------------+---------------------+---------------------+
|          14134.2609 |                3498 |               25773 |
+---------------------+---------------------+---------------------+
```
**Query 3: Descriptive Statistics for `total_weekly_vigorous_minutes`**
```sql
SELECT 
    AVG(total_weekly_vigorous_minutes) AS avg_weekly_vigorous_minutes,
    MIN(total_weekly_vigorous_minutes) AS min_weekly_vigorous_minutes,
    MAX(total_weekly_vigorous_minutes) AS max_weekly_vigorous_minutes
FROM 
    final_weekly_summary;
```
**Output**:
```sql
+-----------------------------+-----------------------------+-----------------------------+
| avg_weekly_vigorous_minutes | min_weekly_vigorous_minutes | max_weekly_vigorous_minutes |
+-----------------------------+-----------------------------+-----------------------------+
|                    134.5652 |                           0 |                         775 |
+-----------------------------+-----------------------------+-----------------------------+
```
**Query 4: Descriptive Statistics for `total_weekly_moderate_minutes`**
```sql
SELECT 
    AVG(total_weekly_moderate_minutes) AS avg_weekly_moderate_minutes,
    MIN(total_weekly_moderate_minutes) AS min_weekly_moderate_minutes,
    MAX(total_weekly_moderate_minutes) AS max_weekly_moderate_minutes
FROM 
    final_weekly_summary;
```
**Output**:
```sql
+-----------------------------+-----------------------------+-----------------------------+
| avg_weekly_moderate_minutes | min_weekly_moderate_minutes | max_weekly_moderate_minutes |
+-----------------------------+-----------------------------+-----------------------------+
|                     87.3043 |                           0 |                         283 |
+-----------------------------+-----------------------------+-----------------------------+
```
**Query 5: Descriptive Statistics for `total_weekly_time_in_bed`**
```sql
SELECT 
    AVG(total_weekly_time_in_bed) AS avg_weekly_time_in_bed,
    MIN(total_weekly_time_in_bed) AS min_weekly_time_in_bed,
    MAX(total_weekly_time_in_bed) AS max_weekly_time_in_bed
FROM 
    final_weekly_summary;
```
**Output**:
```sql
+------------------------+------------------------+------------------------+
| avg_weekly_time_in_bed | min_weekly_time_in_bed | max_weekly_time_in_bed |
+------------------------+------------------------+------------------------+
|              2009.2391 |                     61 |                   3747 |
+------------------------+------------------------+------------------------+
```
**Query 6: Descriptive Statistics for `total_weekly_minutes_asleep`**
```sql
SELECT 
    AVG(total_weekly_minutes_asleep) AS avg_weekly_minutes_asleep,
    MIN(total_weekly_minutes_asleep) AS min_weekly_minutes_asleep,
    MAX(total_weekly_minutes_asleep) AS max_weekly_minutes_asleep
FROM 
    final_weekly_summary;
```
**Output**:
```sql
+---------------------------+---------------------------+---------------------------+
| avg_weekly_minutes_asleep | min_weekly_minutes_asleep | max_weekly_minutes_asleep |
+---------------------------+---------------------------+---------------------------+
|                 1885.0652 |                        58 |                      3391 |
+---------------------------+---------------------------+---------------------------+
```
**Query 7: Descriptive Statistics for `average_weekly_heart_rate`**
```sql
SELECT 
    AVG(average_weekly_heart_rate) AS avg_weekly_heart_rate,
    MIN(average_weekly_heart_rate) AS min_weekly_heart_rate,
    MAX(average_weekly_heart_rate) AS max_weekly_heart_rate
FROM 
    final_weekly_summary;
```
**Output**:
```sql
+-----------------------+-----------------------+-----------------------+
| avg_weekly_heart_rate | min_weekly_heart_rate | max_weekly_heart_rate |
+-----------------------+-----------------------+-----------------------+
|     76.92056539784308 |                63.738 |                99.506 |
+-----------------------+-----------------------+-----------------------+
```

## 4.2 Comparing Weekly Activity with NHIS Averages:
1. For each user’s weekly activity data, we compared their vigorous and moderate activity minutes to NHIS averages. The percentage difference was calculated to highlight whether users met, exceeded, or fell short of NHIS recommendations.
2. The `pct_diff_vigorous_minutes` and `pct_diff_moderate_minutes` fields provide these percentage differences.
**Key SQL Queries:**
```sql
SELECT 
    Id,
    year,
    week,
    total_weekly_vigorous_minutes,
    nhis_vigorous_minutes_avg,
    ROUND((total_weekly_vigorous_minutes - nhis_vigorous_minutes_avg) / nhis_vigorous_minutes_avg * 100, 2) AS pct_diff_vigorous_minutes,
    total_weekly_moderate_minutes,
    nhis_moderate_minutes_avg,
    ROUND((total_weekly_moderate_minutes - nhis_moderate_minutes_avg) / nhis_moderate_minutes_avg * 100, 2) AS pct_diff_moderate_minutes
FROM final_weekly_summary;
```

**Output**:
```sql
+------------+------+------+-------------------------------+---------------------------+---------------------------+-------------------------------+---------------------------+---------------------------+
| Id         | year | week | total_weekly_vigorous_minutes | nhis_vigorous_minutes_avg | pct_diff_vigorous_minutes | total_weekly_moderate_minutes | nhis_moderate_minutes_avg | pct_diff_moderate_minutes |
+------------+------+------+-------------------------------+---------------------------+---------------------------+-------------------------------+---------------------------+---------------------------+
| 2026352035 | 2016 |   16 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 2026352035 | 2016 |   17 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 2026352035 | 2016 |   18 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 2026352035 | 2016 |   19 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 2347167796 | 2016 |   15 |                           132 |                   201.071 |                    -34.35 |                           244 |                   235.752 |                       3.5 |
| 2347167796 | 2016 |   16 |                            87 |                   201.071 |                    -56.73 |                           161 |                   235.752 |                    -31.71 |
| 2347167796 | 2016 |   17 |                            41 |                   201.071 |                    -79.61 |                            20 |                   235.752 |                    -91.52 |
| 4020332650 | 2016 |   15 |                           110 |                   201.071 |                    -45.29 |                            61 |                   235.752 |                    -74.13 |
| 4020332650 | 2016 |   18 |                            51 |                   201.071 |                    -74.64 |                            99 |                   235.752 |                    -58.01 |
| 4020332650 | 2016 |   19 |                             5 |                   201.071 |                    -97.51 |                            13 |                   235.752 |                    -94.49 |
| 4388161847 | 2016 |   15 |                            17 |                   201.071 |                    -91.55 |                            58 |                   235.752 |                     -75.4 |
| 4388161847 | 2016 |   16 |                           208 |                   201.071 |                      3.45 |                           108 |                   235.752 |                    -54.19 |
| 4388161847 | 2016 |   17 |                            96 |                   201.071 |                    -52.26 |                           178 |                   235.752 |                     -24.5 |
| 4388161847 | 2016 |   18 |                           263 |                   201.071 |                      30.8 |                           198 |                   235.752 |                    -16.01 |
| 4388161847 | 2016 |   19 |                           134 |                   201.071 |                    -33.36 |                            89 |                   235.752 |                    -62.25 |
| 4558609924 | 2016 |   16 |                            53 |                   201.071 |                    -73.64 |                            94 |                   235.752 |                    -60.13 |
| 4558609924 | 2016 |   17 |                            80 |                   201.071 |                    -60.21 |                           134 |                   235.752 |                    -43.16 |
| 4558609924 | 2016 |   18 |                            58 |                   201.071 |                    -71.15 |                            34 |                   235.752 |                    -85.58 |
| 4558609924 | 2016 |   19 |                            87 |                   201.071 |                    -56.73 |                           114 |                   235.752 |                    -51.64 |
| 5553957443 | 2016 |   15 |                           174 |                   201.071 |                    -13.46 |                           144 |                   235.752 |                    -38.92 |
| 5553957443 | 2016 |   16 |                           141 |                   201.071 |                    -29.88 |                            86 |                   235.752 |                    -63.52 |
| 5553957443 | 2016 |   17 |                           202 |                   201.071 |                      0.46 |                            73 |                   235.752 |                    -69.04 |
| 5553957443 | 2016 |   18 |                           114 |                   201.071 |                     -43.3 |                            76 |                   235.752 |                    -67.76 |
| 5553957443 | 2016 |   19 |                           131 |                   201.071 |                    -34.85 |                            76 |                   235.752 |                    -67.76 |
| 5577150313 | 2016 |   15 |                           637 |                   201.071 |                     216.8 |                           219 |                   235.752 |                     -7.11 |
| 5577150313 | 2016 |   16 |                           705 |                   201.071 |                    250.62 |                           236 |                   235.752 |                      0.11 |
| 5577150313 | 2016 |   17 |                           775 |                   201.071 |                    285.44 |                           283 |                   235.752 |                     20.04 |
| 5577150313 | 2016 |   18 |                           439 |                   201.071 |                    118.33 |                           146 |                   235.752 |                    -38.07 |
| 5577150313 | 2016 |   19 |                           183 |                   201.071 |                     -8.99 |                            63 |                   235.752 |                    -73.28 |
| 6117666160 | 2016 |   15 |                             7 |                   201.071 |                    -96.52 |                            21 |                   235.752 |                    -91.09 |
| 6117666160 | 2016 |   16 |                            37 |                   201.071 |                     -81.6 |                            29 |                   235.752 |                     -87.7 |
| 6117666160 | 2016 |   17 |                             0 |                   201.071 |                      -100 |                             7 |                   235.752 |                    -97.03 |
| 6117666160 | 2016 |   18 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 6117666160 | 2016 |   19 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 6775888955 | 2016 |   15 |                            67 |                   201.071 |                    -66.68 |                            60 |                   235.752 |                    -74.55 |
| 6962181067 | 2016 |   15 |                           148 |                   201.071 |                    -26.39 |                            86 |                   235.752 |                    -63.52 |
| 6962181067 | 2016 |   16 |                           242 |                   201.071 |                     20.36 |                           201 |                   235.752 |                    -14.74 |
| 6962181067 | 2016 |   17 |                           160 |                   201.071 |                    -20.43 |                           115 |                   235.752 |                    -51.22 |
| 6962181067 | 2016 |   18 |                            68 |                   201.071 |                    -66.18 |                           133 |                   235.752 |                    -43.58 |
| 6962181067 | 2016 |   19 |                           129 |                   201.071 |                    -35.84 |                            84 |                   235.752 |                    -64.37 |
| 7007744171 | 2016 |   15 |                           162 |                   201.071 |                    -19.43 |                            75 |                   235.752 |                    -68.19 |
| 7007744171 | 2016 |   18 |                           219 |                   201.071 |                      8.92 |                            81 |                   235.752 |                    -65.64 |
| 8792009665 | 2016 |   15 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
| 8792009665 | 2016 |   16 |                             9 |                   201.071 |                    -95.52 |                            75 |                   235.752 |                    -68.19 |
| 8792009665 | 2016 |   17 |                            19 |                   201.071 |                    -90.55 |                            42 |                   235.752 |                    -82.18 |
| 8792009665 | 2016 |   18 |                             0 |                   201.071 |                      -100 |                             0 |                   235.752 |                      -100 |
+------------+------+------+-------------------------------+---------------------------+---------------------------+-------------------------------+---------------------------+---------------------------+
```


## 4.3	Evaluating Sleep Duration:
1. Sleep is a crucial component of the analysis since **Bellabeat’s IVY+ tracker** focuses on supporting healthy sleep habits. By calculating the percentage difference in sleep duration from the **NHIS** benchmark, we highlighted whether users are meeting recommended sleep levels.
2. The `pct_diff_sleep_minute`s field in the table was essential for this analysis.
**Key SQL Queries:**
```sql
SELECT 
    Id,
    year,
    week,
    total_weekly_minutes_asleep,
    nhis_sleep_minutes_avg,
    ROUND((total_weekly_minutes_asleep - nhis_sleep_minutes_avg) / nhis_sleep_minutes_avg * 100, 2) AS pct_diff_sleep_minutes
FROM final_weekly_summary;
```
**Output**:
```sql
+------------+------+------+-----------------------------+------------------------+------------------------+
| Id         | year | week | total_weekly_minutes_asleep | nhis_sleep_minutes_avg | pct_diff_sleep_minutes |
+------------+------+------+-----------------------------+------------------------+------------------------+
| 2026352035 | 2016 |   16 |                        2915 |                418.835 |                 595.98 |
| 2026352035 | 2016 |   17 |                        3145 |                418.835 |                 650.89 |
| 2026352035 | 2016 |   18 |                        3079 |                418.835 |                 635.13 |
| 2026352035 | 2016 |   19 |                        2408 |                418.835 |                 474.93 |
| 2347167796 | 2016 |   15 |                        1364 |                418.835 |                 225.67 |
| 2347167796 | 2016 |   16 |                        2760 |                418.835 |                 558.97 |
| 2347167796 | 2016 |   17 |                        2578 |                418.835 |                 515.52 |
| 4020332650 | 2016 |   15 |                         578 |                418.835 |                     38 |
| 4020332650 | 2016 |   18 |                        1411 |                418.835 |                 236.89 |
| 4020332650 | 2016 |   19 |                         806 |                418.835 |                  92.44 |
| 4388161847 | 2016 |   15 |                         925 |                418.835 |                 120.85 |
| 4388161847 | 2016 |   16 |                        2470 |                418.835 |                 489.73 |
| 4388161847 | 2016 |   17 |                        2147 |                418.835 |                 412.61 |
| 4388161847 | 2016 |   18 |                        2248 |                418.835 |                 436.73 |
| 4388161847 | 2016 |   19 |                        1414 |                418.835 |                  237.6 |
| 4558609924 | 2016 |   16 |                         126 |                418.835 |                 -69.92 |
| 4558609924 | 2016 |   17 |                         274 |                418.835 |                 -34.58 |
| 4558609924 | 2016 |   18 |                         115 |                418.835 |                 -72.54 |
| 4558609924 | 2016 |   19 |                         123 |                418.835 |                 -70.63 |
| 5553957443 | 2016 |   15 |                        2281 |                418.835 |                 444.61 |
| 5553957443 | 2016 |   16 |                        3237 |                418.835 |                 672.86 |
| 5553957443 | 2016 |   17 |                        3391 |                418.835 |                 709.63 |
| 5553957443 | 2016 |   18 |                        3119 |                418.835 |                 644.68 |
| 5553957443 | 2016 |   19 |                        2340 |                418.835 |                 458.69 |
| 5577150313 | 2016 |   15 |                        2126 |                418.835 |                  407.6 |
| 5577150313 | 2016 |   16 |                        3108 |                418.835 |                 642.06 |
| 5577150313 | 2016 |   17 |                        2974 |                418.835 |                 610.06 |
| 5577150313 | 2016 |   18 |                        2089 |                418.835 |                 398.76 |
| 5577150313 | 2016 |   19 |                         935 |                418.835 |                 123.24 |
| 6117666160 | 2016 |   15 |                         380 |                418.835 |                  -9.27 |
| 6117666160 | 2016 |   16 |                        3248 |                418.835 |                 675.48 |
| 6117666160 | 2016 |   17 |                        1888 |                418.835 |                 350.77 |
| 6117666160 | 2016 |   18 |                        2055 |                418.835 |                 390.65 |
| 6117666160 | 2016 |   19 |                        1047 |                418.835 |                 149.98 |
| 6775888955 | 2016 |   15 |                        1049 |                418.835 |                 150.46 |
| 6962181067 | 2016 |   15 |                        2231 |                418.835 |                 432.67 |
| 6962181067 | 2016 |   16 |                        3167 |                418.835 |                 656.14 |
| 6962181067 | 2016 |   17 |                        3102 |                418.835 |                 640.63 |
| 6962181067 | 2016 |   18 |                        2921 |                418.835 |                 597.41 |
| 6962181067 | 2016 |   19 |                        2467 |                418.835 |                 489.01 |
| 7007744171 | 2016 |   15 |                          79 |                418.835 |                 -81.14 |
| 7007744171 | 2016 |   18 |                          58 |                418.835 |                 -86.15 |
| 8792009665 | 2016 |   15 |                        1838 |                418.835 |                 338.84 |
| 8792009665 | 2016 |   16 |                        1258 |                418.835 |                 200.36 |
| 8792009665 | 2016 |   17 |                        1566 |                418.835 |                 273.89 |
| 8792009665 | 2016 |   18 |                        1873 |                418.835 |                 347.19 |
+------------+------+------+-----------------------------+------------------------+------------------------+
```

## 4.4 Integration of WHO Baselines:
In addition to **NHIS** benchmarks, **WHO** recommendations were incorporated as static values (**150 minutes** of activity per week, **7-8 hours** of sleep per night) for further analysis in the next steps. This will serve as a baseline for assessing health patterns that **IVY+** could target for improvement.

## Key Findings
1.	Activity Levels:
- Many users fell short of the **NHIS**-recommended levels for both vigorous and moderate activity minutes. This suggests an opportunity for **Bellabeat** to promote increased physical activity through motivational features in **IVY+**.
- A subset of users, however, exceeded these benchmarks, indicating variability in activity levels. Tailored goals could encourage less active users to meet recommended levels without alienating those already achieving or surpassing them.

2.	Sleep Patterns:
- Sleep duration was often significantly higher than the NHIS benchmark, with some users exceeding recommended weekly sleep by a wide margin. This could reflect sedentary behavior rather than restorative sleep patterns, so further analysis may be necessary to differentiate quality sleep from merely spending time in bed.
- Some users fell below the recommended sleep levels, suggesting the need for sleep tracking and health prompts to improve rest.

# 1. SHARE PHASE - Visualizing and Analyzing Key Metrics
In the Share phase of our **Bellabeat** case study, we focused on visualizing key health metrics derived from the **Fitbit** dataset. These visualizations provide insights into users' activity levels, sleep patterns, and heart rate trends, which are crucial for understanding user behavior and informing product development strategies. Below is a detailed analysis of the four key visualizations created during this phase.

## Share Phase

## Share Phase

In the Share Phase, we used R to create visualizations that highlight key insights from the Bellabeat data. The visualizations include analyses of user activity, sleep patterns, and heart rate trends. These visualizations help to clearly communicate the findings and recommendations derived from the data.

For the complete R code used in the analysis and to view all visualizations, please refer to the [Bellabeat_Viz.Rmd file](./Bellabeat-Viz.html).

### Key Visualizations and Insights:
1. **Average Daily Steps per User**: A bar chart illustrating average daily steps per user, which helped identify general activity levels.
2. **Sleep Duration Comparison with NHIS Baseline**: A bar plot comparing users’ sleep duration to the NHIS-recommended baseline, revealing trends in sleep patterns.
3. **Heart Rate Analysis for Stress Indicators**: A line plot displaying weekly heart rate patterns to identify possible stress indicators.

These visualizations support Bellabeat's understanding of user wellness behaviors and provide insights for enhancing engagement and health outcomes in their product.


