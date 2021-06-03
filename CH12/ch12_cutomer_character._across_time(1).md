## Chapter 12 Finding the pattern of consumer behavior across time

### 1. The indicator of business performance 

For companies currently managing multiple consumer services online, 
they need to define and track thier peformance indicator to see how each is on the right track. Of many measurements, they could take, 
the number of sign-ups for the website gives a clear picture of thier plans. If its number is proven to decrease over time, they take 
serious consideration to cease providing the service whereas it has shown either a steady or dramatic increase over a given period,
they need to step one further to asses any possibility of customer churn or attrition in the near future. 

#### Data

```sql
DROP TABLE IF EXISTS mst_users;
CREATE TABLE mst_users(
    user_id         varchar(255)
  , sex             varchar(255)
  , birth_date      varchar(255)
  , register_date   varchar(255)
  , register_device varchar(255)
  , withdraw_date   varchar(255)
);

INSERT INTO mst_users
VALUES
    ('U001', 'M', '1977-06-17', '2016-10-01', 'pc' , NULL        )
  , ('U002', 'F', '1953-06-12', '2016-10-01', 'sp' , '2016-10-10')
  , ('U003', 'M', '1965-01-06', '2016-10-01', 'pc' , NULL        )
  , ('U004', 'F', '1954-05-21', '2016-10-05', 'pc' , NULL        )
  , ('U005', 'M', '1987-11-23', '2016-10-05', 'sp' , NULL        )
  , ('U006', 'F', '1950-01-21', '2016-10-10', 'pc' , '2016-10-10')
  , ('U007', 'F', '1950-07-18', '2016-10-10', 'app', NULL        )
  , ('U008', 'F', '2006-12-09', '2016-10-10', 'sp' , NULL        )
  , ('U009', 'M', '2004-10-23', '2016-10-15', 'pc' , NULL        )
  , ('U010', 'F', '1987-03-18', '2016-10-16', 'pc' , NULL        )
  , ('U011', 'F', '1993-10-21', '2016-10-18', 'pc' , NULL        )
  , ('U012', 'M', '1993-12-22', '2016-10-18', 'app', NULL        )
  , ('U013', 'M', '1988-02-09', '2016-10-20', 'app', NULL        )
  , ('U014', 'F', '1994-04-07', '2016-10-25', 'sp' , NULL        )
  , ('U015', 'F', '1994-03-01', '2016-11-01', 'app', NULL        )
  , ('U016', 'F', '1991-09-02', '2016-11-01', 'pc' , NULL        )
  , ('U017', 'F', '1972-05-21', '2016-11-01', 'app', NULL        )
  , ('U018', 'M', '2009-10-12', '2016-11-01', 'app', NULL        )
  , ('U019', 'M', '1957-05-18', '2016-11-01', 'pc' , NULL        )
  , ('U020', 'F', '1954-04-17', '2016-11-03', 'app', NULL        )
  , ('U021', 'M', '2002-08-14', '2016-11-03', 'sp' , NULL        )
  , ('U022', 'M', '1979-12-09', '2016-11-03', 'app', NULL        )
  , ('U023', 'M', '1992-01-12', '2016-11-04', 'sp' , NULL        )
  , ('U024', 'F', '1962-10-16', '2016-11-05', 'app', NULL        )
  , ('U025', 'F', '1958-06-26', '2016-11-05', 'app', NULL        )
  , ('U026', 'M', '1969-02-21', '2016-11-10', 'sp' , NULL        )
  , ('U027', 'F', '2001-07-10', '2016-11-10', 'pc' , NULL        )
  , ('U028', 'M', '1976-05-26', '2016-11-15', 'app', NULL        )
  , ('U029', 'M', '1964-04-06', '2016-11-28', 'pc' , NULL        )
  , ('U030', 'M', '1959-10-07', '2016-11-28', 'sp' , NULL        )
;

SELECT register_date,
         COUNT(register_date) AS register_count,
          --alternatively, LAG(COUNT(register_date),1) OVER(ROWS BETWEEN 1 PRECEDING AND CURRENT ROW ORDER BY register_date)
          LAG(COUNT(register_date),1) OVER(ORDER BY register_date) AS previous_count,
          CAST(COUNT(register_date) AS NUMERIC)/LAG(COUNT(register_date),1) OVER(ORDER BY register_date)
          FROM mst_users
          GROUP BY register_date; 

```


