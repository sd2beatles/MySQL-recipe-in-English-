```
Acrroding to most books and web sties, they state that MySQL is known for not providing a window funciton.However, the version of 8
does support some including ROW_NUMBER,RANK(),DESE_RANK() and so on. In the case where MySQL,you hve no choice but to make use of
user-defined variable to solve the issues. 
```


## CHAPTER 7 MANUPULATION OF DATA TABLE 
### 1) GROUPING 

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
### 2) THE Syntax of OVER CLUASE 
#### * OVER() VS OVER(PARTITION BY)

In order to present individual data and ones after aggreate functions is implemented  all together, OVER is the most suitable function. 
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

#### * OVER(ORDER BY) and Ranking The Rows 
First,we want to arrange rows in either descending or ascedning order  with a help of  ORDER BY clause. In addition, 
there are three separate window functions to label the ranks of rows of a result-set, each of which has a distinct feature. 

- ROW_NUMBER() : ranks the rows with no overlap 
- RANK() : allows for overlap , but 'leaping' or 'skiping' in the rank 
- DENSE_LANK()  : allows for overlap and no 'leaping' or 'skiping' in the rank

Furthermore, it is a worthy of remembering other window functions such as "LEAD","LAG". 

- LEAD() : to return a value from  the next row
- LAG() :  to return a value from the previous row 

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
#### * OVER() clause and Aggregating Functions 

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
       
    






	 
	     



	




 
