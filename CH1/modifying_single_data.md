### Modifying Single Data

#### 1 Change the label of data

Introduction:

When downlaoding any 'raw' files  withouht proper modification, it should be hard for the end-uers 
to understand the contents of information in them. For example, we have a column section called register_device containing the relevant
information- 1 for 'Application', 2 for 'Electronics' ,and 3 for 'Smartphone'. Without taking an appropriate conversion of 
the code values into lables, they would be struggle to find out what they are. 



Codes:

Method 1) preapre the data

```Mysql 
<MySQL,PostGreSQL>

CREATE DATABASE sqldb;
USE sqldb;
DROP TABLE IF EXISTS mst_users;
CREATE TABLE mst_users(
    user_id         varchar(255)
  , register_date   varchar(255)
  , register_device integer
);

INSERT INTO mst_users
VALUES
    ('U001', '2016-08-26', 1)
  , ('U002', '2016-08-26', 2)
  , ('U003', '2016-08-27', 3)
;
```

```sql
<Hive>
CREATE TABLE IF NOT EXISTS emst_users(
    user_id string,
    register_date string,
    register_device tinyint) 
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;
```





Method 2) Conversion of Data Values into Labels 

```mysql
SELECT user_id,  
       CASE WHEN register_device=1 THEN 'Application'
            WHEN register_device=2 THEN 'Electronics'
            WHEN register_device=3 THEN 'SmartPhone'
            ELSE '' 
            END AS 'Device_Name'
		FROM mst_users;
```



#### 2 Extracting elements from URL

#### *  host 

Sometimes, we need a certain part of URL for presenting our data since URL itself should be too long or even complicating during
data analysis. Therefore, in this section, we are trying to practice taking the "host"part from URL and presenting it under the name 
of 'referrer_host'.


method 1) prepare the data
```mysql
DROP TABLE IF EXISTS access_log ;
CREATE TABLE access_log (
    stamp    varchar(255)
  , referrer text
  , url      text
);

INSERT INTO access_log 
VALUES
    ('2016-08-26 12:02:00', 'http://www.other.com/path1/index.php?k1=v1&k2=v2#Ref1', 'http://www.example.com/video/detail?id=001')
  , ('2016-08-26 12:02:01', 'http://www.other.net/path1/index.php?k1=v1&k2=v2#Ref1', 'http://www.example.com/video#ref'          )
  , ('2016-08-26 12:02:01', 'https://www.other.com/'                               , 'http://www.example.com/book/detail?id=002' )
;
```
method 2) Extraction 

```mysql
<mysql Ver>
SELECT stamp, 
       SUBSTRING_INDEX(SUBSTRING_INDEX(referrer,'//',-1),'/',1) AS referrer_host
       FROM access_log;
```
<postgresql ver>
SELECT stamp, 
       SUBSTRING(referrer from '//([^/]*)') as host 
       FROM access_log;
```

```sql
<Hive>
SELECT stamp,
       PARSE_URL(referrer,'HOST') as host,
       PARSE_URL(url,'PATH') as path,
       PARSE_URL(url,'QUERY') as id
       FROM access_log;


```
	
	
#### * path,id 

Method 1)prepare the data

-The same data previously shown- 

Method 2) Extraction

```mysql

# To calculate the size of length of 'http://www.example.com/'
SELECT LENGTH('http://www.example.com/');#Return 23 in total. 

SELECT stamp,
       SUBSTR(SUBSTRING_INDEX(SUBSTRING_INDEX(url,'?id=',1),'#ref',1),23) AS 'path',
       CASE
           WHEN LENGTH(SUBSTRING_INDEX(url,'?id=',-1))=3
		        THEN SUBSTRING_INDEX(url,'?id=',-1)
		   ELSE ''
		END AS 'id'
	FROM access_log;

```
```sql
<posgresql ver>
SELECT url,
       SUBSTRING(url,'//([^/]*)') AS host,
       SUBSTRING(url,'//[^/]*([^#?]*)') AS path,
       SUBSTRING(url,'id=([^&]*)') AS id
       FROM access_log;
```


```sql
<Hive>
SELECT stamp,
       split(parse_url(url,'PATH'),'/')[1] as b,
       split(parse_url(url,'PATH'),'/')[2] as c
       FROM access_log
