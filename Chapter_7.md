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
we could take.Unless OVER funcion is used with specifying any options,the aggreating function will be applied to the whole table(see the column named avg_score in our code where we are actully averaging all the scores on the table). However, f PARTITION BY caluse in the parenthesis of Over will determine which rows will be applied to the given functions. Let's look at how these skills are actually 
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

#### * OVER(ORDER BY)




 
