## Bellabeat Case Study

**Project description:** In this case study I am working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused products for women. 

Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. 

My job is to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices.

### 1. Ask

In this analysis, we study existing trends in smart products and make an informed recommendation to guide the marketing strategy for a specific Bellabeat product. 

The key stakeholders for this analysis are Urška Sršen (Co-founder and CCO) and the Bellabeat marketing analytics team. Sršen assigned this project and will make a final decision based on our analysis. We are a part of the Bellabeat marketing team, and our work reflects on our team members. 

I was given a dataset to work with that contained data on the activity of 30 FitBit users. I found the heartrate table very interesting as it displayed user heartrate for every second over the timeframe of the study. 

My analysis focused on the hypothesis that seemingly healthy users may benefit from medical information about their heart rate.
This could be an important preventative measure against heart-related issues such as Atrial fibrillation. 


### 2. Prepare

I downlaoded the FitBit Fitness Tracker dataset (by Mobius) from Kaggle and stored it in ‘Prepare’ folder within the ‘Case Study 2’ folder on my Desktop. 

The dataset is downloaded as a file named ‘Fitbase Data 4.12.16 – 5.12.16’. The files within are .csv files. Most of the data is stored in the long format, some are in wide format. 

To determine the credibility of the data I looked at reliability, originality, comprehensiveness, how up to date it is, and citability. 

The reliability of the data checks out as it is collected objectively from thirty Fitbit users. My only concern is that this is a small sample size, and we would have liked data from a range of smart fitness devices to get a good overall understanding of the market. 

The data is collected firsthand and therefore I am confident in its originality.

The data has been cleaned and includes all relevant activities we are looking at across different time frames (calories, steps, sleep minutes etc.). 

This data was collected in 2016 however the data is updated annually. The last update was 2 years ago. 

The dataset has been cited to distributed survey via Amazon Mechanical Turk. 

This dataset is also licensed under CC0, meaning it is public domain. This allows us to manipulate and reproduce the data without copy right concerns.


### 3. Process

I primarily worked with three tables from this dataset, each of which needed cleaning. 

The tables I used were heartrate_seconds_merged, daily_steps_merged, and weightLogInfo_merged. 
These tables would give me insight into users' heartrate statistics as well as their general health and activity. 

The first table, heartrate_seconds_merged, was extremely large, containing over 2 million rows of data. This meant I could not initially clean it on Excel.

I chose to upload the data to SQL, however, the date and time column could not be read. I changed all the schema to STRING and proceeded. 

I then changed the schema, separated the date and time, hid the first row since it displayed the column names.


```
-- Casting the rows appropriately
SELECT
  CAST(Id AS INTEGER) AS Id,
  LEFT(Time,9) AS Date,
  RIGHT(TIME, 10) AS Time,
  CAST(Value AS FLOAT64) AS heartrate
FROM
  `adept-student-364007.fitabase_data.heartrate`

-- First column has the column names in it
WHERE
  Id <> "Id"
```

Next, I checked different columns for any null data using the IS NULL function. Later I made sure the dates were inputted correctly using the LENGTH(STRING) function to check if any data didn’t meet the 10-string requirement.


I was able to clean the daily_steps_merged, and weightLogInfo_merged tables in Excel. 
This involved seperating the data and time, changing the date format and checking the data for blanks or errors.


### 4. Analyze

#### A) Heart Rate Boxplot
We started by looking to plot the heartrate data as it had been given. This would give us an idea of how each user’s heartrate fluctuated over the course of the study.

The cleaned heartrate table was saved as heartrate_clean, however, it was too large to download as a local file. 

We saved the table to Google Could Storage using the following code:

```
EXPORT DATA
  OPTIONS (
    uri = 'gs://*****/*****/*.csv',
    format = 'CSV',
    overwrite = true,
    header = true,
    field_delimiter = ';')
AS (
  SELECT Id, Date, Time, heartrate
  FROM `adept-student-364007.fitabase_data.heartrate_clean`
  ORDER BY Id
);
```
(I have hidden the original storage location for privacy)

After uploading the heartrate_clean table to Tableau Public, we made sure the schema was accurate. 

I plotted the heartrate of each individual user vs the date (day). I used a boxplot and whisker plot to be able to visualize the median, interquartile range, maximum, and minimum values during each day of the study. This plot helps us an visualize the resting heartrate and note when the heartrate has gone over/under dangerous levels. 

<img src="images/heartrate_boxplot.png?raw=true"/>

In this plot, I added two reference lines at 60bpm and 120bmp. This is the range of a healthy resting heartrate according to Mayo Clinic .

