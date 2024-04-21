# Marketing-Analysis-Using-SQL-Tableau
<img src = "https://uspto.report/TM/97150481/mark.png" height = "250" width = "900">

## Introduction 
This case study is part of the ```Google Data Analytics Professional Certificate Course```. 
In this project, I act as a data analyst at Bellabeat Marketing and Product departments

## About Bellabeat 
Bellabeat is **high-tech manufacturer** of health-focused products for women. It's a small company, but they have the potential to become a larger player in the gobal smart device market. As Bellabeat products are available through a growing number of online retailers as well as e-commerce channel on their website, they want to **understand better the behavior of their existing users** and use that insight to guide following marketing strategy.

## Method Approach
This project is sectioned into following six parts. 
1. Ask - formulating business question 
2. Prepare - choosing the suitable datasets 
3. Process - ensuring the quality of datasets 
4. Analyze - extracting inights from cleaned datasets 
5. Share - transforming it into interactive formats 
6. Act - suggesting actionable ideas that helps to solve business problem

## Phase 1: Ask 
#### **1.1 Business Task:** 
Identify some trends in how consumers use the Bellabeat devices and how these trends can help improve new opportunities growth for Bellabeat as well as marketing strategy.

## Phase 2: Prepare 

#### 2.1  Dataset Summary   
I select 6 datasets out of 18 datasets for this study. Each dataset represents different quantitative data recorded by Fitbit.    
<img src="https://imgur.com/ASFLrLz.png">   
<sub> The fitbit fitness tracker datasets are published by Möbius in Kaggle under the CC0: Public Domain Creative Common License.</sub>

#### 2.3 Dataset Limitations and Integrity
<sub> The collected data only has a 31-days time span (04-12-2016 to 05-12-2016) and only 33 people took part in the survey. 
The small sample size and short time span of collected data are insufficient to represent all FitBit users. In addition, collected datasets are lacking of demographic information (gender, location, age), so it could lead to bias in this study. </sub>

## Phase 3: Process    

In this project, I choose ```MySQL``` database to process and store datasets and ```Tableau``` as data visualization tool.
To begin, I load six datasets into MySQL database and perform data cleaning process.     
<img src="https://imgur.com/ASFLrLz.png">

#### 3.1 Importing Datasets

Due to large file size, I use ``` Load Data Infile``` command to import all datasets into databases.

``` sql 
/*dailyActivity_merged*/ 
CREATE TABLE dailyActivity_merged (
Id varchar(50), ActivityDate DATE, TotalSteps INT, TotalDistance DOUBLE, TrackerDistance DOUBLE, LoggedActivitiesDistance INT, 
VeryActiveDistance DOUBLE, ModeratelyActiveDistance DOUBLE, LightActiveDistance  DOUBLE, SedentaryActiveDistance INT,VeryActiveMinutes INT, 
FairlyActiveMinutes INT, LightlyActiveMinutes INT, SedentaryMinutes INT, Calories INT); 

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\bellabeat\\dailyActivity_merged.csv'
INTO TABLE dailyActivity_merged
FIELDS TERMINATED BY ','
ENCLOSED BY ""
LINES TERMINATED BY '\n'
IGNORE 1 ROWS; 

/*dailyCalories_merged*/ 
CREATE TABLE dailyCalories_merged(
Id VARCHAR(50), ActivityDay DATE, Calories INT,  time_and_date DATE);

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\bellabeat\\dailyCalories_merged.csv'
INTO TABLE dailyCalories_merged
FIELDS TERMINATED BY ','
ENCLOSED BY ""
LINES TERMINATED BY '\n'
IGNORE 1 ROWS; 

/*dailySteps_merged*/ 
CREATE TABLE dailySteps_merged (
Id VARCHAR(50), ActivityDay DATE, StepTotal INT);

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\bellabeat\\dailySteps_merged.csv'
INTO TABLE dailySteps_merged
FIELDS TERMINATED BY ','
ENCLOSED BY ""
LINES TERMINATED BY '\n'
IGNORE 1 ROWS; 

/*Hourlysteps_merged*/
CREATE TABLE hourlySteps_merged (
Id VARCHAR(50), ActivityHour VARHCAR(50), StepTotal INT); 

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\bellabeat\\hourlySteps_merged.csv'
INTO TABLE hourlySteps_merged
FIELDS TERMINATED BY ','
ENCLOSED BY ""
LINES TERMINATED BY '\n'
IGNORE 1 ROWS; 

/*SleepDay_merged*/ 
CREATE TABLE SleepDay_merged (
Id VARCHAR(50), SleepDay VARCHAR(50), TotalSleepRecords INT, TotalMinutesAsleep INT, TotalTimeInBed INT);

LOAD DATA INFILE ''C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\bellabeat\\sleepDay_merged.csv'
INTO TABLE SleepDay_merged
FIELDS TERMINATED BY ','
ENCLOSED BY ""
LINES TERMINATED BY '\n'
IGNORE 1 ROWS; 
```

