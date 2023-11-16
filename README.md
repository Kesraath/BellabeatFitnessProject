# Bellabeat Data Analysis Project
Bellabeat Fitness Tracking Data Project

### Project Overview

This data analysis project aims to provide fitness data insights for a fashionable health company that sells trackers designed and engineered for women, Bellabeat. By analysing various aspects of the daily use case of fitness trackers by users, to find trends and make data-driven recommendations for new products or areas for targeted marketing. 

### Data Source
[Kaggle Source Data](https://www.kaggle.com/datasets/arashnic/fitbit)
The data sourced is personal tracker data, including minute level heart rate, physical activity and sleep monitoring from thirty consented Fitbit Users.
The data were submitted by respondents through a distributed survey via Amazon Mechanical Turk. 
The data is organised always by ID in Column A and timestamp in Column B with corresponding data in the following columns. 

As this is user-submitted data from data that is collected via a smartwatch, it is difficult to fabricate the minute-level data that the trackers record. However, the method of collecting data has a monetary gain for those who opt to share their data, as such this is something to be considered when looking at the data. Further, the data is from a competing fitness tracking device, so data directly reflect the users currently using Bellabeat products.

The Data was sourced between 12 March 2016 and 12 May 2016

Data that will be used for the analysis:

Fitness Tracking Data used for this project includes:
- The Name of each file was altered when entered into Bigquery Dataset for clarity. 

"dailyActivity_merged.csv" contains detailed information regarding the amount of activity each user did in a day. This can be from nothing, sedentary, to high activity. 

"sleepDay_merged.csv" contains the sleep log for each user, including the number of times the device registered sleep, total minutes asleep, and total time in bed.

"weightLogInfo_merged.csv" contains the weight log for the participants, including their weight in both kilograms and pounds, BMI, and whether the reporting was manual. 

"dailyCalories_merged.csv" contains the day data was logged and the calories burned by a user (data also found in "dailyActicity_merged.csv")

"dailySteps_merged.csv" contains the daily amount of steps and the day the activity was logged by a user (data also found in "dailyActicity_merged.csv")

All files used and displayed can be found here: [Project Files](https://github.com/Kesraath/BellabeatFitnessProject/tree/BellabeatProjectFiles)
### Tools

- Google Sheets - Used to link to bigquery 
- [Google Bigquery](https://cloud.google.com/bigquery) - Data Cleaning & Analysis
- [Tableau](https://public.tableau.com/app/discover) - Visualisations

I chose these tools as they were easy to integrate with one another. Loading the CSV files to Google Sheets via upload and then loading them into BigQuery using the connected sheets tool. In a scenario where changes or additional data were to be added to the spreadsheet, the BigQuery-connected file can be updated and be working on the latest updated spreadsheet. After which, generating/ saving the query output as a table in the BigQuery Dataset or new dataset for queried outputs, Tableau Desktop can be used to link directly to the output file from BigQeury using the connect to server method and logging into the Google account. This means that any changes anywhere in the pipeline can be seen quickly further down the chain. 

### Data Cleaning & Preparation

1. Downloading data from kaggle
2. Load csv into google sheets for inspection
    - Each individual spreashsheet was relatively clean without missing values
3. Data is loaded into Bigquery with connected sheets

After scanning through the data in Google Sheets, there seemed to be no outliers or errors in the input of the data in the individual datasets. 
Using filter by and count functions to check the number of logs, find null data, and check for IDs that don't have the exact number of characters.

However, I did notice that some data sets had more logged data than others. From this, we can assume that not all participants recorded their data in every category. 

### Exploratory Data Analysis

To get a better understanding of the participants and how many logged data for each data set

I used the following query
_____

<details>
    <summary> SQL Query Finding the Quantity of Participants for Each Data Set </summary>

It creates a LogType for the name of the data set as column A and counts all distinct participant's ID. 
This is then joined with the UNION ALL function as the data will be all joined in the same format as the first output for all the following datasets.

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


This gives us an idea of how many participants are in each data set. It shows clearly that only 8 users participated in the weight log. 
Going back to look at the data set it shows that users had to manually input their weight and after some research, the Fitbit watches only track data if the user uses a certified scale or links their data to other software i.e. Apple health where the data will sync, but even this data will be shown as manually entered. 

_____

To get a better understanding of the amount of individual days logged by each unique Id in each data set allows us to know how often the user's data was logged in the 2 months data was collected. 

We see that there are 33 participants in the daily steps, activity, and calorie logs, 24 in the sleep log but only 8 in the weight log. With only 8 participants it makes it really difficult to make recommendations with so few data points. 

To further ensure the data is adequate we can check to see how often each user logged data for each dataset.

<details> 
    <summary>SQL Query Logged Data by User </summary>

- The query first declares that unqId (unique Id) is in a string format
- Next create a table if one doesn't exist of a combination of all days logged by each user across all data sets except the weight log data, as it lacks participants, counting the unique dates of logged data grouped by the participants' Id. This creates long data as seen below.

Output LIMIT 5
|Id        |type     |days_logged|
|----------|---------|-----------|
|2320127002|Sleep Log|1          |
|7007744171|Sleep Log|2          |
|8053475328|Sleep Log|3          |
|6775888955|Sleep Log|3          |
|1844505072|Sleep Log|3          |

- After this pivot, the created table with column names iterated through the 'unqId' assigned string this was then pivoted with the column names using a dynamic iteration process declared unqId (unique Id) at the beginning of the query with the rows being the days logged with column A being the type of Activity. To fix any data type issues CAST function is used to convert Id from INT64 to STR. 

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
We see that several although they did participate in logging their sleep data did not log every single day. With 6 participants logging less than 5 days. This will be kept in mind going forward. 

__________

To get a general understanding of how users sit in terms of activity and sleep levels, statistical data is needed to see where the averages and 1st/3rd quartile data are sitting. 

<details> 
    <summary> SQL Query for Stats Summary </summary>

   
The query below is a bit of a complicated mess, it calculates the Minimum, 1st Quartile, Average, 3rd Quartile, and Maximum of the logged data in each section of the Daily Activity Log.
This likely could have been done with the following code in R

``` R
x <- read.csv("dailyActivity_merged.csv")
summary(x)
```
However, this is the SQL code below to find stats and put them in a readable table format, using the same method as earlier using UNION ALL and labeling each corresponding 'stat' row with a string.
    
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
- The average participant is lightly active as the time in light activity is the highest amongst the activity categories (excluding sedentary)

- Collating information from a [The Lancet](https://www.thelancet.com/journals/lanpub/article/PIIS2468-2667(21)00302-9/fulltext) study that sampled from over 47000 adults and 3000 deaths to provide evidence-based guidelines on the daily amount of steps an adult should take. This includes potential health risks if not taken and The [National Insitutes Of Health's](https://www.nih.gov/news-events/nih-research-matters/number-steps-day-more-important-step-intensity) recommendation, benefits can be found at 8000 steps having 50% lower risk of dying as compared to those who averaged 4000 steps a day, and those with above 12,000 steps had a 65% lower risk of dying compared to averaging 4000 steps.
    - The participants are averaging about 7637 steps per day which is not significantly lower than the recommendation

- The average sleep is about 7 hours daily with total time in bet at around 7 hours and 38 minutes

and to confirm the average number of days sleep was logged, I used the following query:

 ``` sql
SELECT type,
ROUND(AVG(days_logged)) AS average_days_logged,
FROM `project-1-396413.Fitbit.days_logged`
GROUP BY type
```
Output:

|type|average_days_logged|
|----|-------------------|
|Sleep Log|17.0               |
|Activity Log| 28.0              |
|Daily Step Log| 28.0              |
|Calorie Log| 28.0              |

From this table, we see that even when users did log their sleep data, it was an average of 17 days with only 3 users recording for a full 31 days. It is reasonable to say that the sleep data is missing a lot of data points compared to the other datasets even with 24 participants logging data.

### Merging Data

To merge the relevant data, we know that the "dailyActivity_merged.csv" dataset is the most full with all the users Id's as well as the most averagely logged. 

Using the following SQL query below to create a new table with merged data using LEFT JOIN on the 'activity log' as the activity log is the most populated table that leaves null data from the Sleep and Weight Log where data was not collected. (The null data will be used later)

``` sql
CREATE TABLE IF NOT EXISTS `project-1-396413.Fitbit.FitbitDataMerged` AS
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
This query outputs the file "FitbitDataMerged.csv" when the dataset is downloaded

### Visualisation

[Tableau Public Link](https://public.tableau.com/app/profile/kes.andrew.raath/viz/FitbitFitnessTrackingDataViz_16998663196860/TotalStepsvsCalories)
Click the above to view the visualisations in full

The following CSV's were used: 
- Using "sleepDay_merged.csv" for sleep data and the calendar viz
- Using the "FitbitDataMerged.csv" for Sleep vs Total Time in Bed and Total Steps vs Calories Vizzes

To confirm the data is making sense, more activity, in this case, steps lead to more calories burned. 

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/0d72fa5a41940a8dd509ba1d3268f23d332104dd/TotalSteps_vs_Calories.png?raw=true)

From the table above we see that there is a positive correlation between the number of steps taken and the number of calories burned. The more activity in this case walking, or steps taken the more calories burned.

________

To learn more about the habits of users in a visual way by comparing how the amount of time in bed relates to the amount of sleep one has.

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/BellabeatProjectFiles/TotalSleep_vs_TotalTimeInBed.png?raw=true)

We can see there is a linear correlation between the amount of time in bed and the amount of sleep one has. This suggests that users who are in bed longer sleep more. 
A notification to get in bed earlier could be helpful here.

_________

Looking for other aspects of a user's day that could be affecting their sleep I looked at how the lack of movement, sedentary time, affects one's ability to sleep.

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/BellabeatProjectFiles/Sleep_vs_Sedentary_Time.png?raw=true)

The graph above shows a trend that users who have more sedentary minutes in a day are more likely to have less sleep. 

This is possibly from unused energy making users less tired at the end of a day, suggesting users move more, go for a walk, or exercise may help with this. However further data may be needed to confirm that more exercise leads to better sleep. 

__________

To visualise better the users who do log their sleep vs those who don't across the days of the data. This gives a clear understanding of how often users are logging their data, on which days as well as how many days consecutively or not at all.

![alt text](https://github.com/Kesraath/BellabeatFitnessProject/blob/0d72fa5a41940a8dd509ba1d3268f23d332104dd/User%20Sleep%20Log%20Days.png?raw=true)

This used the "sleepDay_merged.csv" and does not include users who didn't log sleep data.

As noted earlier, even with 24 participants, sleep was logged very occasionally. This could suggest that participants aren't wearing their devices to sleep, the device requires charging as it was used throughout the day, or the participants didn't consider logging their sleep data.

### Recommendations for Bellabeat

disclaimer/limitations:
- As the data is from 2016, user usage may have changed not to mention the capability of devices may have changed completely.
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
3. Bellebeat can increase the users' awareness of exercise and the amount of sleep or sleep quality by suggesting users reduce sedentary time.
    - Keeping in mind, however, there is a possibility that the device did not register the sleep and instead recorded it as sedentary time.
4. Create an easier method to sync or record weight data
    - As seen by the lack of participants, Bellabeat could cater to this area by integrating weight tracking better.
    - Create banners on the first start-up of the app after download or awareness if not already connected that users should allow Bellabeat to access other Health app data to allow for weight tracking.
      -    Note that a disclaimer to users on the data collected and how it will be used/protected. 

Outside of app-based changes, possible hardware recommendations are that Bellabeat:
1. Increases the comfort so that users wear their devices to sleep
2. Reduces the time it takes to charge the device
3. Have a device that is able to run for multiple days
4. Bundle in a scale for users as part of a promotion to promote a better understanding of one's weight and how to use it to get fit. 

These hardware change recommendations will need to work in conjunction with the software suggestion above, such as informing users about collecting sleep data and how they can use that to assist in their sleep habits.


Further analysis of specific data is required to have a full conclusion, as such using Bellabeat's own consented user data to corroborate the findings above will be the best way to correct for the limitation of a small sample size in the data from Fitbit.

Thank you for reading my case study.