The visualization helps us see that some users in the study show possibly concerning heartrate levels. User ID 4388161847 (second from the top) recorded a median heartrate that was consistently close to the 60bmp lower limit. This means the lower hinge and lower whisker were constantly below 60bpm.


User ID 8877689391 (bottom user) had interesting heartrate fluctuations. Most of the IQR data was within the healthy limit, however there were days where the heartrate was surprisingly high. April 30th in particular saw an upper hinge of 158 and an upper whisker of 170. These isolated instances could potentially be cause for concern. 


#### B) Heart Rate vs Steps

Next, I wanted to understand what average heartrate looked like compared to user activity. This would give us insight into whether more active users were less at risk of heart disease. 

I uploaded the steps table as daily_steps_clean. I wanted to create a table with daily steps data merged with the average, minimum, and maximum daily heartrate for each user. 

I queried the heartrate_ clean table to find the daily heartrate data. 

```
SELECT 
  Id,
  Date,
  Max(heartrate) AS Maximum_Heartrate,
  MIN(heartrate) AS Minimum_Heartrate,
  AVG(heartrate) AS Average_Heartrate
FROM 
  `adept-student-364007.fitabase_data.heartrate_clean`
GROUP BY 
  Id,
  Date
ORDER BY
  Id,
  Date;
```

I then saved this query as a new table named daily_heartrate.

Before joining daily_steps and daily_heartrate, I cleaned the Date Column in the two tables to make the schema, format, and column name uniform. I saved the new tables as daily_steps_clean and daily_heartrate_clean

To join the data I used the following code:

```
SELECT
  daily_heartrate_clean.Id,
  daily_heartrate_clean.Date,
  daily_heartrate_clean.Maximum_Heartrate,
  daily_heartrate_clean.Minimum_Heartrate,
  daily_heartrate_clean.Average_Heartrate,
  daily_steps_clean.Steps
FROM 
  fitabase_data.daily_heartrate_clean
JOIN
  fitabase_data.daily_steps_clean ON
  daily_heartrate_clean.Id = daily_steps_clean.Id AND
  daily_heartrate_clean.Date = daily_steps_clean.Date;
```

The next step was uploading the new table to Tableau and create a visualization. 

I looked at the correlation between average daily heart rate and daily steps.

<img src="images/average_heartrate_vs_steps.png?raw=true"/>

The data showed very little correlation between the two variables. 


When looking at min/max daily heartrate vs steps we also see negligible correlation. 


<img src="images/max_min_heartarte_vs_steps.png?raw=true"/>

#### C) Heart Rate vs BMI

Lastly, I looked at how a user’s weight affected their heartrate. 

I started by uploading the cleaned weight data to SQL as weight_log. 

To get an idea of the average weight and BMI for each user, I queried the table.

```
SELECT
  DISTINCT ID,
  AVG(WeightKg) AS Average_WeightKg, 
  AVG(WeightPounds) AS Average_WeightPounds,
  AVG(BMI) AS Average_BMI
FROM 
  `adept-student-364007.fitabase_data.weight_log`
GROUP BY
  ID
ORDER BY
  ID
```

Later, I joined the daily_heartrate and weihgt_log tables before exporting them to Tableau. 

```
SELECT
  daily_heartrate_clean.Id,
  daily_heartrate_clean.Date,
  daily_heartrate_clean.Maximum_Heartrate,
  daily_heartrate_clean.Minimum_Heartrate,
  daily_heartrate_clean.Average_Heartrate,
  weight_log.weightKg,
  weight_log.weightPounds,
  weight_log.BMI
FROM 
  fitabase_data.daily_heartrate_clean
JOIN
  fitabase_data.weight_log ON
  daily_heartrate_clean.Id = weight_log.ID AND
  daily_heartrate_clean.Date = weight_log.Date;
```

The viz I created again showed little correlation between BMI and heart rate.

<img src="images/average_heartrate_vs_bmi.png?raw=true"/>

The plot includes reference lines to show the healthy BMI range and the average of the heart rate data being situated at 80bpm (healthy). 
It is important to note that we did not have a large range of BMI and these users were generally fit. 


### 5. Conclusion

In my analysis I found that some users had a dangerously low blood rate over the period of the study. The heartrate data was independent of the user’s overall health and not generally affected by their physical activity.

The data we have can be used to communicate if a user may need to consult a doctor about their heartrate. Seemingly healthy users can show signs of Afib or other heart related conditions. 

This data needs to include the user’s age to be more relevant.

Making use of heartrate data would give us a more holistic approach as a transparent fitness tool. It can also open us up to a newer market segment for older women.


