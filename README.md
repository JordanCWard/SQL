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

4.  Tesla

Tesla is investigating production bottlenecks and they need your help to extract the relevant data. Write a query to determine which parts have begun the assembly process but are not yet finished.

Assumptions:

parts_assembly table contains all parts currently in production, each at varying stages of the assembly process.
An unfinished part is one that lacks a finish_date.

``` sql
SELECT 
  part, 
  assembly_step 
FROM 
  parts_assembly 
WHERE 
  finish_date IS NULL;
```

5. NY Times

Assume you're given the table on user viewership categorised by device type where the three types are laptop, tablet, and phone.

Write a query that calculates the total viewership for laptops and mobile devices where mobile is defined as the sum of tablet and phone viewership. Output the total viewership for laptops as laptop_reviews and the total viewership for mobile devices as mobile_views.

``` sql
SELECT 
  SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views, 
  SUM(CASE WHEN device_type IN ('tablet', 'phone') THEN 1 ELSE 0 END) AS mobile_views 
FROM 
  viewership;
```



6. Facebook

Given a table of Facebook posts, for each user who posted at least twice in 2021, write a query to find the number of days between each user’s first post of the year and last post of the year in the year 2021. Output the user and number of the days between each user's first and last post.

``` sql
SELECT 
  user_id, 
  EXTRACT(days FROM max(post_date) - min(post_date)) AS days_between 
FROM 
  posts 
WHERE 
  post_date BETWEEN '01/01/2021' and '12/31/2021' 
GROUP BY 
  user_id 
HAVING 
  COUNT(post_id) > 1;
```

7. Microsoft

Write a query to identify the top 2 Power Users who sent the highest number of messages on Microsoft Teams in August 2022. Display the IDs of these 2 users along with the total number of messages they sent. Output the results in descending order based on the count of the messages.

Assumption: No two users have sent the same number of messages in August 2022.

``` sql
SELECT 
  sender_id, 
  COUNT(message_id) AS total_messages 
FROM 
  messages 
WHERE 
  sent_date BETWEEN '08/01/2022' AND '08/31/2022' 
GROUP BY 
  sender_id 
ORDER BY 
  total_messages DESC 
LIMIT 
  2;
```

8. LinkedIn

Assume you're given a table containing job postings from various companies on the LinkedIn platform. Write a query to retrieve the count of companies that have posted duplicate job listings.

Definition: Duplicate job listings are defined as two job listings within the same company that share identical titles and descriptions.

``` sql
WITH duplicates AS (
  SELECT 
    count(title) AS same_job 
  FROM 
    job_listings 
  GROUP BY 
    company_id, 
    title, 
    description
) 
SELECT 
  count(same_job) AS duplicate_companies 
FROM 
  duplicates 
WHERE 
  same_job > 1;
```

9. Robinhood

Assume you're given the tables containing completed trade orders and user details in a Robinhood trading system.

Write a query to retrieve the top three cities that have the highest number of completed trade orders listed in descending order. Output the city name and the corresponding number of completed trade orders.

``` sql
SELECT 
  city, 
  count(status) AS total_orders 
FROM 
  trades 
  JOIN users ON trades.user_id = users.user_id 
WHERE 
  status = 'Completed' 
GROUP BY 
  city 
ORDER BY 
  total_orders DESC 
LIMIT 
  3;
```

10. Amazon

Given the reviews table, write a query to retrieve the average star rating for each product, grouped by month. The output should display the month as a numerical value, product ID, and average star rating rounded to two decimal places. Sort the output first by month and then by product ID.

``` sql
SELECT 
  EXTRACT(MONTH FROM submit_date) AS submit_month, 
  product_id, 
  ROUND(AVG(stars), 2) AS rounded_stars 
FROM 
  reviews 
GROUP BY 
  submit_month, 
  product_id 
ORDER BY 
  submit_month ASC, 
  product_id ASC;
```






