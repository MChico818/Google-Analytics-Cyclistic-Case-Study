
### Case Background
  As a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago, I am tasked with understanding how casual riders and annual members use Cyclistic bikes differently. The marketing director theorizes that the company's future success depends on maximizing the number of yearly memberships by converting casual riders into annual members. Pending executive approval, my team will be designing a new marketing strategy that pursues this idea.
  
  To inform any decision-making behind Cyclistic's new marketing strategy, the goal of this project will be to uncover and convey actionable insights.
  

  
### Ask
 - **Business Task**: How do annual members and casual riders use Cyclistic bikes differently?
 - **Goal** : Analyz the Cyclistic historical bike data to identify trends to then design marketing strategies aimed at converting casual riders into annual members
 - **Stakeholders** :   Director of marketing & other executives/key decision makers

### Prepare
**Step 1**: Downloaded previous 12 months of historical trip data from January 2024 - December 2024

**Step 2**: Saved original .csv files all into one folder

--**Data was sourced from first-party group**

### Process
  Used Excel for cleaning the data for each month. Wanted to organize each months data before combining all months.
  - **Deleted any duplicates**
  - **Checked for empty values**
  - **Made sure dates were not overlapping into other months**
  - **Deleted columns: start_station, end_station, start_lat, end_lat, start_lng, end_lng, rideable_type**
  - **Added columns: ride_time, weekday, month**
  - **Filtered ride_times and deleted ride_times that were < 1min**
  - **Changed column names to make readability simplier: ride_id, member_casual, date, start_time, end_time, ride_time, weekday, month**

### Analyze
Imported each month into Big Query since dataset was so large and then combined all months into a full-year data table

```sql
create table `Bike_Final_Data.combined_data` as 
with combined_data as (
  select * from `Bike_Final_Data.Jan_Bike`
  union all
  select * from `Bike_Final_Data.Feb_Bike`
  union all
  select * from `Bike_Final_Data.Mar_Bike`
  union all
  select * from `Bike_Final_Data.April_Bike`
  union all
  select * from `Bike_Final_Data.May_Bike`
  union all
  select * from `Bike_Final_Data.June_Bike`
  union all
  select * from `Bike_Final_Data.July_Bike`
  union all
  select * from `Bike_Final_Data.Aug_Bike`
  union all
  select * from `Bike_Final_Data.Sept_Bike`
  union all
  select * from `Bike_Final_Data.Oct_Bike`
  union all
  select * from `Bike_Final_Data.Nov_Bike`
  union all
  select * from `Bike_Final_Data.Dec_Bike`
)
select * from combined_data;
```
- Result: Combined table contains = **5,724,330 rows**


Now I was able to explore trends and relationships in bike data from Cyclistic

**Total number of members**
```sql
  select
  member_casual,
  count(member_casual) as total_riders
from `Bike_Final_Data.combined_data`
group by
  member_casual;
```
  - Annual Members = **3,641,109**
  - Casual Members = **2,083,221**

**Average Ride Time**
```sql
select
  member_casual,
  round(avg(TIME_DIFF(ride_time, TIME(0,0,0), second)/60)) as avg_ride_time
from
  `Bike_Final_Data.combined_data`
group by
  member_casual;
```
- Annual Members = **12 min**
- Casual Members = **22 min**

**Total Weekday Rides**
```sql
SELECT
  weekday,
  member_casual,
  count(ride_id) as total_rides
FROM
  `Bike_Final_Data.combined_data`
GROUP BY
  weekday,
  member_casual
ORDER BY
  CASE
    WHEN weekday = 'Sunday' THEN 1
    WHEN weekday = 'Monday' THEN 2
    WHEN weekday = 'Tuesday' THEN 3
    WHEN weekday = 'Wednesday' THEN 4
    WHEN weekday = 'Thursday' THEN 5
    WHEN weekday = 'Friday' THEN 6
    WHEN weekday = 'Saturday' THEN 7
  END;
```
- **Members**:
    - Most popular day = **Wednesday**
    - Least popular day = **Sunday**
- **Casual**:
    - Most popular day = **Saturday**
    - Least popular day = **Tuesday**

**Rides Per Hour**
```sql
select
  member_casual,
  EXTRACT(HOUR FROM PARSE_TIMESTAMP('%I:%M:%S %p', start_time)) as hour_of_day,
  count(ride_id) as number_of_rides
from
  `Bike_Final_Data.combined_data`
where
  member_casual = 'member'
group by
  member_casual, hour_of_day
order by
  number_of_rides desc;
```
- **Members**: Busiest time of day: **7am-8am** & **4pm-5pm**
- **Casual**: Busiest time of day: **5pm**

**# of Rides Per Month**
```sql
select
  month,member_casual,
  count(ride_id) as total_rides
from
  `Bike_Final_Data.combined_data`
where
  member_casual = 'casual'
group by
  month, member_casual
order by
  total_rides desc;
```
- **Members**:
    - Most popular months: **July, August, September**
    - Least popular months: **November, December, January**
- **Casual**:
    - Most popular months: **July, August, September**
    - Least popular months: **December, January, February**

 Summer months are when people are riding more and of course during the winter is when are not going to be riding as much

## Share
![image](https://github.com/user-attachments/assets/7dc66749-a3c8-477f-915f-d98a1bfe7077)
![image](https://github.com/user-attachments/assets/53ce1c15-5159-428b-8584-ec9d2c911c4b)
![image](https://github.com/user-attachments/assets/2f60bf8a-68b3-4b3a-b286-e59e9bd1280c)
![image](https://github.com/user-attachments/assets/5263d814-5e4a-4eb1-a8e2-906379564e80)
![image](https://github.com/user-attachments/assets/2cf8da26-36f3-41bf-9479-f61fd9318a4e)

  
## Act

**Key Findings**
  - The analysis on casual and member riders mainly show the behavioral differences between the two groups. Annual members consistently ride more frequently since bikes are their means of transportation. Hence why peak riding times are beginning of work day **(7am-8am)** and end of work day **(4pm-5pm)**. **Weekdays** are the busiest riding days for members; while **weekends** are busiest for the casual riders.
  - Even during the winter months, annual members far outride casual members which shows the weight of bikes as their main souce of transportation.
  - Casual members trips are longer which shows they are riding for leisure when they have free time and not rushing to and from work

Since casual members don't use bikes as their daily source of transportation, there is no significant motivation to commit to the annual membership

**Possible Recommendations**
1) Promote membership discounts during the high riding season (Summer) to entice casual members to joining the annual membership plans
2) Promote weekend discount campaigns on weekends when casual riders ride the most
3) Creating referral systems that benefit both annual and casual members and the company