#### 3.2 Converting DataType 

I find ```date format``` columns are unable to read my relational database, therefore, those date format in VARHCAR(50) must convert to default dateformat.    
<img src="https://imgur.com/Fudwb5C.png">     

``` sql 
-- Converting Data Type --
Add a new column and insert the converted old column value into new column.  

Alter table bellabeat.dailyactivity
add Activity_Date VARCHAR(50) ;
UPDATE bellabeat.dailyactivity
SET Activity_Date = cast(str_to_date(ActivityDate,"%m/%d/%Y")as date); 

Alter table bellabeat.dailyCalories
add Activity_Date VARCHAR(50) ;
UPDATE bellabeat.dailyCalories
SET Activity_Date = cast(str_to_date(ActivityDate,"%m/%d/%Y")as date); 

Alter table bellabeat.dailySteps
add Activity_Date VARCHAR(50) ;
UPDATE bellabeat.dailySteps
SET Activity_Date = cast(str_to_date(ActivityDate,"%m/%d/%Y")as date); 

<img src="https://imgur.com/undefined.png">

```sql 
Alter table bellabeat.hourlysteps_merged
ADD Activity_Hour VARCHAR(50) ;
UPDATE bellabeat.hourlysteps_merged
SET Activity_Hour = cast(str_to_date(ActivityHour,"%m/%e/%Y %r")as datetime); 

Alter table bellabeat.sleepday_merged
ADD SleepDatetime VARCHAR(50) ;
UPDATE bellabeat.sleepday_merged
SET SleepDatetime = cast(str_to_date(SleepDay,"%m/%e/%Y %r")as datetime); 
```

#### 3.3 Comparing Datasets 

I notice across these three datasets ,```dailyActivity_merged``` ```dailyCalories``` ```dailySteps``` , have shared identical column information (Id) among one another. To ensure the data are free from bias, I use ```inner join``` to cross check the datasets.      

```sql
SELECT
A.Id, A.ActivityDate, A.TotalSteps, A.TotalDistance, A.TrackerDistance,
A.LoggedActivitiesDistance, A.VeryActiveDistance,  A.ModeratelyActiveDistance,  
A.LightActiveDistance, A.SedentaryActiveDistance, A.VeryActiveMinutes, A.FairlyActiveMinutes, 
A.LightlyActiveMinutes, A.SedentaryMinutes, A.Calories
FROM bellabeat.dailyactivity_merged A 
INNER JOIN bellabeat.dailycalories_merged C
ON A.Id = C.Id and A.ActivityDate = C.ActivityDay and A.Calories = C.Calories
INNER JOIN bellabeat.dailysteps_merged S 
ON A.Id = S.Id  and A.ActivityDate = S.ActivityDay 
and A.TotalSteps = S.StepTotal; 
```      

After running the inner join query, it returns 940 rows, which is same number of rows across datasets.    
<img src="https://imgur.com/uhrA0Bc.png">

#### 3.4 Checking Start-End Date and Length of Id


```sql
-- Find the start and end date of each activity-- 
SELECT MIN(Activity_Date) as start_date, MAX(Activity_Date) as end_date
FROM bellabeat.dailyActivity_merged;
-- start date : 2016-04-12 and end date = 2016-05-12

SELECT MIN(Activity_Hour ) as start_date, MAX(Activity_Hour) as end_date
FROM bellabeat.hourlysteps_merged; 
-- start date : 4/12/2016 and end date = 5/9/2016-- 

SELECT MIN(SleepDatetime) as start_date, MAX(SleepDatetime) as end_date
FROM bellabeat.sleepday_merged;
-- start date : 4/12/2016 and end date = 5/9/2016

-- Check all ids have the same length-- 
SELECT Id
FROM bellabeat.dailyActivity_merged
WHERE Length(Id)> 10 OR Length(Id )< 10;
-- No values returned; all IDs in Daily Activity have 10 characters
SELECT Id
FROM bellabeat.sleepday_merged
WHERE Length(Id) > 10 OR Length(Id )<10;
-- No values returned; all IDs in Sleep Day have 10 characters
SELECT Id
FROM bellabeat.hourlysteps_merged
WHERE Length(Id)>10 OR Length(Id )<10;
-- No values returned; all IDs in Weight log have 10 characters
```

#### 3.5 Find Duplicate Rows 

```sql
SELECT Id, ActivityDate, Count(*) as num_of_ids
FROM bellabeat.dailyactivity_merged
GROUP BY Id, ActivityDate
Having num_of_ids >1 ;
-- no data to display / no duplicates in daily_activity

