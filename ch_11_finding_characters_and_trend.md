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


### 2. Table conating a separate section "Status of login"

#### 2.1  Login/Guest 

Just in case where there is no entry for the section of user_id, I have used
COALESCE function to distignusih  data into login and guest.

- comparision operation
<>  or != means 'not equl to'.

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
#### 2.2 The Treatment of Subtotal 

In this section,  we would like to group our data by on two selected columns (ie) action and  login_status and count numbers of each group to present subtotal and total in the table simultaneously. 
That is, we can show a  numberson those who placed the goods in shopping charts with not log in and  the total number of action right next to it  

When we inted to insert a subtotal for each field and name it "all", use COALESCE(name of column, "all") and ROllUP functions. Remind that he ROLLUP is an extension of the GROUP BY clause. The ROLLUP option allows you to include extra rows that represent the subtotals along with the grand total row. By using the ROLLUP option, you can use a single query to generate _"multiple grouping sets"_. Then, the resulting tables contain the column 'action' including indiviaul types and "all", and 'login_status' containing 'login','guest',and 'all'. 


* Information on Attributes

 1) action
 2) login_status
 3) action_uu  : number of either login or guest in each action category
 4) action_count : the total counts of each action 


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

- result

![image](https://user-images.githubusercontent.com/53164959/64014817-15865400-cb5e-11e9-86f1-9fec80c7b2ac.png)


#### 2.3 Managing Membership Status 

From immediate supervisors, you are required to change the status of membership of customers to "member" if they have logged in the website once .  Regardless of whether users visit the website with login or not, the data of member status will keep appearing as a member once they log onto the site. Let's take this change in account when coding. 

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
    ('989004ea', NULL, 'purchase', 'drama' , 'D001,D002', 2000, '2016-03-01 18:10:00')
  , ('989004ea', 'U001', 'view'    , NULL    , NULL       , NULL, '2016-04-04 18:00:00')
  , ('989004ea', NULL, 'favorite', 'drama' , 'D001'     , NULL, '2016-10-23 18:00:00')
  , ('989004ea', 'U001', 'review'  , 'drama' , 'D001'     , NULL, '2016-02-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-04-11 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-09-13 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-10-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001'     , NULL, '2016-12-03 18:00:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D002'     , NULL, '2016-12-21 18:01:00')
  , ('989004ea', 'U001', 'add_cart', 'drama' , 'D001,D002', NULL, '2016-12-23 18:02:00')
  , ('989004ea', 'U001', 'purchase', 'drama' , 'D001,D002', 2000, '2016-12-25 18:10:00')
  , ('47db0370', NULL, 'add_cart', 'drama' , 'D001'     , NULL, '2016-02-03 19:00:00')
  , ('47db0370', 'U002', 'purchase', 'drama' , 'D001'     , 1000, '2016-05-03 20:00:00')
  , ('47db0370', NULL, 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-12 20:30:00')
  , ('87b5725f', NULL, 'add_cart', 'action', 'A004'     , NULL, '2016-01-04 12:00:00')
  , ('87b5725f', 'U001', 'add_cart', 'action', 'A005'     , NULL, '2016-03-04 12:00:00')
  , ('87b5725f', NULL, 'add_cart', 'action', 'A006'     , NULL, '2016-05-04 12:00:00')
  , ('9afaf87c', 'U002', 'purchase', 'drama' , 'D002'     , 1000, '2016-12-04 13:00:00')
  , ('9afaf87c', 'U002', 'purchase', 'action', 'A005,A006', 1000, '2016-12-24 15:00:00')
;

WITH action_log_detail AS(
      SELECT  session,
	      user_id,
              action,
              CASE WHEN COALESCE(MAX(user_id) OVER(PARTITION BY session
              ORDER BY  stamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
              ,'')<>'' THEN 'memeber' ELSE 'none' END AS member_status,
              stamp
              FROM action_log)
              SELECT  * FROM action_log_detail;
```

- Tips for SQL query

Up to the current row, if one customer has ever used log_in, there must appear a user_id in the previous section. When he places an order without login after that, the NULL value is automatically assigned to the section of user_id. Then, Using MAX(user_id)  function makes the program to search over  max number of string for the user_id from the very first row to the current one if user_id  is ever recorded. If nothing is recorded up to the point of search, then it returns  NULL. Right After that,  we use COALESCE clause to specify the status of members. 





### 3. Data Collection By Age


Based on information on age and sex of customers, we can set up five different categories to indicate which 
age group each belongs to. Look at the table below, 

![image](https://user-images.githubusercontent.com/53164959/62816405-c30ad680-bb61-11e9-83f8-f2963f6cbe2d.png)


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
;


WITH age_cal AS(
   SELECT user_id, 
          sex,
          birth_date,
          (CAST(REPLACE(CAST(CURRENT_DATE AS TEXT),'-','') AS INT)-CAST(REPLACE(birth_date,'-','') AS INT))/10000 AS age
          FROM mst_users)
    ,age_classification AS(
    SELECT user_id,
           sex,
           birth_date,
           FLOOR(age) AS age,
           CONCAT(CASE WHEN age<20 THEN ''
                  ELSE sex END,
                  CASE WHEN age BETWEEN 4 AND 12 THEN  'C'
                       WHEN age BETWEEN 13 AND 19 THEN 'T'
                       WHEN age BETWEEN 20 AND 34 THEN '1'
                       WHEN age BETWEEN 35 AND 49 THEN '2'
                       ELSE '3' END) AS category
             FROM age_cal)
     SELECT * FROM  age_classification;
                       
            
  ```
![image](https://user-images.githubusercontent.com/53164959/64020700-31452680-cb6d-11e9-9840-5edfd4cc7fcc.png)

Now we want to group consumers by category and count them to produce the total sum of each group. 

```sql 
WITH mst_users_temp AS(
     SELECT user_id,
			sex,
		    birth_date,
			TIMESTAMPDIFF(YEAR,birth_date,CURDATE()) AS age
            FROM mst_users),
	 mst_users_caetgory AS(SELECT user_id,
           CONCAT(
           CASE WHEN age<=20 THEN '' ELSE sex END ,
		   CASE WHEN age BETWEEN 4 AND 12 THEN 'C'
                WHEN age BETWEEN 13 AND 19 THEN 'T'
                WHEN age BETWEEN 20 AND 34 THEN '1'
		WHEN age BETWEEN 35 AND 49 THEN '2'
	        WHEN age>=50 THEN '3' END) AS category
           FROM mst_users_temp)
           SELECT category, 
                  COUNT(1) AS user_count 
		  FROM mst_users_caetgory
                  GROUP BY category;
```
![image](https://user-images.githubusercontent.com/53164959/64020818-7ec19380-cb6d-11e9-9d6d-035ddf093be8.png)


### 4. Understanding Different Aged Groups and thier preferences 

AS sales manager, you should probably wonder which product lines appeal most among the various age groups and which one could score a poor sales score. To better insight into the scale of sales of a product among the age brackets, it is a common practice to adopt bar charts to suggest the difference among the groups.  Then we need to do some manipulation for data visulization.


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
    ('989004ea', NULL, 'purchase', 'drama' , 'D001,D002', 2000, '2016-03-01 18:10:00')
  , ('989004ea', 'U001', 'view'    , 'animation'    , NULL       , NULL, '2016-04-04 18:00:00')
  , ('989004ea', NULL, 'favorite', 'drama' , 'D001'     , NULL, '2016-10-23 18:00:00')
  , ('889004ea', 'U002', 'review'  , 'drama' , 'D001'     , NULL, '2016-02-03 18:00:00')
  , ('889004ea', 'U002', 'add_cart', 'drama' , 'D001'     , NULL, '2016-04-11 18:00:00')
  , ('789004ea', 'U003', 'add_cart', 'drama' , 'D001'     , NULL, '2016-09-13 18:00:00')
  , ('789004ea', 'U003', 'add_cart', 'drama' , 'D001'     , NULL, '2016-10-03 18:00:00')
  , ('1089004ea', 'U004', 'add_cart', 'drama' , 'D001'     , NULL, '2016-11-03 18:00:00')
  , ('1089004ea', 'U004', 'add_cart', 'drama' , 'D001'     , NULL, '2016-12-03 18:00:00')
  , ('1189004ea', 'U005', 'add_cart', 'drama' , 'D002'     , NULL, '2016-12-21 18:01:00')
  , ('1189004ea', 'U005', 'add_cart', 'drama' , 'D001,D002', NULL, '2016-12-23 18:02:00')
  , ('1289004ea', 'U006', 'purchase', 'drama' , 'D001,D002', 2000, '2016-12-25 18:10:00')
  , ('47db0370', 'U007', 'add_cart', 'drama' , 'D001'     , NULL, '2016-02-03 19:00:00')
  , ('47db0370', 'U007', 'purchase', 'drama' , 'D001'     , 1000, '2016-05-03 20:00:00')
  , ('57db0370', 'U008', 'add_cart', 'drama' , 'D002'     , NULL, '2016-11-12 20:30:00')
  , ('87b5725f', 'U009', 'add_cart', 'action', 'A004'     , NULL, '2016-01-04 12:00:00')
  , ('87b5725f', 'U009', 'add_cart', 'action', 'A005'     , NULL, '2016-03-04 12:00:00')
  , ('87b5725f', NULL, 'add_cart', 'action', 'A006'     , NULL, '2016-05-04 12:00:00')
  , ('9afaf87c', 'U0010', 'purchase', 'drama' , 'D002'     , 1000, '2016-12-04 13:00:00')
  , ('9afaf87c', 'U0010', 'purchase', 'action', 'A005,A006', 1000, '2016-12-24 15:00:00')
;


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
;

WITH user_data AS(
     SELECT s.session,
            --replace null values with user_id if avaialbe or leave it null
            MAX(s.user_id) OVER(PARTITION BY session ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS user_id,
            s.category
            FROM action_log AS s)
     ,merge_data AS(            
     SELECT s.session,
            s.user_id,
            s.category,
            t.sex,
            t.birth_date
            FROM user_data AS s 
            JOIN mst_users AS t --enforcing a link between two tables
                 ON s.user_id=t.user_id)
     ,cal_age AS(
      SELECT user_id,
             category,
             sex,
             FLOOR((CAST(REPLACE(CAST(CURRENT_DATE AS TEXT),'-','') AS INT)-CAST(REPLACE(birth_date,'-','') AS INT))/10000) AS age
             FROM merge_data)
     ,age_group AS(
             SELECT category,
             CONCAT(CASE WHEN age<20 THEN ''
                         ELSE sex END,
                     CASE WHEN age BETWEEN 4 AND 12 THEN 'C'
                          WHEN age BETWEEN 13 AND 19 THEN 'T'
                          WHEN age BETWEEN 20 AND 34 THEN '1'
                          WHEN age BETWEEN 35 AND 49 THEN '2'
                          ELSE  '3' END) AS age_group
              FROM cal_age)
       SELECT category,
              age_group,
              count(1) AS count
              FROM age_group
              GROUP BY category,age_group
              ORDER BY age_group;

```
![image](https://user-images.githubusercontent.com/53164959/64059427-61d1a280-cbf7-11e9-8264-8a54d3d3f452.png)

### 5. Venn Diagram Analysis 

Venn Diagram is a statistical tool to visualize the proportion of each group and overlaps among two or more datasets. The diagram oftenuses a circle or ellipse to represent a segment or how much similar or different 
segments are from one another. 



With the use of flag variable, grant 1 if there is a record existing for each segment.  Otherwise, give 0.
For example, a consumer whose user_id is U001 has a historical record of all types of action. Then grant 1 for all the segments. 


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

Collecting data For action and converting it into flag variable. 
WITH flag_variable AS(
    SELECT session,
           --to perform group by session then we need aggregate function  on action
           --SIGN returns 1 if any positive value , 0 if value is zero
           SIGN(SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)) AS has_purchase,
           SIGN(SUM(CASE WHEN action='add_cart' THEN 1 ELSE 0 END)) AS has_add,
           SIGN(SUM(CASE WHEN action='review' THEN 1 ELSE 0 END)) AS has_chart
           FROM action_log
           GROUP BY session)
     SELECT * FROM flag_variable 
     
```


Additionally, to draw venn didagram, we should preapre a separte section to indicate the subtotal of all
combinatin of grouping columns. We use CUBE cluase to obtain this desired information.

Note that if flag variable is 1, then  name the slot the corresponding name of action or 
          if falg variable is zero, put 'not' in front of the name
	  The flag variable is filled with null value, not sure what the user has done. So,
	  put 'any' to the slot. If all of colums contain 'any', the total counts of it is
	  _the total number of users involved in our analysis_. 
      
          

```sql
WITH flag_variable AS(
    SELECT session,
           SIGN(SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)) AS has_purchase,
           SIGN(SUM(CASE WHEN action='add_cart' THEN 1 ELSE 0 END)) AS has_add,
           SIGN(SUM(CASE WHEN action='review' THEN 1 ELSE 0 END)) AS has_review
           FROM action_log
           GROUP BY session)
    ,action_venn_diagram AS(
     SELECT has_purchase,
            has_add,
            has_review,
            COUNT(1) AS users
            FROM flag_variable
            GROUP BY CUBE(has_purchase,has_add,has_review))
            
    SELECT -- converting the string variable into appropriate string label
           CASE WHEN has_purchasE=0 THEN 'Not Purchase'
                WHEN has_purchase=1 THEN 'purhcase'
                ELSE 'any' END AS has_purchase,
           CASE WHEN has_add=0 THEN 'NOT add chart'
                WHEN has_add=1 THEN 'add chart'
                ELSE 'any' END AS has_add,
            CASE WHEN has_review=0 THEN 'NOT review'
                 WHEN has_review=1 THEN 'review'
                 ELSE 'any' END AS has_review,
            users,
            --all columns WITH NULL is the total number of users in our analysis
            --NULLIF if exp1 and exp2 are same, return NULL.Otherwise, return exp1
            100*users/NULLIF(SUM(CASE WHEN has_purchase IS NULL AND has_add IS NULL AND has_review IS NULL THEN users ELSE 0 END) OVER(),0) AS ratio
            FROM action_venn_diagram
            ORDER BY has_purchase,has_add,has_review;
                   
 
```


![image](https://user-images.githubusercontent.com/53164959/64060591-efb68900-cc09-11e9-843b-b374a6f523e2.png)







7. Decile Aanalysis 

Decile analysis is a popular segmentation tool that dvide the whole data into equally sized groups of 10%. 
The steps are so following beow


```sql

DROP TABLE IF EXISTS blackfriday;
CREATE TABLE blackfriday
(user_id varchar(20),
 product_id varchar(20),
 gender  char(5),
 age    varchar(10),
 occupation SMALLINT,
 city_category CHAR(5),
 stay_in_current_city_years VARCHAR(5),
 martial_status SMALLINT,
 product_category1 SMALLINT,
 product_category2 SMALLINT,
 product_category3 SMALLINT,
 purchase_amount INT);

WITH user_purchase_amount AS(
SELECT user_id,
       SUM(purchase_amount) AS purchase_amount
       FROM blackfriday
       GROUP BY user_id),
      decile_user AS(
       SELECT user_id, 
              purchase_amount,
              ntile(30) OVER(ORDER BY purchase_amount DESC) AS decile
              FROM blackfriday
              )
         ,decile_with_amount AS(
         SELECT decile,
                SUM(purchase_amount) AS amount,
                ROUND(AVG(purchase_amount),2) AS average_amount,
                SUM(SUM(purchase_amount)) OVER(ORDER BY decile) AS cumulative_sum,
                SUM(SUM(purchase_amount)) OVER()AS total_sum
                FROM decile_user
                GROUP BY decile
                )
           SELECT decile,
                  amount,
                  round(100*amount/total_sum,2) AS ratio,
                  round(100*cumulative_sum/total_sum,2) AS cumulative_ratio
                  FROM decile_with_amount;

```



![image](https://user-images.githubusercontent.com/53164959/62863999-74b62d00-bd45-11e9-8568-8d11a23c1dd9.png)

I inteded to use data reflecting a sesonal effect to make it clear to explain the drawbacks of Decile analysis. 

First, VIPs, the top 10% of consumers, are generating 20% of profit which is still higher than other groups but not being perceived as significant a number as we expected it would be. The possible reason is that relatively overall consumers are willing to purchase items because massive advertising and discounts are offered to attract thier consumers during Black Friday.  Of course, the group of decile 1 is sill having a greater purchasing power and gives a huge impact on the financial performance of the surveyed company. However, since this seasonal effect may boost the total amount of sales, the proportion of spending made by Vips should shrink accordingly. 

Secondly,we should not put aside the possibility of counting, to the group of long term customers, the relatively large number of consumers who buy in bulk or buy up many items only this time of the year. This seasonal effect may conceal the true sales figure of each group.

### 8. RFM analysis

#### 8.1 Introduction

The alternative approach to Decile is RFM analysis which can allow us to quantify  the group of consumers in a delicate manner. 
We need three major components to perform the analysis ;

- Recency: the recent date of purchase.(Give more weight   to those who makes the purchase lately) 

- Frequency: the frequency of purchasing 
  (The more frequently they make purchases, the more important companies consider them)

- monetary: the total amount purchasing  by the customer


#### 8.2 Retail_Data_transaction.cvs From Kaggle

[https://www.kaggle.com/regivm/retailtransactiondata/version/1?login=true#Retail_Data_Transactions.csv]


![image](https://user-images.githubusercontent.com/53164959/62907540-b7601f80-bdae-11e9-9362-073e090f64b2.png)


```sql
WITH purchase_log AS(
SELECT user_id,
       SUBSTRING(date,1,10) AS dt,
       amount
       FROM retail_data)
       ,user_rfm AS(
       SELECT user_id,
              MAX(dt) AS recent_date,
              CURRENT_DATE-MAX(dt::date) AS recency,
              COUNT(dt) AS frequency,
              SUM(amount) AS monetary
              FROM purchase_log
              GROUP BY user_id)
         /*     
         Finding the range for each field
        SELECT min(recency), 
               max(recency),
               min(frequency),
               max(frequency),
               min(monetary),
               max(monetary)
                FROM user_rfm;
               */
         ,user_rfm_rank AS(
         SELECT user_id,
                recent_date,
                recency,
                frequency,
                monetary,
                CASE WHEN recency <1800 THEN 5
                      WHEN recency <1900 THEN 4
                      WHEN recency <2000 THEN 3
                      WHEN recency <2100 THEN 2
                      ELSE 1 END AS r
                 ,CASE WHEN 20<=frequency THEN 5
                       WHEN 10<=frequency THEN 4
                       WHEN 5<=frequency THEN 3
                       WHEN 2<=frequency THEN 2
                       WHEN 1<=frequency THEN 1 END AS f
                 ,CASE WHEN 2000<=monetary THEN 5
                       WHEN 1500<=monetary THEN 4
                       WHEN 1000<=monetary THEN 3
                       WHEN 500<=monetary THEN 2
                       ELSE 1 END AS m
                  FROM user_rfm )
                  SELECT * FROM user_rfm_rank  LIMIT 5;

```

![image](https://user-images.githubusercontent.com/53164959/62880025-6c6ee980-bd67-11e9-93b9-0b921ebcb67d.png)

Within the RFM model, each consumer has a score from 1111 to 5555 , a total of 125 combinations. That gives us discrete 125 consumers persona based on three indicators of the sales data. It is a good practice of carefully monitoring the clusters by plotting individual scores on a scatter plot graph and looking for patterns. For example, a lot of big business will have a lot of consumer with scores 111, due to a large amount of consumer churn. 

#### 8.3  Table FOR THE NUMBER OF CONUMSERS GROUPED BY rfm index 
Based on the result above, we are interested in the number of consumers belonging to each parameter ranged from 5 to 1. 
For example, what is the number of customers whose rfm_index is r and the scale they belong to is 5?

```sql
WITH purchase_log AS(
SELECT user_id,
       SUBSTRING(date,1,10) AS dt,
       amount
       FROM retail_data)
       ,user_rfm AS(
       SELECT user_id,
              MAX(dt) AS recent_date,
              CURRENT_DATE-MAX(dt::date) AS recency,
              COUNT(dt) AS frequency,
              SUM(amount) AS monetary
              FROM purchase_log
              GROUP BY user_id)
         /*     
         Finding the range for each field
        SELECT min(recency), 
               max(recency),
               min(frequency),
               max(frequency),
               min(monetary),
               max(monetary)
                FROM user_rfm;
               */
         ,user_rfm_rank AS(
         SELECT user_id,
                recent_date,
                recency,
                frequency,
                monetary,
                CASE WHEN recency <1800 THEN 5
                      WHEN recency <1900 THEN 4
                      WHEN recency <2000 THEN 3
                      WHEN recency <2100 THEN 2
                      ELSE 1 END AS r
                 ,CASE WHEN 20<=frequency THEN 5
                       WHEN 10<=frequency THEN 4
                       WHEN 5<=frequency THEN 3
                       WHEN 2<=frequency THEN 2
                       WHEN 1<=frequency THEN 1 END AS f
                 ,CASE WHEN 2000<=monetary THEN 5
                       WHEN 1500<=monetary THEN 4
                       WHEN 1000<=monetary THEN 3
                       WHEN 500<=monetary THEN 2
                       ELSE 1 END AS m
                  FROM user_rfm )
                 ,mst_rfm_index AS(
                 SELECT 1 AS rfm_index
                 UNION ALL SELECT 2 AS rfm_index
                 UNION ALL SELECT 3 AS rfm_index
                 UNION ALL SELECT 4 AS rfm_index
                 UNION ALL SELECT 5 AS rfm_index
                 )
                 ,rfm_flag AS(
                 SELECT
                 m.rfm_index, 
                 CASE WHEN m.rfm_index=r.r THEN 1 ELSE 0 END AS r_flag,
                 CASE WHEN m.rfm_index=r.f THEN 1 ELSE 0 END AS f_flag,
                 CASE WHEN m.rfm_index=r.m THEN 1 ELSE 0 END AS m_flag
                 FROM mst_rfm_index AS m
                 CROSS JOIN user_rfm_rank AS r
                 )
                 SELECT rfm_index,
                        SUM(r_flag) AS r,
                        SUM(f_flag) AS f,
                        SUM(m_flag) AS m
                        FROM rfm_flag
                        GROUP BY rfm_index
                        ORDER BY rfm_index DESC;
                      
```
![image](https://user-images.githubusercontent.com/53164959/62881723-5e22cc80-bd6b-11e9-986c-a9339af250dd.png)

### 8.4 RFM based on one variable(ie) the total sum of r,f,and m

The terminology of one variable may give a rise to confusion to some of readers, but not propert terminology comes out. The key point 
here is to peform RFM analysis subject to the total sum of r,f,and m.

The major problem with rfm analysis in the three-dimensional graphical dimension is a quite large number of groups we take into our consideration. In the previous case, each consumer should belong to one of 125 combinations of three indicators(5*5*5). Therefore, 
an analysis based on the total sum of these three indicators is needed to determine which group they are assigned to.


```sql

WITH purchase_log AS(
   SELECT user_id,
          SUBSTRING(date,1,10) AS dt,
          amount
          FROM retail_data),
          user_rfm AS(
          SELECT user_id,
                 MAX(dt::date) AS date,
                 CURRENT_DATE-MAX(dt::date) AS recency,
                 COUNT(dt) AS frequency,
                 SUM(amount) AS monetary
                 FROM purchase_log
                 GROUP BY user_id)
          ,user_rank AS(
          SELECT user_id, 
                  recency,
                  frequency,
                  monetary,
                  CASE WHEN recency<1800 THEN 5
                       WHEN recency<1900 THEN 4
                       WHEN recency<2000 THEN 3
                       WHEN recency<2100 THEN 2
                       ELSE 1 END AS r,
                  CASE WHEN 20<=frequency THEN 5
                       WHEN 10<=frequency THEN 4
                       WHEN 5<=frequency THEN 3
                       WHEN 3<=frequency THEN 2
                       ELSE 1 END AS f,
                   CASE WHEN 20000<=monetary THEN 5
                        WHEN 1500<=monetary THEN 4 
                        WHEN 1000<=monetary THEN 3
                        WHEN 500<=monetary THEN 2
                        ELSE 1 END AS m
                   FROM user_rfm)
                   SELECT r+f+m AS total_rank
                          ,r,f,m
                          ,COUNT(user_id) 
                          FROM user_rank
                          GROUP BY r,f,m
                          ORDER BY total_rank DESC,r DESC,f DESC,m DESC;
 ```
 ![image](https://user-images.githubusercontent.com/53164959/62908165-87fee200-bdb1-11e9-981d-ed1120e93e1c.png)
 
 ### 8.5 Selecting The Number of Indicators
 
 
 ```sq1
 WITH purchase_log AS(
   SELECT user_id,
          SUBSTRING(date,1,10) AS dt,
          amount
          FROM retail_data),
          user_rfm AS(
          SELECT user_id,
                 MAX(dt::date) AS date,
                 CURRENT_DATE-MAX(dt::date) AS recency,
                 COUNT(dt) AS frequency,
                 SUM(amount) AS monetary
                 FROM purchase_log
                 GROUP BY user_id)
          ,user_rank AS(
          SELECT user_id, 
                  recency,
                  frequency,
                  monetary,
                  CASE WHEN recency<1800 THEN 5
                       WHEN recency<1900 THEN 4
                       WHEN recency<2000 THEN 3
                       WHEN recency<2100 THEN 2
                       ELSE 1 END AS r,
                  CASE WHEN 20<=frequency THEN 5
                       WHEN 10<=frequency THEN 4
                       WHEN 5<=frequency THEN 3
                       WHEN 3<=frequency THEN 2
                       ELSE 1 END AS f,
                   CASE WHEN 20000<=monetary THEN 5
                        WHEN 1500<=monetary THEN 4 
                        WHEN 1000<=monetary THEN 3
                        WHEN 500<=monetary THEN 2
                        ELSE 1 END AS m
                   FROM user_rfm)
                   SELECT CONCAT('r_',r) AS r_rank,
                          COUNT(CASE WHEN f=5 THEN 1 END) as f_5,
                          COUNT(CASE WHEN f=4 THEN 1 END) as f_4,
                          COUNT(CASE WHEN f=3 THEN 1 END) as f_3,
                          COUNT(CASE WHEN f=2 THEN 1 END) as f_2,
                          COUNT(CASE WHEN f=1 THEN 1 END) as f_1
                          FROM user_rank
                          GROUP BY r
                          ORDER BY r_rank DESC;
         
     ```
 
It is also possible to select the number of indicators in your analysis to your case. Just think of a table with rows
indicating recency and columns consisting of the range of frequency.

Right after preapreing the table it is the time when you should classify the consumer persona you uncover during RFM analysis and 
come up with proper strategies implmented specific to each group.

- Brand Champion ( R=14 ,F=20)

These are your best customers and you want to keep it that way. It is the group that you should come up with a variety of marketing strategies to make them feel satisfied with the service and product you offer because they would bring your potential targets to your door. For example, Auction, one of the biggest e-commerce in South Korea, grants thier loyal customers  5 coupons for a free delivery service a month and a big discount on thier purchase if they invite thier neighborhoods to join the service and buy the item. 


- Loyal Customers ( R=28, F=10)

These groups are still one of your valuable customers and there should be a good chance of increasing thier lifetime value with the right offers. Therefore,  you may be alert at what he or she wants and a thorough survey and examination on thier purchasing behavior are needed on request. The possible business strategies are volume discounts and loyalty scheme.  

- Possibly Alienated 

A mismatch between recency and frequency is not a good sign where some was a regular customer but stopped using your service or the recently joined may find your service not much satisfactory and turn away. Regardless of either case, something has gone wrong and there is a need to bring some measures to fix it.  The possible strategy includes welcome back offers and satisfaction surveys. 

- New Customers (R=14, F=1)

People in this category have just recently discovered or rediscovered you after some time away. Our attention is placed on the ways of building a long-lasting relationship with them such as introductory offers, hints, tips and useful content. 

- One-off Big Spender 

Some customers buy up a great amount and then disappear. The possible reason is that they come to you with a specific need and realize that any other offers do not seem relevant to thier taste. To encourage them to revisit your, upgraded and maintenance offers are provided on the condition that they place an order for assigned number of times. 

- Expired leads (R=1, F=1, M=1)

Low-scoring customers are the least promising prospects in your database. They don’t have a significant purchase history with your company and there have been no recent interactions. These customers will fall outside of the scope of most marketing campaigns. By focusing on more promising leads, you can invest resources where they’re likely to lead to a result
 

                               
                   
         
                 
                   
          







