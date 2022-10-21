## Bellabeat Case Study

**Project description:** In this case study I am a data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused products for women. 

Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. 

My job is to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices.

### 1. Ask

In this analysis, we study existing trends in smart products and make an informed recommendation to guide the marketing strategy for a specific Bellabeat product. 

The key stakeholders for this analysis are Urška Sršen (Co-founder and CCO) and the Bellabeat marketing analytics team. Sršen assigned this project and will make a final decision based on our analysis. We are a part of the Bellabeat marketing team, and our work reflects on our team members. 

I was given a dataset to work with that contained data on the activity of 30 FitBit users. I found the heartrate table very interesting as it displayed user heartrate for every second over the timeframe of the study. 

My analysis focused on the hypothesis that seemingly healthy users may benefit from medical information about their heart rate.
This could be an important preventative measure against heart-related issues such as Atrial fibrillation. 



```javascript
if (isAwesome){
  return true
}
```

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


<img src="images/dummy_thumbnail.jpg?raw=true"/>

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



For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
