## CHAPTER 6 Manupulation of Various Values

### 1) Concatenating letters
The Section itself implies that I have tried to link all the given letters in an manner we want to present to end users. 
The 'concat; is used for linking letters and I have put ' ' for leaving a space betweeen pref_name and city_name. 

method 1) prepare the data
```mysql
DROP TABLE IF EXISTS mst_user_location;
CREATE TABLE mst_user_location
(userID  VARCHAR(7) NOT NULL PRIMARY KEY,
 pref_name CHAR(30) NOT NULL,
 city_name CHAR(30) NOT NULL);
 
INSERT INTO mst_user_location 
       VALUES('U001','42 Parkstone Ave','ChristChurch'),
             ('U002','12 Queenswood St','Auckland'),
             ('U003','14-A Queens St','Auckland');
```
method 2) concating the letters

```mysql
SELECT CONCAT(pref_name,' ',city_name) AS pref_city 
       FROM mst_user_location;
```

### 2) Comparing Values 

The table shows the sale of each quater from 2015 to 2017. Assuming there were no track records  in the 3 and 4 quarter of 2017, I have inserted NULL for these periods. As for 2018, all columns consist of NULL. 

```mywsql
CREATE TABLE quarterly_sales (
    year integer
  , q1   integer
  , q2   integer
  , q3   integer
  , q4   integer
);

INSERT INTO quarterly_sales
VALUES
    (2015, 82000, 83000, 78000, 83000)
  , (2016, 85000, 85000, 80000, 81000)
  , (2017, 92000, 81000, NULL ,NULL)
  ,(2018,NULL,NULL,NULL,NULL)
;

```

#### * Measuremnet for Sale's Movement
It is crucial for mangers to see there is any change in sale's revenue from thier company. Therefore, I have made 
there separate sections ,which make it more convenient and handy to conduct thier quarterly analysis. 

- _movment_q2_q1_   '-' : loss in sale's compared to previous quarter , '+'  gain , ' ' no change at all 

- _diff_q2_q1_   the name implies absolute value of the diffrence between two quarters. 

- _sign_q2_q1_  '-1' , loss in sale's compared to previous quarter , '1'  gain , ' 0' no change at all 

```mysql
SELECT year,q1,q2,
       CASE 
           WHEN q1<q2 THEN '+'
           WHEN q1=q2 THEN ' '
           ELSE '-'
           END AS movement_q2_q1
		,ABS(q2-q1) AS diff_q2_q1
        ,SIGN(q2-q1) AS sign_q2_q1
        FROM quarterly_sales
        ORDER BY year;
```

#### * Finding The Lowest and Greatest Sale among The Qauaters 

This taks took me a quite longer than What I had previously expected . This is primarly beacuse  MySQL is not currently providing
any functions which generates either lowest,greatest value among 'non-null' columns. Therefore, I come out with indirect ways to
get those values with the same codes repeated over and over. 

As for finding the greast value, since the sale can not be lower than zero, the value automatically returns zero if any column on the talbe contains zero by using 'COALESCE' ,which is followed by finding the greatest value among four non-Null values. Otherwise,'NULL' automatically would appear.


In obtaining the lowest values in each year,four cases are considered.
First, if all of the columns do not contain null value, then simply implment the least function where values are not affected by NULL.
For the  next two remaining cases, ifnull is employed to find the least value. If a case does not fit to any of them, simply return 'NULL'. 


```mysql
<MySQL version>
SELECT year,
     /* finding the greatest values*/
    case
    WHEN GREATEST(COALESCE(q1,0),COALESCE(q2,0),COALESCE(q3,0),COALESCE(q4,0))=0 THEN 'NULL'
    ELSE GREATEST(COALESCE(q1,0),COALESCE(q2,0),COALESCE(q3,0),COALESCE(q4,0)) 
    END AS greatest_value,
	
    /* obtaining the lowest values*/
    CASE
    WHEN (q1 is NOT NULL AND q2 is NOT NULL AND q3 IS NOT NULL AND q4 IS NOT NULL) THEN least(q1,q2,q3,q4)
    WHEN ifnull(ifnull(q1,q2),ifnull(q3,q4))=IFNULL(q1,q2) then least(q1,q2) 
    WHEN ifnull(ifnull(q1,q2),ifnull(q3,q4))=IFNULL(q3,q4) then least(q3,q4)
    ELSE 'NULL'
    END AS least_value
    from quarterly_sales
    ORDER BY year;
```

