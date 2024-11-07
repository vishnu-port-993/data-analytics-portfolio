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
1.1: Create table and import data set to `daily_activity` :
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
-- Import dailyActivity_merged.csv for timeframe, but for the time frame 4.12.16-5.12.16 into the same table "daily_activity" which will merge both the csv files into one table.
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
Ensuring that all records for this import fall within the overall observation time frame (2016-03-12 to 2016-05-12)
DELETE FROM daily_activity
WHERE ActivityDate NOT BETWEEN '2016-03-12' AND '2016-05-12';
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


