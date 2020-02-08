# Introduction

It is not unusual for market researchers to actively seek out ways to understand their customers' actions and behavior 
for sustainable business success either in the short or long term. Up to this point, we have seen many marketing measures 
to check out the current status of the service to customers.  

In this section,  we are to suggest indicators of the entire of the supply chain of an organization, which are based on the customer lead time. 
The "lead time"  is often defined as the time it takes for customers to receive a good or service from the time the customer places the order or performs a purchase transaction. This spans will help us to retrieve 
a  lot of information including the customer purchasing pattern, any problems associated with the current business plans and so on. 


### 1. Time Diffrence Between Registration and Action Date 
In our case, our mearues is concerned with only two records in the table. The calculation is so easy and simple to compute. 

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

### 2. 
