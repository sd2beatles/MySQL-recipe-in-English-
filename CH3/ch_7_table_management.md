
Acrroding to most books and web sties, they state that MySQL is known for not providing a window funciton.However, the version of 8
does support some including ROW_NUMBER,RANK(),DESE_RANK() and so on. 



## CHAPTER 7 MANUPULATION OF DATA TABLE 
### 7.1 GROUPING 

GROUP BY function is very often used for grouping rows that have the same values into summary rows and accompanied 
with aggregate function(MAX,MIN,AVG etc) to group the result set by one or more columns. 
  

method 1) preparing the 'raw'data
```MySQL
DROP TABLE IF EXISTS review;
CREATE TABLE review (
    user_id    varchar(255)
  , product_id varchar(255)
  , score      numeric
);

INSERT INTO review
VALUES
    ('U001', 'A001', 4.0)
  , ('U001', 'A002', 5.0)
  , ('U001', 'A003', 5.0)
  , ('U002', 'A001', 3.0)
  , ('U002', 'A002', 3.0)
  , ('U002', 'A003', 4.0)
  , ('U003', 'A001', 5.0)
  , ('U003', 'A002', 4.0)
  , ('U003', 'A003', 4.0)
;
```
method 2) computing 
```MySQL
SELECT user_id,
	   COUNT(*) AS total_count,
	   COUNT(DISTINCT product_id) AS product_count,
	   SUM(score) as sum,
       ROUND(AVG(score),2) as avg,
       MAX(score) as max,
       MIN(score) as min 
	   FROM review
       GROUP  BY user_id;
```
### 7.2 THE Syntax of OVER CLUASE 
#### 7.2.1 OVER() VS OVER(PARTITION BY)

In order to present data before and after aggreate functions  are implemented all together, OVER is the most suitable function. 
we could take.Unless OVER funcion is used with specifying any options,the aggreating function will be applied to the whole table(see the column named avg_score in our code where we are actully averaging all the scores on the table). However,  PARTITION BY caluse in the parenthesis of Over will determine which rows will be applied to the given functions. Let's look at how these concepts are actually 
implemented in our case stduy. 


```MySQL
SELECT user_id,
       product_id, 
       score,
       ROUND(AVG(score) OVER(),2)  AS 'avg_score', #since no specification, the AVG function is applied to 9 rows. 
       ROUND(AVG(score) OVER(PARTITION BY user_id),2) AS 'user_avg_score', #return averge values based on the particular usr_id
       ROUND(score-AVG(score) OVER(PARTITION BY user_id),2) AS 'use_avg+diff'
       FROM review;
```