![image](https://user-images.githubusercontent.com/53164959/66262935-ea5de700-e824-11e9-8721-7abbf9437cd0.png)



#### 2. Total counts of creating accounts via various devices 

At first, it may seem dubitable to classify the whole consumer into different sub-groups based on the electrical device they use 
to access the online platform.  In doing so, we can capture and analyze data to market to each group differently and focus on what 
each kind of customer needs at given moment. 

```sql

WITH month_date AS(
    SELECT SUBSTRING(register_date,1,7) AS month_year,
           register_device
           FROM mst_users
           GROUP BY month_year,register_device
           ORDER BY month_year)
     SELECT month_year,
           count(month_year) AS register_count,
           COUNT(CASE WHEN register_device='pc' THEN 1 END)AS register_count,
           COUNT(CASE WHEN register_device='sp' THEN 1 END) AS register_sp,
           COUNT(CASE WHEN register_device='app' THEN 1 END) AS register_sp
           FROM month_date
           GROUP BY month_year;
```
#### 4. Extra measurements for performance 

Solely relying on the registration date as a parameter for your analysis model, we could not make capture an accurate picture of the 
firm's latest performance. It is a good practice to give our mode two extra measurements-consistency rate and tenure( a period based
on arbitrarily assigned days). 

##### 4.1 consistent rate vs rentation rate

#### - consistent rate

The consistent rate refers to how long the customers stay with a company. The unique feature of this measure is that the customer 
counts into the n-th consecutive days based on the latest use of service. For example, we limit our scope of analysis to 5 days
after the initial registration. One consumer had never logged in but accessed your flatform service on the last day. The person 
adds to the group 5-day-consistency. 


#### - rentation rate 

The mechanism is pretty much same except for the followings;

First, the time period is every 7 day. 

Second, if consumer use the service at least once in the given period, he/she automatically counts to
        the total number. Number of Frequecny in this analysis does not matter at all. 

### 4.2 the first day conistency rate

We can easily see that a vast number of consumers cease to use the service the very next day after they sign up for the service. 
There is some time necessary to keep thier consumers satfisifed with the service provided and revisting. To see whether the continuing 
relationship with consumers occurs, we compute the consistent rate for one day right after the registration.


'One day after registration' refers to the proportion of the consumers who keep using the service provided by thier chosen companies.

There are a couple of technical tips to take into account.

First of all,  it would be probably okay to bring down to the number by
dividing the consumers accessing the service by the total numbers who signed up on the specific date. Alternatively,  it is
recommendable to use a flag variable; 1 for those available the very next or 0 for those who are absent. 

Your analysis should keep proceding. However, there is one condition to be met that all log data are accessible to distinguish between 
where the consumer fails to be with the company one day after the registration and where the company is waiting for consumers to get
back. 

Last but not least, our last date for analysis is the latest date when one ofusers revisit. Any date later would treat as NULL values.

