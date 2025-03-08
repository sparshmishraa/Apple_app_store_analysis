## Project Context

AppleStore.csv dataset covers information about applications available on the Apple Store.

There is more to an application that’s why we have another dataset called appleStore_description.csv which will give us an overview of each application description.

The way application is described can tell us a lot about how it interferes with its user base and how it might be perceived in the market.


## Stakeholders

- The personal group who has interested in the outcome of the analysis.
- An aspiring application developer who needs data-driven insights to decide what type of application to built.


## Key Questions for which we are seeking Answers

- What application categories are most popular?
- What price should I set?
- How can I maximize user ratings?


## Code

- Creating Apple Store Table
```
  CREATE TABLE 'AppleStore' ('' INTEGER,'id' INTEGER,'track_name' TEXT,'size_bytes' INTEGER,'currency' TEXT,'price' REAL,'rating_count_tot' INTEGER,
                             'rating_count_ver' INTEGER,'user_rating' REAL,'user_rating_ver' REAL,'ver' TEXT,'cont_rating' TEXT,'prime_genre' TEXT,
                             'sup_devices_num' INTEGER,'ipadSc_urls_num' INTEGER,'lang_num' INTEGER,'vpp_lic' INTEGER)
```
- Creating Apple Store Description Table
```
  CREATE TABLE 'appleStore_description' ('id' INTEGER,'track_name' TEXT,'size_bytes' INTEGER,'app_desc' TEXT)
```

## Exploratory Data Analysis

- Check the number of unique applications in AppleStore & appleStore_description table.
```
SELECT COUNT(DISTINCT id) AS Unique_App_Ids
FROM AppleStore;

SELECT COUNT(DISTINCT id) AS Unique_App_Ids
FROM appleStore_description;
```


- Check for missing values in some of the key fields of the table (Both the tables).
```
SELECT COUNT(*) AS MissingValues
FROM AppleStore
WHERE track_name ISNULL OR user_rating ISNULL OR prime_genre ISNULL;

SELECT COUNT(*) AS MissingValues
FROM appleStore_description
WHERE track_name ISNULL OR app_desc ISNULL;
````


- Finding out the number of applications per genre.
- This gives us an overview of the types & genre distribution in the Apple Store helping us identify dominant genres.
```
SELECT prime_genre, COUNT(*) AS number_of_apps
FROM AppleStore
GROUP BY prime_genre
ORDER BY number_of_apps DESC;
```


- Get an overview of app ratings
```
SELECT min(user_rating) AS min_rating,
       max(user_rating) AS max_rating,
       avg(user_rating) AS avg_rating
FROM AppleStore;
```


## Finding the Insights

- Determine whether paid applications have better ratings than free applications.
```
SELECT 
    CASE
    WHEN price > 0 THEN 'Paid'
    ELSE 'Free'
    END AS app_type,
    avg(user_rating) As avg_rating
FROM AppleStore
GROUP BY app_type;
```


- Check if applications with more supported languages have higher ratings.
```
SELECT
      CASE
      WHEN lang_num < 10 THEN '<10 languages'
      WHEN lang_num BETWEEN 10 AND 30 THEN '10-30 languages'
      ELSE '>30 languages'
      END AS language_bucket,
      avg(user_rating) AS avg_rating
FROM AppleStore
GROUP BY language_bucket
ORDER BY avg_rating DESC;
```


- Check genres with low ratings.
```
SELECT
      prime_genre,
      avg(user_rating) AS avg_rating
FROM AppleStore
GROUP BY prime_genre
ORDER BY avg_rating ASC
LIMIT 10;
```


- Check if there is correlation between the length of the application description and the user rating.
```
SELECT
      CASE
      WHEN length(b.app_desc) < 500 THEN 'Short'
      WHEN length(b.app_desc) BETWEEN 500 AND 1000 THEN 'Medium'
      ELSE 'Long'
      END AS description_length_bucket,
      avg(a.user_rating) AS average_rating
FROM AppleStore AS a
JOIN appleStore_description AS b
ON a.id = b.id
GROUP BY description_length_bucket
ORDER BY average_rating DESC;
```


- Check the top rated applications for each genre.
```
SELECT
      prime_genre, track_name, user_rating
FROM (
  SELECT
  prime_genre,
  track_name,
  user_rating,
  RANK() OVER(PARTITION BY prime_genre
              ORDER BY user_rating DESC,
              rating_count_tot DESC) AS rank
  FROM AppleStore 
  )AS a
WHERE a.rank = 1;
```


















