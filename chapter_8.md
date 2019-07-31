## Chapter 8 Management of Various Tables 

### 1) Combing Tables Vertically : Combing rows of one table to those of another.

Since each SELECT Statement must have the same number of fields(or columns), we  have two options left,one of which is simply to put it aside and the other is to grant default value. 

```MySQL
DROP TABLE IF EXISTS app1_mst_users;
CREATE TABLE app1_mst_users (
    user_id varchar(255)
  , name    varchar(255)
  , email   varchar(255)
);

INSERT INTO app1_mst_users
VALUES
    ('U001', 'Sato'  , 'sato@example.com'  )
  , ('U002', 'Suzuki', 'suzuki@example.com')
;

DROP TABLE IF EXISTS app2_mst_users;
CREATE TABLE app2_mst_users (
    user_id varchar(255)
  , name    varchar(255)
  , phone   varchar(255)
);

INSERT INTO app2_mst_users
VALUES
    ('U001', 'Ito'   , '080-xxxx-xxxx')
  , ('U002', 'Tanaka', '070-xxxx-xxxx')
;



SELECT 'app1' AS app_name,user_id,name,email FROM app1_mst_users
UNION ALL 
SELECT 'app2' AS app_name,user_id,name, NULL AS email FROM app2_mst_users;

```
- UNION ALL

(1)The SQL UNION ALL operator is used to combine the result sets of 2 or more SELECT statements.

(2)It does not remove duplicate rows between the various SELECT statements (all rows are returned).

(3)Each SELECT statement within the UNION ALL must have the same number of fields(or columns) in the result sets with similar data types.

- What is the difference between UNION and UNION ALL?

1)UNION removes duplicate rows.

2)UNION ALL does not remove duplicate rows.

### 2) Combing The Tables horizontally: adding extra columns to one existing table. 

category_id<center>|name<center>
|---------------:|-----------:|
1|dvd|
2|cd|
3|book|


category_id<center>|sales<center>
|---------------:|-----------:|
1|850000|
2|500000|

category_id<center>|rank<center>|prodcut_id<center>|sale<center>|
|----------------:|------------:|-----------------:|------------:|
1|1|D001|50000
1|2|D002|20000
1|3|D003|10000
2|1|C001|30000
2|2|C002|20000
2|3|C003|10000

Based on sales from each category, We want to create a separte reuslt set which includes category_id,name of category,
total sale made by each category,product_id of the top sale product. 


Method 1) LEFT JOIN
```MySQL
DROP TABLE IF EXISTS mst_categories;
CREATE TABLE mst_categories (
    category_id integer
  , name        varchar(255)
);

INSERT INTO mst_categories
VALUES
    (1, 'dvd' )
  , (2, 'cd'  )
  , (3, 'book')
;

DROP TABLE IF EXISTS category_sales;
CREATE TABLE category_sales (
    category_id integer
  , sales       integer
);

INSERT INTO category_sales
VALUES
    (1, 850000)
  , (2, 500000)
;

DROP TABLE IF EXISTS product_sale_ranking;
CREATE TABLE product_sale_ranking (
    category_id integer
  , ranks        integer
  , product_id  varchar(255)
  , sales       integer
);

INSERT INTO product_sale_ranking
VALUES
    (1, 1, 'D001', 50000)
  , (1, 2, 'D002', 20000)
  , (1, 3, 'D003', 10000)
  , (2, 1, 'C001', 30000)
  , (2, 2, 'C002', 20000)
  , (2, 3, 'C003', 10000)
;

SELECT m.category_id,m.name,s.sales,r.product_id AS top_sale_produt
       FROM mst_categories AS m
       LEFT JOIN category_sales AS s  
             ON m.category_id=s.category_id
	   LEFT JOIN product_sale_ranking AS r
              ON m.category_id=r.category_id
		         AND r.ranks=1;
```
If JOIN were used insteda of LEFT JOIN, there would be unexpceted problems arising: dupcliacte rows and omission of
unmatched ones.


method 2) Subquery

```MySQL
SELECT m.category_id,
	   m.name,
       (SELECT s.sales FROM category_sales AS s
		WHERE m.category_id=s.category_id)AS sales,
        (SELECT r.product_id FROM product_sale_ranking AS r
		 WHERE m.category_id=r.category_id
         ORDER BY sales DESC
         LIMIT 1 ) AS top_sale_product
         FROM mst_categories AS m;
 ```

- _what is good about Subquery way?_

The use of ORDER BY AND LIMIT in the subquery would relieve the burden of storing ranks of each cateogry in the pre-processed
result set. Therefore, in our case, we do not see any need of stroing the field named rank in product_sale_ranking.



user_id<center>|car_number<center>
|---------------:|-----------:|
U001|123-XXXX-XXXX|
U002||
U003|5678-XXXX-XXXX|
	
purchase_id<center>|user_id<center>|amount<center>|stamp<center>|
|----------------:|------------:|-----------------:|------------:|
|1001|U001|200|2017-01-30 10:00:00|
|1002|U001|500|2017-02-10 10:00:00|
|1003|U001|200|2017-02-12 10:00:00|
|1004|U002|800|2017-03-01 10:00:00|
|1005|U002|400|2017-03-02 10:00:00|
	

### 3) Creating Flag Columns 

In this section, we are trying to put a flag column to our desired result set - a filed containing value iether 0 or 1. 
The fields we will take to our resulting table are following below

