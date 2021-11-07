# Elevator Data Modeling

## Entity Relationship Diagram (ERD)

![elevator_data_model](https://user-images.githubusercontent.com/20394555/140633325-252723e9-15a6-4063-88d3-2ca94c6b5681.jpeg)

- `up_down_status` field in `elevator_cage` table, is **bool(True/False)**. Up is **True**, down is **false**
- `elevator_status` field in `elevator_cage` table, is **bool(True/False)**. For example If someone is using elevator should be **True**.
- `door_status` field in `elevator_bank` table, is **bool(True/False)**. Door opened **True**, closed **False**.
- Elevator can not be response back if `is_maintenance` field in `elevator_cage` table, is **bool(True/False)** 
- Elevator can be empty/idle. This is the why `weight`is **Nullable**
- Elevator can be empty/idle. This is the why `arrival_floor` will be **Nullable**
- When elevator moving and haven't reach to the destination `arrived_at` can be **Null**

`elevator_cage` and `elevator_bank` (all elevators have only one door **one-to-one**)
`elevator_cage` and `elevator_button` (all elevators have buttons **one-to-many**)

1) List other performance measures that would be useful or important to measure

- Which elevator have the highest number of activities? 
- Elevator usage statistics by floor
- Average time spent in the elevator
- Average passenger count on hourly based
- Elevator travelling speed.
- Elevator response time.
- Average number of trips requiring maintenance
- Duration of maintenance

2) What would a suitable data representation look like

I think denormalized tables are more efficient in situations like this. But there is no one-size-fits-all approach and there is no one-size-fits-all solution.😂
We can easily change our model to any point of interest we want to focus on.

### Pros
-  Enhance query performance
-  Makes a database more suitable to manage
-  Facilitate reporting
-  No need to calculation for every query or report

### Cons
- Data inconsistencies because of the data duplication
- Extra column requires additional working and disk space
- Recoding required if look-up values are altered
- Extra afford to ensure consistency of data

On the other hand, we can think another dimensional model too. Which has 1 dimensional 1 fact table like this:

<img width="744" alt="Screen Shot 2021-11-07 at 09 17 24" src="https://user-images.githubusercontent.com/20394555/140634620-121623f9-524c-4bed-a4c7-cc5bc94bd4d7.png">


### 3) For “Average waiting time per passenger” and at least 2 other performance measures, describe how they can be easily calculated from your data model. 

- Average waiting time per passenger

  ```sql
  SELECT AVG(arrive_timestamp - request_timestamp) AS avg_waiting_time
    FROM elevator_cage
  ```

- Average journey time per passenger

  ```sql
  SELECT AVG(arrived_at - requested_at) AS avg_journey_time
    FROM elevator_cage
  ```

- Elevator ranking by most activities

  ```sql
  SELECT elevator_id, count(id) AS events_count
    FROM elevator_cage
   GROUP BY elevator_id
   ORDER BY events_count DESC
  ```

- Floor with the most elevator calls

  ```sql
  SELECT arrival_floor, count(id) AS events_count
    FROM elevator_button
   GROUP BY arrival_floor
   ORDER BY events_count DESC
  ```
- The hours in which the elevator is used the least in a day
  
  ```sql
  SELECT min(weight), elevator_id, hour(to_date(requested_at)) AS hour 
  FROM elevator_cage
  WHERE min(weight) > 0
  GROUP BY elevator_id, hour(to_date(requested_at))
  ```
- Elevator ranking by most maintenance time

  ```sql
  SELECT elevator_id, count(id) AS events_count
    FROM elevator_cage
   GROUP BY elevator_id
   ORDER BY events_count DESC
  ```
  
References: https://rubygarage.org/blog/database-denormalization-with-examples
