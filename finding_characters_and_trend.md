Finding Characters and Trends 
=============================

### 1. Introduction 

In the business world, business managers are actively seeking relevant information on customers, from which they 
can take an investigation on the behaviors of their customers, predict the pattern of their spending, and 
even get some useful tips on how to improve thier peformance of sales. 


The sample tables we will use for our data analysis are following; 


![image](https://user-images.githubusercontent.com/53164959/62774808-e6417180-bae0-11e9-9d8c-56dbd2f69355.png)

[table 1]  'mst_user' table


![image](https://user-images.githubusercontent.com/53164959/62774912-2e609400-bae1-11e9-94da-bb5205a38942.png)

[table 2] 'action_log' table 

You can find that there are some missing entries on the column of 'user_id' and this refers to those
who visits the website without any log in. 


### 2. Investigation On User's  Behavior Pattern

Now,we are preparing a descriptive report consisting of  

```
action        : view, add_cart, purchase 
action_uu     : view , favorite :  myshop_list , add_cart , purchase, review)
total_uu      : the total sum of each action_uu
usage_rate    : action_uu/total_uu
count_per_use : the rate of action per user 
```

```sql

