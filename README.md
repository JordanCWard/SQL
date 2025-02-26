# SQL Practice

<!-- Hidden text for templates

https://datalemur.com/questions?category=SQL

/*

*/

``` sql

```


-->

40. Wayfair

Assume you're given a table containing information about Wayfair user transactions for different products. Write a query to calculate the year-on-year growth rate for the total spend of each product, grouping the results by product ID.

``` sql
WITH product_spending AS (
  SELECT
    EXTRACT (YEAR FROM transaction_date) AS year,
    product_id,
    SUM(spend) AS total_spent
  FROM
    user_transactions
  GROUP BY
    year, product_id
  ORDER BY
    product_id ASC, year ASC
)
  
SELECT
  a.year,
  a.product_id,
  a.total_spent AS current_year_spend,
  b.total_spent AS prev_year_spend,
  ROUND(100.0*((a.total_spent-b.total_spent)/b.total_spent), 2) AS yoy_rate
FROM
  product_spending a
LEFT JOIN
  product_spending b ON a.year = b.year + 1 AND a.product_id = b.product_id
;
```
<br>


39. Facebook
Assume you're given a table containing information on Facebook user actions. Write a query to obtain number of monthly active users (MAUs) in July 2022, including the month in numerical format "1, 2, 3". An active user is defined as a user who has performed actions such as 'sign-in', 'like', or 'comment' in both the current month and the previous month.


``` sql
SELECT
  EXTRACT(MONTH FROM event_date) AS month_num,
  COUNT(DISTINCT user_id) AS monthly_active_users
FROM
  user_actions
WHERE
  event_date BETWEEN '07/01/2022' AND '07/31/2022'
  AND user_id IN (
    SELECT
      user_id
    FROM
      user_actions
    WHERE
      event_date BETWEEN '06/01/2022' AND '06/30/2022'
  )
GROUP BY
  month_num
;
```
<br>


38. UnitedHealth

UnitedHealth Group (UHG) has a program called Advocate4Me, which allows policy holders (or, members) to call an advocate and receive support for their health care needs – whether that's claims and benefits support, drug coverage, pre- and post-authorisation, medical records, emergency assistance, or member portal services. <br>

Calls to the Advocate4Me call centre are classified into various categories, but some calls cannot be neatly categorised. These uncategorised calls are labeled as “n/a”, or are left empty when the support agent does not enter anything into the call category field. <br>

Write a query to calculate the percentage of calls that cannot be categorised. Round your answer to 1 decimal place.

``` sql
SELECT
  ROUND(
    100.0 * COUNT(*) FILTER(
      WHERE call_category = 'n/a' OR call_category IS NULL) /
      COUNT(*), 1
      ) AS uncategorised_call_pct
FROM
  callers
;
```
<br>


37. Verizon

A phone call is considered an international call when the person calling is in a different country than the person receiving the call. What percentage of phone calls are international? Round the result to 1 decimal.

``` sql
SELECT 
  ROUND(
    100.0 * COUNT(*) FILTER(
      WHERE b.country_id != c.country_id) /
      COUNT(*), 1
      ) 
FROM 
  phone_calls AS a 
JOIN
    phone_info AS b ON a.caller_id = b.caller_id 
JOIN
    phone_info AS c ON a.receiver_id = c.caller_id
;

```
<br>


36. JPMorgan

Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month. Before you can answer this question, you want to first get some perspective on how well new credit card launches typically do in their first month. Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the monthly_cards_issued table for a given card. Order the results starting from the biggest issued amount.

``` sql
/*
Rank the dates to find which month is the first month for each card.
Use first value to return the issued amount for the first month.
Partition by card name, order by issue year and issue month.
*/

SELECT DISTINCT
  card_name, 
  FIRST_VALUE(issued_amount) OVER (
    PARTITION BY card_name 
    ORDER BY issue_year, issue_month
  ) AS issued_amount 
FROM 
  monthly_cards_issued 
ORDER BY 
  issued_amount DESC;
```
<br>


35. Alibaba

You're given a table containing the item count for each order on Alibaba, along with the frequency of orders that have the same item count. Write a query to retrieve the mode of the order occurrences. Additionally, if there are multiple item counts with the same mode, the results should be sorted in ascending order.

``` sql
/*
to find the mode 

CTE
count the number of times each order occurrence appears
group by order occurrences

join new table with original table 
only want rows with max occurrence value 
*/

WITH all_orders AS (
SELECT
  order_occurrences,
  count(*) AS occurrences
FROM
  items_per_order
GROUP BY
  order_occurrences
) 
  
SELECT
  item_count
FROM
  all_orders
FULL OUTER JOIN
  items_per_order ON all_orders.order_occurrences = items_per_order.order_occurrences
WHERE
  occurrences = (SELECT MAX(occurrences) FROM all_orders)
;
```
<br>


34. Walmart

Assume you're given a table on Walmart user transactions. Based on their most recent transaction date, write a query that retrieve the users along with the number of products they bought. Output the user's most recent transaction date, user ID, and the number of products, sorted in chronological order by the transaction date.

