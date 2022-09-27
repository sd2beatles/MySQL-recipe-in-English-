 CHAPTER 9 Reporting Time Based Data
 =========================
 
#### 10.1 Aggregation Function on Multiple Rows

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


#### 10.2 Moving Average : A Indicator of Trend 

There are many types of data reporting a seasonal increase in sales in a weekend or during the specified interval of a year. 
This seasonal 'noise' could adversely affect our critical insight into the trend of business performance. Therefore, a seasonal average comes in handy to help data analysts the track of sale's movement. In our example, we use 'seven-day moving average' .


_Why do we need to maintain two separate sections, seven_day_avg and Seven_day_avg_specific?_

Seven_day_avg is computed out when you can not obtain the full track of records for seven days. This output is measured in sales up
to the first six days. However, if you restrict the number of dates to 7 days and calculate the average, then use seven_day_strict. 
 
 ```sql
 
SELECT dt, 
       AVG(SUM(purchase_amount)) AS total_amount,
       AVG(SUM(purchase_amount)) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS seven_day_avg,
       CASE 
          WHEN 7=COUNT(*) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) 
          THEN 
          AVG(SUM(purchase_amount)) OVER(ORDER BY dt ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
          END AS seven_day_strict
		FROM purchase_log
        GROUP BY dt
        ORDER BY dt;
 ```
 
 #### 10.3 Cummulate Sum 
 
 
 ![image](https://user-images.githubusercontent.com/53164959/192448584-b6fa2b8d-0d38-46d3-a715-49ae99dce196.png)

 
 case 1) daily cumulate sum 
 
 In practice, we are often asked to prepare a separate section indicating the accumulated sum as well as the total amount daily.
 In this simple example, there are four sections consisting of 
 
 - dt : date
 - year_month : year/month
 - total_amount : the total revenue earned on each date
 - agg_amount : the revene earned up to the date of interest
 
 
 ```sql 
 SELECT dt, 
       SUBSTRING_INDEX(dt,'-',1) AS year,
       SUBSTR(dt,6,3) AS month,
       SUBSTRING_INDEX(dt,'-',-1) AS date,
       SUM(purchase_amount) AS total_amount,
       SUM(SUM(purchase_amount)) OVER(ORDER BY dt ROWS BETWEEN UNBOUNDED PRECEDING) AS agg_amount
       FROM  purchase_log
       GROUP BY dt
       ORDER BY dt;
 ```
Alternatively, it would enhance the easiness of reading the data if we want to split date-format data into the year, month, and date fields.

case 2) monthly cumulate sum

Let's record the total revenue generated in each month in the section of purchase_amount and its aggregating sum in agg_amount. 


```sql
WITH temp_purchase AS(
     SELECT dt,
            SUBSTRING_INDEX(dt,'-',1) AS year,
			SUBSTR(dt,6,2) AS month,
            SUBSTRING_INDEX(dt,'-',-1) AS date,
            SUM(purchase_amount) AS purchase_amount
	FROM purchase_log
    GROUP BY dt
    ORDER BY dt
    )
    SELECT dt,
           concat(year,'-',month) AS 'year_month',
           purchase_amount,
           SUM(purchase_amount) OVER(PARTITION BY year,month ORDER BY  dt ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
           AS agg_amount
	FROM temp_purchase
    ORDER BY dt;
```
- sepcial Note

If you take a closer examination on the code above,   the integral information is first set up and stored in 'temp_purchase' with the help of WITH AS funcion and reused in the later code.  This practice may slow the performance of the data-process. However, in the custom of big data management, the clarity the code line gives readers should outweigh the level of its performance. 


#### 10.4 Comparison of Sales on Year Base  
It is worthy of comparing the sales made in the specific month of a year to that in the same month of the previous year.

