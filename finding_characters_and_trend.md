Finding Characters and Trends 
=============================

### 1. Introduction 

In the business world, business managers are actively seeking relevant information on customers, from which they 
can take an investigation on the behaviors of their customers, predict the pattern of their spending, and 
even get some useful tips on how to improve thier peformance of sales. 


The sample tables we will use for our data analysis are following; 


![image](https://user-images.githubusercontent.com/53164959/62774808-e6417180-bae0-11e9-9d8c-56dbd2f69355.png)

[table 1]  'mst_user' table


![image](https://user-images.githubusercontent.com/53164959/62774912-2e609400-bae1-11e9-94da-bb5205a38942.png)

[table 2] 'action_log' table 

You can find that there are some missing entries on the column of 'user_id' and this refers to those
who visits the website without any log in. 


### 3. Table Data Categorized By The Type of Uers

3.1  Login/Guest 
Just in case where there is no entry for the section of user_id, I have used
COALESCE function to categorize the data into login and guest. Note that <> means not equal to, != also means not equal to.

```sql
DROP TABLE IF EXISTS action_log;
CREATE TABLE action_log(
    session  varchar(255)
  , user_id  varchar(255)
  , action   varchar(255)
  , category varchar(255)
  , products varchar(255)
  , amount   integer
  , stamp    varchar(255)
);

INSERT INTO action_log
VALUES
    ('989004ea', 'U001', 'purchase', 'drama' , 'D001,D002', 2000, '2016-11-03 18:10:00')
  , ('989004ea', 'U001', 'view'    , NULL    , NULL       , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'favorite', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'review'  , 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-03 18:01:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001,D002', NULL, '2016-11-03 18:02:00')
  , ('989004ea', 'U001', 'purchase', 'drama' , 'D001,D002', 2000, '2016-11-03 18:10:00')
  , ('47db0370', 'U002', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 19:00:00')
  , ('47db0370', 'U002', 'purchase', 'drama' , 'D001'     , 1000, '2016-11-03 20:00:00')
  , ('47db0370', 'U002', 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-03 20:30:00')
  , ('87b5725f', 'U001', 'add_cart', 'action', 'A004'     , NULL, '2016-11-04 12:00:00')
  , ('87b5725f', 'U001', 'add_cart', 'action', 'A005'     , NULL, '2016-11-04 12:00:00')
  , ('87b5725f', 'U001', 'add_cart', 'action', 'A006'     , NULL, '2016-11-04 12:00:00')
  , ('9afaf87c', 'U002', 'purchase', 'drama' , 'D002'     , 1000, '2016-11-04 13:00:00')
  , ('9afaf87c', 'U001', 'purchase', 'action', 'A005,A006', 1000, '2016-11-04 15:00:00')
  ,('9afaf87c', NULL, 'purchase', 'action', 'A005,A006', 1000, '2016-11-04 15:00:00')
;

WITH action_log_with_status AS(
 SELECT session,
        user_id,
        action,
		CASE WHEN COALESCE(user_id,"")<> "" THEN 'login' ELSE 'guest' END
        AS login_status
        FROM action_log)
 SELECT * FROM action_log_with_status;
```        
3.2 The Treatment of Subtotal 

When we inted to insert a subtotal for each field and name it "all", use COALESCE(name of column, "all") and ROllUP functions. Remind that he ROLLUP is an extension of the GROUP BY clause. The ROLLUP option allows you to include extra rows that represent the subtotals along with the grand total row. By using the ROLLUP option, you can use a single query to generate _"multiple grouping sets"_. Then, the resulting tables contain the column 'action' including indiviaul types and "all", and 'login_status' containing 'login','guest',and 'all'. 

```sql
SELECT * FROM action_log limit 21;
WITH
login_status AS (    
  SELECT
    session,
    action,
    CASE WHEN COALESCE(user_id,'')<>'' THEN 'login' ELSE 'guest' END AS login_status
  FROM action_log
)
SELECT
  COALESCE(action, 'all') AS action,
  COALESCE(login_status, 'all') AS login_status,
  COUNT(session) AS action_unique_users,
  COUNT(action) AS action_count
FROM login_status
GROUP BY action, login_status WITH ROLLUP;
```

3.3 Managing Membership Status 


```sql
DROP TABLE IF EXISTS action_log;
CREATE TABLE action_log(
    session  varchar(255)
  , user_id  varchar(255)
  , action   varchar(255)
  , category varchar(255)
  , products varchar(255)
  , amount   integer
  , stamp    varchar(255)
);

INSERT INTO action_log
VALUES
    ('989004ea', NULL, 'purchase', 'drama' , 'D001,D002', 2000, '2016-11-03 18:10:00')
  , ('989004ea', 'U001', 'view'    , NULL    , NULL       , NULL, '2016-11-03 18:00:00')
  , ('989004ea', NULL, 'favorite', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'review'  , 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-03 18:01:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001,D002', NULL, '2016-11-03 18:02:00')
  , ('989004ea', 'U001', 'purchase', 'drama' , 'D001,D002', 2000, '2016-11-03 18:10:00')
  , ('47db0370', NULL, 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 19:00:00')
  , ('47db0370', 'U002', 'purchase', 'drama' , 'D001'     , 1000, '2016-11-03 20:00:00')
  , ('47db0370', NULL, 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-03 20:30:00')
  , ('87b5725f', NULL, 'add_cart', 'action', 'A004'     , NULL, '2016-11-04 12:00:00')
  , ('87b5725f', 'U001', 'add_cart', 'action', 'A005'     , NULL, '2016-11-04 12:00:00')
  , ('87b5725f', NULL, 'add_cart', 'action', 'A006'     , NULL, '2016-11-04 12:00:00')
  , ('9afaf87c', 'U002', 'purchase', 'drama' , 'D002'     , 1000, '2016-11-04 13:00:00')
  , ('9afaf87c', 'U002', 'purchase', 'action', 'A005,A006', 1000, '2016-11-04 15:00:00')
;
```










