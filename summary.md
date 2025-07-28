# San Francisco Traffic Crashes Resulting In Injury Analysis (2005–2025)
This repository documents my process for formatting/analyzing a dataset with over 60,000 crash records in San Francisco using PostgreSQL and Excel. Original Dataset: [SF Traffic Crashes Resulting in Injuries](https://data.sfgov.org/Public-Safety/Traffic-Crashes-Resulting-in-Injury/ubvf-ztfx/about_data)

## Database Setup

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

## Data Exploration

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
Output:
![yearly_craashes](https://github.com/matthewcendana/sf-car-collision-analysis/blob/main/images/yearly_crashes.jpeg)
2019 had the most total crashes, while 2020 had the fewest (excluding 2025).



