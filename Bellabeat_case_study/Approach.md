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

## 2.1 Processing and cleaning `daily_activity` table:
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

## 2.2 Processing and cleaning `heartrate_seconds` table:
After importing the data, I conducted multiple cleaning and validation steps to ensure the integrity of the heart rate data.

### 2.2.1 Key SQL queries used for cleaning:
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
## 2.3 Processing and cleaning `sleep_day` table:

### 2.3.1 Key SQL queries used for cleaning:
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


## 2.4 Processing and cleaning `hourly_steps` and `hourly_calories` tables:
### 2.4.1 Key SQL queries used for cleaning:
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
## 2.5 Processing and cleaning `minute_sleep` table:
The following steps ensured data quality and integrity within the minute_sleep table:
### 2.5.1 Key SQL queries used for cleaning:
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
4. **Sample Output Verification**: The following random samples verify that the data in the ` minute_sleep` table is correctly formatted and fall within the specified timeframe.
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

## 2.6 Processing and cleaning `minute_sleep` table:
### 2.6.1 Key SQL queries used for cleaning:
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