For more detail about OVER cluase, visit [https://www.sqlservercentral.com/articles/understanding-the-over-clause]

#### 7.2.2 OVER(ORDER BY) and Ranking The Rows 
First,we want to arrange rows in either descending or ascedning order  with a help of  ORDER BY clause. In addition, 
there are three separate window functions to label the ranks of rows of a result-set, each of which has a distinct feature. 

- ROW_NUMBER() : ranks the rows with no overlap 
- RANK() : allows for overlap , but 'leaping' or 'skiping' in the rank 
- DENSE_LANK()  : allows for overlap and no 'leaping' or 'skiping' in the rank

Furthermore, it is a worthy of remembering other window functions such as "LEAD","LAG". 

- LEAD() : to return a a vlue that current row follows
- LAG() :  to return a value that comes before the ccurrent row 

You can also specify the number of rows either forwarding or backwarding from the current row. For example,
LEAD(usre_id,2) means yo want to get an access to the row after the next row
LAG(user_id,2) indicatest an acess to the row before the previous row.

To see how these all window functions are acually implmented int the case of popular_products

```MySql
DROP TABLE IF EXISTS popular_products;
CREATE TABLE popular_products (
    product_id varchar(255)
  , category   varchar(255)
  , score      numeric
);

INSERT INTO popular_products
VALUES
    ('A001', 'action', 94)
  , ('A002', 'action', 81)
  , ('A003', 'action', 78)
  , ('A004', 'action', 64)
  , ('D001', 'drama' , 90)
  , ('D002', 'drama' , 82)
  , ('D003', 'drama' , 78)
  , ('D004', 'drama' , 58)
;

SELECT product_id,
       score,
       ROW_NUMBER() OVER(ORDER BY score DESC) AS 'row_number',
	   RANK() OVER(ORDER BY score DESC) AS 'rank',
       DENSE_RANK() OVER(ORDER BY score DESC) AS 'density_rank',
       LAG(product_id) OVER(ORDER BY score DESC) AS 'lag1',
       LAG(product_id,2) OVER(ORDER BY score DESC) AS 'lag2', # a row before the previous raw 
	   LEAD(product_id) OVER(ORDER BY score DESC) AS lead1,
       LEAD(product_id,2) OVER(ORDER BY score DESC) as lead2 # a raw after the next raw
	   FROM popular_products;
```
#### 7.2.3 OVER() clause and Aggregating Functions (1)

<Descrition of Each Label>

- row : rank the data on score withouth any overlap
- cum_sum : compute cumulative sum up to k th index 
- local_avg : return an average value of three 'local' values
             (the preceding value right before the current index, the current index,and the follwing value)
- first_value : find the product_id whose socre is the least 
- last_value :  Return the product_id whose score is the highest 


```MySQL
SELECT product_id,
       score, 
	   ROW_NUMBER() OVER(ORDER BY score DESC) AS 'row', 
       SUM(score) OVER(ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_sum,
       AVG(score) OVER(ORDER BY score DESC ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS loca_avg,
       FIRST_VALUE(product_id) OVER(ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'first_value',
       LAST_VALUE(product_id) OVER(ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'last_value'
       FROM popular_products
       ORDER BY ROW_NUMBER() OVER(ORDER BY score DESC) ; 
	   
       ;
 ```
 
 #### 7.2.4 Using PARTITION BY AND ORDER BY in THE OVER() function at The same time

Like what we have done on the previous section, we are still interested in ranking the rows based on the given scores. Howeever, there
is one more prerequisite at this time - ranking the orders based on scores with the rows being partitioned by category. 

```MySQL
SELECT category,product_id,score,
       ROW_NUMBER() OVER(PARTITION BY category ORDER BY score DESC) AS 'row',
       RANK() OVER(PARTITION BY category ORDER BY score DESC) AS 'rank',
       DENSE_RANK() OVER(PARTITION BY category ORDER BY score DESC) AS 'dense_rank'
       FROM popular_products
       ORDER BY category,'row'
       ;
 ```
 
 #### 7.2.5 K-th higheset Rnaks 
 
 Now, let's extract k-th highest ranks from the sorted rows with all the rows still being partitioned by category. 
 Then, WHERE is an essential clause to meet the task but the problem beings as fllowing 
 
 _" Unser SQL implementing a window funciton is prohibited in WHERE clause"_
 
 To handle this issue, we should use a sub-query,indstead. 
 
```MySQL
SELECT * 
	   FROM(
            SELECT 
                 category,
		 score,
		 ROW_NUMBER() OVER(PARTITION BY category ORDER BY score DESC) AS ranks
                 FROM popular_products
                 )AS popluar_proudcts_k_st_rank
                 WHERE ranks<=2
                 ORDER BY category,ranks;
```

#### 7.2.6 Find The First Rank and Least Rank

```MySQL

SELECT DISTINCT category,
       FIRST_VALUE(product_id) OVER(PARTITION BY category ORDER BY score DESC
       ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS first_rank,
       LAST_VALUE(product_id) OVER(PARTITION BY category ORDER BY score DESC 
       ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as rank_last
       FROM popular_products
       ORDER BY category;

```

#### 7.2.7 Quiz

Prepare a table satisfying the following pre-conditions. 

1) The columns must include 
           
	   - user_id
	   
	   - array_id : group proudct_id by usre_id and return it in an array
	   
	   - cum_array_id : return a cumulative array 
	   
	   - total_avg : average out all the values regardless of user_id and product_id and round it to two deciamal points
	   
	   - rank1 : rank the elements within subgroupo of user_id based on thier local average 
	         : this ranking does not allow for overlap and leap in the ranks
		 
            - rank 2: the same as rank 1 except allow for leap or skip in the ranks
	    
	    - rank 3: allow for overlap and skip or leap in the ranks
	    
	    - lag1 : obtain access to a row of product_id before the current row
	    
	    - lag2 : get an acess to a row of product_id before the previous row
	    
	    - lead1 : obtain access to a row of product_id after the current row
	    
            - lead 2: get an acess to a row of product_id after the next row

 2) Just show rank1 up to 3
 
 ```sql
 DROP TABLE IF EXISTS review;
CREATE TABLE review (
    user_id    varchar(255)
  , product_id varchar(255)
  , score      numeric
);

INSERT INTO review
VALUES
    ('U001', 'A001', 4.0)
  , ('U001', 'A002', 5.0)
  , ('U001', 'A003', 5.0)
  , ('U001', 'A004', 5.0)
  , ('U002', 'A001', 3.0)
  , ('U002', 'A002', 3.0)
  , ('U002', 'A003', 4.0)
  , ('U002', 'A004', 4.0)
  , ('U003', 'A001', 5.0)
  , ('U003', 'A002', 4.0)
  , ('U003', 'A003', 4.0)
  , ('U003', 'A004', 4.0)
;
WITH temp_statics AS(
SELECT user_id,
       ARRAY_AGG(product_id) OVER(PARTITION BY user_id) AS array_id,
       ARRAY_AGG(product_id) OVER(PARTITION BY user_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_array_id,
       AVG(score) OVER() AS total_avg,
       ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY score) AS rank1,
       RANK() OVER(PARTITION  BY user_id ORDER BY score) AS rank2,
       DENSE_RANK() OVER(PARTITION BY user_id ORDER BY score) AS rank3,
       LAG(product_id,1) OVER(PARTITION BY user_id) AS lag1,
       LAG(product_id,2) OVER(PARTITION BY user_id) AS lag2,
       LEAD(product_id,1) OVER(PARTITION BY user_id) AS lead1,
       LEAD(product_id,2) OVER(PARTITION BY user_id) AS lead2
       FROM review)
       SELECT *
       FROM temp_statics
       WHERE rank1<=3
      ; 
 ```



