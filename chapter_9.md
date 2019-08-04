### Chapter 9 Reporting Time Based Data

### 1) Aggregation Function on Multiple Rows

we are now intrested in aggregating multiple rows to calculate the total number of purchases, the amount in total, and the avergage of the 
spending made on each date. 

```MySQL
DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log(
    dt              varchar(255)
  , order_id        integer
  , user_id         varchar(255)
  , purchase_amount integer
);

INSERT INTO purchase_log
VALUES
    ('2014-01-01',  1, 'rhwpvvitou', 13900)
  , ('2014-01-01',  2, 'hqnwoamzic', 10616)
  , ('2014-01-02',  3, 'tzlmqryunr', 21156)
  , ('2014-01-02',  4, 'wkmqqwbyai', 14893)
  , ('2014-01-03',  5, 'ciecbedwbq', 13054)
  , ('2014-01-03',  6, 'svgnbqsagx', 24384)
  , ('2014-01-03',  7, 'dfgqftdocu', 15591)
  , ('2014-01-04',  8, 'sbgqlzkvyn',  3025)
  , ('2014-01-04',  9, 'lbedmngbol', 24215)
  , ('2014-01-04', 10, 'itlvssbsgx',  2059)
  , ('2014-01-05', 11, 'jqcmmguhik',  4235)
  , ('2014-01-05', 12, 'jgotcrfeyn', 28013)
  , ('2014-01-05', 13, 'pgeojzoshx', 16008)
  , ('2014-01-06', 14, 'msjberhxnx',  1980)
  , ('2014-01-06', 15, 'tlhbolohte', 23494)
  , ('2014-01-06', 16, 'gbchhkcotf',  3966)
  , ('2014-01-07', 17, 'zfmbpvpzvu', 28159)
  , ('2014-01-07', 18, 'yauwzpaxtx',  8715)
  , ('2014-01-07', 19, 'uyqboqfgex', 10805)
  , ('2014-01-08', 20, 'hiqdkrzcpq',  3462)
  , ('2014-01-08', 21, 'zosbvlylpv', 13999)
  , ('2014-01-08', 22, 'bwfbchzgnl',  2299)
  , ('2014-01-09', 23, 'zzgauelgrt', 16475)
  , ('2014-01-09', 24, 'qrzfcwecge',  6469)
  , ('2014-01-10', 25, 'njbpsrvvcq', 16584)
  , ('2014-01-10', 26, 'cyxfgumkst', 11339)
;



SELECT DISTINCT dt,
       COUNT(*) AS purchase_count,
	   SUM(purchase_amount) AS total_amount,
       AVG(purchase_amount) AS avg_amount
       FROM purchase_log
       GROUP BY dt
       ORDER by dt;
```


Without taking a consideration of who made a purchse on the specific date, our focus is placed on the sum or average of epxpeditures 
on evry date in a given interval. Then, using a COUNT() function will do for the case. 


### 2) Moving Average : A Indator of Trend 

There are many types of data reporting a seasonal increase in sales in a weekend or during the specified interval of a year. 
This seasonal 'noise' could adversely affect our critical insight into the trend of business performance. Therefore, a seasonal average comes in handy to help data analysts the track of sale's movement. In our example, we use 'seven-day moving average' .


_Why do we need to maintain two separate sections, seven_day_avg and Seven_day_avg_specific?_

Seven_day_avg is computed out when you can not obtain the full track of records for seven days. This output is measured in sales up
to the first six days. However, if you restrict the number of dates to 7 days and calculate the average, then use seven_day_strict. 
 
 ```sql
 
SELECT dt, 
       SUM(purchase_amount) AS total_amount,
       AVG(purchase_amount) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS seven_day_avg,
       CASE 
          WHEN 7=COUNT(*) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) 
          THEN 
          AVG(purchase_amount) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
          END AS seven_day_strict
		FROM purchase_log
        GROUP BY dt
        ORDER BY dt;
 ```
 
 ### 3) Cummulate Sum 
 
 In practice, we are often asked to prepare a separate section indicating the accumulated sum as well as the total amount daily.
 In this simple example, there are four sections consisting of 
 
 - dt : date
 - year_month : year/month
 - total_amount : the total revenue earned on each date
 - agg_amount : the revene earned up to the date
 
 
 ```sql 
 SELECT dt, 
       SUBSTRING_INDEX(dt,'-',2) AS date_record,
       SUM(purchase_amount) AS total_amount,
       SUM(SUM(purchase_amount)) OVER(ORDER BY dt ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS agg_amount
       FROM  purchase_log
       GROUP BY dt
       ORDER BY dt;
 ```
Alternatively, it would enhance the easiness of reading the data if we want to split date-format data into the year, month, and date fields.


