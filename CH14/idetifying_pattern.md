### Chater 14 Introduction 
There are some useful metrics we should track to find any hidden purchasing patterns of our customers. In this section, we will explore how to compute these metrics and add explanations on how to obtain useful insights based on them. 


#### 14.1 <Metric Part 1> Number of Vistors,Number of Frequencies and Page Views. 

Unfortunately, in web-log analysis, these two terms are interchangeably used to lead to an unintentional confusion to the third users. To prevent this from happening,  we need to clarify some of the most widely used terms.

1)Clicks VS Vist

We need to make a difference between the two terminologies.

As for clicks, the clicks in the column indicate the number of times a visitor clicked. Please note that more than one click by each customer in a given data counts toward the total. On the other hand, the visit is simply the number of net sessions initiated by the visitor.  


2) page views

Page Views indicates the number of times for which each user visits a tracked website. When a visitor reaches the page and clicks the refresh button, an additional pageview is recorded. A new pageview is also recorded when the user navigates to another page and then returns to the original page.


```sql

drop table if exists visits_website;
CREATE TABLE  visits_website (
    id int,
    browser_short VARCHAR(45) NOT NULL,
    browser_long VARCHAR(255) NOT NULL,
	ip varchar,
	create_at date NOT NULL,
	url varchar,
	referrer varchar);

INSERT INTO visits_website VALUES ('1', 'ip1', 'ip1', '', '2016-08-31 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('2', 'ip1', 'ip1', '', '2016-08-31 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('3', 'ip2', 'ip2', '', '2016-08-31 19:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('4', 'ip1', 'ip1', '', '2016-08-31 19:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('5', 'ip2', 'ip2', '', '2016-08-31 18:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('6', 'ip3', 'ip3', '', '2016-08-31 18:30:00','http://www.example.com/list/cd','');
INSERT INTO visits_website VALUES ('7', 'ip1', 'ip1', '', '2016-08-31 17:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('8', 'ip2', 'ip2', '', '2016-08-31 17:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('9', 'ip3', 'ip3', '', '2016-08-31 16:30:00','http://www.example.com/list/cd','');
INSERT INTO visits_website VALUES ('10', 'ip4', 'ip4', '', '2016-08-31 16:30:00','http://www.example.com/list/newly','http://search.yahoo.co.jp/xxx');
INSERT INTO visits_website VALUES ('11', 'ip1', 'ip1', '', '2016-08-30 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('12', 'ip2', 'ip2', '', '2016-08-30 20:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('13', 'ip3', 'ip3', '', '2016-08-30 20:30:00','http://www.example.com/list/cd','');
INSERT INTO visits_website VALUES ('14', 'ip4', 'ip4', '', '2016-08-30 20:30:00','http://www.example.com/list/newly','http://search.yahoo.co.jp/xxx');
INSERT INTO visits_website VALUES ('15', 'ip5', 'ip5', '', '2016-08-30 20:30:00', 'http://www.example.com/?utm_source=mynavi&utm_medium=affiliate','http://www.mynavi.jp/xxx');
INSERT INTO visits_website VALUES ('16', 'ip1', 'ip1', '', '2016-08-30 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('17', 'ip2', 'ip2', '', '2016-08-30 20:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('18', 'ip3', 'ip3', '', '2016-08-30 20:30:00','http://www.example.com/list/cd','');
INSERT INTO visits_website VALUES ('19', 'ip4', 'ip4', '', '2016-08-30 20:30:00','http://www.example.com/?utm_source=mynavi&utm_medium=affiliate','http://www.mynavi.jp/xxx');
INSERT INTO visits_website VALUES ('20', 'ip5', 'ip5', '', '2016-08-30 20:30:00','http://www.example.com/list/dvd', 'https://www.facebook.com/xxx' );
INSERT INTO visits_website VALUES ('21', 'ip6', 'ip6', '', '2016-08-29 20:30:00','http://www.example.com/detail?id=2', 'https://twitter.com/xxx'    );
INSERT INTO visits_website VALUES ('22', 'ip1', 'ip1', '', '2016-08-29 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx');
INSERT INTO visits_website VALUES ('23', 'ip2', 'ip2', '', '2016-08-29 20:30:00','http://www.example.com/detail?id=1','');
INSERT INTO visits_website VALUES ('24', 'ip3', 'ip3', '', '2016-08-29 20:30:00','http://www.example.com/list/cd','');
INSERT INTO visits_website VALUES ('25', 'ip4', 'ip4', '', '2016-08-29 20:30:00','http://www.example.com/list/newly','http://search.yahoo.co.jp/xxx');
INSERT INTO visits_website VALUES ('26', 'ip5', 'ip5', '', '2016-08-29 20:30:00','http://www.example.com/?utm_source=mynavi&utm_medium=affiliate','http://www.mynavi.jp/xxx');
INSERT INTO visits_website VALUES ('27', 'ip6', 'ip6', '', '2016-08-29 20:30:00','http://www.example.com/list/dvd', 'https://www.facebook.com/xxx' );
INSERT INTO visits_website VALUES ('28', 'ip7', 'ip7', '', '2016-08-29 20:30:00','http://www.example.com/detail?id=1','https://twitter.com/xxx'    );
INSERT INTO visits_website VALUES ('29', 'ip1', 'ip1', '', '2016-08-29 20:30:00','http://www.example.com/?utm_source=google&utm_medium=search','http://www.google.co.kr/xxx'  );
INSERT INTO visits_website VALUES ('30', 'ip2', 'ip2', '', '2016-08-29 20:30:00','http://www.example.com/detail?id=1','');
```
```sql

select  distinct create_at as dt,
        count(distinct browser_long) as access_visits,
		count(distinct browser_short) as access_counts,
		count(*) as page_view,
		count(*)/count(distinct browser_long) as page_per_user
		from visits_website
		group by dt
		order by dt;
		

```