#### 7.2.8 GROUP_CONCAT CALUSE 

Just image a vast of consumers logged on Amazone with thier own user id and puchased a variety of products on display. You may present 
all the purchaseses and the total amount each user spent as two separate fields on the table. This can be done through use of GROUP_CONCAT. 

```MySQL
DROP TABLE IF EXISTS purchase_detail_log;
CREATE TABLE purchase_detail_log (
    purchase_id integer
  , product_id  varchar(255)
  , price       integer
);

INSERT INTO purchase_detail_log
VALUES
    (100001, 'A001', 3000)
  , (100001, 'A002', 4000)
  , (100001, 'A003', 2000)
  , (100002, 'D001', 5000)
  , (100002, 'D002', 3000)
  , (100003, 'A001', 3000)
;


SELECT DISTINCT purchase_id,
       GROUP_CONCAT(product_id),
       SUM(PRICE)
       FROM purchase_detail_log
       GROUP BY purchase_id
       ;
```
As a default, the separtor of GROUP_CONCAT is ','. If you want to change it, put whatever you want to the second part of the clause. 
GROUP_CONCAT(EXP1,'|') 

### 7.3 Transfomration of Rows into Columns

### _This special 'transposition' is conditional on the fact that you know exactly the number of rows and thier type._

First, we need a base column which is used to group the values. In our case, dt is refered to as a base column. 

Second, count the number of columns you want to add to the existing table. The number is exactly the same as    
        that of unquie values under the column of indicator. 

Third, set up condition to assign a value to the proper column. 
       This process will iterate over until it reaces the last row. 
       In this stage, you have to be cautious to handle the date.That is, use MAX/MIN function to extract each value for newly created          fields. On every dt there is the only one TRUE value in response to CASE cluase available for each of newly created                      fileds;'impressions','sessions',and'users'. Use MAX/MIN to extract the value for every dt


 __"Note that the resulting output from CASE clause is a list. Even for only one single scalar, the resulting foramt is still
  list not a scalar.In order to extract the elment from the list contining the only one value, we often use MAX/MIN function"__
       

```MySQL
DROP TABLE IF EXISTS daily_kpi;
CREATE TABLE daily_kpi (
    dt        varchar(255)
  , indicator varchar(255)
  , val       integer
);

INSERT INTO daily_kpi
VALUES
    ('2017-01-01', 'impressions', 1800)
  , ('2017-01-01', 'sessions'   ,  500)
  , ('2017-01-01', 'users'      ,  200)
  , ('2017-01-02', 'impressions', 2000)
  , ('2017-01-02', 'sessions'   ,  700)
  , ('2017-01-02', 'users'      ,  250)
;


SELECT dt,
       MAX(CASE WHEN indicator='impressions' THEN val END) AS impressions,
	   MAX(CASE WHEN indicator='sessions' THEN val END) AS sessions,
       MAX(CASE WHEN indicator='users' THEN val END) AS users
       FROM daily_kpi
       GROUP BY dt
       ORDER BY dt;
```    
### 7.4 Concatenating Selected Rows into A Single String

The resulting outcome consists of the following fileds 

- purchase_id 

- product_id_array : to pick up the selected items by each consumer and return it in a array by using ARRAY_AGG() 

- product_id_string : to select all the products by each user_id and concatenate them into a string 

- amount : the total amount of spendings made by one cusotmer. 

This way of expression would make the end-users of your table more easily percevie what items each user has purchased. 

```sql
DROP TABLE IF EXISTS purchase_detail_log;
CREATE TABLE purchase_detail_log (
    purchase_id integer
  , product_id  varchar(255)
  , price       integer
);

INSERT INTO purchase_detail_log
VALUES
    (100001, 'A001', 3000)
  , (100001, 'A002', 4000)
  , (100001, 'A003', 2000)
  , (100002, 'D001', 5000)
  , (100002, 'D002', 3000)
  , (100003, 'A001', 3000)
;

SELECT purchase_id,
       ARRAY_AGG(product_id) AS product_id_array,
       STRING_AGG(product_id,',') AS product_id_string,
       SUM(price) AS total_sum
       FROM purchase_detail_log
       GROUP BY purchase_id
       ORDER BY purchase_id;
	
	
	
-- mysql version
SELECT purchase_id,
       GROUP_CONCAT(purchase_id SEPARATOR ',') AS product_ids,
       SUM(price) AS total_purchase
       FROM purchase_detail_log
       GROUP BY purchase_id;
		
```

