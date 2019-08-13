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

From immediate supervisors, you are required to change the status of membership of customers to "member" if they have logged in the website once in thier lifetime.  Regardless of whether users visit the website with login or not, the data of member status will keep appearing as a member once they log onto the site. Let's take this change into
our consideration when coding. 
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

3.4 Speacial Treatment On user_id 

When carefully examing all the sections of the table thoroughly, you should be aware that some sections are containing  NULL.  To treat the missing value,  you are required to replace it with 0. Otherwise, return user_id. 
This time COALESCE function could have alternatively used. However, this time I take NULLIF function.. 

_"NULLIF(exp1,exp2) function returns NULL if exp1 and exp2 are same.Otherwise, return exp1."_

```sql
 
WITH action_log_detail AS(
      SELECT  session,
			  CASE WHEN NULLIF(user_id,NUll)<>'' THEN user_id ELSE '0' END AS user_id,
              action,
              CASE WHEN COALESCE(MAX(user_id) OVER(PARTITION BY session
              ORDER BY  stamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
              ,'')<>'' THEN 'memeber' ELSE 'none' END AS member_status,
              stamp
              FROM action_log)
              SELECT  * FROM action_log_detail;
```


4. Data Collection By Age


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


WITH mst_users_with_birth_date AS(
    SELECT
    user_id,
    sex,
	birth_date,
    TIMESTAMPDIFF(YEAR,birth_date,CURDATE()) AS age
    FROM mst_users)
    SELECT user_id,
           sex,
           birth_date,
           age,
		   CONCAT(
           CASE WHEN 20<=age THEN sex
                ELSE ''END, 
		   CASE WHEN age BETWEEN 4 AND 12 THEN 'C' #C stands for children
                WHEN age BETWEEN 13 AND 19 THEN 'T'
                WHEN age BETWEEN 20 AND 34 THEN  '1'
                WHEN age BETWEEN 35 AND 49 THEN '2'
                WHEN age>=50 THEN '3'
                END) AS category
		FROM mst_users_with_birth_date;
  ```
  
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

5. Understanding Different Aged Groups and thier preferences 

AS sales manager, you should probably wonder which product lines appeal most to the teenagers these days and
which one could get the worst reviews by 3o's with the higher purchasing power. It seems to be inevitable that we should take an investigation on the liking among different groups of consumers. 

```sql
WITH mst_age AS(
    SELECT user_id,
           sex, 
           TIMESTAMPDIFF(YEAR,birth_date,CURDATE()) AS age
           FROM mst_users),
           mst_age_group AS(
           SELECT   user_id,
					CONCAT(CASE WHEN age<=20 THEN '' ELSE sex END,
					    CASE WHEN age BETWEEN 4 AND 12 THEN  'C'
                             WHEN age BETWEEN 13 AND 19 THEN 'T'
                             WHEN age BETWEEN 20 AND 34 THEN '1'
                             WHEN age BETWEEN 35 AND 49 THEN '2'
                             WHEN age>=50 THEN '3' END) AS category
				FROM mst_age)
			SELECT p.register_device,l.category AS user_category,COUNT(1) AS purchase_count
			FROM mst_users AS p
			LEFT JOIN mst_age_group  AS l
			      ON p.user_id=l.user_id
                  GROUP BY p.register_device,l.category
				  ORDER BY purchase_count DESC;
```


![image](https://user-images.githubusercontent.com/53164959/62818253-18ee7700-bb80-11e9-8647-71764cef4f54.png)

5. Vitor Frequency Table Downlading The Data Called public_center_vistors.csv

```sql
DROP TABLE IF EXISTS public_vistors;
CREATE TABLE public_vistors
(date varchar(50),
 career_exploration INT,
 child_center INT,
 exhibition INT,
 total INT);
 
LOAD DATA  INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/visit_day.csv' INTO TABLE public_vistors 
fields terminated by ','; 
SELECT * FROM public_vistors;


DROP TABLE IF EXISTS public_vistors;
CREATE TABLE public_vistors
(visit_date varchar(50),
 career_exploration INT,
 child_center INT,
 exhibition INT,
 total INT);
 
LOAD DATA  INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/visit_day.csv' INTO TABLE public_vistors 
fields terminated by ','; 
SELECT * FROM public_vistors LIMIT 10;


