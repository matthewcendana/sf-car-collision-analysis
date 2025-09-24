# San Francisco Traffic Crashes Resulting In Injury Analysis (2005–2025)
This repository documents my  exploratory data analysis process for a dataset containing over 60,000 car collision records in San Francisco, utilizing PostgreSQL and Excel. In addition to the SQL queries, I also created a dashboard in Tableau for visualizing the data. Check out the links below for more info!

[Original Dataset: SF Traffic Crashes Resulting in Injuries](https://data.sfgov.org/Public-Safety/Traffic-Crashes-Resulting-in-Injury/ubvf-ztfx/about_data)

[View the dashboard on Tableau Public](https://public.tableau.com/views/sf-collision-dashboard/CollisionDatetimeDashboard?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

**Note:** I used an extension for a treemap that cannot be viewed on Tableau Public without a license. I currently don't have one, but please check the bottom of this repository for more information on the Tableau dashboard.

## Database Setup:

The dataset was imported from a CSV into a PostgreSQL table using DBeaver. Since the dataset contains 63 columns, I first created the table manually with defined data types, then imported the CSV into the existing table to prevent any column type mismatches.

**Table name**: `sf_crashes`

```sql
CREATE TABLE sf_crashes (
    unique_id BIGINT PRIMARY KEY,
    cnn_intrsctn_fkey BIGINT,
    cnn_sgmt_fkey BIGINT,
    case_id_pkey BIGINT,
    tb_latitude REAL,
    tb_longitude REAL,
    geocode_source TEXT,
    geocode_location TEXT,
    collision_datetime TIMESTAMP,
    collision_date TIMESTAMP,
    collision_time TEXT,
    accident_year TEXT,
    month TEXT,
    day_of_week TEXT,
    time_cat TEXT,
    juris TEXT,
    officer_id TEXT,
    reporting_district TEXT,
    beat_number TEXT,
    primary_rd TEXT,
    secondary_rd TEXT,
    distance INTEGER,
    direction TEXT,
    weather_1 TEXT,
    weather_2 TEXT,
    collision_severity TEXT,
    type_of_collision TEXT,
    mviw TEXT,
    ped_action TEXT,
    road_surface TEXT,
    road_cond_1 TEXT,
    road_cond_2 TEXT,
    lighting TEXT,
    control_device TEXT,
    intersection TEXT,
    vz_pcf_code TEXT,
    vz_pcf_group TEXT,
    vz_pcf_description TEXT,
    vz_pcf_link TEXT,
    number_killed INTEGER,
    number_injured INTEGER,
    street_view TEXT,
    dph_col_grp TEXT,
    dph_col_grp_description TEXT,
    party_at_fault TEXT,
    party1_type TEXT,
    party1_dir_of_travel TEXT,
    party1_move_pre_acc TEXT,
    party2_type TEXT,
    party2_dir_of_travel TEXT,
    party2_move_pre_acc TEXT,
    point TEXT,
    data_as_of TIMESTAMP,
    data_updated_at TIMESTAMP,
    data_loaded_at TIMESTAMP,
    analysis_neighborhood TEXT,
    supervisor_district TEXT,
    police_district TEXT,
    current_police_districts TEXT,
    current_supervisor_districts TEXT,
    analysis_neighborhoods TEXT,
    neighborhoods TEXT,
    sf_find_neighborhoods TEXT
);
```

## Data Exploration:

```sql
SELECT unique_id, COUNT(*) 
FROM sf_crashes 
WHERE unique_id IS NULL
GROUP BY unique_id;
```
There were no null values for `unique_id`, meaning each crash in the dataset has a primary key. 

```sql
SELECT cnn_sgmt_fkey, COUNT(*) 
FROM sf_crashes 
WHERE cnn_sgmt_fkey IS null
GROUP BY cnn_sgmt_fkey; 
```
There are 34,650 null values in `cnn_sgmt_fkey` (nearest street centerline segment key). According to the metadata, this field is left empty when a crash occurs at an intersection, indicating that 34,650 crashes in the dataset took place at intersections.

```sql
SELECT accident_year, COUNT(*) AS total_crashes
FROM sf_crashes
GROUP BY accident_year
ORDER BY accident_year;
```

### Exploring Datetimes: 
The earliest collision was recorded on January 1st, 2005, and the latest was on June 30th, 2025 (found via Excel).

Finding the total number of crashes per year:
```sql
SELECT accident_year, COUNT(*) AS total_crashes
FROM sf_crashes
GROUP BY accident_year
ORDER BY accident_year;
```

### Output:

<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/crashes-by-year.jpeg" alt="Yearly crashes" width="600">
</div>

<div align="center">
<em>2019 had the most total crashes, while 2020 had the fewest likely due to quarantine (excluding 2025).</em>
</div>

### Crashes by Month:
```sql
SELECT 
    TO_CHAR(collision_datetime, 'Month') AS accident_month,
    COUNT(*) AS crashes_by_month
FROM sf_crashes
GROUP BY accident_month
ORDER BY 
  CASE TRIM(TO_CHAR(collision_datetime, 'Month'))
    WHEN 'January'   THEN 1
    WHEN 'February'  THEN 2
    WHEN 'March'     THEN 3
    WHEN 'April'     THEN 4
    WHEN 'May'       THEN 5
    WHEN 'June'      THEN 6
    WHEN 'July'      THEN 7
    WHEN 'August'    THEN 8
    WHEN 'September' THEN 9
    WHEN 'October'   THEN 10
    WHEN 'November'  THEN 11
    WHEN 'December'  THEN 12
  END;
```
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/crashes-by-month.jpg" alt="Monthly crashes" width="600">
</div>

<div align="center">
<em>October had the most total crashes, while July had the fewest.</em>
</div>

### Crashes By Day of The Week:
```sql
SELECT day_of_week, COUNT(*) AS crashes_by_day
FROM sf_crashes
GROUP BY day_of_week
--This is just for formatting the table from Monday - Sunday
ORDER BY 
  CASE day_of_week
    WHEN 'Monday' THEN 1 
    WHEN 'Tuesday' THEN 2
    WHEN 'Wednesday' THEN 3
    WHEN 'Thursday' THEN 4
    WHEN 'Friday' THEN 5
    WHEN 'Saturday' THEN 6
    WHEN 'Sunday' THEN 7
  END;
```
Friday had the most crashes of any day of the week (9,316), while Sunday had the fewest (7,683). 9 crashes had no recorded weekday:

```sql
SELECT *
FROM sf_crashes
WHERE day_of_week IS NULL
   OR TRIM(day_of_week) = '';
```

Each of these datapoints was a valid entry with a unique_id. Using the collision_datetime variable, we can pinpoint what day of the week the crash occurred on and fill in these empty values:

```sql
UPDATE sf_crashes
SET day_of_week = TRIM(TO_CHAR(collision_datetime, 'Day'))
WHERE day_of_week IS NULL OR TRIM(day_of_week) = '';
```

<div align="center">
<em>Final Output: </em>
</div>

<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/crashes-by-day.jpg" alt="Crashes by day" width="600">
</div>


### Crashes by Hour of the Day:
```sql
SELECT 
    TO_CHAR(collision_datetime, 'HH12 AM') || ' - ' ||
    TO_CHAR(collision_datetime + interval '1 hour', 'HH12 AM') AS hour_range,
    COUNT(*) AS total_crashes
FROM sf_crashes
GROUP BY hour_range
ORDER BY total_crashes DESC;
```
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/crashes-by-hour.jpg">
</div>

<div align="center">
<em>*Ordered from most frequent hour intervals to least frequent, collisions peaked between 5–6 PM and during typical rush hour periods, highlighting the increased risk associated with heavy commuter traffic.</em>
</div>

### Exploring Weather Conditions:
```sql
SELECT 
    TRIM(weather_1) AS weather_condition,
    COUNT(*) AS count
FROM sf_crashes
GROUP BY TRIM(weather_1)
ORDER BY count DESC;
```
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/weather1.jpg">
</div>

<div align="center">
    <em> Clear was the most frequent weather condition. To understand whether weather actually affects collision risk, additional analysis would be needed since "clear" may simply be the most common overall condition. </em>
</div>


```sql
SELECT 
    TRIM(weather_2) AS weather_condition,
    COUNT(*) AS count
FROM sf_crashes
GROUP BY TRIM(weather_2)
ORDER BY count DESC;
```
Most of the collisions (60,678) had a 2nd weather condition of 'Not Stated'

### Collision Severity:
```sql
SELECT 
    collision_severity,
    COUNT(*) AS crash_count
FROM sf_crashes
GROUP BY collision_severity
ORDER BY crash_count DESC;
```

<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/severity.jpg">
</div>


### Type of Collision:
```sql
SELECT 
    type_of_collision,
    COUNT(*) AS crash_count
FROM sf_crashes
GROUP BY type_of_collision
ORDER BY crash_count DESC;
```

<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/type%20of%20collision.jpg">
</div>

<div align="center">
    <em> Broadside collisions were the most frequent. A broadside collision, also called a T-bone, happens when the front of one vehicle strikes the side of another, often at intersections. </em>
</div>

### Probable Cause:
```sql
SELECT 
    TRIM(vz_pcf_description) AS cause_description,
    COUNT(*) AS crash_count
FROM sf_crashes
GROUP BY TRIM(vz_pcf_description)
ORDER BY crash_count DESC;
```
There were 249 unique cause descriptions. 'Unsafe speed for prevailing conditions' had the highest count of 11,703.


### Collision Hotspots:
```sql
SELECT 
    analysis_neighborhood AS neighborhood,
    COUNT(*) AS total_crashes,
    SUM(CASE WHEN collision_severity = 'Fatal' THEN 1 ELSE 0 END) AS fatal_crashes,
    SUM(CASE WHEN collision_severity = 'Injury (Complaint of Pain)' THEN 1 ELSE 0 END) AS injury_complaint_pain,
    SUM(CASE WHEN collision_severity = 'Injury (Other Visible)' THEN 1 ELSE 0 END) AS injury_visible,
    SUM(CASE WHEN collision_severity = 'Injury (Severe)' THEN 1 ELSE 0 END) AS injury_severe
FROM sf_crashes
GROUP BY analysis_neighborhood
ORDER BY fatal_crashes DESC, injury_severe DESC, injury_visible DESC, injury_complaint_pain DESC
LIMIT 15;
```

<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/collision-hotspots.jpg">
</div>

<div align="center">
    <em> This query counts the total number of collisions and breaks them down by severity (fatal, severe injury, etc.). Mission Street had the highest number of collisions. </em>
</div>



## Tableau Screenshots:

### Collisions by Year Line Graph: 
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/tableau-1.jpg">
</div>

### Collisions by Hour Bar Chart:
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/tableau-2.jpg">
</div>

### Collisions by Day of Week + Severity Grouped Bar Chart: 
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/tableau-3.jpg">
</div>

###Type of Collision Tree Map: 
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/tableau-4.jpg">
</div>

### Top 15 Collision Neighborhoods Tree Map:
<div align="center">
  <img src="https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/tableau-5.jpg">
</div>