### 7.5 Stacking Columns 

#### 7.5.1 A fixed number of columns in rows 

![image](https://user-images.githubusercontent.com/53164959/191708937-ee9fbed5-d04c-4cd7-914c-e01e5ca20730.pn

	
	
 In this section, we will introduce how to stack the values of columns into serveral rows into a group. Look at the table below, 
 and you can easily be aware that there are fixed number of columns in rows. Here create a pivot table
 whose serial number is listed in the same number as that of the columns and CROSS JOIN it. 
 
![image](https://user-images.githubusercontent.com/53164959/63136668-11aaeb80-c00e-11e9-986c-dce59e939523.png)

```sql
DROP TABLE IF EXISTS quarterly_sales;
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
  , (2017, 92000, 81000, NULL , NULL )
;

DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log (
    purchase_id integer
  , product_ids varchar(255)
);

INSERT INTO purchase_log
VALUES
    (100001, 'A001,A002,A003')
  , (100002, 'D001,D002')
  , (100003, 'A001')
;

SELECT q.year AS year,
       CASE WHEN p.idx=1 THEN 'q1'
            WHEN P.idx=2 THEN 'q2'
            WHEN p.idx=3 THEN 'q3'
            WHEN p.idx=4 THEN 'q4'
            END AS quarter,
       CASE WHEN p.idx=1 THEN q.q1
            WHEN p.idx=2 THEN q.q2
            WHEN P.idx=3 THEN q.q3
            WHEN p.idx=4 THEN q.q4 
            END AS sales
       FROM quarterly_sales AS 	q
       CROSS JOIN(
       SELECT 1 AS idx
       UNION ALL SELECT 2 AS idx
       UNION ALL SELECT 3 AS idx
       UNION ALL SELECT 4 AS idx
       )AS p

```

#### 7.5.2 Variable length of a column in rows 

Let's suppose that we have a table with two columns and one of them refers to uers id and the colum next to it tells you
the code of items each user has placed an order for. Numer of items will vary from one to another,thereby the length of 
a string in which selected items are stored being also different. We need to take a different approach to deal with this case. 

![image](https://user-images.githubusercontent.com/53164959/63137477-6734c780-c011-11e9-9694-48dedddba0b1.png)

First there is a need to convert string to an array. We can perform this conversion by using STRING_TO_ARRAY. 
Ater that, employ UNNEST function to take an array and create a table with a single row for each element in an array. 
The last step is simply CROSS JOIN it if you want to show purchase_id and product_id altoegher.


```sql
DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log (
    purchase_id integer
  , product_ids varchar(255)
);

INSERT INTO purchase_log
VALUES
    (100001, 'A001,A002,A003')
  , (100002, 'D001,D002')
  , (100003, 'A001')
;

SELECT purchase_id,
       product_id
       FROM purchase_log AS p
       CROSS JOIN UNNEST(STRING_TO_ARRAY(product_ids,',')) AS product_id ;
```

#### 7.5.3 put serial number on each product_ids 

```sql

WITH statics AS(
     SELECT purchase_id,
            product_id,
            LENGTH(product_id)-LENGTH(REPLACE(product_id,',',''))+1 AS len --to find out the total number of product in each array
            FROM purchase_log)
     SELECT DISTINCT s.purchase_id,
            s.product_id,
            p.index,
            SPLIT_PART(product_id,',',p.index) --placing product up to the total number
            FROM statics AS s
            --we need to make a series with index up to the greast number of product_ids 
            JOIN(SELECT GENERATE_SERIES(1,MAX(LENGTH(product_id)-LENGTH(REPLACE(product_id,',',''))+1) OVER()) AS index
                 FROM purchase_log) AS p
                 ON p.index<=s.len ; --to place on limit that the index can not exceed over the total number of prodcuts 
```

_NOTE_

An SQL JOIN clause is used to combine rows from two or more tables, based on a common field between them. There are different types of joins available in SQL: INNER JOIN: returns rows when there is a match in both tables. LEFT JOIN: returns all rows from the left table, even if there are no matches in the right table.

![image](https://user-images.githubusercontent.com/53164959/63138725-7cf8bb80-c016-11e9-8cd2-a774a14000a1.png)






       














       
   
 
 
 
 
 





 
 
       
    






	 
	     



	




 