### 14.2 <Metric 2> URL Path

We may come up with another method to segmentize the data. You may consider taking the URL as a metric to divide the data into a variety of groups and compute its counts. But the problem here is that the number of classification should be small and make it worse to develop a better insight into the consumer pattern. Instead, it seems to be better for us to make a url path as a standard.


```sql
with acess_log_path as(
   select distinct substring(url from '//[^/]+([^?#]+)') as url_path,
	      count(distinct browser_short) as user_counts,
	      count(distinct browser_long) as user_freq
	      from visits_website
	      group by url_path)
   select * from acess_log_path;

```
![image](https://user-images.githubusercontent.com/53164959/87244027-d612de80-c475-11ea-82f0-4719936f5e86.png)


The approach we have taken seems to be sound and logical to begin. However, it has its drawbacks in that some of the rows are designated as a list. We are currently looking at a big picture of how many customers visit the given website through a given path. Either too specialized or categorized URL path would bring out an unintentional difficulty and confusion to the analysts. We need to conglomerate the relevant items into one and assign a proper name.

```sql

with prep as(
 select distinct substring(url from '//[^/]+([^?#]+)') as url_path,
	    count(distinct browser_short) as counter_access,
	    count(distinct browser_long) as counter_vistors
	    from visits_website
	    group by url_path),
    split_part as(
       select *,
        split_part(url_path,'/',2) as path1,
        split_part(url_path,'/',3) as path2
        from prep),
    assign_access_log as(
	select *,
           case when path1='list' then
		        case when path2='newly' then 'newly'
		             else 'category_list' end
		        else url_path end as page_name
	      from split_part)
	select page_name,
	       count(counter_access) as access_count,
		   count(counter_vistors) as access_vistors
		   from assign_access_log
		   group by page_name;
```

![image](https://user-images.githubusercontent.com/53164959/87244579-1162dc80-c479-11ea-9627-ad06b609fe0c.png)

### 14.3 Logical Ways of Counting The Data

If your generated URL contains extra parameters such as campaign, medium, or so on, we need to seek out the more sophisticated ways to count the data. Here is a simple logic we could take to make the groups and count the number of visitors for each corresponding group.


```sql
with prep as(
  select *,
	     substring(url from 'http://([^/]*)') as url_domain,
	     substring(url from 'utm_source=([^&]*)') as utm_source,
	     substring(url from 'utm_medium=([^&]*)') as utm_medium,
	     substring(referrer from 'https?://([^/]*)') as referrer_domain
	     from visits_website)
  ,access_via as(
   select *,
	      row_number() over(order by create_at) as log_id ,
	      case when utm_source<>'' and utm_medium<>'' then concat(utm_source,'-',utm_medium) 
               when referrer_domain in ('www.facebook.com','twitter.com') then 'social'
	           when referrer_domain in ('search.yahoo.co.jp','www.google.co.kr') then 'search'
	           else 'others' end as via
	  from prep)
  select via,
         count(1) as acess_count
		 from access_via
		 group by via
		 order by acess_count desc;

```
![image](https://user-images.githubusercontent.com/53164959/87295975-03788e80-c541-11ea-915d-5797a272fa34.png)

### 14.4 Conversion,CVR,and Amount Purchased

Let's assume we have two separate tables, one of which is on the login infomrton where the other for purchase history. 
```sql


INSERT INTO access_log
VALUES
    ('2016-10-01 12:00:00', '0CVKaz', '1CwlSX', 'http://www.example.com/?utm_source=google&utm_medium=search'       , 'http://www.google.co.jp/xxx'      )
  , ('2016-10-01 13:00:00', '0CVKaz', '1CwlSX', 'http://www.example.com/detail?id=1'                                , ''                                 )
  , ('2016-10-01 13:00:00', '1QceiB', '3JMO2k', 'http://www.example.com/list/cd'                                    , ''                                 )
  , ('2016-10-01 14:00:00', '1QceiB', '3JMO2k', 'http://www.example.com/detail?id=1'                                , 'http://search.google.co.jp/xxx'   )
  , ('2016-10-01 15:00:00', '1hI43A', '6SN6DD', 'http://www.example.com/list/newly'                                 , ''                                 )
  , ('2016-10-02 16:00:00', '1hI43A', '6SN6DD', 'http://www.example.com/list/cd'                                    , 'http://www.example.com/list/newly')
  , ('2016-10-01 17:00:00', '2bGs3i', '1CwlSX', 'http://www.example.com/'                                           , ''                                 )
  , ('2016-10-01 18:00:00', '2is8PX', '7Dn99b', 'http://www.example.com/detail?id=2'                                , 'https://twitter.com/xxx'          )
  , ('2016-10-02 12:00:00', '2mmGwD', 'EFnoNR', 'http://www.example.com/'                                           , ''                                 )
  , ('2016-10-02 13:00:00', '2mmGwD', 'EFnoNR', 'http://www.example.com/list/cd'                                    , 'http://search.google.co.jp/xxx'   )
  , ('2016-10-02 14:00:00', '3CEHe1', 'FGkTe9', 'http://www.example.com/list/dvd'                                   , ''                                 )
  , ('2016-10-02 15:00:00', '3Gv8vO', '1CwlSX', 'http://www.example.com/detail?id=2'                                , ''                                 )
  , ('2016-10-02 16:00:00', '3cv4gm', 'KBlKgT', 'http://www.example.com/list/newly'                                 , 'http://search.yahoo.co.jp/xxx'    )
  , ('2016-10-02 17:00:00', '3cv4gm', 'KBlKgT', 'http://www.example.com/'                                           , 'https://www.facebook.com/xxx'     )
  , ('2016-10-02 18:00:00', '690mvB', 'FGkTe9', 'http://www.example.com/list/dvd?utm_source=yahoo&utm_medium=search', 'http://www.yahoo.co.jp/xxx'       )
  , ('2016-10-03 12:00:00', '6oABhM', '3JMO2k', 'http://www.example.com/detail?id=3'                                , 'http://search.yahoo.co.jp/xxx'    )
  , ('2016-10-03 13:00:00', '7jjxQX', 'KKTw9P', 'http://www.example.com/?utm_source=mynavi&utm_medium=affiliate'    , 'http://www.mynavi.jp/xxx'         )
  , ('2016-10-03 14:00:00', 'AAuoEU', '6SN6DD', 'http://www.example.com/list/dvd'                                   , 'https://www.facebook.com/xxx'     )
  , ('2016-10-03 15:00:00', 'AAuoEU', '6SN6DD', 'http://www.example.com/list/newly'                                 , ''                                 )
;

insert into purchase_log 
   values ('2016-10-01 15:00:00', '0CVKaz', '1CwlSX',1,10000)
          ,('2016-10-01 16:00:00', '1QceiB', '3JMO2k',2,10000)
		  ,('2016-10-01 20:00:00', '1QceiB', '3JMO2k',3,10000)
		  ,('2016-10-02 14:00:00', '1QceiB', '3JMO2k',4,10000)
          ,('2016-10-01 17:00:00', '1hI43A', '6SN6DD',5,20000)
		  ,('2016-10-02 16:40:00', '1hI43A', '6SN6DD',6,10000)
		  ,('2016-10-01 17:20:00', '2bGs3i', '1CwlSX',7,14000)
		  ,('2016-10-01 18:10:00', '2is8PX', '7Dn99b',8,20000)
		  ,('2016-10-02 12:10:00', '2mmGwD', 'EFnoNR',9,4000)
		  ,('2016-10-02 13:50:00', '2mmGwD', 'EFnoNR',10,2200)
		  ,('2016-10-02 15:00:00', '3Gv8vO', '1CwlSX',11,2100)
		  ,('2016-10-02 18:00:00', '3cv4gm', 'KBlKgT',12,2200)
		  ,('2016-10-02 19:00:00', '690mvB', 'FGkTe9',13,100)
		  ,('2016-10-03 13:00:00', '6oABhM', '3JMO2k',14,2200)
		  ,('2016-10-03 15:00:00', 'AAuoEU', '6SN6DD',15,2100);

```


```sql

with access_log_with_parse_info as(
 select *,
	   substring(url from 'https?://([^/]*)') as url_domain,
	   substring(url from 'utm_source=([^&]*)') as url_source,
	   substring(url from 'utm_medium=([^&]*)') as url_medium,
	   substring(referrer from 'https?://([^/]*)') as referrer_domain
	   from access_log)
  ,access_log_with_via_info as(
   select *,
	     row_number() over(order by stamp) as log_id,
	     case when url_source<>'' and url_medium<>'' then concat(url_source,'-',url_medium) 
              when referrer_domain in ('google.co.jp','yahoo.co.jp') then 'search'
	          when referrer_domain in ('facebook.com','twitter.com') then 'social' 
	          else 'others' end as via 
	    from access_log_with_parse_info)
  ,access_log_with_purchase_amount as(
   select a.log_id,
	      a.via,
	      sum(case when p.stamp::date between a.stamp::date and a.stamp::date+'1 day'::interval then p.amount end) as amount
	  
	  
	  from access_log_with_via_info as a
	  left join purchase_log as p
	  on a.long_session=p.long_session
	  group by a.log_id,a.via
  )


select via,
       count(1) as count_via,
	   count(amount) as conversions,
	   avg(100*sign(coalesce(amount,0))) as cvr,
	   sum(coalesce(amount,0)) as amount,
	   avg(1.0*coalesce(amount,0)) as avg_amount
	   from access_log_with_purchase_amount
	   group by vi
```

### 14.5 Conversion,CVR,and Amount Purchased



### 14.6 Discovering Click Patterns Based on Days and Time 

We are still looking for many ways to discover teh click patterns of our customers. In this part, we can analyize it using the day and time of using service recorded in our database. Let's demonstrate it in codes. 

```sql
with access_log_with_dow as(
 select stamp,
	    date_part('dow',stamp::timestamp) as dow,
	    cast(substring(stamp,12,2) as int)*60*60+
	    cast(substring(stamp,15,2) as int)*60+
	    cast(substring(stamp,18,2) as int) as whole_seconds,
	    30*60 as interval_seconds
        from access_log)
 ,access_log_with_floor_seconds as(
    select stamp,
	       dow,
	       cast(floor(whole_seconds/interval_seconds)*interval_seconds as int) as floor_seconds
    from access_log_with_dow)
  ,access_log_with_index as(select stamp,
         dow,
		 lpad(floor(floor_seconds/(60*60))::text,2,'0')||':'
		     ||lpad(floor(floor_seconds%(60*60)/60)::text,2,'0')||':'
			 ||lpad(floor(floor_seconds%60)::text,2,'0') as index_time
			 from access_log_with_floor_seconds)
	select index_time,
	       count(case when dow=0 then 1 end) as Sun,
		   count(case when dow=1 then 1 end ) as Mon,
		   count(case when dow=2 then 1 end) as Tue,
		   count(case when dow=3 then 1 end) as Wed,
		   count(case when dow=4 then 1 end) as Thur,
		   count(case when dow=5 then 1 end) as Fri,
		   count(case when dow=6 then 1 end) as Sat
		   from access_log_with_index
		   group by index_time;
```
![image](https://user-images.githubusercontent.com/53164959/87594201-9115d300-c727-11ea-87c0-c3578dded792.png)

