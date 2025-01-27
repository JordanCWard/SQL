# SQL_practice

https://datalemur.com/questions?category=SQL
 <br> <br>


23. Snapchat

Assume you're given tables with information on Snapchat users, including their ages and time spent sending and opening snaps. <br>
Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group. <br>
Round the percentage to 2 decimal places in the output.

``` sql
SELECT 
  age_bucket, 
  ROUND(100.0*
    SUM(CASE WHEN activity_type = 'send' THEN time_spent ELSE 0 END)/
    SUM(CASE WHEN activity_type = 'send' OR activity_type = 'open' THEN time_spent ELSE 0 END), 2) AS send_perc, 
  ROUND(100.0*
    SUM(CASE WHEN activity_type = 'open' THEN time_spent ELSE 0 END)/
    SUM(CASE WHEN activity_type = 'send' OR activity_type = 'open' THEN time_spent ELSE 0 END), 2) AS open_perc
FROM 
  activities 
  LEFT JOIN age_breakdown ON activities.user_id = age_breakdown.user_id 
GROUP BY 
  age_bucket;
```
<br>


22. FAANG

Imagine you're an HR analyst at a tech company tasked with analyzing employee salaries. <br>
Your manager is keen on understanding the pay distribution and asks you to determine the second highest salary among all employees. <br>
It's possible that multiple employees may share the same second highest salary. In case of duplicate, display the salary only once.

``` sql
WITH salary_ranking AS (
  SELECT
    row_number() OVER (ORDER BY salary DESC) AS ranking,
    salary
  FROM
    employee
)

SELECT
  salary
FROM
  salary_ranking
WHERE
  ranking = 2
;
```
<br>


21. Uber

Assume you are given the table below on Uber transactions made by users. <br>
Write a query to obtain the third transaction of every user. <br>
Output the user id, spend and transaction date.

``` sql
WITH ranked_transactions AS (
  SELECT
    row_number() OVER (PARTITION BY user_id ORDER BY transaction_date ASC) AS row_rank,
    user_id,
    spend,
    transaction_date
  FROM
    transactions
)

SELECT
  user_id,
  spend,
  transaction_date
FROM
  ranked_transactions
WHERE
  row_rank = 3
;
```
<br>


20. UnitedHealth

UnitedHealth Group (UHG) has a program called Advocate4Me, which allows policy holders (or, members) to call an advocate and receive support for their health care needs – whether that's claims and benefits support, drug coverage, pre- and post-authorisation, medical records, emergency assistance, or member portal services. <br>
Write a query to find how many UHG policy holders made three, or more calls, assuming each call is identified by the case_id column.

``` sql
WITH calls_by_id AS (
  SELECT
    policy_holder_id,
    COUNT(case_id) AS total_calls
  FROM
    callers
  GROUP BY
    policy_holder_id
)

SELECT
  COUNT(policy_holder_id) AS policy_holder_count
FROM
  calls_by_id
WHERE
  total_calls > 2
;
```
<br>


19. CVS Health

CVS Health wants to gain a clearer understanding of its pharmacy sales and the performance of various products. <br>
Write a query to calculate the total drug sales for each manufacturer. <br>
Round the answer to the nearest million and report your results in descending order of total sales. <br>
In case of any duplicates, sort them alphabetically by the manufacturer name. <br>
Since this data will be displayed on a dashboard viewed by business stakeholders, please format your results as follows: "$36 million".

``` sql
WITH manufacturer_sales AS (
  SELECT
    manufacturer,
    ROUND((SUM(total_sales))/1000000, 0) AS sum_of_profit
  FROM
    pharmacy_sales
  GROUP BY
    manufacturer
  ORDER BY
    sum_of_profit DESC, manufacturer ASC
)

SELECT
  manufacturer,
  CONCAT('$', sum_of_profit, ' million') AS sales_mil
FROM
  manufacturer_sales
;
```
<br>


18. CVS Health

CVS Health is analyzing its pharmacy sales data, and how well different products are selling in the market. <br>
Each drug is exclusively manufactured by a single manufacturer. <br>
Write a query to identify the manufacturers associated with the drugs that resulted in losses for CVS Health and calculate the total amount of losses incurred. <br>
Output the manufacturer's name, the number of drugs associated with losses, and the total losses in absolute value. <br>
Display the results sorted in descending order with the highest losses displayed at the top.