```sql
DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log(
    dt              varchar(255)
  , order_id        integer
  , user_id         varchar(255)
  , purchase_amount integer
);

INSERT INTO purchase_log
VALUES
    ('2014-01-01',    1, 'rhwpvvitou', 13900)
  , ('2014-01-01',    1, 'rhwpvvitou', 13900)
  , ('2014-01-03',    1, 'rhwpvvitou', 13900)
  ,('2014-01-03',    1, 'rhwpvvitou', 13900)
  , ('2014-02-08',   95, 'chtanrqtzj', 28469)
  ,('2014-02-08',   95, 'chtanrqtzj', 28469)
  ,('2014-02-28',   95, 'chtanrqtzj', 28469)
  ,('2014-02-28',   95, 'chtanrqtzj', 28469)
  , ('2014-03-09',  168, 'bcqgtwxdgq', 18899)
  , ('2014-04-11',  250, 'kdjyplrxtk', 12394)
  , ('2014-05-11',  325, 'pgnjnnapsc',  2282)
  , ('2014-06-12',  400, 'iztgctnnlh', 10180)
  , ('2014-07-11',  475, 'eucjmxvjkj',  4027)
  , ('2014-08-10',  550, 'fqwvlvndef',  6243)
  , ('2014-09-10',  625, 'mhwhxfxrxq',  3832)
  , ('2014-10-11',  700, 'wyrgiyvaia',  6716)
  , ('2014-11-10',  775, 'cwpdvmhhwh', 16444)
  , ('2014-12-10',  850, 'eqeaqvixkf', 29199)
  , ('2015-01-09',  925, 'efmclayfnr', 22111)
  , ('2015-02-10', 1000, 'qnebafrkco', 11965)
  , ('2015-03-12', 1075, 'gsvqniykgx', 20215)
  , ('2015-04-12', 1150, 'ayzvjvnocm', 11792)
  , ('2015-05-13', 1225, 'knhevkibbp', 18087)
  , ('2015-06-10', 1291, 'wxhxmzqxuw', 18859)
  , ('2015-07-10', 1366, 'krrcpumtzb', 14919)
  , ('2015-08-08', 1441, 'lpglkecvsl', 12906)
  , ('2015-09-07', 1516, 'mgtlsfgfbj',  5696)
  , ('2015-10-07', 1591, 'trgjscaajt', 13398)
  , ('2015-11-06', 1666, 'ccfbjyeqrb',  6213)
  , ('2015-12-05', 1741, 'onooskbtzp', 26024)
;
```

- LEFT JOIN apporach 

```sql
WITH temp_purchase AS(
      SELECT dt, 
      SUBSTRING_INDEX(dt,'-',1) AS year,
      SUBSTR(dt,6,2) AS month,
      SUBSTRING_INDEX(dt,'-',-1) AS date,
      SUM(purchase_amount) AS purchase_amount
      FROM purchase_log
      GROUP BY dt
      ORDER by dt),
      
      temp_2014 AS(SELECT
      dt, SUM(CASE WHEN year='2014' THEN purchase_amount END) AS amount_2014
      FROM temp_purchase
      GROUP BY month),
      temp_2015 AS(SELECT
      dt, SUM(CASE WHEN year='2015' THEN purchase_amount END) AS amount_2015
      FROM temp_purchase
      GROUP BY month)
      SELECT p.month,t1.amount_2014 ,t2.amount_2015
      FROM temp_purchase AS p
      LEFT JOIN temp_2014 AS t1
      ON p.dt=t1.dt
      LEFT JOIN temp_2015 AS t2
      ON p.dt=t2.dt
      GROUP BY p.month
      ORDER BY p.month;

```
However, this approach should put any users into the problem in reading the codes. 
Therefore,we need to explode another method to simplify it. 

- Without LEFT JOIN approach

