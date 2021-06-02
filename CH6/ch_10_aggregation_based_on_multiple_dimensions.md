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
 

WITH all_category AS(
     SELECT CAST('all' AS VARCHAR) AS category,
            CAST('all' AS VARCHAR) AS sub_category,
            SUM(price) AS sale
            FROM purchase_detail_log),
     sub_category AS(
     SELECT category AS category,
           CAST('all' AS VARCHAR) AS sub_category,
           SUM(price) AS sale
           FROM purchase_detail_log
           GROUP BY category),
      sub_all_category AS(
      SELECT category AS category,
             sub_category AS sub_category,
             SUM(price) OVER(PARTITION BY category,sub_category) AS sale
             FROM purchase_detail_log)
      SELECT * FROM sub_all_category
            UNION ALL SELECT * FROM sub_category
            UNION ALL SELECT * FROM all_category;
             
                
	 
	 
The problems with this approach is 
- The query is quite lengthy.
- The performance of the query may not be good since the database engine has to internally execute two separate       queries and         combine the result sets into one.
- To prevent mismatch in data-type, we need to cast the section into proper type. 
  
1.2 ROLL UP method 

We use ROLL up cluase (The ROLLUP generates multiple grouping sets based on the columns or expression specified in the GROUP BY clause.) to simplify our previous code. If either category or sub_category is NULL, then COALESCE
returns 'all'. 

```sql
SELECT COALESCE(category,'all') AS category,
       COALESCE(sub_category,'all') AS sub_category,
       SUM(price) AS amount
       FROM purchase_detail_log
       GROUP BY ROLLUP(category,sub_category)
       ORDER BY category='all',sub_category='all';
```
ROLLUP operators let you extend the functionality of GROUP BY clauses by calculating subtotals and grand totals for a set of columns. The CUBE operator is similar in functionality to the ROLLUP operator; however, the CUBE operator can calculate subtotals and grand totals for all permutations of the columns specified in it.
### 2 ABC Analysis

ABC analysis is commonly used in the management of inventory at companies. The basic method is to categorize the products into ranks from A to C based on some factors including the proportion of sales revenue, market share and so on. After placing priorities on each category, we will plan a proper strategy for each section. 

The steps for preparing ABC analysis. 
First, arrange data to sales revenue in descending order.
Second, compute the total sum of all categories
Third, using the total sum to calculate a proportion of each category.
Lastly, prepare the cumulate sum and set the arbitrary intervals at your
            own disposal or accordingly to rank the listed categories. 


```sql
WITH category_info AS(
     SELECT category,
           SUM(price) AS amount
           FROM purchase_detail_log
           GROUP BY category),
      cal_index AS(
      SELECT category,
             amount, 
             100*amount/SUM(amount) OVER() AS rate
             FROM category_info
             ORDER BY amount DESC)
      ,cum_sum AS(
       SELECT category,
              amount,
              rate,
              SUM(rate) OVER(ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum_rate
              FROM cal_index)
     SELECT category,
            amount,
            rate,
            cum_rate,
            CASE WHEN cum_rate BETWEEN 0 AND 75 THEN 'A'
                 WHEN cum_rate BETWEEN 75 AND 90 THEN 'B'
                 WHEN cum_rate BETWEEN 90 AND 101 THEN 'C'
                 END AS class
            FROM cum_sum;
```
![image](https://user-images.githubusercontent.com/53164959/63639407-901e2200-c6cd-11e9-8446-588f4589f705.png)

Export the outcome to csv file and draw a barchart using the exported infomrtion. 

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

data=pd.read_csv('test.csv')
sns.set()
category=data.category
ax=data.pivot_table("rate",index='category',columns='class')
ax=ax.reindex(category)
ax.plot(kind="bar")
```
![image](https://user-images.githubusercontent.com/53164959/63639437-002ca800-c6ce-11e9-87b3-1f51c2bef59b.png)



### 3.Fan chart fro sales' growth
A fan chart is a chart that joins a line graph for observed past data and gives some hints of the movement of sales along with the given time interval. At your dispoal, pick any month of the year as a reference point and the amount of revenue generated in that month is now being considered as a based amount. Now, a rate is now computed 
with the following formula ;

rate = current_amount/ base_amount *100 

If the rate returns 100 %, there is no change in Sales as of the base amount. If it is more than 100 %, Sales have improved over the period where the opposite is true.


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


SELECT * FROM purchase_log;
WITH daily_category AS(
     SELECT dt,
            category,
            SUBSTRING_INDEX(dt,'-',1) AS year,
            SUBSTR(dt,6,2) AS month,
            SUBSTRING_INDEX(dt,'-',-1) AS date,
            SUM(purchase_amount) AS amount 
            FROM purchase_log
            GROUP BY dt,category 
            ORDER BY dt,category),
		monthly_category AS(
        SELECT dt,
        CONCAT(year,'-',month) AS year_months,
        SUM(amount) AS amount,
        category
        FROM daily_category
        GROUP BY year,month,category
        ORDER BY year,month,category
        )
        SELECT  year_months,
                category,
				amount,
                FIRST_VALUE(amount) OVER(PARTITION BY category ORDER BY year_months ROWS 
                BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS base_amount,
                100.0*amount/FIRST_VALUE(amount) OVER(PARTITION BY category ORDER BY year_months ROWS 
                BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rate
                FROM monthly_category 
                ORDER BY year_months,category;
```

### 4. Histogram 

4.1 Histogram Analaysis

- The shape of a histogram is symmetrically distributed around the central value,
  which indicates that there should a peak(ie the highest value of all the data) 
  occur around the mean and mode. 

-The graph is skewed left since a few small values are driven by outliers, but
 do not affect the median. The mode i is said to be greater than mean. 
 
 -This is the case because skewed-right data have a few large values that drive the mean upward but do not affect 
  where the exact middle of the data is (that is, the median).

- When the graph is bimodal, the only defining chracteristics about this distribution is that it has 2 peaks of 
  the same height.

4.2 Ways to Construct a Freuqency Distribution Table 

Step 1) Calculte an range(ie) range =maximum - minimum