```






#### * spliting one section into many sub-divsions
In this section, the path is further taken apart into two subgroups, path1 and path 2. 
Unlike PostGreSQL that provides split_part function, MySQL dose not have it, which leads the codes to look a little messy and 
less efficient in terms of readability. 

```mysql
SELECT stamp,url,SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTR(url,LENGTH(SUBSTRING_INDEX(url,'/',3))+2),'/',1),'#',1) AS path1,
		CASE
        WHEN SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTR(url, LENGTH(SUBSTRING_INDEX(url,'/',3))+2), '?', 1), '#', 1) RLIKE '\/'
            THEN SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTR(url, LENGTH(SUBSTRING_INDEX(url,'/',3))+2), '?', 1), '#', 1), '/', -1)
		    ELSE ''
            END AS path2
			FROM access_log;
```

```SQL
<postgresql>
WITH access_time AS(
     SELECT SUBSTRING(stamp,1,4) AS year,
            SUBSTRING(stamp,6,2) AS month,
            SUBSTRING(stamp,9,2) AS date,
            url
            FROM access_log)
	    
     SELECT CONCAT(year,'-',month) AS year_month,
            SUBSTRING(url,'//([^/]*)') AS host,
            SUBSTRING(url,'//[^/]+([^#?]+)') AS path,
            SPLIT_PART(SUBSTRING(url,'//[^/]+([^#?]+)'),'/',2) AS path1,
            SPLIT_PART(SUBSTRING(url,'//[^/]+([^?#]+)'),'/',3) AS path2,
            SUBSTRING(url,'id=([^&]*)') AS id
            FROM access_time;
 ```

#### 4) Handling Date and Time Stamp of Data
#### *Printing current-time and time stamp 
```mysql
SELECT CURRENT_DATE() AS dt, current_time() AS dh ,current_timestamp() AS stamp;
```
#### *Conversion of 'char' into 'date' type
Making use of 'Cast'function is the most universay way to handle the task.

```mysql
SELECT CAST('2019-7-23' AS DATE) AS dt, CAST('2019-7-23 17:20:13' AS DATETIME ) AS stamp;
```

#### *Extraction of specific parts from the date
```mysql
DROP TABLE IF EXISTS practbl;
CREATE TABLE practbl
(stamp datetime);

INSERT INTO practbl 
       VALUES ('2019-7-23 17:25:01');

SELECT stamp, 
       EXTRACT(YEAR FROM stamp) AS YEAR, 
	   EXTRACT(MONTH FROM stamp) AS MONTH, 
       EXTRACT(DAY FROM stamp)  AS DAY,
       EXTRACT(HOUR FROM stamp) AS HOUR
       FROM practbl;
```

```sql
<postgresql>
SELECT CAST(stamp AS date) AS dt, --return "2019-07-23"
       CAST(stamp AS timestamp) AS stamp, -- return "2019-07-23 17:25:01"
       stamp::date,  --return "2019-07-23"
       stamp::timestamp
       FROM practbl;



SELECT SUBSTRING(stamp,1,4) AS year,
       SUBSTRING(stamp,6,2) AS month,
       SUBSTRING(stamp,9,2) AS date,
       SUBSTRING(stamp,12,2) AS hour,
       SUBSTRING(stamp,15,2) AS min,
       SUBSTRING(stamp,18,2) AS sec
       from practbl;
```



#### 5)Handling Default Value 
NULL as default value  is an 'infectious' one ; anything it touches will lead to NULL in the end. Therefore, we need a measure
to deal with this problem. I have used 'coalesce' to prevent it from happening. 

method 1)prepare the data
```mysql
CREATE TABLE purchase_log_with_coupon (
    purchase_id varchar(255)
  , amount      integer
  , coupon      integer
);

INSERT INTO purchase_log_with_coupon
VALUES
    ('10001', 3280, NULL)
  , ('10002', 4650,  500)
  , ('10003', 3870, NULL)
;
```
method 2) handling the default values.
```mysql
SELECT purchase_id,amount,coupon,
       amount-COALESCE(coupon,0) AS actual_price
       FROM purchase_log_with_coupon;
```










