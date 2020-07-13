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


