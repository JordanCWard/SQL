# SQL_practice

https://datalemur.com/questions?category=SQL

1. Histogram of Tweets

Assume you're given a table Twitter tweet data, write a query to obtain a histogram of tweets posted per user in 2022. Output the tweet count per user as the bucket and the number of Twitter users who fall into that bucket.
In other words, group the users by the number of tweets they posted in 2022 and count the number of users in each group.

``` sql
WITH user_tweet_counts AS (
  SELECT 
    user_id, 
    COUNT(tweet_id) AS tweet_count 
  FROM 
    tweets 
  WHERE 
    tweet_date BETWEEN '2022-01-01' 
    AND '2022-12-31' 
  GROUP BY 
    user_id
) 
SELECT 
  tweet_count, 
  COUNT(user_id) as number_of_users 
FROM 
  user_tweet_counts 
GROUP BY 
  1;
```


2. LinkedIn

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. You want to find candidates who are proficient in Python, Tableau, and PostgreSQL.
Write a query to list the candidates who possess all of the required skills for the job. Sort the output by candidate ID in ascending order.

Method 1 (CTE):
``` sql
WITH candidate_skills AS (
  SELECT 
    candidate_id, 
    string_agg(skill, ', ') as all_skills 
  FROM 
    candidates 
  GROUP BY 
    candidate_id
) 
SELECT 
  candidate_id 
FROM 
  candidate_skills 
WHERE 
  all_skills LIKE '%Python%' 
  AND all_skills LIKE '%Tableau%' 
  AND all_skills LIKE '%PostgreSQL%' 
ORDER BY 
  candidate_id ASC;
```

Method 2:
``` sql
SELECT 
  candidate_id 
FROM 
  candidates 
WHERE 
  skill IN (
    'Python', 'Tableau', 'PostgreSQL'
  ) 
GROUP BY 
  candidate_id 
HAVING 
  COUNT(skill) = 3 
ORDER BY 
  candidate_id;
```


3. Facebook

Assume you're given two tables containing data about Facebook Pages and their respective likes (as in "Like a Facebook Page").

Write a query to return the IDs of the Facebook pages that have zero likes. The output should be sorted in ascending order based on the page IDs.

Method 1: Except
``` sql
SELECT 
  page_id 
FROM 
  pages 
EXCEPT 
SELECT 
  page_id 
FROM 
  page_likes 
ORDER BY 
  page_id ASC;
```

Method 2: Null
``` sql
SELECT 
  pages.page_id 
FROM 
  pages 
  LEFT OUTER JOIN page_likes AS likes ON pages.page_id = likes.page_id 
WHERE 
  likes.page_id IS NULL;
```