SELECT Id, Activity_Hour, Count(*) as num_of_ids
FROM bellabeat.hourlysteps_merged
GROUP BY Id, Activity_Hour
Having num_of_ids >1; 
-- no data to display / no duplicates in hourlysteps

SELECT Id, SleepDay, Count(*) as num_of_ids
FROM bellabeat.sleepday_merged
GROUP BY Id, SleepDay
Having num_of_ids >1;
-- There are 3 duplicate rows returned
``` 

#### 3.6 Find Unsuitable Data 

```sql 
-- check if total steps - 0 in daily activity table-- 
SELECT Id, count(*) as num_of_zero_steps
FROM bellabeat.dailyactivity_merged
WHERE TotalSteps
group by Id
order by num_of_zero_steps;

-- create new daily activity table -- 
CREATE TABLE bellabeat.dailyactivty_new 
AS SELECT * FROM bellabea.dailyacitivty_merged
WHERE TotalSteps != 0 ; 
```

#### 3.7 Find Null Data

```sql 
-- check for null data -- 
SELECT* FROM bellabeat.dailyactivity_merged 
WHERE Id is null; 
-- 0 datapoint are null -- 

SELECT* FROM bellabeat.hourlysteps
WHERE Id is null; 
-- No datapoint are null -- 

SELECT* FROM bellabeat.sleepday
WHERE Id is null; 
-- No datapoint are null-- 
```

## Phase 4: Analyze

Once turning datasets into usable format, I export it into a ```csv.file``` and then import into Tableau software to visualise data. The ***following diagrams*** are the trends that I've identified during data visualization process.

### 4.1 Overview of ```User Activity``` Class   
<img src="https://imgur.com/lO4ZLZD.png" height="450" width="600">

### 4.2 ```Daily Activity Behavior``` of a Smart Device User     
<img src="https://imgur.com/wZJ3Bsb.png" height="450" width="600">

### 4.3 ```Day of Week Activity Behavior``` of Smart Device Users     
<img src="https://imgur.com/HOJsZOH.png" height="450" width="600"> 

### 4.4 Number of ```Sleep Night``` being tracked in a month      
<img src = "https://imgur.com/t99ZzuO.png" height="450" width="600"> 

### 4.5 Correlation between ```calories and Level of active activity```     
<img src = "https://imgur.com/RRRb9ZM.png" height="450" width="600">

### 4.6 ```User Type Distribution ```    
<img src= "https://imgur.com/yFGIb5Z.png" height="450" width="600">

## 5. Share   
### ```Findings```
-	***User experience*** of bellabeat product is positive as 90% of existing users wear for more than 21 days in a month.
-	90% users ***wear for more than 21 days in a month.*** 
-	On average, ***each user spends half their day on sedentary activities*** and less than 2 hours altogether on very active and fairly active activities (1 hour 12 mins).
-	***Evening period, 5pm to 7pm, has the highest average steps count throughout a day*** followed by afternoon to noon period, 12pm to 2pm, which has the second highest count. 
-	***Users are less likely to wear during sleep***, where only 10 users, out of 24 users, have worn more than 15 nights. 
-	There is a ***positive correlation between activity level and calories burned.*** The more activity you are, the more calories you burn   

## 6. Act   
### ```Recommendations```     

The insufficient data has resulted less credibility in our finding. 
Based on the findings, I recommend adding notification system to the products, promoting point-award incentive system and marketing campaign for white-collar group. 

1. ```Add Notification System to products/apps ``` For users who have not reached the daily minimum step count or been on sedentary activities continually, products/apps will notify either through reminder notification or vibration from smart products. 

2. ``` Promote point-award incentive system ``` Based on our analysis, we found that 90% users wear fitbit regularly. Users who completing bellabeat exercise weekly challenge can redeem points on products/memberships. Those points can use on exchanging/discount on bellabeat new products. 

3. ``` Marketing Campaign__ pinpointing the perks of having Health Assitant and targeting white-collar worker```    
Based on our findings, we observed the users in our study, on average spend 12 hours on sedentary activity and they are most active in evening and afternoon. Plus, 90% users wear more than 21 days in a month. This group of people tend to lose track of time due to their job nature and less-physical consuming activity, so the marketing content could cater and tailor for them. 