```sql
<postgreSQL version>

SELECT years, 
       GREATEST(q1,q2,q3,q4) AS  greatest,
       LEAST(q1,q2,q3,q4) AS least
       FROM quarterly_sales
       ORDER BY years;
```



#### * Finding Avaerage 

```sql
DROP TABLE IF EXISTS quarterly_sales;
CREATE TABLE quarterly_sales (
    years integer
  , q1   integer
  , q2   integer
  , q3   integer
  , q4   integer
);

INSERT INTO quarterly_sales
VALUES
    (2015, 82000, 83000, 78000, 83000)
  , (2016, 85000, 85000, 80000, 81000)
  , (2017, 92000, 81000, NULL ,NULL)
  ,(2018,NULL,NULL,NULL,NULL)
;

WITH stat_sales AS(
        SELECT years, 
        q1,
        q2,
        q3,
        q4,
       SIGN(COALESCE(q1,0))+SIGN(COALESCE(q2,0))+SIGN(COALESCE(q3,0))+SIGN(COALESCE(q4,0)) AS denominator
       FROM quarterly_sales)
       SELECT years, 
       CASE WHEN denominator>0 THEN (COALESCE(q1,0)+COALESCE(q2,0)+COALESCE(q3,0)+COALESCE(q4,0))/denominator 
            ELSE NULL  END AS avg
       FROM stat_sales;
```



### 3) Manipulation of Integer Data

```MySQL
CREATE TABLE advertising_stats (
    dt          varchar(255)
  , ad_id       varchar(255)
  , impressions integer
  , clicks      integer
);

INSERT INTO advertising_stats
VALUES
    ('2017-04-01', '001', 100000,  3000)
  , ('2017-04-01', '002', 120000,  1200)
  , ('2017-04-01', '003', 500000, 10000)
  , ('2017-04-02', '001',      0,     0)
  , ('2017-04-02', '002', 130000,  1400)
  , ('2017-04-02', '003', 620000, 15000)
;
Impressions represents the total number of advertisments made on each given data. 
Now, I will add two additional columns 'ctr' for the rate of actual clicks on the specific date and 'crt_as_percent'
that is measured in percentage with 2 decimal points.

SELECT dt,ad_id, ROUND(clicks/impressions,2) AS ctr,
	   100*(clicks/impressions) AS crt_as_percent 
       FROM advertising_stats
       ORDER BY dt,ad_id
       ;

```
<Special Note 1> It must be unnatural to assume that any number is divisible by zero.
The meritness of emplyoing MySQL is we don't need to take an extra measure to handle this issue.
However, if other queries are taken for this situation, a little trick should kick in to avoid an error message ; 

```
SELECT ROUND(clcks/NULLIF(impression,0),2) AS ctr, 100*(clicks/NULLIF(impression,0)) AS crt_as_percent 
       FROM advertising_stats
       ORDER BY dt,ad_id;
```       
<Special Note 2> NULLIF VS IFNULL
- NULLIF(exp1,exp2) if exp1 and exp2 are same to one another, it returns NULL. 
                  Otherwise,the firste expression is returned. 

- IFNULL(exp1,exp2) if exp1 is null,then it returns exp2 and if two values are all NULL,it returns NULL. 
                  Other than these two cases, it always return the first expression. 
                  
		  
### 4) Computation of Distance 
#### * Absolute Value and Squared Root 

```MySQL
DROP TABLE IF EXISTS location_1d;
CREATE TABLE location_1d (
    x1 integer
  , x2 integer
);

INSERT INTO location_1d
VALUES
    ( 5 , 10)
  , (10 ,  5)
  , (-2 ,  4)
  , ( 3 ,  3)
  , ( 0 ,  1)
;

SELECT ABS(X1-X2) AS abs,
       SQRT(POW(x1-x2,2)) AS sqrt
	   FROM location_1d;
```

#### * Eucleidan Distance 

```MySQL
DROP TABLE IF EXISTS location_2d;
CREATE TABLE location_2d (
    x1 integer
  , y1 integer
  , x2 integer
  , y2 integer
);

INSERT INTO location_2d
VALUES
    (0, 0, 2, 2)
  , (3, 5, 1, 2)
  , (5, 3, 2, 1)
;

SELECT ROUND(SQRT(POW(x1-x2,2)+POW(y1-y2,2)),2) AS dist
	   FROM location_2d;
```