``` sql
SELECT
  manufacturer,
  COUNT(drug),
  SUM((cogs - total_sales)) AS total_loss
FROM
  pharmacy_sales
WHERE
  cogs > total_sales
GROUP BY
  manufacturer
ORDER BY
  total_loss DESC
;
```
<br>


17. CVS Health

CVS Health is trying to better understand its pharmacy sales, and how well different products are selling. <br>
Each drug can only be produced by one manufacturer. <br>
Write a query to find the top 3 most profitable drugs sold, and how much profit they made. <br>
Assume that there are no ties in the profits. <br>
Display the result from the highest to the lowest total profit.

``` sql
SELECT
  drug,
  (total_sales - cogs) AS total_profit
FROM
  pharmacy_sales
ORDER BY
  total_profit DESC
LIMIT
  3
;
```
<br>


16. Alibaba

You're trying to find the mean number of items per order on Alibaba, rounded to 1 decimal place using tables which includes information on the count of items in each order (item_count table) and the corresponding number of orders for each item count (order_occurrences table).

``` sql
SELECT
  ROUND(
    SUM(item_count::DECIMAL * order_occurrences)
    /SUM(order_occurrences)
    , 1) AS mean
FROM
  items_per_order
;
```
<br>


15. JPMorgan

Your team at JPMorgan Chase is preparing to launch a new credit card, and to gain some insights, you're analyzing how many credit cards were issued each month. <br>
Write a query that outputs the name of each credit card and the difference in the number of issued cards between the month with the highest issuance cards and the lowest issuance. <br>
Arrange the results based on the largest disparity.

``` sql
SELECT
  card_name,
  MAX(issued_amount) - MIN(issued_amount) AS difference
FROM
  monthly_cards_issued
GROUP BY
  card_name
ORDER BY
  difference DESC
;
```
<br>


14. IBM

IBM is analyzing how their employees are utilizing the Db2 database by tracking the SQL queries executed by their employees. <br>
The objective is to generate data to populate a histogram that shows the number of unique queries run by employees during the third quarter of 2023 (July to September). <br>
Additionally, it should count the number of employees who did not run any queries during this period. <br>
Display the number of unique queries as histogram categories, along with the count of employees who executed that number of unique queries.

``` sql
WITH query_count AS (
SELECT
  employee_id,
  COUNT(DISTINCT query_id) AS total_queries
FROM
  queries
WHERE
  query_starttime >= '2023-07-01T00:00:00Z'
  AND query_starttime < '2023-10-01T00:00:00Z'
GROUP BY
  employee_id
ORDER BY
  total_queries DESC
)

SELECT
  COALESCE(total_queries, '0') AS unique_queries,
  COUNT(COALESCE(total_queries, '0')) AS employee_count
FROM
  employees
LEFT JOIN
  query_count ON employees.employee_id = query_count.employee_id
GROUP BY
  unique_queries
ORDER BY
  unique_queries
;
```
<br>


13. TikTok

Assume you're given tables with information about TikTok user sign-ups and confirmations through email and text. <br>
New users on TikTok sign up using their email addresses, and upon sign-up, each user receives a text message confirmation to activate their account. <br>
Write a query to display the user IDs of those who did not confirm their sign-up on the first day, but confirmed on the second day.

``` sql
SELECT
  user_id
FROM
  emails
INNER JOIN
  texts ON emails.email_id = texts.email_id
WHERE
  DATE_PART('day', action_date - signup_date) = 1
  AND texts.signup_action = 'Confirmed'
;
```
<br>


12. Facebook

Assume you have an events table on Facebook app analytics. <br>
Write a query to calculate the click-through rate (CTR) for the app in 2022 and round the results to 2 decimal places. <br>
Definition and note: Percentage of click-through rate (CTR) = 100.0 * Number of clicks / Number of impressions. <br>
To avoid integer division, multiply the CTR by 100.0, not 100.

``` sql
SELECT
  app_id,
  ROUND((100.0*SUM(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END)) /
  (SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END)), 2) AS ctr
FROM
  events
WHERE
  EXTRACT(year FROM timestamp) = 2022
GROUP BY
  app_id;
```
<br>


