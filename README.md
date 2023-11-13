# Bellabeat Data Analysis Project
Bellabeat Fitness Tracking Data Project

### Project Overview

This data analysis project aims to provide fitness data insights for a fashionable health company that sells trackers designed and engineered for women, Bellabeat. By analysing various aspects of the daily use case of users, to find trends and make data-driven recommendations for new products or areas for targeted marketing. 

### Data Source
[Kaggle Source Data](https://www.kaggle.com/datasets/arashnic/fitbit)

Fitness Tracking Data used for this project includes:
"dailyActivity_merged.csv" This file contains detailed information regarding the amount of activity each user did in a day. This can be from nothing, sedentary, to high activity. 

"sleepDay_merged.csv" This file contains the sleep log for each user, including the number of times the device registered sleep, total minutes asleep, and total time in bed.

"weightLogInfo_merged.csv" This file contains the weight log for the participants, including their weight in both kilograms and pounds, BMI, and whether the reporting was manual. 

### Tools

- Google Sheets - Used to link to bigquery 
- [Google Bigquery](https://cloud.google.com/bigquery) - Data Cleaning & Analysis
- [Tableau](https://public.tableau.com/app/discover) - Visualisations

### Data Cleaning & Preparation

1. Downloading data from kaggle
2. Load csv into google sheets for inspection
    - Each individual spreashsheet was relatively clean without missing values
3. Data is loaded into Bigquery with connected sheets

### Exploratory Data Analysis

To get a better understanding of the participants and how many took data for each piece of data

_____

<details>
    <summary> SQL Query Finding the Quantity of Participants for Each Data Set </summary>
    
```sql
SELECT 'Activity Log' AS LogType,
COUNT(DISTINCT Id) AS Participants
FROM `project-1-396413.Fitbit.daily_activity`

UNION ALL 

SELECT 'Calorie Log' AS LogType,
COUNT(DISTINCT Id) AS Participants
FROM `project-1-396413.Fitbit.daily_calories`

UNION ALL 

SELECT 'Daily Steps Log' AS LogType,
COUNT(DISTINCT Id) AS Participants
FROM `project-1-396413.Fitbit.daily_steps`

UNION ALL 

SELECT 'Sleep Log' AS LogType,
COUNT(DISTINCT Id) AS Participants
FROM `project-1-396413.Fitbit.sleep_day`

UNION ALL 

SELECT 'Weight Log' AS LogType,
COUNT(DISTINCT Id) AS Participants
FROM `project-1-396413.Fitbit.weight_log`
```
</details>

Output:

|LogType           |Participants|
|---------------|------------|
|Weight Log     |8           |
|Daily Steps Log|33          |
|Sleep Log      |24          |
|Calorie Log    |33          |
|Activity Log   |33          |


This gives us an idea of how many participants are in each data set. 

_____

To get a better understanding of the amount of individual days logged by each unique Id in each data set allows us to know how often the user's data was logged in the 2 months data was collected. 

We see that there are 33 participants in the daily steps, activity, and calorie logs, 24 in the sleep log but only 8 in the weight log. With only 8 participants it makes it really difficult to make recommendations with so few data points. 

To further ensure the data is adequate we can check to see how often each user logged data for each dataset.

<details> 
    <summary>SQL Query Logged Data by User </summary>

- The query first declares that unqId (unique Id) is in a string format
- Next create a table if one doesn't exist of a combination of all days logged by each user across all data sets except the weight log data, as it lacks participants, counting the unique dates of logged data grouped by the participants' Id. This creates long data

Output LIMIT 5
|Id        |type     |days_logged|
|----------|---------|-----------|
|2320127002|Sleep Log|1          |
|7007744171|Sleep Log|2          |
|8053475328|Sleep Log|3          |
|6775888955|Sleep Log|3          |
|1844505072|Sleep Log|3          |


```sql
DECLARE unqId STRING;

CREATE TABLE IF NOT EXISTS `project-1-396413.Fitbit.days_logged` AS (

SELECT Id, 'Activity Log' AS type, 
COUNT(DISTINCT ActivityDate) AS days_logged
FROM `project-1-396413.Fitbit.daily_activity`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Calorie Log' AS type,
COUNT(DISTINCT ActivityDay) AS days_logged
FROM `project-1-396413.Fitbit.daily_calories`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Daily Steps Log' AS type,
COUNT(DISTINCT ActivityDay) AS days_logged
FROM `project-1-396413.Fitbit.daily_steps`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Sleep Log' AS type,
COUNT(DISTINCT SleepDay) AS days_logged
FROM `project-1-396413.Fitbit.sleep_day`

WHERE Id IS NOT NULL
GROUP BY Id
);

EXECUTE IMMEDIATE """
  SELECT 
STRING_AGG(CONCAT("'",CAST(Id AS STRING),"'")) 

FROM (
  SELECT
  DISTINCT Id
  FROM `project-1-396413.Fitbit.days_logged`
) """ INTO unqId;

EXECUTE IMMEDIATE FORMAT("""
SELECT *
FROM `project-1-396413.Fitbit.days_logged`
PIVOT(SUM(days_logged) FOR CAST(Id AS STRING) IN(%s))""", unqId);
```
</details>

Output
|type           |2320127002|7007744171|8053475328|6775888955|1844505072|1644430081|4057192912|4558609924|1927972279|4020332650|2347167796|8792009665|6117666160|8253242879|3372868164|4388161847|7086361926|1503960366|4319703577|5577150313|4702921684|4445114986|2026352035|3977333714|6290855005|8877689391|5553957443|2873212765|8378563200|8583815059|2022484408|6962181067|1624580081|
|---------------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|----------|
|Sleep Log      |1         |2         |3         |3         |3         |4         |          |5         |5         |8         |15        |15        |18        |          |          |23        |24        |25        |26        |26        |27        |28        |28        |28        |          |          |31        |          |31        |          |          |31        |          |
|Activity Log   |31        |26        |31        |26        |31        |30        |4         |31        |31        |31        |18        |29        |28        |19        |20        |31        |31        |31        |31        |30        |31        |31        |31        |30        |29        |31        |31        |31        |31        |31        |31        |31        |31        |
|Daily Steps Log|31        |26        |31        |26        |31        |30        |4         |31        |31        |31        |18        |29        |28        |19        |20        |31        |31        |31        |31        |30        |31        |31        |31        |30        |29        |31        |31        |31        |31        |31        |31        |31        |31        |
|Calorie Log    |31        |26        |31        |26        |31        |30        |4         |31        |31        |31        |18        |29        |28        |19        |20        |31        |31        |31        |31        |30        |31        |31        |31        |30        |29        |31        |31        |31        |31        |31        |31        |31        |31        |

This gives us a better understanding of how the data was logged by each user. We can see the users who didn't log data where the data is empty.

<details> 
    <summary>SQL Query Logs by Participant and Data Set</summary>
Long data of each user was created using the create table if not exists query, this was then pivoted using dynamic iteration through declared unqId (unique Id) at the beginning of the query. To fix any data type issues CAST function is used to convert Id from INT64 to STR.

``` sql
DECLARE unqId STRING;

CREATE TABLE IF NOT EXISTS `project-1-396413.Fitbit.days_logged` AS (

SELECT Id, 'Activity Log' AS type, 
COUNT(DISTINCT ActivityDate) AS days_logged
FROM `project-1-396413.Fitbit.daily_activity`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Calorie Log' AS type,
COUNT(DISTINCT ActivityDay) AS days_logged
FROM `project-1-396413.Fitbit.daily_calories`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Daily Steps Log' AS type,
COUNT(DISTINCT ActivityDay) AS days_logged
FROM `project-1-396413.Fitbit.daily_steps`

WHERE Id IS NOT NULL
GROUP BY Id

UNION ALL 

SELECT Id, 'Sleep Log' AS type,
COUNT(DISTINCT SleepDay) AS days_logged
FROM `project-1-396413.Fitbit.sleep_day`

WHERE Id IS NOT NULL
GROUP BY Id
);

EXECUTE IMMEDIATE """
  SELECT 
STRING_AGG(CONCAT("'",CAST(Id AS STRING),"'")) 

FROM (
  SELECT
  DISTINCT Id
  FROM `project-1-396413.Fitbit.days_logged`
) """ INTO unqId;

EXECUTE IMMEDIATE FORMAT("""
SELECT *
FROM `project-1-396413.Fitbit.days_logged`
PIVOT(SUM(days_logged) FOR CAST(Id AS STRING) IN(%s))""", unqId);
```
</details>

To get a better idea of what's happening, statistical data is needed to see where the averages and 1st/3rd quartile data are sitting. 

<details> 
    <summary> SQL Query for Stats Summary </summary>
    <details>
    <summary>Stats for Activity Log</summary>

``` sql
SELECT 1 AS list, 'Min' AS stat,
  MIN(TotalSteps) AS total_steps,
  MIN(TotalDistance) AS total_distance,
  MIN(SedentaryMinutes) AS sedentary_minutes,
  MIN(VeryActiveMinutes) AS very_active_minutes,
  MIN(FairlyActiveMinutes) AS fairly_active_minutes,
  MIN(LightlyActiveMinutes) AS lightly_active_minutes,
  MIN(Calories) AS calories_da

FROM `project-1-396413.Fitbit.daily_activity` 

UNION ALL

SELECT 2 AS list,'1st Quarter' AS stat, 
  AVG(total_steps), 
  AVG(total_distance), 
  AVG(sedentary_minutes), 
  AVG(very_active_minutes), 
  AVG(fairly_active_minutes), 
  AVG(lightly_active_minutes), 
  AVG(calories)
 
FROM ( 
  SELECT 
  ROUND(PERCENTILE_CONT(TotalSteps, 0.25) OVER(),2) AS total_steps,
  ROUND(PERCENTILE_CONT(TotalDistance, 0.25) OVER(),2) AS total_distance,
  ROUND(PERCENTILE_CONT(SedentaryMinutes, 0.25) OVER(),2) AS sedentary_minutes,
  ROUND(PERCENTILE_CONT(VeryActiveMinutes, 0.25) OVER(),2) AS very_active_minutes,
  ROUND(PERCENTILE_CONT(FairlyActiveMinutes, 0.25) OVER(),2) AS fairly_active_minutes,
  ROUND(PERCENTILE_CONT(LightlyActiveMinutes, 0.25) OVER(),2) AS lightly_active_minutes,
  ROUND(PERCENTILE_CONT(Calories, 0.25) OVER(),2) AS calories

FROM `project-1-396413.Fitbit.daily_activity`
)


UNION ALL

SELECT 3 AS list, 'Average' AS stat,
  ROUND(AVG(TotalSteps),2) AS total_steps,
  ROUND(AVG(TotalDistance),2) AS total_distance,
  ROUND(AVG(SedentaryMinutes),2) AS sedentary_minutes,
  ROUND(AVG(VeryActiveMinutes),2) AS very_active_minutes,
  ROUND(AVG(FairlyActiveMinutes),2) AS fairly_active_minutes,
  ROUND(AVG(LightlyActiveMinutes),2) AS lightly_active_minutes,
  ROUND(AVG(Calories),2) AS calories_da

FROM `project-1-396413.Fitbit.daily_activity` 

UNION ALL

SELECT 4 AS list,'3rd Quarter' AS stat, 
  AVG(total_steps), 
  AVG(total_distance), 
  AVG(sedentary_minutes), 
  AVG(very_active_minutes), 
  AVG(fairly_active_minutes), 
  AVG(lightly_active_minutes), 
  AVG(calories)
 
FROM ( 
  SELECT 
  ROUND(PERCENTILE_CONT(TotalSteps, 0.75) OVER(),2) AS total_steps,
  ROUND(PERCENTILE_CONT(TotalDistance, 0.75) OVER(),2) AS total_distance,
  ROUND(PERCENTILE_CONT(SedentaryMinutes, 0.75) OVER(),2) AS sedentary_minutes,
  ROUND(PERCENTILE_CONT(VeryActiveMinutes, 0.75) OVER(),2) AS very_active_minutes,
  ROUND(PERCENTILE_CONT(FairlyActiveMinutes, 0.75) OVER(),2) AS fairly_active_minutes,
  ROUND(PERCENTILE_CONT(LightlyActiveMinutes, 0.75) OVER(),2) AS lightly_active_minutes,
  ROUND(PERCENTILE_CONT(Calories, 0.75) OVER(),2) AS calories

FROM `project-1-396413.Fitbit.daily_activity`
)

UNION ALL

SELECT 5 AS list, 'Max' AS stat,
  ROUND(MAX(TotalSteps),2) AS total_steps,
  ROUND(MAX(TotalDistance),2) AS total_distance,
  ROUND(MAX(SedentaryMinutes),2) AS sedentary_minutes,
  ROUND(MAX(VeryActiveMinutes),2) AS very_active_minutes,
  ROUND(MAX(FairlyActiveMinutes),2) AS fairly_active_minutes,
  ROUND(MAX(LightlyActiveMinutes),2) AS lightly_active_minutes,
  ROUND(MAX(Calories),2) AS calories_da

FROM `project-1-396413.Fitbit.daily_activity`

ORDER BY 1;
```
</details>

<details>
    
<summary>Stats for Sleep Log</summary>

``` sql
SELECT 1 AS list, 'Min' AS stat,
  MIN(TotalSleepRecords) AS sleep_records,
  MIN(TotalMinutesAsleep) AS minutes_asleep,
  MIN(TotalTimeInBed) AS time_in_bed,

FROM `project-1-396413.Fitbit.sleep_day` 

UNION ALL

SELECT 2 AS list,'1st Quarter' AS stat, 
  AVG(sleep_records), 
  AVG(minutes_asleep), 
  AVG(time_in_bed), 
 
FROM ( 
  SELECT 
  ROUND(PERCENTILE_CONT(TotalSleepRecords, 0.25) OVER(),2) AS sleep_records,
  ROUND(PERCENTILE_CONT(TotalMinutesAsleep, 0.25) OVER(),2) AS minutes_asleep,
  ROUND(PERCENTILE_CONT(TotalTimeInBed, 0.25) OVER(),2) AS time_in_bed,


FROM `project-1-396413.Fitbit.sleep_day`
)


UNION ALL

SELECT 3 AS list, 'Average' AS stat,
  ROUND(AVG(TotalSleepRecords),2) AS sleep_records,
  ROUND(AVG(TotalMinutesAsleep),2) AS minutes_asleep,
  ROUND(AVG(TotalTimeInBed),2) AS time_in_bed,

FROM `project-1-396413.Fitbit.sleep_day` 

UNION ALL

SELECT 4 AS list,'3rd Quarter' AS stat, 
  AVG(sleep_records), 
  AVG(minutes_asleep), 
  AVG(time_in_bed), 
 
FROM ( 
  SELECT 
  ROUND(PERCENTILE_CONT(TotalSleepRecords, 0.75) OVER(),2) AS sleep_records,
  ROUND(PERCENTILE_CONT(TotalMinutesAsleep, 0.75) OVER(),2) AS minutes_asleep,
  ROUND(PERCENTILE_CONT(TotalTimeInBed, 0.75) OVER(),2) AS time_in_bed,

FROM `project-1-396413.Fitbit.sleep_day`
)

UNION ALL

SELECT 5 AS list, 'Max' AS stat,
  ROUND(MAX(TotalSleepRecords),2) AS sleep_records,
  ROUND(MAX(TotalMinutesAsleep),2) AS minutes_asleep,
  ROUND(MAX(TotalTimeInBed),2) AS time_in_bed,

FROM `project-1-396413.Fitbit.sleep_day`

ORDER BY 1;
```
</details>

</details>
Output:

|list           |stat|total_steps|total_distance|sedentary_minutes|very_active_minutes|fairly_active_minutes|lightly_active_minutes|calories_da|
|---------------|----|-----------|--------------|-----------------|-------------------|---------------------|----------------------|-----------|
|1              |Min |0.0        |0.0           |0.0              |0.0                |0.0                  |0.0                   |0.0        |
|2              |1st Quarter|3789.75    |2.62          |729.75           |0.0                |0.0                  |127.0                 |1828.5     |
|3              |Average|7637.91    |5.49          |991.21           |21.16              |13.56                |192.81                |2303.61    |
|4              |3rd Quarter|10727.0    |7.71          |1229.5           |32.0               |19.0                 |264.0                 |2793.25    |
|5              |Max |36019.0    |28.03         |1440.0           |210.0              |143.0                |518.0                 |4900.0     |

|list           |stat|sleep_records|minutes_asleep|time_in_bed|
|---------------|----|-------------|--------------|-----------|
|1              |Min |1.0          |58.0          |61.0       |
|2              |1st Quarter|1.0          |361.0         |403.0      |
|3              |Average|1.12         |419.47        |458.64     |
|4              |3rd Quarter|1.0          |490.0         |526.0      |
|5              |Max |3.0          |796.0         |961.0      |

From this summary, we can ascertain the following:
- Among the participants, the average sedentary time is extremely high at 991 minutes (16 hours) daily
- The average participant is lightly active as the time in light activity is the highest

- Collating information from a [The Lancet](https://www.thelancet.com/journals/lanpub/article/PIIS2468-2667(21)00302-9/fulltext) study that sampled from over 47000 adults and 3000 deaths to provide evidence based guidelines on the daily amount of steps an adult should take. This includes potential health risks if not taken and The [National Insitutes Of Health's](https://www.nih.gov/news-events/nih-research-matters/number-steps-day-more-important-step-intensity) recommendation, benefits can be found at 8000 steps having 50% lower risk of dying as compared to those who averaged 4000 steps a day, and those with above 12,000 steps had a 65% lower risk of dying compared to averaging 4000 steps.
    - The participants are averaging about 7637 steps per day which is not significantly lower than the recommendation

- The average sleep is about 7 hours daily with total time in bet at around 7 hours and 38 minutes

- Lastly, that even when users did log their sleep data, it was an average of 17 days with only 3 users recording for a full 31 days.

 Pulling Directly from the table we created
 ``` sql
SELECT type,
ROUND(AVG(days_logged)) AS average_days_logged,
FROM `project-1-396413.Fitbit.days_logged`
GROUP BY type
```
|type|average_days_logged|
|----|-------------------|
|Sleep Log|17.0               |
|Activity Log| 28.0              |
|Daily Step Log| 28.0              |
|Calorie Log| 28.0              |

With this, it is clear that the sleep data is missing a lot of data points compared to the other datasets even with 24 participants logging data.

### Merging Data

``` sql
SELECT 
  daily_activity.Id,
  CAST(daily_activity.ActivityDate AS DATE) AS date,
  ROUND(weight_log.WeightKg, 2) AS weight_kg_rounded,
  sleep_day.TotalMinutesAsleep AS sleep_minutes,
  sleep_day.TotalTimeInBed AS total_time_in_bed,
  sleep_day.TotalTimeInBed - sleep_day.TotalMinutesAsleep AS time_in_bed_not_sleeping,
  daily_activity.TotalSteps,
  ROUND(daily_activity.TotalDistance, 2) AS total_distance_rounded_km,
  daily_activity.Calories,
  daily_activity.VeryActiveMinutes,
  daily_activity.FairlyActiveMinutes,
  daily_activity.LightlyActiveMinutes,
  daily_activity.SedentaryMinutes,
  
  /* Below is a CASE query for activity levels based on information from NIH and Study*/
  CASE
    WHEN daily_activity.TotalSteps > 12000 THEN 'Extremely High Activity'
    WHEN daily_activity.TotalSteps > 9000 AND daily_activity.TotalSteps <= 12000 THEN 'High Activity'
    WHEN daily_activity.TotalSteps > 7000 AND daily_activity.TotalSteps <= 9000 THEN 'Ideal Activity'
    WHEN daily_activity.TotalSteps > 5000 AND daily_activity.TotalSteps <= 7000 THEN 'Moderate Activity'
    WHEN daily_activity.TotalSteps > 3000 AND daily_activity.TotalSteps <= 5000 THEN 'Low Activity'
    WHEN daily_activity.TotalSteps > 1000 AND daily_activity.TotalSteps <= 3000 THEN 'Extremely Low Activity'
  ELSE "Sedentary"
  END AS activity_level_steps

FROM `project-1-396413.Fitbit.daily_activity` AS daily_activity

LEFT JOIN `project-1-396413.Fitbit.sleep_day` AS sleep_day ON
  daily_activity.Id = sleep_day.Id AND daily_activity.ActivityDate = sleep_day.SleepDay

LEFT JOIN `project-1-396413.Fitbit.weight_log` AS weight_log ON
  daily_activity.Id = weight_log.Id AND daily_activity.ActivityDate = weight_log.Date
```
Merging data using LEFT JOIN on the 'activity log' as the activity log is the most populated table that leaves null data from the Sleep and Weight Log where data was not collected.

### Visualisation

[Tableau Public Link](null) <--- add link
Click the above to view the visualisations in full

Note: 
- Using "sleepDay_merged.csv" for sleep data and the calendar viz
- Using the "FitbitDataMerged.csv" for Sleep vs Total Time in Bed and Total Steps vs Calories Vizzes


![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/0d72fa5a41940a8dd509ba1d3268f23d332104dd/TotalSteps_vs_Calories.png?raw=true)

There is a positive correlation between the number of steps taken and the number of calories burned which makes sense. The more activity in this case walking, or steps taken the more calories burned. This indicates that the data doesn't have any outliers.

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/BellabeatProjectFiles/TotalSleep_vs_TotalTimeInBed.pngraw=true)

With regard to sleep, there is a linear correlation between the amount of time in bed and the amount of sleep one has. This suggests that if users wish to have more sleep, a notification to get in bed earlier could be helpful. 

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/BellabeatProjectFiles/Sleep_vs_Sedentary_Time.png?raw=true)

The graph above shows a trend that users who have more sedentary minutes in a day are more likely to have less sleep.

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/0d72fa5a41940a8dd509ba1d3268f23d332104dd/User%20Sleep%20Log%20Days.png?raw=true)


The graphic above compares the days users logged their sleep.

As noted earlier, even with 24 participants, sleep was logged very occasionally. This can suggest that participants aren't wearing their devices to sleep, the device requires charging as it was used throughout the day, or the participants didn't consider logging their sleep data.

### Recommendations for Bellabeat

disclaimer/limitations:
- As the data is from 2016, user usage may have changed not to mention the capability of devices in terms of holding charge and collecting data.
- With only 33 participants, data can be biased towards this specific group of undisclosed participants. No user information was included, thus the following recommendation assumes that participants are equal in numbers of males and females.
- This data exploration gives an idea of how users of a competing fitness tracker use their devices. With that, the recommendations will also assume that Bellabeat products are identical to the Fitbit devices or are competitive in features.

##Target Audience

Bellabeat is a brand that develops wearables and accompanying products that monitor biometric and lifestyle data to help women better understand how their bodies work and make healthier choices. With a mission to empower women to reconnect with themselves, unleash their inner strengths, and be what they were meant to be.

As such the main target audience will be women who are less active (seen in the sedentary time) BUT want to improve their health, and do exercise but lightly to stay active.

Although these women try to stay active, they may need a push in the right direction. 

These can include:
1. Increasing the average number of daily steps to that of at least 8000, with more being better.
    - With 8000 steps daily, lowering the risk of mortality (by any cause) by 50% 
2. Further, assist users in understanding the importance of sleep and help set sleep targets with notification reminders.
    - From the available sleep logs, the more time users spend in bed the more they slept
          - with the assumption that users who choose to wind down earlier are able to get more sleep
3. Bellebeat can increase the users' awareness of exercise and amount of sleep or sleep quality by suggesting users reduce sedentary time.
    - Keeping in mind, however, there is a possibility that the device did not register the sleep and instead recorded it as sedentary time.

Outside of app-based changes, possible hardware recommendations are that Bellabeat:
1. Increases the comfort so that users wear their devices to sleep
2. Reduces the time it takes to charge the device
3. Have a device that is able to run for multiple days

These hardware change recommendations will need to work in conjunction with the software suggestion above, such as informing users about collecting sleep data and how they can use that to assist in their sleep habits.


Further analysis of specific data is required to have a full conclusion, as such using Bellabeat's own consented user data to corroborate the findings above will be the best way to correct for the limitation of a small sample size in the data from Fitbit.

Thank you for reading my case study.