``` sql
/*
output
  last transaction date for each user
  user id
  purchase count

find the last purchase date for each user (CTE)
inner join with original table to remove extra rows 
count all purchases
group by user_id and transaction_date
order by transaction_date asc
*/

WITH max_dates AS (
  SELECT
    user_id,
    MAX(transaction_date) AS last_transaction
  FROM
    user_transactions
  GROUP BY
    user_id
)

SELECT
  transaction_date,
  u.user_id,
  count(product_id) AS purchase_count
FROM
  user_transactions u
INNER JOIN
  max_dates m ON u.user_id = m.user_id AND u.transaction_date = m.last_transaction
GROUP BY
  u.user_id, transaction_date
ORDER BY
  transaction_date ASC
```
<br>


33. Amazon

In an effort to identify high-value customers, Amazon asked for your help to obtain data about users who go on shopping sprees. A shopping spree occurs when a user makes purchases on 3 or more consecutive days. List the user IDs who have gone on at least 1 shopping spree in ascending order.

``` sql
WITH consecutive_days AS (
    SELECT 
        user_id,
        transaction_date,
        LEAD(transaction_date, 1) OVER (PARTITION BY user_id ORDER BY transaction_date) AS next_day1,
        LEAD(transaction_date, 2) OVER (PARTITION BY user_id ORDER BY transaction_date) AS next_day2
    FROM transactions
)

SELECT
  user_id,
  transaction_date,
  next_day1,
  next_day2
FROM
  consecutive_days
WHERE
  next_day1 = transaction_date + INTERVAL '1 day' AND
  next_day2 = transaction_date + INTERVAL '2 days'
ORDER BY
  user_id
;
```
<br>


32. Bloomberg

The Bloomberg terminal is the go-to resource for financial professionals, offering convenient access to a wide array of financial datasets. As a Data Analyst at Bloomberg, you have access to historical data on stock performance. Currently, you're analyzing the highest and lowest open prices for each FAANG stock by month over the years. For each FAANG stock, display the ticker symbol, the month and year with the corresponding highest and lowest open prices. Ensure that the results are sorted by ticker symbol.

``` sql
SELECT 
  ticker, 
  TO_CHAR(MAX(date) FILTER (WHERE open = (
          SELECT 
            MAX(open) 
          FROM 
            stock_prices t2 
          WHERE 
            t2.ticker = t1.ticker)), 'Mon-YYYY') AS highest_mth, 
  MAX(open) AS highest_open, 
  TO_CHAR(MIN(date) FILTER (WHERE open = (
          SELECT 
            MIN(open) 
          FROM 
            stock_prices t2 
          WHERE 
            t2.ticker = t1.ticker)), 'Mon-YYYY') AS lowest_mtn, 
  MIN(open) AS lowest_open 
FROM 
  stock_prices t1 
GROUP BY 
  ticker
ORDER BY
  ticker
;
```
<br>


31. Zomato

Zomato is a leading online food delivery service that connects users with various restaurants and cuisines, allowing them to browse menus, place orders, and get meals delivered to their doorsteps. Recently, Zomato encountered an issue with their delivery system. Due to an error in the delivery driver instructions, each item's order was swapped with the item in the subsequent row. As a data analyst, you're asked to correct this swapping error and return the proper pairing of order ID and item. If the last item has an odd order ID, it should remain as the last item in the corrected data. For example, if the last item is Order ID 7 Tandoori Chicken, then it should remain as Order ID 7 in the corrected data. In the results, return the correct pairs of order IDs and items.

``` sql
WITH modified_orders AS (
SELECT
  ROW_NUMBER() OVER () AS row_num,
  COUNT(*) OVER () AS total_rows,
  order_id,
  item,
  LEAD(item) OVER (ORDER BY order_id) AS new_item,
  LAG(item) OVER (ORDER BY order_id) AS prev_item
FROM
  orders
)

SELECT
  order_id AS corrected_order_id,
    CASE
      WHEN row_num = total_rows AND row_num % 2 != 0 THEN item
      WHEN row_num % 2 = 1 THEN new_item
      ELSE prev_item
    END AS item
FROM modified_orders
ORDER BY row_num;
;
```
<br>


30. Google

Assume you're given a table with measurement values obtained from a Google sensor over multiple days with measurements taken multiple times within each day. Write a query to calculate the sum of odd-numbered and even-numbered measurements separately for a particular day and display the results in two different columns. <br>

Definition: Within a day, measurements taken at 1st, 3rd, and 5th times are considered odd-numbered measurements, and measurements taken at 2nd, 4th, and 6th times are considered even-numbered measurements.

``` sql
WITH ranked_measurements AS (
  SELECT
    row_number() OVER (PARTITION BY DATE_TRUNC('day', measurement_time) ORDER BY measurement_time) AS ranking,
    measurement_value,
    DATE_TRUNC('day', measurement_time) AS measurement_day
  FROM
    measurements
)

SELECT
  measurement_day,
  SUM(CASE WHEN ranking % 2 != 0 THEN measurement_value ELSE 0 END) AS odd_sum,
  SUM(CASE WHEN ranking % 2 = 0 THEN measurement_value ELSE 0 END) AS even_sum
FROM
  ranked_measurements
GROUP BY
  measurement_day
;
```
<br>



