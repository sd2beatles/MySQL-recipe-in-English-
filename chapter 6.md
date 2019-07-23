## CHAPTER 6 Manupulation of Various Values

### 1) Concatenating letters
The Section itself implies that I have tried to link all the given letters in an manner we want to present to end users. 
The 'concat; is used for linking letters and I have put ' ' for leaving a space betweeen pref_name and city_name. 

method 1) prepare the data
```mysql
DROP TABLE IF EXISTS mst_user_location;
CREATE TABLE mst_user_location
(userID  VARCHAR(7) NOT NULL PRIMARY KEY,
 pref_name CHAR(30) NOT NULL,
 city_name CHAR(30) NOT NULL);
 
INSERT INTO mst_user_location 
       VALUES('U001','42 Parkstone Ave','ChristChurch'),
             ('U002','12 Queenswood St','Auckland'),
             ('U003','14-A Queens St','Auckland');
```
method 2) concating the letters

```mysql
SELECT CONCAT(pref_name,' ',city_name) AS pref_city 
       FROM mst_user_location;
```

### 2) Comparing Values 

The table shows the sale of each quater from 2015 to 2017. Assuming there were no track records  in the 3 and 4 quarter of 2017, I have inserted NULL for these periods. As for 2018, all columns consist of NULL. 

```mywsql
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
  , (2017, 92000, 81000, NULL ,NULL)
  ,(2018,NULL,NULL,NULL,NULL)
;

```

#### * Measuremnet for Sale's Movement
It is crucial for mangers to see there is any change in sale's revenue from thier company. Therefore, I have made 
there separate sections ,which make it more convenient and handy to conduct thier quarterly analysis. 

-movment_q2_q1  '-' : loss in sale's compared to previous quarter , '+'  gain , ' ' no change at all 

-diff_q2_q1  the name implies absolute value of the diffrence between two quarters. 

-sign_q2_q1  '-1' , loss in sale's compared to previous quarter , '1'  gain , ' 0' no change at all 

```mysql
SELECT year,q1,q2,
       CASE 
           WHEN q1<q2 THEN '+'
           WHEN q1=q2 THEN ' '
           ELSE '-'
           END AS movement_q2_q1
		,ABS(q2-q1) AS diff_q2_q1
        ,SIGN(q2-q1) AS sign_q2_q1
        FROM quarterly_sales
        ORDER BY year;
```

#### * Finding The Lowest and Greatest Sale among The Qauaters 

This taks took me a quite longer than What I had previously expected. This is primarly beacuse  MySQL does not currently provide
any functions which generates either lowest,greatest value among 'non-null' columns. Therefore, I came out with indirect ways to
get those values with the same codes repeated over and over. 

As for finding the greast value, since the sale can not be lower than zero, the value automatically returns zero if any column on the talbe contains zero by using 'COALESCE' ,which is followed by finding the greatest value among four non-Null values. Otherwise,'NULL' automatically would appear.


In obtaining the lowest values in each year,four cases are considered.
First, if all of the columns do not contain null value, then simply implment the least function where values are not affected by NULL.
For the  next two remaining cases, ifnull is employed to find the least value. If a case does not fit to any of them, simply return 'NULL'. 


```mysql
SELECT year,
     /* finding the greatest values*/
    case
    WHEN GREATEST(COALESCE(q1,0),COALESCE(q2,0),COALESCE(q3,0),COALESCE(q4,0))=0 THEN 'NULL'
    ELSE GREATEST(COALESCE(q1,0),COALESCE(q2,0),COALESCE(q3,0),COALESCE(q4,0)) 
    END AS greatest_value,
	
    /* obtaining the lowest values*/
    CASE
    WHEN (q1 is NOT NULL AND q2 is NOT NULL AND q3 IS NOT NULL AND q4 IS NOT NULL) THEN least(q1,q2,q3,q4)
    WHEN ifnull(ifnull(q1,q2),ifnull(q3,q4))=q1 then least(q1,q2) 
    WHEN ifnull(ifnull(q1,q2),ifnull(q3,q4))=q3 then least(q3,q4)
    ELSE 'NULL'
    END AS least_value
    from quarterly_sales
    ORDER BY year;
```

#### * Finding Avaerage 

```MySQL
SELECT year,(COALESCE(q1,0)+COALESCE(q2,0)+COALESCE(q3,0)+COALESCE(q4,0))/4 AS average
FROM quarterly_sales
ORDER BY year;
```