### 5) Hanlding Time and Date Data

#### _Preapring unprocessed data_
```MySQL
DROP TABLE IF EXISTS mst_users_with_dates;
CREATE TABLE mst_users_with_dates (
    user_id        varchar(255)
  , register_stamp varchar(255)
  , birth_date     varchar(255)
);

INSERT INTO mst_users_with_dates
VALUES
    ('U001', '2016-02-28 10:00:00', '2000-02-29')
  , ('U002', '2016-02-29 10:00:00', '2000-02-29')
  , ('U003', '2016-03-01 10:00:00', '2000-02-29')
;
```

#### * Manipulation of Date and Time 

In order to substract from or add cetrain interval to the current data,   I have used the simple operators( + or - ) together with INTERVAL function. Unlike postGresql, MySQL dictates thatINTERVAL be followed by a desginated integer number. For exmaple,  INTERVAL 1 HOUR .

```MySQL
<MySQL version>
SELECT user_id,register_stamp,
       register_stamp+INTERVAL 1 HOUR AS after_1_hour,
       register_stamp-INTERVAL 30 minute AS before_30_minutes,
       SUBSTRING_INDEX(register_stamp,' ',1) AS register_date,
       SUBSTRING_INDEX(register_stamp,' ',1)+INTERVAL 1 DAY AS after_1_day,
       SUBSTRING_INDEX(register_stamp,' ',1)-INTERVAL 1 MONTH AS before_1_month,
       DATEDIFF(CURDATE(),register_stamp) AS diff_days
       FROM mst_users_with_dates;
```

```
<postgresql version>


#### * Age 
It seems to be trivial and struggling for data analysts to compute age based on date-typed data if they decide to use MySQL. Among many alternatives , I have taken TIMESTAMPDIFF.

```MySQL
SELECT user_id,DATE(NOW()) AS today,register_stamp,birth_date,
	   TIMESTAMPDIFF(YEAR,birth_date,CURDATE()) AS register_date,
       /* in order to extract the section of date from register_stamp,
	      DATE is used!*/
       TIMESTAMPDIFF(YEAR,birth_date,DATE(register_stamp)) AS current_age
       FROM mst_users_with_dates;

```

### 6) Management of IP address

Given the dotted-quad representations of IP address as a string, we are curious wheter one is more or less than another,or whether
IP address belongs to the data in the format 'adress/y'. Whatever cases they are, it is not a good idead  to make a judgement
based on 'string' data since the task itself is really inefficient and time-consuming. Therefore, I have employed INET_ATON.

#### * Inequality  

```MySQL
SELECT IF(INET_ATON('123.0.0.1') < INET_ATON('128.0.0.1'),'T','F') AS lt,
       IF(INET_ATON('123.0.0.1')> INET_ATON('123.0.0.1'),'T','F') AS gt;
 ```
 #### * Split an IP address into 4 respective octet
 
 In addition to the four sections of IP address, I pad the table with another column called ip_integer(ie an integer value of ip address).
 
```MySQL
DROP TABLE IF EXISTS iptbl;
CREATE TABLE iptbl
(ip CHAR(20) NOT NULL PRIMARY KEY);
INSERT INTO iptbl
       VALUES ('120.168.0.1'),
              ('179.123.0.2'),
			  ('156.233.0.0'),
              ('123.0.0.2');
              
SELECT ip,SUBSTRING_INDEX(ip,'.',1) AS ip_part1,
          SUBSTRING_INDEX(SUBSTRING_INDEX(ip,'.',-3),'.',1) AS ip_part2,
	      SUBSTRING_INDEX(SUBSTRING_INDEX(ip,'.',-2),'.',1) AS ip_part3,
          SUBSTRING_INDEX(ip,'.',-1) AS ip_part4,
          INET_ATON(ip) AS ip_integer
          FROM iptbl;
```
##### - _special Note_  INET_ATON(EXP1) VS INET_NTOA(EXP1)

INET_ATON(exp1) to return the numerical value of IP address.

INET_NTOA(exp1) to concvert the numerical value into IP address. 
 
       








 