```sql

WITH action_log_mst AS(
   SELECT m.user_id,
          CAST(m.register_date AS DATE) AS register_date,
          CAST(a.stamp AS DATE) AS action_date,
          MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
          CAST(m.register_date AS DATE)+'1 day'::interval AS next_one_day
          FROM mst_user AS m
          LEFT OUTER JOIN action_log AS a
               ON m.user_id=a.user_id)
      ,one_day_consistency AS(
       SELECT user_id,
              register_date,
              --Just in case where a user log in more than one on a specific date, we need to indicate whether 
              --they log on one day after register_date. Using sign function to return 1 to indicate that they have logged in
              --at least one time after register
              SIGN(SUM(
                   --removing any date later than latest_date
                   --assign 1  if the day of the first vist after sign up is the fllowing day.Otherwise, 0.
                   CASE WHEN next_one_day <latest_date THEN
                        CASE WHEN action_date=next_one_day THEN 1 ELSE 0 END
                        END)) AS next_one_day
              FROM action_log_mst
              GROUP BY user_id,register_date
              ORDER BY user_id,register_date)
        SELECT register_date,
               AVG(100*next_one_day) AS one_day_consistency
               FROM one_day_consistency
               GROUP BY register_date;
       
 ```
 
 ![image](https://user-images.githubusercontent.com/53164959/64406476-dce5fd80-d0bc-11e9-8d69-debf9030482c.png)
       
### 4.3 the nth day conistency rate

From now, we will take a look at how to compute the first n-th consistent rate in the following section. 
But it is one easy mistake to store consistent rates under the columns of a table.Instead, to avoid any complex and complicated queries, 
we need to store the relevant data row-wise.

```sql
/*
This code itself is really burdensome since we should type it one by one.
WITH repeat_interval(index_name,interval_date) AS(
     VALUES
     ('01 day repeat',1),
     ('02 day repeat',2),
     ('03 day repeat',3),
     ('04 day repeat',4),
     ('05 day repeat',5),
     ('06 day repeat',6),
     ('07 day repeat',7)
     )
  */
  WITH repeate_interval AS(
     SELECT CONCAT(index,'days') AS index_name,
            interval_date
          FROM GENERATE_SERIES(1,7) AS interval_date)
     SELECT * FROM interval_range;
  ,action_log_mst_index AS(
   SELECT m.user_id,
          CAST(m.register_date AS DATE) AS register_date,
          CAST(a.stamp AS DATE) AS action_date,
          MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
          r.index_name,
          CAST(CAST(m.register_date AS DATE)+ '1day'::interval*r.interval_date AS DATE) AS index_date
          FROM mst_user AS m
               LEFT OUTER JOIN action_log AS a
                    ON m.user_id=a.user_id
                    CROSS JOIN repeat_interval as r
                    ORDER BY user_id,action_date,index_name,index_date)
    ,user_action_flag AS(
     SELECT user_id, 
            register_date,
            index_name,
            --assigning the flag variables; 1 for visting webstie n days from the initial sign up and 0 for otherwise
            SIGN(SUM(
            CASE WHEN index_date<=latest_date THEN 
                 CASE WHEN index_date=action_date THEN 1 ELSE 0 END 
                 END)) AS index_date_action
            FROM action_log_mst_index
            GROUP BY user_id,register_date,index_name,index_date)
      SELECT register_date,
             index_name,
             AVG(100*index_date_action) AS repeat_rate
             FROM user_action_flag
             GROUP BY register_date,index_name
             ORDER BY register_date,index_name; 
 ```
 
####  4.4 Rentation Rate

Raw Data 
```sql
DROP TABLE IF EXISTS mst_users;
CREATE TABLE mst_users(
    user_id         varchar(255)
  , sex             varchar(255)
  , birth_date      varchar(255)
  , register_date   varchar(255)
  , register_device varchar(255)
  , withdraw_date   varchar(255)
);

INSERT INTO mst_users
VALUES
    ('U001', 'M', '1977-06-17', '2016-10-01', 'pc' , NULL        )
  , ('U002', 'F', '1953-06-12', '2016-10-01', 'sp' , '2016-10-10')
  , ('U003', 'M', '1965-01-06', '2016-10-01', 'pc' , NULL        )
  , ('U004', 'F', '1954-05-21', '2016-10-01', 'pc' , NULL        )
  , ('U005', 'M', '1987-11-23', '2016-10-01', 'sp' , NULL        )
  , ('U006', 'F', '1950-01-21', '2016-10-01', 'pc' , '2016-10-10')
  , ('U007', 'F', '1950-07-18', '2016-10-01', 'app', NULL        )
  , ('U008', 'F', '2006-12-09', '2016-10-01', 'sp' , NULL        )
  , ('U009', 'M', '2004-10-23', '2016-10-01', 'pc' , NULL        )
  , ('U010', 'F', '1987-03-18', '2016-10-01', 'pc' , NULL        )
  , ('U011', 'F', '1993-10-21', '2016-10-01', 'pc' , NULL        )
  , ('U012', 'M', '1993-12-22', '2016-10-01', 'app', NULL        )
  , ('U013', 'M', '1988-02-09', '2016-10-01', 'app', NULL        )
  , ('U014', 'F', '1994-04-07', '2016-10-01', 'sp' , NULL        )
  , ('U015', 'F', '1994-03-01', '2016-10-01', 'app', NULL        )
  , ('U016', 'F', '1991-09-02', '2016-10-01', 'pc' , NULL        )
  , ('U017', 'F', '1972-05-21', '2016-10-01', 'app', NULL        )
  , ('U018', 'M', '2009-10-12', '2016-10-01', 'app', NULL        )
  , ('U019', 'M', '1957-05-18', '2016-10-01', 'pc' , NULL        )
  , ('U020', 'F', '1954-04-17', '2016-10-02', 'app', NULL        )
  , ('U021', 'M', '2002-08-14', '2016-10-02', 'sp' , NULL        )
  , ('U022', 'M', '1979-12-09', '2016-10-02', 'app', NULL        )
  , ('U023', 'M', '1992-01-12', '2016-10-02', 'sp' , NULL        )
  , ('U024', 'F', '1962-10-16', '2016-10-02', 'app', NULL        )
  , ('U025', 'F', '1958-06-26', '2016-10-02', 'app', NULL        )
  , ('U026', 'M', '1969-02-21', '2016-10-02', 'sp' , NULL        )
  , ('U027', 'F', '2001-07-10', '2016-10-02', 'pc' , NULL        )
  , ('U028', 'M', '1976-05-26', '2016-10-02', 'app', NULL        )
  , ('U029', 'M', '1964-04-06', '2016-10-02', 'pc' , NULL        )
  , ('U030', 'M', '1959-10-07', '2016-10-02', 'sp' , NULL        )

;

DROP TABLE IF EXISTS action_log;
CREATE TABLE action_log(
    session  varchar(255)
  , user_id  varchar(255)
  , action   varchar(255)
  , stamp    varchar(255)
);

INSERT INTO action_log
VALUES
    ('989004ea', 'U001', 'view'   ,'2016-10-01 18:00:00')
  , ('989004ea', 'U001', 'view'   ,'2016-10-02 18:01:00')
  , ('989004ea', 'U001', 'view'   ,'2016-10-03 18:01:00')
  , ('989004ea', 'U001', 'view'   ,'2016-10-04 18:01:00')
  , ('989004ea', 'U001', 'view'   ,'2016-10-05 18:01:00')
  , ('989004ea', 'U001', 'view'   ,'2016-10-06 18:10:00')
  , ('47db0370', 'U001', 'follow' ,'2016-10-05 19:00:00')
  , ('47db0370', 'U001', 'view'   ,'2016-10-07 19:10:00')
  , ('47db0370', 'U001', 'follow' ,'2016-10-06 20:30:00')
  , ('5asfv583', 'U001', 'follow' ,'2016-10-20 19:00:00')
  , ('5asfv583', 'U001', 'view'   ,'2016-10-20 19:10:00')
  , ('5asfv583', 'U001', 'follow' ,'2016-10-21 20:30:00')
  , ('536jdqk2', 'U001', 'view'   ,'2016-12-22 19:00:00')
  , ('1gs7jacx', 'U001', 'view'   ,'2017-01-23 19:00:00')
  , ('87b5725f', 'U002', 'follow' ,'2016-10-01 12:00:00')
  , ('87b5725f', 'U002', 'follow' ,'2016-10-01 12:01:00')
  , ('87b5725f', 'U002', 'follow' ,'2016-10-02 12:02:00')
  , ('9afaf87c', 'U002', 'view'   ,'2016-10-02 13:00:00')
  , ('9afaf87c', 'U002', 'comment','2016-10-02 15:00:00')
  , ('afsd4bag', 'U002', 'view'   ,'2016-10-10 15:00:00')
  , ('ga43q56a', 'U003', 'view'   ,'2016-10-11 12:00:00')
  , ('854jcq5m', 'U004', 'view'   ,'2016-10-12 12:00:00')
;
```



```sql
WITH repeat_interval(index_name,interval_begin,interval_end) AS(  
     VALUES ('07 day rentation',1,7),
            ('14 day rentation',8,14),
            ('21 day rentation',15,22),
            ('28 day rentation',22,28))
      ,action_log_mst AS(
       SELECT m.user_id,
              r.index_name,
              CAST(m.register_date AS DATE) AS register_date,
              CAST(a.stamp AS DATE) AS action_date,
              MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
              CAST(m.register_date AS DATE)+'1 day'::interval*r.interval_begin AS index_begin_date,
              CAST(m.register_date AS DATE)+'1 day'::interval*r.interval_end AS index_end_date
              FROM mst_user AS m
                  LEFT OUTER JOIN action_log AS a
                       ON m.user_id=a.user_id
                       CROSS JOIN repeat_interval AS r)
      ,user_action_flag AS(
      SELECT user_id,
             index_name,
             register_date,
             SIGN(SUM(
             CASE WHEN index_end_date<latest_date THEN
                 CASE WHEN action_date BETWEEN index_begin_date AND index_end_date THEN 1 ELSE 0 END
                 END)) AS index_date_action
             FROM action_log_mst
             GROUP BY user_id,register_date,index_name,index_begin_date,index_end_date
             )
       SELECT  register_date,
               index_name,
               AVG(100*index_date_action) AS index_rate
               FROM user_action_flag 
               GROUP BY register_date,index_name
               ORDER BY register_date,index_name;
 ```
 #### 4.5 Displaying factors affecting customer churn and rentation rate for each factor
 From the last section,  we can easily see the trend of retention rate across the given interval.  However, there is an extra measure to investigate on. That is which part of the services that customers have used mostly contributed to the movement of retention rate. To answer this question, we need to display all the activities they have done and individual relation rate. 

The final result will include

activity name
the total number of users for the service
the rate of usage 
the retention rate of users  one day after initial sign up(you can adjust the date at your disposal) 
the retention rate of non-users one day after registration
(the percentage of people who has not kept using the service from the first sign up )

With common sense, it is so reasonable to say that the greater  retention rate of users but the lower of non-users, the greater it can influence the whole preservation rate. 

  ```sql
WITH repeat_interval(index_name,interval_begin_date,interval_end_date) AS(
VALUES ('01 day report',1,1))
,action_log_index_date AS(
SELECT u.user_id,
       u.register_date,
       CAST(a.stamp AS DATE) AS action_date,
       MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
       r.index_name,
       CAST(u.register_date AS DATE)+'1 day'::INTERVAL*r.interval_begin_date AS begin_date,
       CAST(u.register_date AS DATE)+'1 day'::INTERVAL*r.interval_end_date AS end_date
       FROM mst_users AS u
            LEFT OUTER JOIN action_log AS a
                  ON u.user_id=a.user_id
                  CROSS JOIN repeat_interval AS r)
        , user_action_flag AS(
        SELECT user_id, 
               register_date,
               index_name,
               SIGN(SUM(
                    CASE WHEN end_date<=latest_date THEN
                         CASE WHEN action_date BETWEEN begin_date AND end_date THEN 1 ELSE 0 END 
                         END)) AS index_date_action
               FROM action_log_index_date 
               GROUP BY user_id,register_date,index_name,begin_date,end_date)
         , mst_actions  AS(
           SELECT 'view' AS action
           UNION ALL SELECT 'comment' AS action
           UNION ALL SELECT 'follow' AS action)
         , mst_user_actions AS( -- create the first three columns of the table
           SELECT u.user_id,
                  u.register_date,
                  a.action
                  FROM mst_users AS u
                       CROSS JOIN mst_actions AS a)
          ,register_action_flag AS(
          SELECT DISTINCT m.user_id, 
                          m.register_date,
                          m.action,
                          CASE WHEN a.action IS NOT NULL THEN 1 ELSE 0 END AS do_action, 
                          f.index_name,
                          f.index_date_action
                          FROM mst_user_actions AS m
                          LEFT JOIN action_log AS a
                          ON m.user_id=a.user_id
                             AND CAST(m.register_date AS DATE)=CAST(a.stamp AS DATE)
                             AND m.action=a.action
                           LEFT JOIN user_action_flag AS f
                           ON m.user_id=f.user_id
                           WHERE f.index_date_action IS NOT NULL)--remove data with latest date
                      SELECT action,  
                             COUNT(1) AS users,--the total number of each action
                             AVG(100.0* do_action) AS usage_rate,--the total usage rate of each action
                             index_name,
                             --If user do action one day after initial register, cacluate its rate
                             AVG(CASE WHEN do_action=1 THEN 100*index_date_action END) AS idx_date,
                             AVG(CASE WHEN do_action=0 THEN 100*index_date_action END) AS no_idx_date
                             FROM register_action_flag
                             GROUP BY index_name,action;
                             
```

![image](https://user-images.githubusercontent.com/53164959/64795320-df62be80-d5b8-11e9-920e-d6a894f5cb77.png)