Ste 2) Based on the rage, decide how many mutaully exclusive classes are created.

step 3) Compute the number of occurences in a class. Preapre a table to record all the information. 

To bear in mind that the histogram is preapred based on the continuous variable, we should not place a space between each class or interval. 


4.3 MySQL code

4.3.1 Preparing Data Class

```SQL
DROP TABLE IF EXISTS purchase_log;
CREATE TABLE purchase_log
(dt VARCHAR(255) NULL ,
 order_id INT(11) NULL,
 user_id VARCHAR(255) NULL,
 price INT(11) NULL,
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

WITH stat AS (SELECT MAX(price) AS max_price,
        MIN(price) AS min_price, 
        MAX(price)-MIN(price) AS range_price,
		10 AS bucket_num
        FROM purchase_detail_log
        ),
        purchase_log_with_bucket AS(
        SELECT price,
               min_price,
			   price-min_price AS diff,
               range_price/bucket_num AS bucket_range, #setting equal length for every interval
               FLOOR(((price-min_price)/(range_price/bucket_num))+1) AS bucket #assigning the value to each bucket
               FROM purchase_detail_log,stat)
		SELECT price,
			   min_price,
               diff,
               bucket_range,
               bucket
               FROM purchase_log_with_bucket
               ORDER BY bucket;
```

If you want to divide numbers over which data ranges into an interval of equal length, 
substracting the min_price from max_price is calcuated and divided by a nominal value as diff. 
The assignement of value to the proper class is determined by price_range(price-min_pirce) divided by the nominal value and use FLOOR function to discard decimal points. 

<img width="576" alt="data1" src="https://user-images.githubusercontent.com/53164959/62750257-790aed80-ba9a-11e9-8519-aa23de888bee.png">

The total number of classes is restricted to 10, thereby any values outside this range classified as 11. Now 
Now we make small modifications on our code to include the value into our class levels. 


```sql
WITH stat AS (SELECT MAX(price)+1 AS max_price, # modification here
        MIN(price) AS min_price, 
        MAX(price)+1-MIN(price) AS range_price, # modifiction here 
		10 AS bucket_num
        FROM purchase_detail_log
        ),
        purchase_log_with_bucket AS(
        SELECT price,
               min_price,
			   price-min_price AS diff,
               range_price/bucket_num AS bucket_range,
               FLOOR(((price-min_price)/(range_price/bucket_num))+1) AS bucket
               FROM purchase_detail_log,stat)
		SELECT price,
			   min_price,
               diff,
               bucket_range,
               bucket
               FROM purchase_log_with_bucket
               ORDER BY bucket;

```

4.3.2 Preapre A Table for Histogram
```sql
WITH stat AS (SELECT MAX(price)+1 AS max_price,
        MIN(price) AS min_price, 
        MAX(price)+1-MIN(price) AS range_price,
		10 AS bucket_num
        FROM purchase_detail_log
        ),
        purchase_log_with_bucket AS(
        SELECT price,
               min_price,
			   price-min_price AS diff,
               range_price/bucket_num AS bucket_range,
               FLOOR(((price-min_price)/(range_price/bucket_num))+1) AS bucket
               FROM purchase_detail_log,stat)
		SELECT bucket,
               min_price+bucket_range*(bucket-1) AS lower_lmit
               ,min_price+bucket_range*bucket AS uppper_limit
               ,COUNT(price) AS num_purchase
               ,SUM(price) AS total_amount
               FROM purchase_log_with_bucket
               GROUP BY bucket,min_price,bucket_range
               ORDER by bucket;
```
4.3.3 Control Over the Interval Size

The resulting outcome from the previous section shows that each limit is recorded in a decimal point, which makes end-users of the data hard to understand the defining characteristics. Therefore, it is a good practice of converting them into an integer by modifying max, min, and range at disposal of data analysts

```sql
WITH stats AS(
     SELECT 0 AS min_price,
            30000 AS max_price,
	    30000 AS range_price,
            10 AS bucket_num
            FROM purchase_log),
	  purchase_log_freq AS(
      SELECT price,
             min_price,
             price-min_price AS diff_price,
			 range_price/bucket_num AS interval_value,
             FLOOR((price-min_price)/(range_price/bucket_num))+1 AS bucket
             FROM purchase_log,stats
             ORDER BY bucket,price)
	 SELECT bucket, 
            min_price+interval_value*(bucket-1) AS lower_limit,
            min_price+interval_value*(bucket) AS upper_lmit,
            COUNT(price) AS num_purchase,
            SUM(price) AS total_sum
            FROM purchase_log_freq
            GROUP BY bucket
            ORDER BY bucket;
```
4.4 Pratical Tips

- If the graph looks bimodal, it is always worthwhile to classify the whole population into two specific sub-groups,   a separate histogram being drawn foreach classified one.

- When instructed to take an investigation on causes to either rise or fall in
  the sales revenue, it is strongly advisable to first prepare two histograms
  each based on "recent and past sales, respectively.  Right after the graphs are 
  done, compare one to another to identify if there are any differences occurring
  between them.