```sql
WITH temp_purchase AS(
      SELECT dt, 
      SUBSTRING_INDEX(dt,'-',1) AS year,
      SUBSTR(dt,6,2) AS month,
      SUBSTRING_INDEX(dt,'-',-1) AS date,
      SUM(purchase_amount) AS purchase_amount
      FROM purchase_log
      GROUP BY dt
      ORDER by dt)
      SELECT month,
             SUM(CASE year WHEN  '2014' THEN purchase_amount END) AS amount_2014,
             SUM(CASE year WHEN  '2015' THEN purchase_amount END) AS  amount_2015,
             ROUND(100*SUM(CASE year WHEN '2015' THEN purchase_amount END)/
                 SUM(CASE year WHEN '2014' THEN purchase_amount END),2) AS rate
	  FROM temp_purchase
      GROUP BY month
      ORDER BY month;
```
#### 10.5 Z-chart Analysis

![z_chart](https://user-images.githubusercontent.com/53164959/62454270-f8ed3b00-b7ae-11e9-9800-ad4fc7e4143d.png)

Showing progress over time of either a small project or a whole business can result in a variety of charts to help the business judgment easier. Z can at least three measures into one chart. 

(1) The omponents of Z-chart are 

- Monthly sale: the total amount of sales is recorded each month.
- Aggregated sales: Sales are calculated in a cumulate sum  up to the previous month.
- Moving Sum:  Sum is made based on the last 11 months and a month of interest. 

 (2) Considerations in analyzing z-chart

First, pay attention to the shape of a line,indicating the cumulative sum.
If the line is sloping upward, then there is no noticeable fluctuation in sales over time. However, a curve to the top of the left should give an alarm of a decrease in revenue whereas a curve to the bottom of the right is a opposite sign. 


Secondly, the line of moving sum over time is a horizontal line, which represents constancy in sales. If the line slopes upward, this is a positive sign while if it slopes downward, business managers should be alert in a decrease in sales. 

(3) MySQL codes 

```sql
WITH daily_purchase AS(
      SELECT dt, 
      SUBSTRING_INDEX(dt,'-',1) AS year,
      SUBSTR(dt,6,2) AS month,
      SUBSTRING_INDEX(dt,'-',-1) AS date,
      SUM(purchase_amount) AS purchase_amount
      FROM purchase_log
      GROUP BY dt
      ORDER by dt),
		monthly_purchase AS
        (SELECT year,
                month,
				SUM(purchase_amount) AS amount
		 FROM daily_purchase
         GROUP BY year,month),
         cal_index AS(
         SELECT
         year,
         month,
         amount, #monthly sales
         SUM(CASE  WHEN year= '2015' THEN amount END) 
		     OVER(ORDER BY year,month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS agg_amount,
		 SUM(amount) OVER(ORDER BY year,month ROWS BETWEEN 11 PRECEDING AND CURRENT ROW) AS year_avg_amount
         FROM monthly_purchase
         ORDER BY year,month
         )
         SELECT concat(year,'-',month) AS year_months,
         amount,
         agg_amount,
         year_avg_amount
         FROM cal_index
         WHERE year='2015'
	 ORDER BY year_month;
```

### 10.6 Key Points in Analysing Sales

We should not reach a conclusion on the performance of business solely based on sales record bacause there must be some reason behind it. Therefore, it is a good practice to attach some supporting numbers as factors leading to the result of sale performance. For example,  the financial performance of the shopping mall has shown a steady decrease in revenue earned in a certain year. You could possibly suggest the frequency of purchases made by customers and the average expenditure by him or her: a steady in the level of frequency but a steady decrease in average spending.  By using these 'aids', you now infer that a decrease in spending is playing a major determinant of the poor performance in sales. 

```sql
SELECT * FROM purchase_log limit 3;
WITH daily_purchase AS(
 SELECT dt,
 order_id as orders,
 SUBSTRING_INDEX(dt,'-',1) AS year,
 SUBSTR(dt,6,2) AS month,
 SUBSTRING_INDEX(dt,'-',-1) AS date,
 SUM(purchase_amount) AS purchase_amount
 FROM purchase_log
 GROUP BY dt
 ORDER BY dt)
 ,monthly_purchase AS(
 SELECT year,
        month,
        SUM(orders) AS orders,
        AVG(purchase_amount) AS avg_amount, #average_month_expenditure
	    SUM(purchase_amount) AS monthly #total_month_expenditure 
        FROM daily_purchase
        GROUP BY year,month)
SELECT CONCAT(year,'-',month) AS year_months,
        orders,
        avg_amount,
        monthly,
        SUM(monthly) OVER(PARTITION BY year ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        AS agg_amount,
        LAG(monthly,12) OVER(ORDER BY year,month) AS last_year,
        100.0*monthly/LAG(monthly,12) OVER(ORDER BY year,month) AS rate
        FROM monthly_purchase
        ORDER BY year_months;
```

#### 10.7 Case Study (Penguin Birth and Death)

![image](https://user-images.githubusercontent.com/53164959/63220322-adc82480-c1bf-11e9-8aa5-8c214fabb99e.png)

```sql

WITH stats AS(
SELECT SUBSTRING(quarter,1,4) AS year,
       SUBSTRING(quarter,5,2) AS quart,
       male_birth,
       female_birth,
       male_death,
       female_death
       FROM birth_death),
     cum_sum AS(
     SELECT year,
            quart,
            male_birth,
            SUM(male_birth) OVER(PARTITION BY year)  AS annual_sum_male_birth,
            ROUND(100*CAST(male_birth AS numeric) /SUM(male_birth) OVER(PARTITION BY year),2) AS local_male_birth_rate,
            LAG(male_birth,4) OVER(ORDER BY year,quart) AS last_year_male_birth,
            female_birth,
            SUM(female_birth) OVER(PARTITION BY year)  AS annual_sum_female_birth,
            ROUND(100*CAST(female_birth AS numeric)/SUM(female_birth) OVER(PARTITION BY year),2) AS local_female_birth_rate,
            LAG(female_birth,4) OVER(ORDER BY year,quart) AS last_year_female_birth,
            male_death,
            SUM(male_death) OVER(PARTITION BY year)  AS annual_sum_male_death,
            ROUND(100*CAST(male_death AS numeric)/SUM(female_birth) OVER(PARTITION BY year),2) AS local_male_death_rate,
            LAG(male_death,4) OVER(ORDER BY year,quart) AS last_year_male_death,
            female_death,
            SUM(female_death) OVER(PARTITION BY year) AS annual_sum_female_death,
            ROUND(100*CAST(female_death AS numeric)/SUM(female_death) OVER(PARTITION BY year),2) AS local_female_death_rate,
            LAG(female_death,4) OVER(ORDER BY year,quart) AS last_year_female_death
            FROM stats
            )
            SELECT CONCAT(year||quart) AS year_quart,
                   male_birth,
                   annual_sum_male_birth,
                   local_male_birth_rate,
                   last_year_male_birth,
                   ROUND(100*CAST(male_birth AS numeric)/last_year_male_birth,2) AS year_over_year_rate,
                   female_birth,
                   annual_sum_female_birth,
                   local_female_birth_rate,
                   last_year_female_birth,
                   ROUND(100*CAST(female_birth AS numeric)/last_year_female_birth,2) AS year_over_year_rate,
                   male_death,
                   annual_sum_male_death,
                   local_male_death_rate,
                   last_year_male_death,
                   ROUND(100*CAST(male_death AS numeric)/last_year_male_death,2) AS year_over_year_rate,
                   female_death,
                   annual_sum_female_death,
                   local_female_death_rate,
                   last_year_female_death,
                   ROUND(100*CAST(female_death AS numeric)/last_year_female_death,2) AS year_over_year_rate
                FROM cum_sum;
    
     --postgreSQL does not support round(double precision,integer). For this reason, we should cast the
     --rate into numberic then round it to any decimal points you wish for.
```






