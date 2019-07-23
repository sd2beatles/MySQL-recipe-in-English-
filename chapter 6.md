### CHAPTER 6 Manupulation of Various Values

#### 1) Concatenating letters
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






