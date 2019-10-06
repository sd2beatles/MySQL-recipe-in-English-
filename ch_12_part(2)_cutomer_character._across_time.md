### 4 Consistent Date by number of uses


 4.1 Introduction 

From the last section, we have provided the retention rate for each service customer has used for the given period. 
If further divided into the various levels, each group could give us a view of which group of the service largely contribute to 
the higher customer churn.  

4.2 Method

There is 



We will collect the data for each level of activity during the past seven days and based on it.
Our attention is manly on what is the 14-day-retention rate for each sub-group.
The reason behind this business technique is to find out how the past event could affect the following week's retention. 




```sql
WITH repeat_interval(index_name,interval_begin_date,interval_end_date) AS(
    VALUES('14 day rentation',8,14))
    ,action_log_with_index_date AS(
    SELECT u.user_id,
           CAST(u.register_date AS DATE) AS register_date,
           r.index_name,
           CAST(a.stamp AS DATE) AS action_date,
           MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
           CAST(u.register_date AS DATE)+'1 day'::interval*r.interval_begin_date AS begin_date,
           CAST(u.register_date AS DATE)+'1 day'::interval*r.interval_end_date AS end_date
           FROM mst_user AS u
              LEFT OUTER JOIN action_log AS a
              ON u.user_id=a.user_id
              CROSS JOIN repeat_interval AS r)
       ,user_action_flag AS(
       SELECT user_id,
              register_date,
              index_name,
              SIGN(SUM(
                  CASE WHEN end_date <=latest_date THEN
                       CASE WHEN action_date BETWEEN begin_date AND end_date THEN 1 ELSE 0 
                       END END)) AS index_date_action
              FROM action_log_with_index_date
              GROUP BY user_id,register_date,index_name
              ORDER BY user_id,register_date,index_name)
         ,mst_action_bucket(action,min_count,max_count) AS(
         VALUES ('view',0,0),
                ('view',1,4),
                ('view',5,8),
                ('follow',0,0),
                ('follow',1,3),
                ('follow',4,6),
                ('comment',0,0),
                ('comment',1,3),
                ('comment',4,6))
         ,mst_user_bucket AS(
         SELECT u.user_id, 
                u.register_date,
                a.action,
                a.min_count,
                a.max_count
                FROM mst_users AS u
                CROSS JOIN mst_action_bucket AS a)
         ,register_action_flag AS(
          SELECT m.user_id,
                 m.action,
                 m.min_count,
                 m.max_count,
                 COUNT(a.action) AS action_count, --recording the level of each activity for the past 7 days.
                 CASE WHEN COUNT(m.action) BETWEEN m.min_count AND m.max_count
                      THEN 1 ELSE 0 END AS achieve,
                 index_name,
                 index_date_action
                 FROM mst_user_bucket AS m
                 LEFT JOIN action_log AS a
                      ON m.user_id=a.user_id
                         --restricting the data based on the past 7days
                         AND CAST(a.stamp AS DATE)  
                         BETWEEN CAST(m.register_date AS DATE) AND CAST(m.register_date AS DATE)+'7 day'::interval
                         AND m.action=a.action
                 LEFT JOIN user_action_flag AS f
                 ON m.user_id=f.user_id
                 WHERE F.index_date_action IS NOT NULL
                 GROUP BY m.user_id,
                          m.action,
                          m.min_count,
                          m.max_count,
                          f.index_name,
                          f.index_date_action)
              SELECT action,
                     min_count||'~'||max_count AS count_range,
                     SUM(CASE WHEN achieve=1 then 1 ELSE 0 END) AS achieve,
                     index_name,
                     AVG(CASE WHEN achieve=1 THEN 100*index_date_action END) as achieve_index_rate 
                     FROM register_action_flag
                     GROUP BY index_name,action,min_count,max_count
                     ;
```
![image](https://user-images.githubusercontent.com/53164959/64873763-c0c9f980-d684-11e9-9d66-5752f91f8772.png)

5. Consistent Rate by daily uses

5.1 Introductiion

We are sometimes curious about how the first nth of days could affect the long-term usage rate. The result is highly uncertain in terms of business we are currently analyzing. For example, if you are currently on the analysis of social network services, the longer period of the first few days could likely lead to a higher possibility of a consistent rate in the future date. However, this is not always a case.  We need to prepare a table showing the first assigned dates and 'settlement' rate after some specific dates later.  

 ```sql
WITH repeat_interval(index_name,interval_begin_date,interval_end_date) AS(
    VALUES('28_day_rentation',22,28))
    ,action_log_with_date AS(
     SELECT u.user_id,
            CAST(u.register_date AS DATE) AS register_date,
            r.index_name,
            CAST(a.stamp AS DATE) AS action_date,
            MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
            CAST(u.register_date AS DATE)+'1 day'::interval*r.interval_begin_date AS begin_date,
            CAST(u.register_date AS DATE)+'1 day'::interval*r.interval_end_date AS end_date
            FROM mst_user AS u
                 LEFT OUTER JOIN action_log AS a
                 ON u.user_id=a.user_id
                 CROSS JOIN repeat_interval AS r)
     ,user_action_flag AS(
     --count 
     SELECT user_id,
            register_date,
            index_name,
            SIGN(SUM(
                 CASE WHEN end_date<latest_date THEN
                 CASE WHEN action_date BETWEEN begin_date AND end_date THEN 1 ELSE 0 END END)) AS index_date_action
            FROM action_log_with_date
            GROUP BY user_id,register_date,index_name)
    ,register_action_flag AS(
     SELECT m.user_id,
            --here,we want to track out how many consecutive days user keep log-in for during the first 8 days
            COUNT(DISTINCT CAST(a.stamp AS DATE)) AS dt_count
            --From now on, we wnat to show information in the period between 22 and 28 days after registration
            ,f.index_name
            ,f.index_date_action
            FROM mst_users AS m
                LEFT OUTER JOIN action_log AS a
                ON m.user_id=a.user_id
                   AND CAST(a.stamp AS DATE) BETWEEN 
                       CAST(m.register_date AS DATE)+'1 day'::interval AND CAST(m.register_date AS DATE)+'7 day'::interval
                 LEFT JOIN user_action_flag AS f
                      ON m.user_id=f.user_id
                 WHERE f.index_date_action IS NOT NULL
                 GROUP BY m.user_id,
                          f.index_name,
                          f.index_date_action)
            SELECT dt_count AS dates, -- number of days from 1 to 7
                   COUNT(user_id) AS users,-- number of users who keep using services from day 1 to day 7
                   100*COUNT(user_id)/SUM(COUNT(user_id)) OVER() AS user_ratio,
                   100*SUM(COUNT(user_id)) OVER(ORDER BY dt_count ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)/
                       SUM(COUNT(user_id)) OVER() AS cum_ratio,
                   --to create a column to indicate those who are using services up to somewhere between 22 and 28 days
                   SUM(index_date_action) AS achieve_users,
                   AVG(100*index_date_action) AS achieve_ratio
                   FROM register_action_flag
                        GROUP BY dt_count
                        ORDER by dt_count;
  ```
  
