### Chater 14 Introduction 
There are some useful metrics we should track to find any hidden purchasing patterns of our customers. In this section, we will explore how to compute these metrics and add explanations on how to obtain useful insights based on them. 


#### 14.1 [Metric Part 1] Number of Vistors,Number of Frequencies and Page Views. 

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
    ip VARCHAR NOT NULL,
    browser_short VARCHAR(45) NOT NULL,
    browser_long VARCHAR(255) NOT NULL,
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