WITH term_data AS(
     SELECT visit_date,
     SUBSTRING_INDEX(visit_date,'-',1) AS year,
     SUBSTR(visit_date,6,2) AS month,
     SUBSTRING_INDEX(visit_date,'-',-1) AS date,
     career_exploration,
     child_center
     FROM public_vistors
     GROUP BY MONTH)
     SELECT month,
            career_exploration,
	    child_center,
	    SUM(career_exploration) OVER() AS total_career_exploration,
            SUM(child_center) OVER() AS total_child_center, 
            SUM(career_exploration) OVER(ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS
	    cum_sum_ce,
            SUM(child_center) OVER(ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_sum_cc,
            ROUND(100*career_exploration/sum(career_exploration) OVER(),2) AS rate_ce,
            ROUND(100*child_center/SUM(child_center) OVER(),2) AS rate_cc
    FROM term_data
    GROUP BY month;
 ```
6. Venn Diagram Analysis 

Venn Diagram is a statistical tool to visualize the proportion of each group and overlaps among two or more datasets. The diagram oftenuses a circle or ellipse to represent a segment or how much similar or different 
segments are from one another. 


6.1 Flag Variable 

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
WITH action_log_users AS(
     SELECT    # first sum up all the purchases made by one user and if the count
               # is non-zero, return 1. Otherwise, 0. Same goes to the other segments.
		    SIGN(SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)) AS has_purchase,
		    SIGN(SUM(CASE WHEN action='review' THEN 1 ELSE 0 END)) AS has_review,
            SIGN(SUM(CASE WHEN action='favorite' THEN 1 ELSE 0 END)) AS has_favorite
            FROM action_log
            GROUP BY user_id)
	 SELECT * 
	 FROM action_log_users;
```

Additionally, to draw venn didagram, we should preapre a separte section to indicate the subtotal of all
combinatin of grouping columns. We use CUBE cluase to obtain the combinations. 

```sql
WITH user_action_flag AS(
     SELECT CASE WHEN COALESCE(user_id,'')<>'' THEN user_id ELSE 'Guest' END AS user_id,
            SIGN(SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)) AS has_purchase,
            SIGN(SUM(CASE WHEN action='review' THEN 1 ELSE 0 END)) AS has_review,
            SIGN(SUM(CASE WHEN action='favorite' THEN 1 ELSE 0 END)) AS has_favorite
            FROM action_log
            GROUP BY user_id) -- produce fullstatics based on each user_id
            SELECT has_purchase,
                   has_review,
                   has_favorite
                   FROM user_action_flag
                   GROUP BY CUBE(has_purchase,has_review,has_favorite)
		   ORDER BY has_purchase,has_review,has_favorite;
                   
  ```


The empty entries are NULL to indicate that it is unclear wheter consumers has taken any action on each category or not.

```SQL
WITH user_action_flag AS(
     SELECT CASE WHEN COALESCE(user_id,'')<>'' THEN user_id ELSE 'Guest' END AS user_id,
            SIGN(SUM(CASE WHEN action='purchase' THEN 1 ELSE 0 END)) AS has_purchase,
            SIGN(SUM(CASE WHEN action='review' THEN 1 ELSE 0 END)) AS has_review,
            SIGN(SUM(CASE WHEN action='favorite' THEN 1 ELSE 0 END)) AS has_favorite,
            count(1) AS users
            FROM action_log
            GROUP BY user_id) -- produce fullstatics based on each user_id
                  ,action_venn_diagram AS(SELECT 
                   has_purchase,
                   has_review,
                   has_favorite,
                   COUNT(1) AS users
                   FROM user_action_flag
                   GROUP BY CUBE(has_purchase,has_review,has_favorite)
		   ORDER BY has_purchase,has_review,has_favorite)
            SELECT CASE WHEN has_purchase=1 THEN 'purchase' WHEN has_purchase=0 THEN 'not purchase' ELSE 'any' END AS has_purchse,
                   CASE WHEN has_review=1 THEN 'review' WHEN has_review=0 THEN 'not review' ELSE 'any' END AS has_review,
                   CASE WHEN has_favorite=1 THEN 'favorite' WHEN has_favorite=0 THEN 'not favorite' ELSE 'any' END AS has_favorite,
                   100.0*users/NULLIF(SUM(CASE WHEN has_purchase IS NULL AND has_review IS NULL AND has_favorite IS NULL THEN users ELSE                    0 END) OVER(),0) AS ratio
		   -- every action having null value represents the total amount of consumers
		   -- using window function to calculate the sum 
                   FROM action_venn_diagram
		   ORDER by has_purchase,has_review,has_favorite ;
```


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




