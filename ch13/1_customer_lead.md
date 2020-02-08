# Introduction

It is not unusual for market researchers to actively seek out ways to understand their customers' actions and behavior 
for sustainable business success either in the short or long term. Up to this point, we have seen many marketing measures 
to check out the current status of the service to customers.  

In this section,  we are to suggest indicators of the entire of the supply chain of an organization, which are based on the customer lead time. 
The "lead time"  is often defined as the time it takes for customers to receive a good or service from the time the customer places the order or performs a purchase transaction. This spans will help us to retrieve 
a  lot of information including the customer purchasing pattern, any problems associated with the current business plans and so on. 


### 1. Time Diffrence Between Registration and Action Date 
In our case, our table only contains two relevant columns,and this makes it  so easy and simple to compute the measure. 

```sql
WITH reservation(reservation_id,register_date,visit_date,days) AS(
	VALUES(1,DATE '2016-09-01',DATE '2016-10-01',3),
	      (2,DATE '2016-09-20',DATE '2016-10-01',2),
	      (3,DATE '2016-09-30',DATE '2016-11-20',2),
	      (4,DATE '2016-10-01',DATE '2017-01-03',2),
          (5,DATE '2016-11-01',DATE '2016-12-28',3))
    SELECT reservation_id,
	       register_date,
		   visit_date,
		   CONCAT(CAST(visit_date-register_date AS text),' days') AS lead_time
		   FROM reservation;
```

### 2. Columns Across The tables

It is more than often to see that columns of our interest are spread across the tables. In this case, we need to capitalize on one of the SQL queries to bring them altogether.  

```sql
WITH request(user_id,product_id,request_date) AS(
   VALUES('U001','1',DATE '2016-09-01'),
	      ('U002','2',DATE '2016-09-02'),
	      ('U003','3',DATE '2016-09-30'),
	      ('U004','4',DATE '2016-10-01'),
	      ('U005','5',DATE '2016-11-01'))
   ,estimates(user_id,product_id,estimate_date) AS(
     VALUES('U001','1',DATE '2016-09-21'),
           ('U002','2',DATE '2016-10-15'),
           ('U003','3',DATE '2016-10-15'),
           ('U004','4',DATE '2016-12-01'),
            ('U005','5',DATE '2016-12-12')) 
    ,orders(user_id,product_id,order_date) AS(
	  VALUES('U001','1',DATE '2016-10-01'),
			 ('U004','4',DATE '2016-12-05'),
			 ('U005','5',DATE '2016-12-23'))
	SELECT r.user_id,
	       r.product_id,
		   e.estimate_date-r.request_date AS estimate_lead_time,
		   o.order_date-r.request_date AS total_lead_time
	      FROM request AS r
		  LEFT JOIN estimates AS e
		  ON r.user_id=e.user_id AND r.product_id=e.product_id
		  LEFT JOIN orders AS o
		  ON r.user_id=o.user_id AND r.product_id=o.product_id;

```

### 3. Time Between Orders 

To understand how frequently customers make purchases, marketers often rely on time purchase analysis. 
Before implementing it, we should manage the data in a manner to extract the information for this analysis. 
We would not dive into the detailed procedure for the analysis. Instead, we just prepare a lead time which represents time difference
between purchase dates. 

```sql
WITH purchase_log(user_id,product_id,purchase_date) AS(
    VALUES('U001','1','2016-09-01'),
	      ('U002','2','2016-09-20'),
	      ('U003','3','2016-09-30'),
	      ('U001','4','2016-10-01'),
	      ('U002','5','2016-11-01'),
	      ('U002','6','2016-11-23')
  )
  SELECT user_id,
         purchase_date,
		 purchase_date::DATE - LAG(purchase_date::DATE) OVER(PARTITION BY user_id ORDER BY purchase_date) AS lead_time
		 FROM purchase_log;

```
