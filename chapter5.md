### chapter 5 Modifying Single Data

#### 5.1 change the label of data

Introduction:

When downlaoding any 'raw' files  withouht proper modification, it should be hard for the end-uers 
to understand the contents of information in them. For example, we have a column section called register_device containing the relevant
information- 1 for 'Application', 2 for 'Electronics' ,and 3 for 'Smartphone'. Without taking an appropriate conversion of 
the code values into lables, they would be struggle to find out what they are. 



Codes:

Method 1) preapre the data

```Mysql 
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

