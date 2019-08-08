Aggregation Based On Multiple Dimensions
========================================

Depending on the nature of dimensions, we could implement different types of aggregation for the only single measure. Therefore,  we should first take a step of classifying the objects into proper categories and decide which aggregation is most plausible for each subgroup. 

### 1.Drill Down Approach

1.1 UNION ALL method
Here Drill Down approach-breaking down the complex system into progressively smaller parts-will kick in to help us design database systems for products labeled with subcategories as well as categories. 

First, we need a section for accommodating the fields -category, and subcategories for all products. 
The Nex step category section is classified into smaller parts with the subcategory section keeping the same as previous.  Lastly, we assigned specialized product lines to the sub category.   
All this process can be characterized as the table below.


<img width="576" alt="data1" src="https://user-images.githubusercontent.com/53164959/62681821-8a4dee80-b9f5-11e9-8d73-5bfc0f3d0ce7.png">

```sql
DROP TABLE IF EXISTS purchase_detail_log;
CREATE TABLE purchase_detail_log(
    dt           varchar(255)
  , order_id     integer
  , user_id      varchar(255)
  , item_id      varchar(255)
  , price        integer
  , category     varchar(255)
  , sub_category varchar(255)
);

INSERT INTO purchase_detail_log
VALUES
    ('2017-01-18', 48291, 'usr33395', 'lad533', 37300,  'ladys_fashion', 'bag')
  , ('2017-01-18', 48291, 'usr33395', 'lad329', 97300,  'ladys_fashion', 'jacket')
  , ('2017-01-18', 48291, 'usr33395', 'lad102', 114600, 'ladys_fashion', 'jacket')
  , ('2017-01-18', 48291, 'usr33395', 'lad886', 33300,  'ladys_fashion', 'bag')
  , ('2017-01-18', 48292, 'usr52832', 'dvd871', 32800,  'dvd'          , 'documentary')
  , ('2017-01-18', 48292, 'usr52832', 'gam167', 26000,  'game'         , 'accessories')
  , ('2017-01-18', 48292, 'usr52832', 'lad289', 57300,  'ladys_fashion', 'bag')
  , ('2017-01-18', 48293, 'usr28891', 'out977', 28600,  'outdoor'      , 'camp')
  , ('2017-01-18', 48293, 'usr28891', 'boo256', 22500,  'book'         , 'business')
  , ('2017-01-18', 48293, 'usr28891', 'lad125', 61500,  'ladys_fashion', 'jacket')
  , ('2017-01-18', 48294, 'usr33604', 'mem233', 116300, 'mens_fashion' , 'jacket')
  , ('2017-01-18', 48294, 'usr33604', 'cd477' , 25800,  'cd'           , 'classic')
  , ('2017-01-18', 48294, 'usr33604', 'boo468', 31000,  'book'         , 'business')
  , ('2017-01-18', 48294, 'usr33604', 'foo402', 48700,  'food'         , 'meats')
  , ('2017-01-18', 48295, 'usr38013', 'foo134', 32000,  'food'         , 'fish')
  , ('2017-01-18', 48295, 'usr38013', 'lad147', 96100,  'ladys_fashion', 'jacket')
 ;
 
 
 SELECT * FROM purchase_detail_log limit 5;
 WITH sub_category_amount AS
 (SELECT category,
         sub_category,
         SUM(price) AS amount
         FROM purchase_detail_log
         GROUP BY category,sub_category),
         category_amount AS(
         SELECT category,
         'all' as sub_category,
         SUM(price) AS amount
         from purchase_detail_log
         GROUP BY category),
         total_amount AS(
         SELECT
         'all' AS category,
         'all' AS sub_category,
         SUM(price) AS amount
         from purchase_detail_log)
         SELECT category,sub_category,amount FROM sub_category_amount
         UNION ALL SELECT category,sub_category,amount FROM category_amount
         UNION ALL SELECT category,sub_category,amount FROM total_amount;
         ```
The problems with this approach is 
- The query is quite lengthy.
- The performance of the query may not be good since the database engine has to internally execute two separate       queries and combine the result sets into one.
         
1.2 ROLL UP method 

We use ROLL up cluase (The ROLLUP generates multiple grouping sets based on the columns or expression specified in the GROUP BY clause.) to simplify our previous code. If either category or sub_category is NULL, then COALESCE
returns 'all'. 

```sql
SELECT COALESCE(category,'all') AS category,
        COALESCE(sub_category,'all') AS sub_category,
        SUM(price) AS amount
        FROM purchase_detail_log
        GROUP BY category,sub_category WITH ROLLUP;
```

### 3.Fan chart fro sales' growth

```sql
DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log
(dt VARCHAR(255) NULL ,
 order_id INT(11) NULL,
 user_id VARCHAR(255) NULL,
 purchase_amount INT(11) NULL,
 category varchar(20) NULL);
 
INSERT INTO purchase_log
VALUES
    ('2014-01-01',    1, 'rhwpvvitou', 13900,'DVD')
  , ('2014-02-08',   95, 'chtanrqtzj', 28469,'BOOK')
  , ('2014-03-09',  168, 'bcqgtwxdgq', 18899,'DVD')
  , ('2014-04-11',  250, 'kdjyplrxtk', 12394,'GAME')
  , ('2014-05-11',  325, 'pgnjnnapsc',  2282,'FOOD')
  , ('2014-06-12',  400, 'iztgctnnlh', 10180,'FASHION_MEN')
  , ('2014-07-11',  475, 'eucjmxvjkj',  4027,'GAME')
  , ('2014-08-10',  550, 'fqwvlvndef',  6243,'DVD')
  , ('2014-09-10',  625, 'mhwhxfxrxq',  3832,'FOOD')
  , ('2014-10-11',  700, 'wyrgiyvaia',  6716,'FASHION_WOMEN')
  , ('2014-11-10',  775, 'cwpdvmhhwh', 16444,'DVD')
  , ('2014-12-10',  850, 'eqeaqvixkf', 29199,'FOOD')
  , ('2015-01-09',  925, 'efmclayfnr', 22111,'FASHION_MEN')
  , ('2015-02-10', 1000, 'qnebafrkco', 11965,'FASHION_WOMEN')
  , ('2015-03-12', 1075, 'gsvqniykgx', 20215,'GAME')
  , ('2015-04-12', 1150, 'ayzvjvnocm', 11792,'DVD')
  , ('2015-05-13', 1225, 'knhevkibbp', 18087,'BOOK')
  , ('2015-06-10', 1291, 'wxhxmzqxuw', 18859,'FOOD')
  , ('2015-07-10', 1366, 'krrcpumtzb', 14919,'FASHION_MEN')
  , ('2015-08-08', 1441, 'lpglkecvsl', 12906,'DVD')
  , ('2015-09-07', 1516, 'mgtlsfgfbj',  5696,'FOOD')
  , ('2015-10-07', 1591, 'trgjscaajt', 13398,'GAME')
  , ('2015-11-06', 1666, 'ccfbjyeqrb',  6213,'FASHION_WOMEN')
  , ('2015-12-05', 1741, 'onooskbtzp', 26024,'FASHION_MEN')
;
```




