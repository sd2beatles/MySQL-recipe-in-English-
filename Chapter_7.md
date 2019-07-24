## CHAPTER 7 MANUPULATION OF DATA TABLE 
### 1) GROUPING 

GROUP BY function is very often used for grouping rows that have the same values into summary rows and accompanied 
with aggregate function(MAX,MIN,AVG etc) to group the result set by one or more columns. 


#### * Groupping The Result Set  

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
#### * Handling Before and After Groupping Data at A Time


 
