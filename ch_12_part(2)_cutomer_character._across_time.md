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
  
#### Practical Test
```sql

WITH modified_data AS(
 SELECT user_id,
        CAST(date AS DATE) AS action_date,
        CAST(MIN(date) OVER(PARTITION BY user_id) AS DATE)  AS register_date
        FROM retail
        GROUP BY user_id,date)
   ,repeat_interval (index_name,interval_begin,interval_end) AS(
    VALUES('12-36 months rentation',12,36))
   ,action_log_with_date AS(
    SELECT m.user_id,
           m.action_date,
           m.register_date,
           m.register_date+'1 month'::interval*r.interval_begin AS begin_date,
           m.register_date+'1 month'::interval*r.interval_end AS end_date,
           MAX(m.action_date) OVER() AS latest_date,
           r.index_name
           FROM modified_data AS m
           CROSS JOIN repeat_interval AS r)
    ,user_action_flag AS(
    SELECT user_id, 
           register_date,
           index_name,
           SIGN(SUM(CASE WHEN end_date<=latest_date THEN
                         CASE WHEN action_date BETWEEN begin_date AND end_date THEN 1 ELSE 0 END END)) AS index_date_action
           FROM action_log_with_date
           GROUP BY user_id,register_date,index_name)
    ,register_action_flag AS(
    SELECT m.user_id,
           COUNT(DISTINCT(m.action_date)) AS dt_counts,
           u.index_name,
           u.index_date_action
           FROM modified_data AS m
           LEFT JOIN user_action_flag AS u
           ON m.user_id=u.user_id
           AND m.action_date BETWEEN m.register_date AND m.register_date+'12 month'::INTERVAL
           WHERE u.index_date_action IS NOT NULL
           GROUP BY m.user_id,
                    u.index_name,
                    u.index_date_action)
    SELECT dt_counts AS months, 
           COUNT(user_id) AS users,
           100*COUNT(user_id)/SUM(COUNT(user_id)) OVER() AS user_ration,
           100*SUM(COUNT(user_id)) OVER(ORDER BY dt_counts ROWs BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)/SUM(COUNT(user_id)) OVER() AS cum_ratio,
           SUM(index_date_action) AS achieve_users,
           AVG(100*index_date_action) AS achieve_ratio
           FROM register_action_flag
           WHERE dt_counts BETWEEN 1 AND 12
           GROUP by dt_counts
           ORDER by dt_counts;
           
 ```
 
 ### 5.2 MAU(Monthly Active Users) part1
 
 MAU stands for Monthly Active Users, commonly used to indicate the number of users for a given month.  The figure is meaningless unless this can be sub-divided into some categories for further analysis.  For example, 300 thousand for this month is the simple number that does not help us to read what is happening on the service of the company. To assess the health performance of  our comany and used to a basis of other metrics, the classificaiton is now necessary. 
  
Now, we will divide the whole number into three groups which are

new-comer: those who begin to start using our service from this month
revisitor:those who has kept using our service up to this month
comback user: those who has been away for a while but came back 

```sql
WITH monthly_user_action AS(
  SELECT m.user_id,
         SUBSTRING(m.register_date,1,7) AS register_month,
         SUBSTRING(a.stamp,1,7) AS action_month,
         SUBSTRING(CAST(CAST(a.stamp AS DATE)-'1 month'::INTERVAL AS TEXT),1,7) AS pre_month 
         FROM mst_users AS m
         LEFT JOIN action_log AS a
         ON m.user_id=a.user_id)
   ,monthly_user_with_type AS(
   SELECT --each users can use the service more than once in a given month.
          --To prevent double-counting, we should implement disetinct before user_id 
          DISTINCT user_id,
          action_month,
          CASE WHEN register_month=action_month THEN 'new_user'
               WHEN pre_month=LAG(action_month) OVER(PARTITION BY user_id ORDER BY action_month) THEN 'repeated_user'
               ELSE 'comeback_user' END AS c,
          pre_month
          FROM monthly_user_action
          WHERE action_month IS NOT NULL)
       SELECT action_month,
           COUNT(user_id),
           COUNT(CASE WHEN c='new_user' THEN 1 END) AS new_users,
           COUNT(CASE WHEN c='repeated_user' THEN 1 END) AS repeated_users,
           COUNT(CASE WHEN c='comeback_user' THEN 1 END) AS comeback_users
           FROM monthly_user_with_type 
           GROUP BY action_month;
```

The outcome is 