11. FAANG

Companies often perform salary analyses to ensure fair compensation practices. <br>
One useful analysis is to check if there are any employees earning more than their direct managers. <br>
As a HR Analyst, you're asked to identify all employees who earn more than their direct managers. <br>
The result should include the employee's ID and name.

``` sql
SELECT 
  emp.employee_id AS employee_id, 
  emp.name AS employee_name 
FROM 
  employee AS mgr 
INNER JOIN
  employee AS emp ON mgr.employee_id = emp.manager_id 
WHERE 
  emp.salary > mgr.salary;
```
<br>


10. Amazon

Given the reviews table, write a query to retrieve the average star rating for each product, grouped by month. <br>
The output should display the month as a numerical value, product ID, and average star rating rounded to two decimal places. <br>
Sort the output first by month and then by product ID.

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
<br>


9. Robinhood

Assume you're given the tables containing completed trade orders and user details in a Robinhood trading system. <br>
Write a query to retrieve the top three cities that have the highest number of completed trade orders listed in descending order. <br>
Output the city name and the corresponding number of completed trade orders.

``` sql
SELECT 
  city, 
  count(status) AS total_orders 
FROM 
  trades 
JOIN
  users ON trades.user_id = users.user_id 
WHERE 
  status = 'Completed' 
GROUP BY 
  city 
ORDER BY 
  total_orders DESC 
LIMIT 
  3;
```
<br>


8. LinkedIn

Assume you're given a table containing job postings from various companies on the LinkedIn platform. <br>
Write a query to retrieve the count of companies that have posted duplicate job listings. <br>
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
<br>


7. Microsoft

Write a query to identify the top 2 Power Users who sent the highest number of messages on Microsoft Teams in August 2022. <br>
Display the IDs of these 2 users along with the total number of messages they sent.  <br>
Output the results in descending order based on the count of the messages. <br>
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
<br>


6. Facebook

Given a table of Facebook posts, for each user who posted at least twice in 2021, write a query to find the number of days between each user’s first post of the year and last post of the year in the year 2021. <br>
Output the user and number of the days between each user's first and last post.

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
<br>


5. NY Times

Assume you're given the table on user viewership categorised by device type where the three types are laptop, tablet, and phone. <br>
Write a query that calculates the total viewership for laptops and mobile devices where mobile is defined as the sum of tablet and phone viewership. <br>
Output the total viewership for laptops as laptop_reviews and the total viewership for mobile devices as mobile_views.

``` sql
SELECT 
  SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views, 
  SUM(CASE WHEN device_type IN ('tablet', 'phone') THEN 1 ELSE 0 END) AS mobile_views 
FROM 
  viewership;
```
<br>


4.  Tesla

Tesla is investigating production bottlenecks and they need your help to extract the relevant data. <br>
Write a query to determine which parts have begun the assembly process but are not yet finished. <br>

Assumptions <br>
parts_assembly table contains all parts currently in production, each at varying stages of the assembly process. <br>
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
<br>


3. Facebook

Assume you're given two tables containing data about Facebook Pages and their respective likes (as in "Like a Facebook Page"). <br>
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
LEFT OUTER JOIN
  page_likes AS likes ON pages.page_id = likes.page_id 
WHERE 
  likes.page_id IS NULL;
```
<br>


2. LinkedIn

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. <br>
You want to find candidates who are proficient in Python, Tableau, and PostgreSQL. <br>
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
  skill IN ('Python', 'Tableau', 'PostgreSQL') 
GROUP BY 
  candidate_id 
HAVING 
  COUNT(skill) = 3 
ORDER BY 
  candidate_id;
```
<br>


1. Histogram of Tweets

Assume you're given a table Twitter tweet data, write a query to obtain a histogram of tweets posted per user in 2022. <br>
Output the tweet count per user as the bucket and the number of Twitter users who fall into that bucket. <br>
In other words, group the users by the number of tweets they posted in 2022 and count the number of users in each group.

``` sql
WITH user_tweet_counts AS (
  SELECT 
    user_id, 
    COUNT(tweet_id) AS tweet_count 
  FROM 
    tweets 
  WHERE 
    tweet_date BETWEEN '2022-01-01' AND '2022-12-31' 
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