- user_id
- car_number : NULL is permissible 
- purchase_count : indicates the number of purchases
- card_registration : 1 for registered / 0 for non-registered
- purchase_history  : 1 for yes  / 0 for no


```
USE sqldb;
DROP TABLE IF EXISTS mst_users_with_card_number;
CREATE TABLE mst_users_with_card_number (
    user_id     varchar(255)
  , card_number varchar(255)
);

INSERT INTO mst_users_with_card_number
VALUES
    ('U001', '1234-xxxx-xxxx-xxxx')
  , ('U002', NULL                 )
  , ('U003', '5678-xxxx-xxxx-xxxx')
;

DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log (
    purchase_id integer
  , user_id     varchar(255)
  , amount      integer
  , stamp       varchar(255)
);

INSERT INTO purchase_log
VALUES
    (10001, 'U001', 200, '2017-01-30 10:00:00')
  , (10002, 'U001', 500, '2017-02-10 10:00:00')
  , (10003, 'U001', 200, '2017-02-12 10:00:00')
  , (10004, 'U002', 800, '2017-03-01 10:00:00')
  , (10005, 'U002', 400, '2017-03-02 10:00:00')
;



SELECT m.user_id,m.card_number,COUNT(p.purchase_id) AS purchase_count,
       CASE WHEN card_number IS NOT NULL THEN 1
            ELSE 0 END AS card_registration,
		SIGN(COUNT(p.purchase_id)) AS purchase_history
        FROM mst_users_with_card_number AS m
        LEFT JOIN purchase_log AS p
              ON m.user_id=p.user_id
		GROUP BY m.user_id;
```

### Special Note
Let's take a look at SIGN(COUNT(p.purchase_id)) in pieces. First Count return either 0 or integer value if purchse_id is more than one.
Then, using the function of SIGN where it returns zero for zero parameter whearas returing 1 for any positive integer. 


### 4) Avoiding Recursive and Lengthy Subquries
In mangaing somewhat complicating quries, repeating the same subquires over and over seems to be inevitable. It thie repetitive code
appears often in our code, it would be unpleasant and hard for readers to catch up the lines, leading to thier lower readability. 
Thefore, CTE ,which stands for Common TABLE Expression, kicks in to deal with this issue -simplifying complex joins and subqueries.

CTE follows the below caluse 
 _"with <talbe name> AS (SELECT ~)"_

step 1 ) preapre the 'raw' data
```MySQL
DROP TABLE IF EXISTS product_sales;
CREATE TABLE product_sales (
    category_name varchar(255)
  , product_id    varchar(255)
  , sales         integer
);

INSERT INTO product_sales
VALUES
    ('dvd' , 'D001', 50000)
  , ('dvd' , 'D002', 20000)
  , ('dvd' , 'D003', 10000)
  , ('cd'  , 'C001', 30000)
  , ('cd'  , 'C002', 20000)
  , ('cd'  , 'C003', 10000)
  , ('book', 'B001', 20000)
  , ('book', 'B002', 15000)
  , ('book', 'B003', 10000)
  , ('book', 'B004',  5000)
;
```
step 2) create tables for 'multiple uses'

- rank
- dvd
- dvd_sale
- cd
- cd_sales
- book
 book_sales

_"If you want to create more than one table by using 'WITH', comma must be defined before preapring the next one."_

```MySQL
WITH
product_sale_ranking AS(
SELECT
     category_name,
	 product_id,
     sales,
     ROW_NUMBER() OVER(PARTITION BY category_name ORDER BY sales DESC) AS ranks
     FROM product_sales)
     ,mst_ranks AS(
     SELECT DISTINCT ranks
     FROM product_sale_ranking
	)
SELECT 
	 m.ranks
     ,r1.product_id AS dvd
     ,r1.sales AS dvd_sale
     ,r2.product_id AS cd
     ,r2.sales AS cd_sale
     ,r3.product_id AS book
     ,r3.sales AS book_sale
     FROM mst_ranks AS m
     LEFT JOIN 
     product_sale_ranking AS r1
     ON m.ranks=r1.ranks AND r1.category_name='dvd'
     LEFT JOIN
     product_sale_ranking AS r2
     ON m.ranks=r2.ranks AND r2.category_name='cd'
     LEFT JOIN
     product_sale_ranking AS r3
     ON m.ranks=r3.ranks and r3.category_name='book'
     ORDER BY m.ranks;

```

### 5) Multiple Similar Table
Let's suppose you have a great diversity of orignal thoughts and ides about how to design your own database tables. Unfortunately,
you are denied an access or right to create them. In this cse, constructing "similar" tables will statify your desire.

- CASE STUDY 1) Arranging Code Values After Thier Lables ( LEFT JOIN apporach) 

method 1) prepare the pre-proccessed data
```sql
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

method 2) create a similar table 
```sql
WITH mst_id_labels AS(
     SELECT 1 AS device_id , 'PC' AS device_name
         UNION ALL SELECT   2 AS device_id , 'SP' AS  device_name
         UNION ALL SELECT   3 AS device_id , 'APP' AS device_name)
         SELECT
              distinct user_id,
              d.device_id,
              d.device_name
		FROM mst_users AS u
       LEFT JOIN mst_id_labels AS d
            ON u.register_device=d.device_id;
```
|usr_id<center>|device_id<center>|device_name<center>|
|------------:|-----------------:|-----------------:|
 |U001|1|PC|
 |U002|2|SP|
 |U003|3|APP|
	
		 
	
	
 