29. Microsoft

A Microsoft Azure Supercloud customer is defined as a customer who has purchased at least one product from every product category listed in the products table. Write a query that identifies the customer IDs of these Supercloud customers.

``` sql
WITH customer_categories AS (
  SELECT
    customer_contracts.customer_id,
    COUNT(DISTINCT products.product_category) AS product_count
  FROM
    customer_contracts
  LEFT JOIN
    products ON customer_contracts.product_id = products.product_id
  GROUP BY
    customer_contracts.customer_id
)

SELECT
  customer_id
FROM
  customer_categories
WHERE
  product_count = (
  SELECT COUNT(DISTINCT product_category) FROM products
  )
;
```
<br>


28. TikTok

New TikTok users sign up with their emails. They confirmed their signup by replying to the text confirmation to activate their accounts. Users may receive multiple text messages for account confirmation until they have confirmed their new account. A senior analyst is interested to know the activation rate of specified users in the emails table. Write a query to find the activation rate. Round the percentage to 2 decimal places.

Assumptions: <br>
The analyst is interested in the activation rate of specific users in the emails table, which may not include all users that could potentially be found in the texts table. For example, user 123 in the emails table may not be in the texts table and vice versa.

``` sql
SELECT 
  ROUND(COUNT(texts.email_id)::DECIMAL
    /COUNT(DISTINCT emails.email_id),2) AS activation_rate
FROM emails
LEFT JOIN texts
  ON emails.email_id = texts.email_id
  AND texts.signup_action = 'Confirmed';
;
```
<br>


27. Spotify

Assume there are three Spotify tables: artists, songs, and global_song_rank, which contain information about the artists, songs, and music charts, respectively. Write a query to find the top 5 artists whose songs appear most frequently in the Top 10 of the global_song_rank table. Display the top 5 artist names in ascending order, along with their song appearance ranking. If two or more artists have the same number of song appearances, they should be assigned the same ranking, and the rank numbers should be continuous (i.e. 1, 2, 2, 3, 4, 5).

``` sql
WITH top_ten_artists AS (
  SELECT
    artist_name,
    dense_rank() OVER (ORDER BY COUNT(rank) DESC) as artist_rank
  FROM
    global_song_rank
  LEFT JOIN
    songs ON songs.song_id = global_song_rank.song_id
  LEFT JOIN
    artists ON artists.artist_id = songs.artist_id
  WHERE
    rank <= 10
  GROUP BY
    artist_name
)

SELECT
  artist_name,
  artist_rank
FROM
  top_ten_artists
WHERE
  artist_rank <= 5
;
```
<br>


26. FAANG

As part of an ongoing analysis of salary distribution within the company, your manager has requested a report identifying high earners in each department. A 'high earner' within a department is defined as an employee with a salary ranking among the top three salaries within that department. You're tasked with identifying these high earners across all departments. Write a query to display the employee's name along with their department name and salary. In case of duplicates, sort the results of department name in ascending order, then by salary in descending order. If multiple employees have the same salary, then order them alphabetically.

``` sql
WITH ranked_departments AS (
  SELECT
    dense_rank() OVER (PARTITION BY e.department_id ORDER BY salary DESC) AS ranking,
    name,
    salary,
    department_name
  FROM
    employee AS e
  LEFT JOIN
    department AS d ON e.department_id = d.department_id
)

SELECT
  department_name,
  name,
  salary
FROM
  ranked_departments
WHERE
  ranking < 4
ORDER BY
  department_name,
  salary DESC,
  name
;
```
<br>


25. Amazon

Assume you're given a table containing data on Amazon customers and their spending on products in different category, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

``` sql
WITH ranked_products AS (
  SELECT
    category,
    product,
    sum(spend) as total_spent,
    row_number() OVER (
      PARTITION BY category
      ORDER BY sum(spend) DESC) AS ranking
  FROM
    product_spend
  WHERE
    transaction_date BETWEEN '01/01/2022' AND '12/31/2022'
  GROUP BY
    category,
    product
)

SELECT
  category,
  product,
  total_spent
FROM
  ranked_products
WHERE
  ranking < 3
;
```
<br>


24. Twitter

Given a table of tweet data over a specified time period, calculate the 3-day rolling average of tweets for each user. Output the user ID, tweet date, and rolling averages rounded to 2 decimal places.

``` sql
SELECT
  user_id,
  tweet_date,
  ROUND(AVG(tweet_count) OVER
    (PARTITION BY user_id
    ORDER BY tweet_date ASC
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS rolling_avg_3d
FROM
  tweets
ORDER BY
  user_id ASC,
  tweet_date ASC
;
```
<br>


23. Snapchat

Assume you're given tables with information on Snapchat users, including their ages and time spent sending and opening snaps. Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group. Round the percentage to 2 decimal places in the output.

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