![image](https://user-images.githubusercontent.com/53164959/72214229-3796d680-3541-11ea-85e0-675e580ec16e.png)


### 5.3 (MAU part 2)

Our main interest is in the repeated group. To have much knowledge, we should move one step further to subdivide the group into three parts. The breif disccusion is following 

 * repeated_user(new_user) :Those who was classifed as newcomer in the previous month but stil use the service for the given month 
 * repeated_user(repeat_user):Tho
 
 
 ```sql
 WITH monthly_user_action AS(
  SELECT m.user_id,
         SUBSTRING(m.register_date,1,7) AS register_month,
         SUBSTRING(a.stamp,1,7) AS action_month,
         SUBSTRING(CAST(CAST(a.stamp AS DATE)-'1 month'::INTERVAL AS TEXT),1,7) AS pre_month 
         FROM mst_users AS m
         LEFT JOIN action_log AS a
         ON m.user_id=a.user_id)
   ,monthly_user_with_type AS(
   SELECT --each users can use the service more than once in a given month.
          --To prevent double-counting, we should implement disetinct before user_id 
          DISTINCT user_id,
          action_month,
          CASE WHEN register_month=action_month THEN 'new_user'
               WHEN pre_month=LAG(action_month) OVER(PARTITION BY user_id ORDER BY action_month) THEN 'repeated_user'
               ELSE 'comeback_user' END AS c,
          pre_month
          FROM monthly_user_action
          WHERE action_month IS NOT NULL)
     ,monthly_users AS(
     SELECT m1.action_month,
            COUNT(m1.user_id) AS mau,
            COUNT(CASE WHEN m1.c='new_user' THEN 1 END) AS new_users,
            COUNT(CASE WHEN m1.c='repeat_user' THEN 1 END) AS repeat_users,
            COUNT(CASE WHEN m1.c='comeback_user' THEN 1 END) AS comeback_users,
            COUNT(CASE WHEN m1.c='new_user' AND m0.c='new_user' THEN 1 ELSE 0 END) AS new_repeat_users,
            COUNT(CASE WHEN m1.c='repeat_user' AND m0.c='repeat_user' THEN 1 ELSE 0 END) AS continuous_users,
            COUNT(CASE WHEN m1.c='repeat_user' AND m0.c='comeback_user' THEN 1 ELSE 0 END) AS come_back_users
        FROM monthly_user_with_type AS m1
        LEFT JOIN monthly_user_with_type AS m0 
        ON m1.user_id=m0.user_id AND 
           m1.pre_month=m0.action_month
        GROUP BY m1.action_month)
      SELECT * FROM monthly_users;
  ```
  
  ![image](https://user-images.githubusercontent.com/53164959/72214448-729b0900-3545-11ea-962e-c285db478018.png)

5.3 MAU (3)
WITH monthly_user_action AS(
 SELECT m.user_id,
        SUBSTRING(m.register_date,1,7) AS register_month,
        SUBSTRING(a.stamp,1,7) AS action_month,
        SUBSTRING(CAST(CAST(a.stamp AS DATE)-'1 month'::INTERVAL AS TEXT),1,7) AS pre_month 
        FROM mst_users AS m
        LEFT JOIN action_log AS a
        ON m.user_id=a.user_id)
  ,monthly_user_with_type AS(
  SELECT --each users can use the service more than once in a given month.
         --To prevent double-counting, we should implement disetinct before user_id 
         DISTINCT user_id,
         action_month,
         CASE WHEN register_month=action_month THEN 'new_user'
              WHEN pre_month=LAG(action_month) OVER(PARTITION BY user_id ORDER BY action_month) THEN 'repeated_user'
              ELSE 'comeback_user' END AS c,
         pre_month
         FROM monthly_user_action
         WHERE action_month IS NOT NULL)
    ,monthly_users AS(
    SELECT m1.action_month,
           COUNT(m1.user_id) AS mau,
           COUNT(CASE WHEN m1.c='new_user' THEN 1 END) AS new_users,
           COUNT(CASE WHEN m1.c='repeat_user' THEN 1 END) AS repeat_users,
           COUNT(CASE WHEN m1.c='comeback_user' THEN 1 END) AS comeback_users,
           COUNT(CASE WHEN m1.c='new_user' AND m0.c='new_user' THEN 1 ELSE 0 END) AS new_repeat_users,
           COUNT(CASE WHEN m1.c='repeat_user' AND m0.c='repeat_user' THEN 1 ELSE 0 END) AS continuous_repeat_users,
           COUNT(CASE WHEN m1.c='repeat_user' AND m0.c='comeback_user' THEN 1 ELSE 0 END) AS come_back_repeat_users
       FROM monthly_user_with_type AS m1
       LEFT JOIN monthly_user_with_type AS m0 
       ON m1.user_id=m0.user_id AND 
          m1.pre_month=m0.action_month
       GROUP BY m1.action_month)
       SELECT *,
              --Labeled as the new_users in the previous month but repeated_users for this month
              100*new_repeat_users/NULLIF(LAG(new_users) OVER(ORDER BY action_month),0) AS priv_new_repeat_ratio,
              --Labeled as the repeated_users in the previous month and still classifed as the repeated
              100*continuous_repeat_users/NULLIF(LAG(repeat_users) OVER(ORDER BY action_month),0) AS priv_new_repeat_ratio,
              --Labeled as the comeback users in the previous month and classifed as repeated_comeback_users
              100*come_back_repeat_users/NULLIF(LAG(comeback_users) OVER(ORDER BY action_month),0) AS priv_come_back_ratio
              FROM monthly_users
        
  
