# SQL Practice

<!-- Hidden text for templates

/*

MySQL (others available)
https://platform.stratascratch.com/coding/9622-number-of-bathrooms-and-bedrooms?code_type=3

MySQL
https://platform.stratascratch.com/coding?code_type=1

MySQL
https://www.hackerrank.com/domains/sql

MySQL
https://leetcode.com/studyplan/top-sql-50/

PostgreSQL
https://datalemur.com/questions?category=SQL

*/

/*

*/

``` sql

```
<br>

-->









<!--
ALWAYS ADD COMMENTS
-->

191. Game Play Analysis

``` sql
/*
    Calculates the fraction of players who logged in again the day after their
    first login. The result is rounded to two decimal places.
    
    - Numerator: Number of players who logged in the day after their first login.
    - Denominator: Total number of distinct players.
*/

SELECT
    ROUND(
        COUNT(DISTINCT player_id) /     -- Numerator: players meeting the condition
        (SELECT COUNT(DISTINCT player_id) FROM Activity),  -- Denominator: all players
        2
    ) AS fraction
FROM
    Activity
WHERE
    -- Check if (player_id, previous_day) matches (player_id, first_login)
    (player_id, DATE_SUB(event_date, INTERVAL 1 DAY)) IN (
        /*
            Subquery: Finds each player's first login date.
            - MIN(event_date) ensures only the earliest login per player.
            - Used to verify if a later login happened exactly one day after.
        */
        SELECT
            player_id,
            MIN(event_date) AS first_login
        FROM
            Activity
        GROUP BY
            player_id
    )
;
```
<br>


190. Income By Title and Gender

``` sql
/*
    Calculates the average total compensation (salary + bonus) for
    employees, grouped by their job title and sex.
    
    Since employees may have multiple bonus records, we first aggregate 
    bonuses per worker. This prevents double-counting and ensures accurate 
    total compensation before averaging.
*/

-- Aggregate bonuses per worker before joining to employee data
WITH bonus_summary AS (
    SELECT
        worker_ref_id,
        SUM(bonus) AS total_bonus
    FROM sf_bonus
    GROUP BY worker_ref_id
)
SELECT
    e.employee_title,
    e.sex,
    AVG(e.salary + b.total_bonus) AS avg_total_compensation
FROM bonus_summary b
JOIN sf_employee e
    ON b.worker_ref_id = e.id
GROUP BY
    e.employee_title,
    e.sex;
```
<br>


189. Reviews of Categories

``` sql
/*
This query takes a semicolon-delimited list of categories, splits them into individual category
values, and then aggregates the total review counts for each category across all businesses.

Steps:
1. Generate a sequence of numbers using a recursive CTE (acts as category index positions).
2. Use SUBSTRING_INDEX to extract the nth category from each row’s semicolon-delimited string.
3. Join numbers to each row so that every category gets expanded into its own row.
4. Sum the review_count for each category across all businesses.
5. Return categories ordered by their total review_count (highest first).
*/

-- Recursive CTE to generate a sequence of numbers (1-20)
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM numbers
    WHERE n < 20
)

-- Expand categories and compute weighted totals
SELECT 
    -- Extract the nth category value
    TRIM(
        SUBSTRING_INDEX(
            SUBSTRING_INDEX(y.categories, ';', n), 
            ';', 
            -1
        )
    ) AS item,

    -- Total review count summed across all businesses containing this category
    SUM(y.review_count) AS total_qty

FROM yelp_business y

-- Ensure we only generate rows up to the number of categories in each string
JOIN numbers 
  ON n <= 1 + LENGTH(y.categories) - LENGTH(REPLACE(y.categories, ';', ''))

-- Group results by unique category
GROUP BY item

-- Exclude empty strings (in case of trailing semicolons)
HAVING item <> ''

-- Order categories by total review count, highest first
ORDER BY total_qty DESC;
```
<br>


188. Occupations

``` sql
/* 
Pivot occupations: each occupation becomes a column, 
with names aligned row by row in alphabetical order 
*/
WITH ranked AS (
    SELECT 
        name,
        occupation,
        -- Assign row numbers within each occupation (alphabetical by name)
        ROW_NUMBER() OVER (PARTITION BY occupation ORDER BY name) AS rn
    FROM occupations
)
SELECT
    -- Conditional aggregation: pivot occupations into columns
    MAX(CASE WHEN occupation = 'Doctor' THEN name END)    AS Doctor,
    MAX(CASE WHEN occupation = 'Professor' THEN name END) AS Professor,
    MAX(CASE WHEN occupation = 'Singer' THEN name END)    AS Singer,
    MAX(CASE WHEN occupation = 'Actor' THEN name END)     AS Actor
FROM ranked
GROUP BY rn
ORDER BY rn;
```
<br>


187. Twitter

``` sql
/*
   Compute a 3-day rolling average of tweet counts per user.
   - Window defined by user_id partition, ordered by tweet_date.
   - Frame: 2 preceding rows + current row (non-cumulative moving average).
   - ROUND for presentation only (2 decimal precision).
*/
SELECT
    user_id,
    tweet_date,
    ROUND(
        AVG(tweet_count) OVER (
            PARTITION BY user_id
            ORDER BY tweet_date ASC
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ), 
        2
    ) AS rolling_avg_3d
FROM tweets
ORDER BY user_id, tweet_date;
```
<br>


186. Department top three salaries

``` sql
-- Top-3 salaries per department (ties included via DENSE_RANK)
WITH one_table AS (
  SELECT
    DENSE_RANK() OVER (PARTITION BY e.departmentId ORDER BY e.salary DESC) AS ranked_salary,
    d.name  AS Department,
    e.name  AS Employee,
    e.salary AS Salary
  FROM employee   e
  JOIN department d ON d.id = e.departmentId
)
SELECT
  Department,
  Employee,
  Salary
FROM one_table
WHERE ranked_salary <= 3;  -- keep top 3 per department; includes ties at rank 3
```
<br>


185. Draw the Triangle 2

``` sql
-- Generate a sequence of numbers from 1 to 20 using a recursive CTE
WITH RECURSIVE counter_cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM counter_cte
    WHERE n < 20
)

-- Create a string of asterisks repeated by the counter value
SELECT 
    REPEAT('* ', n) AS output
FROM 
    counter_cte;
```
<br>


184. Top Competitors

``` sql
-- Retrieve hackers who have achieved full scores on more than one challenge
-- The query joins submissions, challenges, difficulty, and hackers tables
-- to identify hackers who solved multiple challenges at the maximum score.

SELECT
    h.hacker_id,  -- Unique identifier for each hacker
    h.name        -- Hacker's name
FROM
    submissions s
    LEFT JOIN challenges c 
        ON s.challenge_id = c.challenge_id
    LEFT JOIN difficulty d 
        ON c.difficulty_level = d.difficulty_level
    LEFT JOIN hackers h 
        ON s.hacker_id = h.hacker_id
WHERE
    s.score = d.score  -- Only consider submissions where the hacker achieved the full score
GROUP BY
    h.hacker_id, 
    h.name            -- Group results by hacker to count solved challenges
HAVING
    COUNT(*) > 1      -- Only include hackers who solved more than one challenge perfectly
ORDER BY
    COUNT(*) DESC,    -- Rank hackers by number of full-score challenges (highest first)
    h.hacker_id ASC   -- Tie-breaker: order by hacker ID in ascending order
;
```
<br>



183. HackerRank

``` sql
/*
    CTE #1: Compute the total challenges created per hacker.
    Assign ranks in descending order of challenge count.
    Hackers with equal counts share the same rank.
*/
WITH ranked_hackers AS (
    SELECT
        h.hacker_id,
        h.name,
        COUNT(*) AS challenge_count,
        RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
    FROM challenges c
    LEFT JOIN hackers h ON c.hacker_id = h.hacker_id
    GROUP BY h.hacker_id, h.name
),

/*
    CTE #2: Calculate the frequency of each rank.
    Used to distinguish between tied and unique ranks.
*/
rank_frequencies AS (
    SELECT
        rnk,
        COUNT(*) AS freq
    FROM ranked_hackers
    GROUP BY rnk
)

/*
    Final result:
    - Always include all top-ranked hackers (rank = 1).
    - Include lower-ranked hackers only if their rank is unique (freq = 1).
    - Exclude tied ranks greater than 1.
*/
SELECT
    rh.hacker_id,
    rh.name,
    rh.challenge_count
FROM ranked_hackers rh
JOIN rank_frequencies rf ON rh.rnk = rf.rnk
WHERE
    rh.rnk = 1
    OR (rh.rnk > 1 AND rf.freq = 1)
ORDER BY
    rh.challenge_count DESC,
    rh.hacker_id;
```
<br>


182. Stripe

``` sql
-- CTE: Calculate the time difference (in minutes) between consecutive transactions
-- for the same merchant, credit card, and transaction amount.
WITH transaction_comparison AS (
    SELECT 
        merchant_id,
        EXTRACT(
            EPOCH FROM transaction_timestamp 
            - LAG(transaction_timestamp) OVER (
                PARTITION BY merchant_id, credit_card_id, amount
                ORDER BY transaction_timestamp
            )
        ) / 60 AS minute_difference
    FROM 
        transactions
)

-- Final query: Count how many transactions occurred within 10 minutes
-- of the previous transaction for the same merchant, credit card, and amount.
SELECT
    COUNT(minute_difference) AS payment_count
FROM
    transaction_comparison
WHERE
    minute_difference <= 10;

```
<br>


181. Bloomberg

``` sql
-- Get each ticker's highest and lowest opening price and the month-year it occurred
SELECT
    ticker,

    -- Find month-year of highest opening price for each ticker
    TO_CHAR(
        MAX(date) FILTER (
            WHERE open = (
                SELECT MAX(open)
                FROM stock_prices t2
                WHERE t2.ticker = t1.ticker
            )
        ),
        'Mon-YYYY'
    ) AS highest_mth,

    -- Highest opening price for the ticker
    MAX(open) AS highest_open,

    -- Find month-year of lowest opening price for each ticker
    TO_CHAR(
        MIN(date) FILTER (
            WHERE open = (
                SELECT MIN(open)
                FROM stock_prices t2
                WHERE t2.ticker = t1.ticker
            )
        ),
        'Mon-YYYY'
    ) AS lowest_mth,

    -- Lowest opening price for the ticker
    MIN(open) AS lowest_open

FROM
    stock_prices t1

-- Group results by ticker symbol
GROUP BY
    ticker

-- Order results alphabetically by ticker
ORDER BY
    ticker;
```
<br>


180. Ollivander's Inventory

``` sql
-- Retrieve details of the most cost-efficient non-evil wands,
-- ensuring that for each (age, power) combination, only the wand(s) 
-- requiring the minimum coins_needed is selected.
-- Results are sorted by power (descending) and then by age (descending).

SELECT
    w.id,               -- Unique wand identifier
    p.age,              -- Age category of the wand
    w.coins_needed,     -- Number of coins required to purchase the wand
    w.power             -- Power rating of the wand
FROM wands AS w
JOIN wands_property AS p
    ON w.code = p.code
WHERE
    p.is_evil = 0  -- Only include non-evil wands
    AND (p.age, w.power, w.coins_needed) IN (
        -- Subquery: for each (age, power) pair, find the minimum coins_needed
        SELECT 
            p2.age,
            w2.power,
            MIN(w2.coins_needed) AS min_coins_needed
        FROM wands AS w2
        JOIN wands_property AS p2
            ON w2.code = p2.code
        WHERE p2.is_evil = 0
        GROUP BY p2.age, w2.power
    )
ORDER BY
    w.power DESC,  -- Higher power first
    p.age DESC;    -- For equal power, older wands first
```
<br>


179. Election results

``` sql
-- 1. Filter out rows where candidate is NULL or blank
WITH filtered AS (
  SELECT 
    voter, 
    candidate
  FROM voting_results
  WHERE NULLIF(TRIM(candidate), '') IS NOT NULL
),

-- 2. Assign each (voter, candidate) row a weight
--    Weight = 1 / total number of votes cast by that voter
scored AS (
  SELECT
    voter,
    candidate,
    1.0 / COUNT(*) OVER (PARTITION BY voter) AS vote_points
  FROM filtered
),

-- 3. Aggregate total points per candidate
agg AS (
  SELECT
    candidate,
    SUM(vote_points) AS total_points
  FROM scored
  GROUP BY candidate
)

-- 4. Rank candidates by total points and pick the top one(s)
SELECT candidate
FROM (
  SELECT
    candidate,
    total_points,
    RANK() OVER (ORDER BY total_points DESC) AS rnk
  FROM agg
) r
WHERE rnk = 1;
```
<br>


178. Weather observation station

``` sql
-- Select the city or cities with the shortest and longest names from the station table
-- Includes ties if multiple cities share the same shortest or longest length

SELECT city, len
FROM (
    SELECT
        city,
        CHAR_LENGTH(city) AS len,
        
        -- Assign rank based on ascending city name length and alphabetical order
        RANK() OVER (ORDER BY CHAR_LENGTH(city), city) AS shortest_rank,
        
        -- Assign rank based on descending city name length and alphabetical order
        RANK() OVER (ORDER BY CHAR_LENGTH(city) DESC, city) AS longest_rank
    FROM
        station
) ranked

-- Filter for cities ranked first in either shortest or longest length
WHERE shortest_rank = 1 OR longest_rank = 1;
```
<br>


177. Friend requests

``` sql
-- Create a unified list of all user IDs involved in friendships,
-- including both requesters and accepters
SELECT
    id,
    COUNT(id) AS num
FROM (
    -- Select requester IDs and alias them as 'id'
    SELECT requester_id AS id
    FROM RequestAccepted

    UNION ALL

    -- Select accepter IDs and alias them as 'id'
    SELECT accepter_id
    FROM RequestAccepted
) AS all_friends

-- Group by user ID to count total occurrences (i.e., total friendships)
GROUP BY
    id

-- Order by the count in descending order to find the most connected user
ORDER BY
    num DESC

-- Return only the top result (user with the most friends)
LIMIT
    1;
```
<br>


176. Amazon

``` sql
-- Define a CTE to identify session intervals
-- For each server_id, pair each session status with the timestamp of the following event
WITH session_info AS (
  SELECT
    server_id,
    session_status,
    status_time AS start_time,
    
    -- Use LEAD to look ahead to the next status_time within the same server_id partition
    -- This allows us to derive the end_time of a session based on when the next event occurs
    LEAD(status_time) OVER (
      PARTITION BY server_id 
      ORDER BY status_time
    ) AS end_time

  FROM
    server_utilization
  ORDER BY
    server_id ASC,
    status_time ASC
)

-- Aggregate total uptime for sessions that started
SELECT
  -- Calculate total duration in seconds, then convert to days by dividing by 86400
  -- EXTRACT(EPOCH FROM interval) returns the duration in seconds
  -- FLOOR is used to return the result as a whole number (i.e., complete days only)
  FLOOR(
    EXTRACT(EPOCH FROM SUM(end_time - start_time)) / 86400
  ) AS total_uptime_days
FROM
  session_info
WHERE
  session_status = 'start';
```
<br>


175. Amazon

``` sql
SELECT
  -- Calculate how many prime_eligible items can fit into 500,000 total square footage
  COUNT(*) FILTER (WHERE item_type = 'prime_eligible') *
    FLOOR(
      500000 /
        SUM(square_footage) FILTER (WHERE item_type = 'prime_eligible')
    ) AS prime_eligible,

  -- Calculate how many not_prime items can fit into the remaining space
  COUNT(*) FILTER (WHERE item_type = 'not_prime') *
    FLOOR(
      (
        500000 - 
          SUM(square_footage) FILTER (WHERE item_type = 'prime_eligible') *
            FLOOR(
              500000 / 
                SUM(square_footage) FILTER (WHERE item_type = 'prime_eligible')
            )
      ) /
        SUM(square_footage) FILTER (WHERE item_type = 'not_prime')
    ) AS not_prime
FROM
  inventory;
```
<br>


174. CVS Health

``` sql
-- Aggregate total sales per manufacturer and convert to millions
WITH manufacturer_sales AS (
  SELECT
    manufacturer,
    ROUND(SUM(total_sales) / 1000000, 0) AS sum_of_profit
  FROM
    pharmacy_sales
  GROUP BY
    manufacturer
  ORDER BY
    sum_of_profit DESC, manufacturer ASC
)

-- Format the result with a dollar sign and 'million' suffix
SELECT
  manufacturer,
  CONCAT('$', sum_of_profit, ' million') AS sales_mil
FROM
  manufacturer_sales;

```
<br>


173. LinkedIn

``` sql
-- Aggregate all skills for each candidate into a comma-separated string
WITH candidate_skills AS (
  SELECT 
    candidate_id, 
    STRING_AGG(skill, ', ') AS all_skills
  FROM 
    candidates
  GROUP BY 
    candidate_id
)

-- Select candidates who have all required skills
SELECT 
  candidate_id
FROM 
  candidate_skills
WHERE 
  all_skills LIKE '%Python%'       -- Candidate must know Python
  AND all_skills LIKE '%Tableau%'  -- Candidate must know Tableau
  AND all_skills LIKE '%PostgreSQL%' -- Candidate must know PostgreSQL
ORDER BY 
  candidate_id ASC;
```
<br>


172. FAANG

``` sql
-- CTE to calculate the average salary per department per month
WITH avg_salary AS (
  SELECT
    e.department_id,  -- Department of the employee
    TO_CHAR(s.payment_date, 'MM-YYYY') AS pay_month,  -- Convert payment date to month-year format
    AVG(s.amount) AS salary  -- Average salary for that department and month
  FROM
    salary s
  LEFT JOIN
    employee e ON s.employee_id = e.employee_id  -- Join to get department information
  GROUP BY
    e.department_id,
    TO_CHAR(s.payment_date, 'MM-YYYY')  -- Group by department and month
)

-- Compare each department's average salary to the overall average salary
SELECT
  department_id,
  pay_month,
  CASE
    WHEN salary < (SELECT AVG(amount) FROM salary) THEN 'lower'  -- Below overall average
    WHEN salary = (SELECT AVG(amount) FROM salary) THEN 'same'   -- Equal to overall average
    WHEN salary > (SELECT AVG(amount) FROM salary) THEN 'higher' -- Above overall average
  END AS comparison
FROM
  avg_salary
WHERE
  pay_month = '03-2024';  -- Filter for the specific month of March 2024
```
<br>


171. Confirmation Rate

``` sql
-- Calculate the confirmation rate (percentage of 'confirmed' actions) per user, including users with no confirmations

WITH user_confirmations AS (
    SELECT
        user_id,
        -- Calculate proportion of 'confirmed' actions, rounded to 2 decimal places
        ROUND(SUM(action = 'confirmed') / COUNT(*), 2) AS rate
    FROM
        confirmations
    GROUP BY
        user_id
)

SELECT
    s.user_id,
    -- Use 0 if no confirmation record exists for the user
    COALESCE(uc.rate, 0) AS confirmation_rate
FROM
    signups s
LEFT JOIN
    user_confirmations uc ON s.user_id = uc.user_id;
```
<br>


170. Consecutive numbers

``` sql
-- Step 1: Generate numbered logs with a sliding window to check the next two numbers
WITH numbered_logs AS (
    SELECT
        num,
        LEAD(num, 1) OVER (ORDER BY id) AS num_lead1,  -- Get the next number
        LEAD(num, 2) OVER (ORDER BY id) AS num_lead2   -- Get the number after the next
    FROM
        logs
),

-- Step 2: Identify numbers that appear three times consecutively
consecutive_nums AS (
    SELECT 
        num AS ConsecutiveNums
    FROM
        numbered_logs
    WHERE 
        num = num_lead1 AND num = num_lead2  -- Check if current and next two numbers are the same
)

-- Step 3: Return distinct values of numbers that appear consecutively three times
SELECT DISTINCT
    ConsecutiveNums
FROM
    consecutive_nums;
```
<br>


169. Find users with valid e-emails

``` sql
-- Select users with valid leetcode.com emails based on several validation rules
SELECT
    user_id,
    name,
    mail
FROM
    users
WHERE
    -- First character must be a letter
    mail REGEXP '^[a-zA-Z]'
    
    -- Must contain exactly one '@' symbol
    AND LENGTH(mail) - LENGTH(REPLACE(mail, '@', '')) = 1

    -- Local part (before '@') can only contain a-z, A-Z, 0-9, _, ., -
    AND SUBSTRING_INDEX(mail, '@', 1) REGEXP '^[a-zA-Z0-9_.-]+$'

    -- Domain must be exactly 'leetcode.com'
    AND SUBSTRING_INDEX(mail, '@', -1) = 'leetcode.com';
```
<br>

Combined into one WHERE statement
``` sql
WHERE
    mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode[.]com$'
```
<br>


168. Product Sales Analysis III

``` sql
-- Rank each product's sales by year (earliest year = rank 1)
WITH ordered_by_years AS (
    SELECT
        RANK() OVER (PARTITION BY product_id ORDER BY year ASC) AS rank_by_year,  -- Rank years per product
        product_id,
        year,
        quantity,
        price
    FROM
        sales
)

-- Select the first year’s sales data for each product
SELECT
    product_id,
    year AS first_year,     -- Rename 'year' to 'first_year' for clarity
    quantity,
    price
FROM
    ordered_by_years
WHERE
    rank_by_year = 1        -- Keep only the earliest year per product
;
```
<br>


167. Google

``` sql
-- CTE to assign row numbers to measurements partitioned by day
WITH ranked_measurements AS (
  SELECT
    ROW_NUMBER() OVER (
      PARTITION BY DATE_TRUNC('day', measurement_time) 
      ORDER BY measurement_time
    ) AS ranking,
    measurement_value,
    DATE_TRUNC('day', measurement_time) AS measurement_day
  FROM
    measurements
)

-- Calculate odd and even ranked sums for each day
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


166. Biggest single number

``` sql
-- Get the largest number in `mynumbers` that appears exactly once
SELECT (
    SELECT
        num
    FROM
        mynumbers
    GROUP BY
        num
    HAVING
        COUNT(num) = 1
    ORDER BY
        num DESC
    LIMIT
        1
) AS num;
```
<br>


165. Snapchat

``` sql
-- Calculate percentage of time spent on 'send' and 'open' activities by age bucket
SELECT
    age_bucket,

    -- Percentage of time spent on 'send' activities
    ROUND(
        100.0 * SUM(CASE WHEN activity_type = 'send' THEN time_spent ELSE 0 END)
        /
        SUM(CASE WHEN activity_type IN ('send', 'open') THEN time_spent ELSE 0 END),
        2
    ) AS send_perc,

    -- Percentage of time spent on 'open' activities
    ROUND(
        100.0 * SUM(CASE WHEN activity_type = 'open' THEN time_spent ELSE 0 END)
        /
        SUM(CASE WHEN activity_type IN ('send', 'open') THEN time_spent ELSE 0 END),
        2
    ) AS open_perc

FROM
    activities

    -- Join age breakdown to map user_id to age_bucket
    LEFT JOIN age_breakdown
        ON activities.user_id = age_breakdown.user_id

GROUP BY
    age_bucket;
```
<br>


164. FAANG

``` sql
-- Rank employees within each department by salary (highest first)
WITH ranked_departments AS (
    SELECT
        dense_rank() OVER (
            PARTITION BY e.department_id  -- Restart ranking for each department
            ORDER BY salary DESC          -- Rank by salary descending
        ) AS ranking,
        name,
        salary,
        department_name
    FROM
        employee AS e
    LEFT JOIN
        department AS d
        ON e.department_id = d.department_id  -- Join to get department names
)

-- Select top 3 highest paid employees per department
SELECT
    department_name,
    name,
    salary
FROM
    ranked_departments
WHERE
    ranking < 4  -- Keep only ranks 1, 2, and 3 (top 3 per department)
ORDER BY
    department_name,  -- Group results by department
    salary DESC,      -- Sort highest salary first within department
    name;             -- Tie-breaker: sort by employee name
```
<br>


163. IBM

``` sql
-- CTE: count distinct queries per employee in Q3 2023
WITH query_count AS (
    SELECT
        employee_id,
        COUNT(DISTINCT query_id) AS total_queries
    FROM
        queries
    WHERE
        query_starttime >= '2023-07-01T00:00:00Z' -- start of Q3
        AND query_starttime < '2023-10-01T00:00:00Z' -- end of Q3
    GROUP BY
        employee_id
    ORDER BY
        total_queries DESC
)

-- Join employees with query counts and group summary
SELECT
    COALESCE(total_queries, '0') AS unique_queries, -- replace NULL with 0
    COUNT(*) AS employee_count -- count employees per query count
FROM
    employees
LEFT JOIN
    query_count ON employees.employee_id = query_count.employee_id
GROUP BY
    unique_queries
ORDER BY
    unique_queries;
```
<br>


162. The PADS

``` sql
-- Select the final result text from the combined subquery
SELECT result_text
FROM (
    -- Get each name with the first letter of their occupation in parentheses
    SELECT 
        CONCAT(name, '(', LEFT(occupation, 1), ')') AS result_text,
        1 AS sort_priority,                -- Set sort priority for individual names
        name AS sort_name,                 -- Used for sorting individual names alphabetically
        NULL AS sort_count                 -- Placeholder for count (not needed in this block)
    FROM occupations

    UNION ALL

    -- Get count of each occupation in lowercase with proper message
    SELECT 
        CONCAT('There are a total of ', COUNT(*), ' ', LOWER(occupation), 's.') AS result_text,
        2 AS sort_priority,                -- Set sort priority for summary counts
        LOWER(occupation) AS sort_name,    -- Used for sorting counts alphabetically if counts are equal
        COUNT(*) AS sort_count             -- Used for sorting by count
    FROM occupations
    GROUP BY occupation
) AS combined_result

-- Order results: first names alphabetically, then counts by count ascending and name alphabetically
ORDER BY 
    sort_priority ASC,                     -- First show individual names (priority 1), then counts (priority 2)
    CASE WHEN sort_priority = 1 THEN sort_name END ASC,  -- Sort names alphabetically
    CASE WHEN sort_priority = 2 THEN sort_count END ASC, -- Sort counts ascending
    CASE WHEN sort_priority = 2 THEN sort_name END ASC   -- If counts equal, sort by occupation name
;
```
<br>


161. Restaurant Growth

``` sql
-- Get dates, total amounts, and 7-day rolling averages for customer activity
SELECT
    visited_on,
    amount,
    average_amount
FROM (
    -- Compute total amount and rolling 7-day average for each date
    SELECT DISTINCT
        visited_on,
        
        -- Sum of amounts over the current day and previous 6 days
        SUM(amount) OVER (
            ORDER BY visited_on
            RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
        ) AS amount,
        
        -- Rolling 7-day average of amounts, rounded to 2 decimal places
        ROUND(
            SUM(amount) OVER (
                ORDER BY visited_on
                RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
            ) / 7, 2
        ) AS average_amount
    FROM
        Customer
) AS whole_totals
WHERE
    -- Only include rows starting from the 7th day (to ensure full 7-day window)
    DATEDIFF(
        visited_on,
        (SELECT MIN(visited_on) FROM Customer)
    ) >= 6;
```
<br>


160. Exchange seats

``` sql
-- Swap adjacent students in the 'seat' table based on their id
SELECT
    id,
    CASE
        -- If id is odd and there is a next student, take the next student's name
        WHEN id % 2 = 1 
             AND LEAD(student) OVER (ORDER BY id) IS NOT NULL
        THEN LEAD(student) OVER (ORDER BY id)
        
        -- If id is even, take the previous student's name
        WHEN id % 2 = 0 
        THEN LAG(student) OVER (ORDER BY id)
        
        -- Otherwise, keep the current student's name
        ELSE student
    END AS student
FROM
    seat
ORDER BY
    id ASC;
```
<br>


159. Wayfair

Assume you're given a table containing information about Wayfair user transactions for different products. Write a query to calculate the year-on-year growth rate for the total spend of each product, grouping the results by product ID.

``` sql
-- Aggregate yearly spend per product
WITH product_spending AS (
  SELECT
    EXTRACT(YEAR FROM transaction_date) AS year,
    product_id,
    SUM(spend) AS total_spent
  FROM user_transactions
  GROUP BY year, product_id
  ORDER BY product_id ASC, year ASC
)

-- Compare each product’s spend to the prior year and calculate YoY change
SELECT
  a.year,
  a.product_id,
  a.total_spent AS current_year_spend,
  b.total_spent AS prev_year_spend,
  ROUND(100.0 * ((a.total_spent - b.total_spent) / b.total_spent), 2) AS yoy_rate
FROM product_spending a
LEFT JOIN product_spending b
  ON a.year = b.year + 1
  AND a.product_id = b.product_id;
```
<br>




158. Amazon

Assume you're given a table containing data on Amazon customers and their spending on products in different category, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

``` sql
-- Rank products within each category by total spend in 2022
WITH ranked_products AS (
    SELECT
        category,
        product,
        SUM(spend) AS total_spent,
        ROW_NUMBER() OVER (
            PARTITION BY category ORDER BY SUM(spend) DESC
        ) AS ranking
    FROM
        product_spend
    WHERE
        transaction_date BETWEEN '2022-01-01' AND '2022-12-31'
    GROUP BY
        category,
        product
)

-- Select top 2 products per category
SELECT
    category,
    product,
    total_spent
FROM
    ranked_products
WHERE
    ranking < 3;
```
<br>


157. User with Most Approved Flags

Which user flagged the most distinct videos that ended up approved by YouTube? Output, in one column, their full name or names in case of a tie. In the user's full name, include a space between the first and the last name.

``` sql
-- CTE to calculate the number of distinct approved videos per user
WITH cte AS (
    SELECT
        -- Build a clean, properly spaced username from first and last names
        -- NULLIF avoids double spaces if a name part is blank
        -- CONCAT_WS automatically skips NULL values
        -- TRIM removes any leading/trailing spaces
        TRIM(CONCAT_WS(' ', NULLIF(u.user_firstname, ''), NULLIF(u.user_lastname, ''))) AS username,

        -- Count distinct videos flagged by this user and approved
        COUNT(DISTINCT u.video_id) AS vids,

        -- Assign a rank based on the count (highest count = rank 1)
        -- DENSE_RANK ensures that ties share the same rank value
        DENSE_RANK() OVER (
            ORDER BY COUNT(DISTINCT u.video_id) DESC
        ) AS rnk
    FROM user_flags u

    -- Join only approved flags from the review table
    -- Filter is applied in the ON clause to ensure only "APPROVED" reviews are counted
    JOIN flag_review f
        ON f.flag_id = u.flag_id
       AND f.reviewed_outcome = 'APPROVED'

    -- Group by the raw first/last name fields to ensure correct counts
    GROUP BY u.user_firstname, u.user_lastname
)

-- Final output: only the user(s) with the top approved video count
SELECT
    username,
    vids
FROM cte
WHERE rnk = 1;
```
<br>


156. Interviews

Samantha interviews many candidates from different colleges using coding challenges and contests. Write a query to print the contest_id, hacker_id, name, and the sums of total_submissions, total_accepted_submissions, total_views, and total_unique_views for each contest sorted by contest_id. Exclude the contest from the result if all four sums are 0.

Note: A specific contest can be used to screen candidates at more than one college, but each college only holds 1 screening contest.

``` sql
-- Select contest details along with aggregated statistics for submissions and views
SELECT
    cn.contest_id,
    cn.hacker_id,
    cn.name,
    SUM(s.total_submissions) AS total_submissions,
    SUM(s.total_accepted_submissions) AS total_accepted_submissions,
    SUM(v.total_views) AS total_views,
    SUM(v.total_unique_views) AS total_unique_views
FROM
    contests cn

-- Join contests with colleges to link contests to their respective colleges
JOIN 
    colleges cl ON cn.contest_id = cl.contest_id

-- Join colleges with challenges to get all challenges for each college
JOIN 
    challenges ch ON cl.college_id = ch.college_id

-- Left join aggregated view statistics for each challenge
LEFT JOIN (
    SELECT
        challenge_id,
        SUM(total_views) AS total_views,
        SUM(total_unique_views) AS total_unique_views
    FROM
        view_stats
    GROUP BY
        challenge_id
) v ON ch.challenge_id = v.challenge_id

-- Left join aggregated submission statistics for each challenge
LEFT JOIN (
    SELECT
        challenge_id,
        SUM(total_submissions) AS total_submissions,
        SUM(total_accepted_submissions) AS total_accepted_submissions
    FROM
        submission_stats
    GROUP BY
        challenge_id
) s ON ch.challenge_id = s.challenge_id

-- Group results by contest to aggregate stats at the contest level
GROUP BY
    cn.contest_id,
    cn.hacker_id,
    cn.name

-- Filter out contests that have no submissions or views at all
HAVING
    total_submissions != 0 OR
    total_accepted_submissions != 0 OR
    total_views != 0 OR
    total_unique_views != 0

-- Order the results by contest_id for better readability
ORDER BY
    cn.contest_id
;
```
<br>



155. Host Popularity Rental Prices

You are given a table named airbnb_host_searches that contains data for rental property searches made by users. Determine the minimum, average, and maximum rental prices for each popularity-rating bucket. A popularity-rating bucket should be assigned to every record based on its number_of_reviews.

The host’s popularity rating is defined as below:
•   0 reviews: "New"
•   1 to 5 reviews: "Rising"
•   6 to 15 reviews: "Trending Up"
•   16 to 40 reviews: "Popular"
•   More than 40 reviews: "Hot"

Output host popularity rating and their minimum, average and maximum rental prices. Order the solution by the minimum price.

``` sql
-- Classify listings into popularity categories based on number_of_reviews
WITH rating AS (
    SELECT
        id,
        price,
        number_of_reviews,
        CASE
            WHEN number_of_reviews = 0 THEN 'New'
            WHEN number_of_reviews BETWEEN 1 AND 5 THEN 'Rising'
            WHEN number_of_reviews BETWEEN 6 AND 15 THEN 'Trending Up'
            WHEN number_of_reviews BETWEEN 16 AND 40 THEN 'Popular'
            ELSE 'Hot'
        END AS pop_rating
    FROM airbnb_host_searches
)

-- Compute price statistics by popularity category
SELECT
    pop_rating,
    MIN(price) AS min_price,              -- Minimum price in category
    ROUND(AVG(price), 2) AS avg_price,    -- Average price (2 decimals)
    MAX(price) AS max_price               -- Maximum price in category
FROM rating
GROUP BY pop_rating
ORDER BY min_price;                       -- Sort results by minimum price
```
<br>





154. Symmetric Pairs

You are given a table, Functions, containing two columns: X and Y. Two pairs (X1, Y1) and (X2, Y2) are said to be symmetric pairs if X1 = Y2 and X2 = Y1. Write a query to output all such symmetric pairs in ascending order by the value of X. List the rows such that X1 ≤ Y1.

``` sql
-- Select pairs of (x, y) from the functions table
SELECT 
    f1.x, 
    f1.y
FROM 
    functions f1

-- Join the table with itself to find reciprocal pairs (where x=y and y=x)
JOIN 
    functions f2 ON f1.x = f2.y AND f1.y = f2.x

-- Group by (x, y) to prepare for aggregation in the HAVING clause
GROUP BY 
    f1.x, 
    f1.y

-- Filter to include only:
-- 1. Pairs appearing more than once (COUNT > 1)
-- 2. Or pairs where x is less than y to avoid duplicates in reciprocal pairs
HAVING 
    COUNT(f1.x) > 1 OR f1.x < f1.y

-- Order the result set by x for better readability
ORDER BY 
    f1.x;
```
<br>


153. Placements

You are given three tables: Students, Friends and Packages. Students contains two columns: ID and Name. Friends contains two columns: ID and Friend_ID (ID of the ONLY best friend). Packages contains two columns: ID and Salary (offered salary in $ thousands per month). Write a query to output the names of those students whose best friends got offered a higher salary than them. Names must be ordered by the salary amount offered to the best friends. It is guaranteed that no two students got same salary offer.

``` sql
-- Select the names of students whose friends have higher package salaries
SELECT
    s.name
FROM
    students s

-- Join students with their own packages
JOIN
    packages p1 ON s.id = p1.id

-- Join students with their friends
JOIN
    friends f ON s.id = f.id

-- Join friends with their respective packages
JOIN
    packages p2 ON f.friend_id = p2.id

-- Filter for cases where a friend's salary is higher than the student's salary
WHERE
    p2.salary > p1.salary

-- Order the results by the friend's salary (ascending)
ORDER BY
    p2.salary;
```
<br>


152. SQL Project Planning

You are given a table, Projects, containing three columns: Task_ID, Start_Date and End_Date. It is guaranteed that the difference between the End_Date and the Start_Date is equal to 1 day for each row in the table.

``` sql
-- Select the earliest start_date and latest end_date for each group of consecutive projects
SELECT 
    MIN(start_date) AS start_date,
    MAX(end_date) AS end_date
FROM (
    -- Assign group IDs to consecutive projects where start_date equals previous end_date
    SELECT 
        start_date,
        end_date,
        @grp := IF(@prev_end = start_date, @grp, @grp + 1) AS group_id,  -- Increment group_id if current start_date does not match previous end_date
        @prev_end := end_date  -- Update @prev_end for the next row
    FROM
        projects
    -- Initialize session variables for tracking previous end_date and group ID
    JOIN
        (SELECT @prev_end := NULL, @grp := 0) vars
    -- Ensure rows are processed in chronological order by start_date
    ORDER BY
        start_date
) AS grouped

-- Group projects by the assigned group_id to find ranges of consecutive projects
GROUP BY
    group_id

-- Order the result by the duration of each group (shortest first), then by start_date
ORDER BY
    DATEDIFF(MAX(end_date), MIN(start_date)) ASC,
    MIN(start_date) ASC
;
```
<br>


151. Contest Leaderboard

The total score of a hacker is the sum of their maximum scores for all of the challenges. Write a query to print the hacker_id, name, and total score of the hackers ordered by the descending score. If more than one hacker achieved the same total score, then sort the result by ascending hacker_id. Exclude all hackers with a total score of 0 from your result.

``` sql
-- Select hacker IDs, names, and their total score across challenges
SELECT
    h.hacker_id,
    h.name,
    SUM(ms.score) AS total_score
FROM
    hackers AS h

-- Join with a subquery that gets each hacker's highest score per challenge
INNER JOIN (
    SELECT
        hacker_id,
        challenge_id,
        MAX(score) AS score
    FROM
        submissions
    GROUP BY
        hacker_id, challenge_id
) AS ms ON h.hacker_id = ms.hacker_id

-- Group by hacker to aggregate their total score
GROUP BY
    h.hacker_id, h.name

-- Include only hackers whose total score is greater than zero
HAVING
    total_score > 0

-- Order the results by total score (descending) and hacker ID (ascending)
ORDER BY
    total_score DESC,
    h.hacker_id;

```
<br>




150. Flags per Video

For each video, find how many unique users flagged it. A unique user can be identified using the combination of their first name and last name. Do not consider rows in which there is no flag ID.

``` sql
-- Count the number of unique users per video_id.
-- A user is identified by the concatenation of user_firstname and user_lastname,
-- with NULL first names replaced by the last name (via COALESCE).
-- Only considers rows where flag_id is not NULL.

SELECT
    video_id,
    COUNT(
        DISTINCT CONCAT(
            COALESCE(user_firstname, user_lastname)
        )
    ) AS num_users
FROM user_flags
WHERE flag_id IS NOT NULL
GROUP BY video_id;
```
<br>


149. Top 10 Songs 2010

Find the top 10 ranked songs in 2010. Output the rank, group name, and song name, but do not show the same song twice. Sort the result based on the rank in ascending order.

``` sql
-- Top 10 songs from the Billboard Year-End Hot 100 for 2010

SELECT DISTINCT
    year_rank,
    group_name,
    song_name
FROM
    billboard_top_100_year_end
WHERE
    year = '2010'
    AND year_rank <= 10   -- Limit to the top 10
ORDER BY
    year_rank ASC;        -- Rank #1 at the top
```
<br>


148. The Report

Ketty gives Eve a task to generate a report containing three columns: Name, Grade and Mark. Ketty doesn't want the NAMES of those students who received a grade lower than 8. The report must be in descending order by grade -- i.e. higher grades are entered first. If there is more than one student with the same grade (8-10) assigned to them, order those particular students by their name alphabetically. Finally, if the grade is lower than 8, use "NULL" as their name and list them by their grades in descending order. If there is more than one student with the same grade (1-7) assigned to them, order those particular students by their marks in ascending order.

``` sql
SELECT
    CASE
        WHEN FLOOR(marks/10) + 1 > 7 THEN name
        ELSE NULL END,
    CASE
        WHEN marks = 100 THEN 10
        ELSE FLOOR(marks/10) + 1 END,
    marks
FROM
    students
ORDER BY
    2 DESC,
    1,
    3 ASC
;
```
<br>


147. Weather Observation Station 20

Query the median of the Northern Latitudes (LAT_N) from STATION and round your answer to 4 decimal places.

``` sql
SELECT
    ROUND(AVG(lat_n), 4) AS median
FROM (
    SELECT
        lat_n,
        ROW_NUMBER() OVER (ORDER BY lat_n) AS rn,
        COUNT(*) OVER () AS cnt
    FROM
        station
    ) AS ranked
WHERE
    rn IN (FLOOR((cnt + 1)/2), CEIL((cnt + 1)/2));
```
<br>


146. New Companies

Amber's conglomerate corporation just acquired some new companies. Each of the companies follows this hierarchy:

![image](https://github.com/user-attachments/assets/611647ec-f9d6-4664-aaf1-6050d8e51524)

Write a query to print the company_code, founder name, total number of lead managers, total number of senior managers, total number of managers, and total number of employees. The tables may contain duplicate records.

``` sql
SELECT
    e.company_code,
    c.founder,
    COUNT(DISTINCT e.lead_manager_code),
    COUNT(DISTINCT e.senior_manager_code),
    COUNT(DISTINCT e.manager_code),
    count(DISTINCT e.employee_code)
FROM
    employee e
JOIN
    company c ON c.company_code = e.company_code
GROUP BY
    1, 2
;
```
<br>


145. Processed Ticket Rate By Type

Find the processed rate of tickets for each type. The processed rate is defined as the number of processed tickets divided by the total number of tickets for that type. Round this result to two decimal places.

``` sql
-- Calculate the average number of processed complaints per type
SELECT
    type,
    ROUND(SUM(processed) / COUNT(processed), 2) AS avg_processed
FROM
    facebook_complaints
GROUP BY
    type;
```
<br>


144. Draw the Triangle 1

P(R) represents a pattern drawn in R rows. Write a query to print the pattern P(20) starting with P(20) and ending with P(1).

``` sql
WITH RECURSIVE counter_cte AS (
    SELECT 20 AS n
    UNION ALL
    SELECT n - 1 FROM counter_cte WHERE n > 1
)

SELECT REPEAT('* ', n) AS output
FROM counter_cte;
```
<br>



143. Google - Salary by Education

Given the education levels and salaries of a group of individuals, find what is the average salary for each level of education.

``` sql
SELECT
    AVG(salary) AS avg_salary,
    education
FROM
    google_salaries
GROUP BY
    education;
```
<br>


142. Airbnb - Find searches with no data for the host_response_rate column

Find all search details where data is missing from the host_response_rate column.

``` sql
SELECT
    *
FROM
    airbnb_search_details
WHERE
    host_response_rate IS NULL
;
```
<br>


141. Amazon - April & May Sign Up's

You have been asked to get a list of all the sign up IDs with transaction start dates in either April or May. Since a sign up ID can be used for multiple transactions only output the unique ID. Your output should contain a list of non duplicated sign-up IDs.

``` sql
SELECT DISTINCT signup_id
FROM transactions
WHERE transaction_start_date BETWEEN '2020-04-01' AND '2020-05-31'
;
```
<br>







140. Average Population of Each Continent

Given the CITY and COUNTRY tables, query the names of all the continents (COUNTRY.Continent) and their respective average city populations (CITY.Population) rounded down to the nearest integer.

``` sql
SELECT
    country.continent,
    FLOOR(AVG(city.population))
FROM
    country
JOIN
    city ON city.countrycode = country.code 
GROUP BY
    country.continent
;
```
<br>


139. African Cities

Given the CITY and COUNTRY tables, query the names of all cities where the CONTINENT is 'Africa'.

``` sql
SELECT
    city.name
FROM
    city
LEFT JOIN
    country ON country.code = city.countrycode
WHERE
    continent = 'Africa'
;
```
<br>


138. Population Census

Given the CITY and COUNTRY tables, query the sum of the populations of all cities where the CONTINENT is 'Asia'.

``` sql
SELECT
    SUM(city.population)
FROM
    country
LEFT JOIN
    city on city.countrycode = country.code
WHERE
    CONTINENT = 'ASIA'
;
```
<br>


137. Weather Observation Station 19

Consider  and  to be two points on a 2D plane where (a,b) are the respective minimum and maximum values of Northern Latitude (LAT_N) and (c,d) are the respective minimum and maximum values of Western Longitude (LONG_W) in STATION. Query the Euclidean Distance between points P_1 and P_2 and format your answer to display 4 decimal digits.

``` sql
SELECT
    ROUND(SQRT(POW(MIN(LONG_W)-MAX(LONG_W), 2) + POW(MIN(LAT_N)-MAX(LAT_N), 2)), 4)
FROM
    STATION
;
```
<br>


136. Weather Observation Station 18

Query the Manhattan Distance between points P1 and P2 and round it to a scale of 4 decimal places.

``` sql
SELECT
    ROUND(
        ABS(MIN(LAT_N)-MAX(LAT_N))+
        ABS(MIN(LONG_W)-MAX(LONG_W)), 
        4)
FROM
    STATION
;
```
<br>


135. Weather Observation Station 17

Query the Western Longitude (LONG_W) where the smallest Northern Latitude (LAT_N) in STATION is greater than 38.7780. Round your answer to 4 decimal places.

``` sql
SELECT
    ROUND(LONG_W, 4)
FROM
    STATION
WHERE
    LAT_N > 38.7780
ORDER BY
    LAT_N ASC
LIMIT
    1
;
```
<br>


134. Weather Observation Station 16

Query the smallest Northern Latitude (LAT_N) from STATION that is greater than 38.7780. Round your answer to 4 decimal places.

``` sql
SELECT
    MIN(ROUND(LAT_N, 4))
FROM
    STATION
WHERE
    LAT_N > 38.7780
;
```
<br>


133. Weather Observation Station 15

Query the Western Longitude (LONG_W) for the largest Northern Latitude (LAT_N) in STATION that is less than 137.2345. Round your answer to 4 decimal places.

``` sql
SELECT
    ROUND(LONG_W, 4)
FROM
    STATION
WHERE
    LAT_N < 137.2345
ORDER BY
    LAT_N DESC
LIMIT
    1
;
```
<br>


132. Weather Observation Station 14

Query the greatest value of the Northern Latitudes (LAT_N) from STATION that is less than 137.2345. Truncate your answer to 4 decimal places.

``` sql
SELECT
    ROUND(MAX(LAT_N), 4)
FROM
    STATION
WHERE
    LAT_N < 137.2345
;
```
<br>


131. Weather Observation Station 13

Query the sum of Northern Latitudes (LAT_N) from STATION having values greater than 38.7880 and less than 137.2345. Truncate your answer to  decimal places.

``` sql
SELECT
    ROUND(SUM(LAT_N), 4)
FROM
    STATION
WHERE
    LAT_N > 38.7880
    AND LAT_N < 137.2345
;
```
<br>


130. Weather Observation Station 2

Query the following two values from the STATION table:

The sum of all values in LAT_N rounded to a scale of 2 decimal places.
The sum of all values in LONG_W rounded to a scale of 2 decimal places.

``` sql
SELECT
    ROUND(SUM(LAT_N), 2),
    ROUND(SUM(LONG_W), 2)
FROM
    STATION
;
```
<br>


129. Top Earners

We define an employee's total earnings to be their monthly salary x months worked, and the maximum total earnings to be the maximum total earnings for any employee in the Employee table. Write a query to find the maximum total earnings for all employees as well as the total number of employees who have maximum total earnings. Then print these values as 2 space-separated integers.

``` sql
SELECT
    salary * months AS earnings,
    COUNT(*)
FROM
    Employee
GROUP BY
    earnings
ORDER BY
    earnings DESC
LIMIT
    1
;
```
<br>


128. The Blunder

Samantha was tasked with calculating the average monthly salaries for all employees in the EMPLOYEES table, but did not realize her keyboard's  key was broken until after completing the calculation. She wants your help finding the difference between her miscalculation (using salaries with any zeros removed), and the actual average salary.

Write a query calculating the amount of error, and round it up to the next integer.

``` sql
SELECT
    CEIL(AVG(salary) - AVG(REPLACE(salary, '0', '')))
FROM
    employees
;
```
<br>


127. Population Density Difference

Query the difference between the maximum and minimum populations in CITY.

``` sql
SELECT
    MAX(population) - MIN(population)
FROM
    city
;
```
<br>


126. Japan Population

Query the sum of the populations for all Japanese cities in CITY. The COUNTRYCODE for Japan is JPN.

``` sql
SELECT
    SUM(population)
FROM
    city
WHERE
    countrycode = 'JPN'
;
```
<br>


125. Average Population

Query the average population for all cities in CITY, rounded down to the nearest integer.

``` sql
SELECT
    ROUND(AVG(population), 0)
FROM
    city
;
```
<br>


124. Revising Aggregations - Averages

Query the average population of all cities in CITY where District is California.

``` sql
SELECT
    AVG(population)
FROM
    city
WHERE
    district = 'California'
;
```
<br>


123. Revising Aggregations - The Sum Function

Query the total population of all cities in CITY where District is California.

``` sql
SELECT
    SUM(population)
FROM
    city
WHERE
    district = 'California'
;
```
<br>


122. Binary tree nodes

You are given a table, BST, containing two columns: N and P, where N represents the value of a node in Binary Tree, and P is the parent of N.

Write a query to find the node type of Binary Tree ordered by the value of the node. Output one of the following for each node: <br>

Root: If node is root node. <br>
Leaf: If node is leaf node. <br>
Inner: If node is neither root nor leaf node.

``` sql
SELECT
    CASE
        WHEN P IS NULL THEN CONCAT(N, ' Root')
        WHEN N IN (SELECT P FROM BST WHERE P IS NOT NULL) THEN CONCAT(N, ' Inner')
        ELSE CONCAT(N, ' Leaf')
        END
FROM
    BST
ORDER BY
    N ASC
;
```
<br>


121. Revising Aggregations - The Count Function

Query a count of the number of cities in CITY having a Population larger than 100k.

``` sql
SELECT
    COUNT(*)
FROM
    city
WHERE
    population > 100000
;
```
<br>


120. Top Businesses With Most Reviews

``` sql
-- Select top 5 businesses with the highest review counts
SELECT
    name,
    review_count
FROM
    yelp_business
ORDER BY
    review_count DESC
LIMIT 5;
```
<br>


119. San Francisco health inspection violations

You are given a dataset of health inspections that includes details about violations. Each row represents an inspection, and if an inspection resulted in a violation, the violation_id column will contain a value.

Count the total number of violations that occurred at 'Roxanne Cafe' for each year, based on the inspection date. Output the year and the corresponding number of violations in ascending order of the year.

``` sql
-- Get yearly count of inspections for Roxanne Cafe
SELECT
    YEAR(inspection_date) AS inspection_year,
    COUNT(*) AS total_inspections
FROM
    sf_restaurant_health_violations
WHERE
    business_name = 'Roxanne Cafe'
GROUP BY
    inspection_year
ORDER BY
    inspection_year;
```
<br>


118. Type of triangle

Write a query identifying the type of each record in the TRIANGLES table using its three side lengths.

``` sql
SELECT
    CASE
        WHEN A+B<=C OR A+C<=B OR B+C<=A THEN 'Not A Triangle'
        WHEN A = B AND B = C THEN 'Equilateral'
        WHEN A = B OR B = C OR A = C THEN 'Isosceles'
        ELSE 'Scalene' END
FROM
    triangles
;
```
<br>


117. Employee salaries

Write a query that prints a list of employee names (i.e.: the name attribute) for employees in Employee having a salary greater than 2000 per month who have been employees for less than 10 months. Sort your result by ascending employee_id.

``` sql
SELECT
    name
FROM
    employee
WHERE
    salary > 2000
    AND months < 10
ORDER BY
    employee_ID ASC
;
```
<br>


116. Employee names

Write a query that prints a list of employee names (i.e.: the name attribute) from the Employee table in alphabetical order.

``` sql
SELECT
    name
FROM
    employee
ORDER BY
    name ASC
;
```
<br>


115. Higher than 75 marks

Query the Name of any student in STUDENTS who scored higher than  Marks. Order your output by the last three characters of each name. If two or more students both have names ending in the same last three characters (i.e.: Bobby, Robby, etc.), secondary sort them by ascending ID.

``` sql
SELECT
    name
FROM
    students
WHERE
    marks > 75
ORDER BY
    RIGHT(name, 3), ID
;
```
<br>


114. Weather observation station 12

Query the list of CITY names from STATION that do not start with vowels and do not end with vowels. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city REGEXP '^[^aeiouAEIOU]'
    AND city REGEXP '.*[^aeiouAEIOU]$'
;
```
<br>


113. Weather observation station 11

Query the list of CITY names from STATION that either do not start with vowels or do not end with vowels. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city REGEXP '^[^aeiouAEIOU]'
    OR city REGEXP '.*[^aeiouAEIOU]$'
;
```
<br>


112. Weather observation station 10

Query the list of CITY names from STATION that do not end with vowels. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city REGEXP '.*[^aeiouAEIOU]$'
;
```
<br>


111. Weather observation station 9

Query the list of CITY names from STATION that do not start with vowels. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city REGEXP '^[^aeiouAEIOU].*$'
;
```
<br>


110. Weather observation station 8

Query the list of CITY names from STATION which have vowels (i.e., a, e, i, o, and u) as both their first and last characters. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city REGEXP '^[aeiou].*[aeiou]$'
;
```
<br>


109. Weather observation station 7

Query the list of CITY names ending with vowels (a, e, i, o, u) from STATION. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city LIKE '%a'
    OR city LIKE '%e'
    OR city LIKE '%i'
    OR city LIKE '%o'
    OR city LIKE '%u'
;
```
<br>


108. Weather observation station 6

Query the list of CITY names starting with vowels (i.e., a, e, i, o, or u) from STATION. Your result cannot contain duplicates.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    city LIKE 'a%'
    OR city LIKE 'e%'
    OR city LIKE 'i%'
    OR city LIKE 'o%'
    OR city LIKE 'u%'
;
```
<br>


107. Share of Active Users

Calculate the percentage of users who are both from the US and have an 'open' status, as indicated in the fb_active_users table.

``` sql
-- Calculate the percentage of active users from the USA with 'open' status
SELECT
    100 * AVG(CASE 
        WHEN country = 'USA' AND status = 'open' THEN 1 
        ELSE 0 
    END) AS usa_open_percentage
FROM
    fb_active_users;
```
<br>


106. Weather observation station 4

Find the difference between the total number of CITY entries in the table and the number of distinct CITY entries in the table.

``` sql
SELECT
    COUNT(city) - COUNT(DISTINCT city)
FROM
    station
;
```
<br>


105. Weather observation station 3

Query a list of CITY names from STATION for cities that have an even ID number. Print the results in any order, but exclude duplicates from the answer.

``` sql
SELECT DISTINCT
    city
FROM
    station
WHERE
    MOD(id, 2) = 0
;
```
<br>


104. Weather observation station 1

Query a list of CITY and STATE from the STATION table.

``` sql
SELECT
    city,
    state
FROM
    station
;
```
<br>


103. Japanese cities' names

Query the names of all the Japanese cities in the CITY table. The COUNTRYCODE for Japan is JPN.

``` sql
SELECT
    name
FROM
    city
WHERE
    countrycode = 'JPN'
;
```
<br>


102. Japanese cities' attributes

Query all attributes of every Japanese city in the CITY table. The COUNTRYCODE for Japan is JPN.

``` sql
SELECT
    *
FROM
    city
WHERE
    countrycode = 'JPN'
;
```
<br>


101. Select by ID

Query all columns for a city in CITY with the ID 1661.

``` sql
SELECT
    *
FROM
    city
WHERE
    id = 1661
;
```
<br>


100. Select all

Query all columns (attributes) for every row in the CITY table.

``` sql
SELECT
    *
FROM
    city
;
```
<br>


99. Revising the select query II

Query the NAME field for all American cities in the CITY table with populations larger than 120000. The CountryCode for America is USA.

``` sql
SELECT
    name
FROM
    city
WHERE
    countrycode = 'USA'
    AND population > 120000
;
```
<br>


98. Revising the select query I

Query all columns for all American cities in the CITY table with populations larger than 100000. The CountryCode for America is USA.

``` sql
SELECT
    *
FROM
    city
WHERE
    population > 100000
    AND
    countrycode = 'USA'
;
```
<br>


97. Customer Revenue In March

Calculate the total revenue from each customer in March 2019. Include only customers who were active in March 2019. An active user is a customer who made at least one transaction in March 2019. Output the revenue along with the customer id and sort the results based on the revenue in descending order.

``` sql
-- Aggregate total order cost per customer for March 1–29, 2019
SELECT
    cust_id,
    SUM(total_order_cost) AS total_cost
FROM orders
WHERE order_date BETWEEN '2019-03-01' AND '2019-03-29'
GROUP BY cust_id
ORDER BY 2 DESC;
```
<br>


96. Churro Activity Date

Find the inspection date and risk category (pe_description) of facilities named 'STREET CHURROS' that received a score below 95.

``` sql
-- Get inspection dates and descriptions for STREET CHURROS with scores below 95
SELECT
    activity_date,
    pe_description
FROM
    los_angeles_restaurant_health_inspections
WHERE
    facility_name = 'STREET CHURROS'
    AND score < 95;
```
<br>


95. Second highest salary

Write a solution to find the second highest distinct salary from the Employee table. If there is no second highest salary, return null.

``` sql
SELECT 
  (
    SELECT DISTINCT salary AS SecondHighestSalary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
  ) AS SecondHighestSalary;
;
```
<br>


94. Investments in 2016

Write a solution to report the sum of all total investment values in 2016 tiv_2016, for all policyholders who have the same tiv_2015 value as one or more other policyholders, and are not located in the same city as any other policyholder (i.e., the (lat, lon) attribute pairs must be unique). Round tiv_2016 to two decimal places

``` sql
SELECT
    ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM
    insurance
WHERE
    tiv_2015 IN (
        SELECT tiv_2015
        FROM insurance
        GROUP BY tiv_2015
        HAVING COUNT(*) > 1
    )
    AND (lat, lon) IN (
        SELECT lat, lon
        FROM insurance
        GROUP BY lat, lon
        HAVING COUNT(*) = 1
    )
;
```
<br>


93. Salaries Differences

Calculates the difference between the highest salaries in the marketing and engineering departments. Output just the absolute difference in salaries.

``` sql
-- Absolute difference between top Marketing and Engineering salaries
SELECT
    ABS(
        MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) -
        MAX(CASE WHEN d.department = 'engineering' THEN e.salary END)
    ) AS salary_difference
FROM
    db_employee e
JOIN
    db_dept d ON e.department_id = d.id;
;
```
<br>


92. List the products ordered in a period (1327)

Write a solution to get the names of products that have at least 100 units ordered in February 2020 and their amount. Return the result table in any order.

``` sql
SELECT
    p.product_name,
    SUM(unit) AS unit
FROM
    orders o
JOIN
    products p ON o.product_id = p.product_id
WHERE
    order_date BETWEEN '2020-02-01' AND '2020-02-29'
GROUP BY
    p.product_name
HAVING
    unit >= 100
;
```
<br>


Combined into one WHERE statement
``` sql
WHERE
    mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode[.]com$'
```
<br>


91. Delete duplicate emails

Write a solution to delete all duplicate emails, keeping only one unique email with the smallest id.

``` sql
WITH ranked AS (
    SELECT
        id,
        email,
        LAG(email) OVER (ORDER BY email, id) AS prev_email
    FROM
        person
)

DELETE
FROM person 
WHERE id IN (
    SELECT id
    FROM ranked
    WHERE email = prev_email
);
```
<br>


90. Group sold products by the date

Write a solution to find for each date the number of different products sold and their names. The sold products names for each date should be sorted lexicographically. Return the result table ordered by sell_date.

``` sql
SELECT
    sell_date,
    COUNT(DISTINCT product) AS num_sold,
    GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') AS products
FROM
    activities a1
GROUP BY
    sell_date
;
```
<br>


89. Patients with a condition

Write a solution to find the patient_id, patient_name, and conditions of the patients who have Type I Diabetes. Type I Diabetes always starts with DIAB1 prefix. Return the result table in any order.

``` sql
SELECT
    patient_id,
    patient_name,
    conditions
FROM
    patients
WHERE
    conditions LIKE 'DIAB1%'
    OR conditions LIKE '% DIAB1%'
;
```
<br>


88. Yelp

Find the review_text that received the highest number of  cool votes.  
Output the business name along with the review text with the highest number of cool votes.

``` sql
-- Retrieve reviews that have the highest 'cool' score among all Yelp reviews
SELECT
    business_name,
    review_text
FROM
    yelp_reviews

-- Filter to only include reviews with the maximum 'cool' value
WHERE
    cool = (
        SELECT MAX(cool)
        FROM yelp_reviews
    );
```
<br>


87. Count Salary Categories

Write a solution to calculate the number of bank accounts for each salary category. The salary categories are: <br>

"Low Salary": All the salaries strictly less than $20000. <br>
"Average Salary": All the salaries in the inclusive range [$20000, $50000]. <br>
"High Salary": All the salaries strictly greater than $50000. <br>
The result table must contain all three categories. If there are no accounts in a category, return 0. <br>

Return the result table in any order.

``` sql
SELECT
    "Low Salary" AS category,
    sum(income < 20000) AS accounts_count
FROM
    Accounts

UNION ALL

SELECT
    "Average Salary" AS category,
    sum(income BETWEEN 20000 AND 50000) AS accounts_count
FROM
    Accounts

UNION ALL

SELECT
    "High Salary" AS category,
    sum(income > 50000) AS accounts_count
FROM
    Accounts
;
```
<br>



86. Artist Appearance Count

Find how many times each artist appeared on the Spotify ranking list.  
Output the artist name along with the corresponding number of occurrences.  
Order records by the number of occurrences in descending order.

``` sql
-- Get the number of songs per artist, ordered by most songs
SELECT
    artist,
    COUNT(*)
FROM
    spotify_worldwide_daily_song_ranking
GROUP BY
    artist
ORDER BY
    2 DESC;
```
<br>


85. Last person to fit in the bus

There is a queue of people waiting to board a bus. However, the bus has a weight limit of 1000 kilograms, so there may be some people who cannot board. Write a solution to find the person_name of the last person that can fit on the bus without exceeding the weight limit. The test cases are generated such that the first person does not exceed the weight limit. Note that only one person can board the bus at any given turn.

V1: Subquery
``` sql
SELECT
    person_name
FROM (
    SELECT
        person_name,
        turn,
        SUM(weight) OVER (ORDER BY turn ASC) AS cumulative_weight
    FROM
        queue
    ) AS weight_sum
WHERE
    cumulative_weight <= 1000
ORDER BY
    turn DESC
LIMIT
    1
;
```
<br>


V2: CTE
``` sql
WITH weight_totals AS (
SELECT
    person_name,
    SUM(weight) OVER (ORDER BY turn ASC) AS cumulative_weight
FROM
    queue
ORDER BY
    turn ASC
)

SELECT
    person_name
FROM
    weight_totals
WHERE
    cumulative_weight <= 1000
ORDER BY
    cumulative_weight DESC
LIMIT
    1
;
```
<br>


84. Customers who bought all products (1045)

Write a solution to report the customer ids from the Customer table that bought all the products in the Product table. Return the result table in any order.

``` sql
SELECT
    customer_id
FROM
    customer
GROUP BY
    customer_id
HAVING
    GROUP_CONCAT(DISTINCT product_key ORDER BY product_key ASC SEPARATOR ',') IN (
        SELECT
            GROUP_CONCAT(DISTINCT product_key ORDER BY product_key ASC SEPARATOR ',')
        FROM
            product
    )
;
```
<br>


83. Matching Similar Hosts and Guests

``` sql
/*
    Retrieves all unique host–guest pairs from Airbnb data where the host
    and guest share the same gender and nationality.
*/

SELECT DISTINCT
    h.host_id,       -- Unique identifier for the host
    g.guest_id       -- Unique identifier for the guest
FROM
    airbnb_hosts h
JOIN
    airbnb_guests g
    ON h.gender = g.gender            -- Match by gender
   AND h.nationality = g.nationality  -- Match by nationality
;
```
<br>


82. Immediate food delivery II

If the customer's preferred delivery date is the same as the order date, then the order is called immediate; otherwise, it is called scheduled. The first order of a customer is the order with the earliest order date that the customer made. It is guaranteed that a customer has precisely one first order. Write a solution to find the percentage of immediate orders in the first orders of all customers, rounded to 2 decimal places.

``` sql
SELECT
    ROUND(AVG(order_date = customer_pref_delivery_date)*100, 2) as immediate_percentage
FROM
    delivery
WHERE
    (customer_id, order_date) IN (
        SELECT
            customer_id, MIN(order_date)
        FROM
            delivery
        GROUP BY
            customer_id
    )
;
```
<br>


81. Monthly transactions 1 (1193)

Write an SQL query to find for each month and country, the number of transactions and their total amount, the number of approved transactions and their total amount. Return the result table in any order.

``` sql
SELECT
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(*) AS trans_count,
    SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM
    transactions
GROUP BY
    1, 2
;
```
<br>


80. The number of employees which report to each employee (1731)

For this problem, we will consider a manager an employee who has at least 1 other employee reporting to them. Write a solution to report the ids and the names of all managers, the number of employees who report directly to them, and the average age of the reports rounded to the nearest integer. Return the result table ordered by employee_id.

``` sql
SELECT
    e1.employee_id,
    e1.name,
    COUNT(e2.reports_to) AS reports_count,
    ROUND(AVG(e2.age), 0) AS average_age
FROM
    employees e1
JOIN
    employees e2 ON e1.employee_id = e2.reports_to
GROUP BY
    1
ORDER BY
    1
;
```
<br>


79. Primary department for each employee (1789)

Employees can belong to multiple departments. When the employee joins other departments, they need to decide which department is their primary department. Note that when an employee belongs to only one department, their primary column is 'N'. Write a solution to report all the employees with their primary department. For employees who belong to one department, report their only department. Return the result table in any order.

``` sql
SELECT
    employee_id,
    department_id
FROM
    employee
WHERE
    primary_flag='Y' OR 
    employee_id IN (
        SELECT
            employee_id
        FROM
            Employee
        GROUP BY
            employee_id
        HAVING
            COUNT(employee_id) = 1
    )
;
```
<br>


78. Movie rating (1341)

Find the name of the user who has rated the greatest number of movies. In case of a tie, return the lexicographically smaller user name. Find the movie name with the highest average rating in February 2020. In case of a tie, return the lexicographically smaller movie name.

``` sql
SELECT name AS results
FROM (
    SELECT u.name, COUNT(*) AS rating_count
    FROM movierating mr
    JOIN users u ON mr.user_id = u.user_id
    GROUP BY u.user_id, u.name
    ORDER BY rating_count DESC, u.name ASC
    LIMIT 1
) AS top_user

UNION ALL

SELECT title
FROM (
    SELECT m.title, AVG(mr.rating) AS avg_rating
    FROM movierating mr
    JOIN movies m ON mr.movie_id = m.movie_id
    WHERE mr.created_at BETWEEN '2020-02-01' AND '2020-02-29'
    GROUP BY m.movie_id, m.title
    ORDER BY avg_rating DESC, m.title ASC
    LIMIT 1
) AS top_movie;
```
<br>


77. Fix names in a table (1667)

Write a solution to fix the names so that only the first character is uppercase and the rest are lowercase. Return the result table ordered by user_id.

``` sql
SELECT
    user_id,
    CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2))) AS name
FROM
    users
ORDER BY
    user_id ASC
;
```
<br>


76. Triangle judgement (610)

Report for every three line segments whether they can form a triangle. Return the result table in any order.

``` sql
SELECT
    x,
    y,
    z,
    CASE
        WHEN x+y>z AND x+z>y AND y+z>x THEN 'Yes'
        ELSE 'No'
    END AS triangle
FROM
    triangle
;
```
<br>


75. Unique Users Per Client Per Month

Write a query that returns the number of unique users per client for each month. Assume all events occur within the same year, so only month needs to be be in the output as a number from 1 to 12.

``` sql
-- Count distinct users per client per month from fact_events
SELECT
    client_id,
    MONTH(time_id) AS month,
    COUNT(DISTINCT user_id) AS users_num
FROM
    fact_events
GROUP BY
    client_id,
    month;
```
<br>


74. Forbes

Find the most profitable company from the financial sector. Output the result along with the continent.

``` sql
-- Get the company and continent of the most profitable company
SELECT
    company,
    continent
FROM
    forbes_global_2010_2014
ORDER BY
    profits DESC
LIMIT 1;
```
<br>


73. Employees whose manager left the company (1978)

Find the IDs of the employees whose salary is strictly less than $30000 and whose manager left the company. When a manager leaves the company, their information is deleted from the Employees table, but the reports still have their manager_id set to the manager that left. Return the result table ordered by employee_id.

``` sql
SELECT
    employee_id
FROM
    employees
WHERE
    salary < 30000
    AND manager_id NOT IN (SELECT employee_id FROM employees)
ORDER BY
    employee_id ASC
;
```
<br>


72. Amazon

Find the number of workers by department who joined on or after April 1, 2014. Output the department name along with the corresponding number of workers. Sort the results based on the number of workers in descending order.

``` sql
-- Get count of workers per department who joined after 2014-04-01
SELECT
    department,
    COUNT(*) AS num_workers
FROM
    worker
WHERE
    joining_date >= '2014-04-01'
GROUP BY
    1
ORDER BY
    2 DESC;
```
<br>


71. Find Followers Count (1729)

Write a solution that will, for each user, return the number of followers. Return the result table ordered by user_id in ascending order.

``` sql
SELECT
    user_id,
    count(follower_id) AS followers_count
FROM
    followers
GROUP BY
    user_id
ORDER BY
    user_id ASC
;
```
<br>


70. Classes More Than 5 Students (596)

Write a solution to find all the classes that have at least five students. Return the result table in any order.

``` sql
SELECT
    class
FROM
    courses
GROUP BY
    class
HAVING
    COUNT(student) >= 5
;
```
<br>


69. User Activity for the Past 30 Days 1 (1141)

Write a solution to find the daily active user count for a period of 30 days ending 2019-07-27 inclusively. A user was active on someday if they made at least one activity on that day. Return the result table in any order.

``` sql
SELECT
    activity_date AS day,
    COUNT(DISTINCT user_id) AS active_users
FROM
    activity
WHERE
    activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY
    activity_date
;
```
<br>


68. Queries Quality and Percentage

We define query quality as the average of the ratio between query rating and its position. We also define poor query percentage as the percentage of all queries with rating less than 3.

Write a solution to find each query_name, the quality and poor_query_percentage. Both quality and poor_query_percentage should be rounded to 2 decimal places. Return the result table in any order.

``` sql
SELECT
    query_name,
    ROUND(AVG(rating/position), 2) AS quality,
    ROUND(100.0*AVG(CASE WHEN rating < 3 THEN 1 ELSE 0 END), 2) AS poor_query_percentage
FROM
    queries
GROUP BY
    query_name
;
```
<br>


67. Percentage of Users Attended a Contest (1633)

Write an SQL query that reports the average experience years of all the employees for each project, rounded to 2 digits. Return the result table in any order.

``` sql
SELECT
    contest_id,
    ROUND(100.0*(COUNT(user_id) / (SELECT COUNT(*) FROM users)), 2) AS percentage
FROM
    register
GROUP BY
    contest_id
ORDER BY
    2 DESC, 1 ASC
;
```
<br>


66. Project Employees 1 (1075)

Write an SQL query that reports the average experience years of all the employees for each project, rounded to 2 digits. Return the result table in any order.

``` sql
SELECT
    project_id,
    ROUND(AVG(experience_years), 2) AS average_years
FROM
    project
JOIN
    employee ON project.employee_id = employee.employee_id
GROUP BY
    project_id
;
```
<br>


65. Average Selling Price (1251)

Write a solution to find the average selling price for each product. average_price should be rounded to 2 decimal places. If a product does not have any sold units, its average selling price is assumed to be 0. Return the result table in any order.

``` sql
SELECT
    p.product_id,
    COALESCE(ROUND(SUM(p.price * u.units) / SUM(u.units), 2), 0) AS average_price
FROM
    prices p
LEFT JOIN
    unitssold u ON u.product_id = p.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY
    p.product_id
;
```
<br>


64. Finding Updated Records

We have a table with employees and their salaries, however, some of the records are old and contain outdated salary information. Find the current salary of each employee assuming that salaries increase each year. Output their id, first name, last name, department ID, and current salary. Order your list by employee ID in ascending order.

``` sql
-- Retrieve the highest salary per employee along with their basic details, ordered by salary descending
SELECT
    id,
    first_name,
    last_name,
    department_id,
    MAX(salary) AS salary
FROM
    ms_employee_salary
GROUP BY
    id, first_name, last_name, department_id
ORDER BY
    salary DESC;
```
<br>


63. Product Price at a Given Date (1164)

Write a solution to find the prices of all products on 2019-08-16. Assume the price of all products before any change is 10. Return the result table in any order.

``` sql
/*
What are the names of the table(s)?
products
What are the columns in the products table?
product_id, new_price, change_date
What are the data types?
int, int, date
What is the format of the date in change_date?
YYYY-MM-DD

I'm going to return two columns from the products table; the product id and the new price with conditions.
The new price will be found using first value function partitioned by product id and ordered by change date descending as price.
The columns will have the condition that the change date is less than or equal to 2019-08-16.
This will get the newest price. However, if an item doesn't have a price, it will be missing.
Thus, we need to use another table.
This table will have a union with the first table.
To find all items without prices and assign them 10, we will have two columns; product id and 10 as price.
This will be from the products table.
We will group by product id.
The condition will be having a minimum change date of later than 2019-08-16.
This will get all the product ids that are missing from the first table and add them on with a price of 10.
*/

SELECT
    product_id,
    FIRST_VALUE(new_price) OVER (PARTITION BY product_id ORDER BY change_date DESC) AS price
FROM
    products
WHERE
    change_date <= '2019-08-16'
UNION
SELECT
    product_id,
    10 AS price
FROM
    products
GROUP BY
    product_id
HAVING
    MIN(change_date) > '2019-08-16'
;
```
<br>


62. Number of Unique Subjects Taught by Each Teacher (2356)

Write a solution to calculate the number of unique subjects each teacher teaches in the university. Return the result table in any order.

``` sql
/*
What is the name of the table with the teachers?
teacher
What columns are in the teacher table?
teacher_id, subject_id, department_id
What are the data types?
All three are integers.
Should we return all columns or only specific columns?
teacher_id and the number of unique subjects

I am going to return the teacher id and the distinct count of the subject id from the teacher table.
I will use an alias for the count of subject id as num_of_classes.
I will group by teacher id.
*/

SELECT
    teacher_id,
    COUNT(DISTINCT subject_id) AS cnt
FROM
    teacher
GROUP BY
    teacher_id
;
```
<br>


61. Not Boring Movies (620)

Write a solution to report the movies with an odd-numbered ID and a description that is not "boring". Return the result table ordered by rating in descending order.

``` sql
/*
What is the name of the table that stores movies?
cinema
what is the grain of the cinema table?
one row per movie
What columns are available in the movie table?
id, movie, description, rating
What are the data types of the columns?
id int, movie varchar (string or text), description varchar, rating float (decimal)
Can any columns contain NULL values?
yes, the id column
Are there any duplicate movie IDs? Is movie IDs a primary key?
no
Are there ties in ratings? How should we break the ties?
no
Should we return all columns or only specific columns?
all columns

I am going to load the data from the cinema table.
Filter rows based on two conditions: The id is odd (MOD(id,2) <> 0). The description is not "boring".
Sort the filtered results by rating in descending order.
Return the final result set.
*/

SELECT
    id,
    movie,
    description,
    rating
FROM
    cinema
WHERE
    MOD(id, 2) <> 0
    AND description <> 'boring'
ORDER BY
    rating DESC
;
```
<br>


60. Managers with at least 5 direct reports (570)

Write a solution to find managers with at least five direct reports. Return the result table in any order.

``` sql
/*
name of the manager

self inner join where id is the same as manager id

group by manager id

having at least 5 direct reports for one manager, use count
*/

SELECT
    a.name
FROM
    employee a
JOIN
    employee b ON a.id = b.managerId
GROUP BY
    b.managerId
HAVING
    COUNT(*) >= 5
;
```
<br>


59. Students and Examinations (1280)

Write a solution to find the number of times each student attended each exam. Return the result table ordered by student_id and subject_name.

``` sql
/*
student id
student name
subject name
count of number of exams for each student in each subject

list all exam subjects for every student, even if it's 0
to do this, crossjoin students and subjects

then left join the crossjoined table above to the examinations table

order by student id asc, student name asc, subject name asc
group by student id, student name, subject name
*/

SELECT
    st.student_id
    ,st.student_name
    ,su.subject_name
    ,COUNT(e.subject_name) AS attended_exams
FROM
    students st
JOIN
    subjects su
LEFT JOIN
    examinations e
        ON st.student_id = e.student_ID
        AND su.subject_name = e.subject_name
GROUP BY
    St.student_id, St.student_name, Su.subject_name
ORDER BY
    St.student_id, St.student_name, Su.subject_name
;
```
<br>


58. Employee Bonus (577)

Write a solution to report the name and bonus amount of each employee with a bonus less than 1000. Return the result table in any order.

``` sql
/*
employee name
bonus

bonus < 1000 or bonus doesn't exist

any order
*/

SELECT
    name,
    bonus
FROM
    employee e
LEFT JOIN
    bonus b ON e.empID = b.empID
WHERE
    bonus < 1000 OR bonus IS NULL
;
```
<br>


57. Average Time of Process per Machine (1661)

There is a factory website that has several machines each running the same number of processes. Write a solution to find the average time each machine takes to complete a process. The resulting table should have the machine_id along with the average time as processing_time, which should be rounded to 3 decimal places. Return the result table in any order.

``` sql
/*
machine id
average difference in start and end time rounded to three places

any order
*/

SELECT
    a.machine_id,
    ROUND(AVG(b.timestamp - a.timestamp), 3) AS processing_time
FROM
    Activity a, 
    Activity b
WHERE 
    a.machine_id = b.machine_id
    AND a.process_id = b.process_id
    AND a.activity_type = 'start'
    AND b.activity_type = 'end'
GROUP BY
    machine_id
;
```
<br>


56. Rising Temperature (197)

Write a solution to find all dates' id with higher temperatures compared to its previous dates (yesterday). Return the result table in any order.

``` sql
/*
table 2 id
where the temperature increased from one day to the next
self join + 1
where table two temperature > table one temperature
any order
*/

SELECT
    w2.id
FROM
    weather w1
JOIN
    weather w2 ON w1.id + 1 = w2.id
WHERE
    w2.temperature > w1.temperature
;
```
<br>


55. Customer Who Visited but Did Not Make Any Transactions (1581)

Write a solution to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits. Return the result table sorted in any order.

``` sql
/*
customer id, count of customer id
for each visit id that doesn't exist in transactions table (subquery)
group by customer id
any order
*/

SELECT
    customer_id,
    COUNT(customer_id) AS count_no_trans
FROM
    visits v
WHERE
    v.visit_id NOT IN (SELECT visit_id FROM transactions)
GROUP BY
    customer_id
;
```
<br>


54. Product Sales Analysis I (1068)

Write a solution to report the product_name, year, and price for each sale_id in the Sales table. Return the resulting table in any order.

``` sql
/*
product name, year, price
for each sale id
left join product table to sales table on product id
any order
*/

SELECT
    product_name,
    year,
    price
FROM
    sales
LEFT JOIN
    product ON sales.product_id = product.product_id
;
```
<br>


53. Replace Employee ID With The Unique Identifier (1378)

Write a solution to show the unique ID of each user, If a user does not have a unique ID replace just show null. Return the result table in any order.

``` sql
/*
unique id, name
all names, whether they have a unique id or not
any order
*/

SELECT
    unique_id,
    name
FROM
    employees
LEFT JOIN
    employeeuni ON employees.id = employeeuni.id
;
```
<br>


52. Invalid Tweets (1683)

Write a solution to find the IDs of the invalid tweets. The tweet is invalid if the number of characters used in the content of the tweet is strictly greater than 15. Return the result table in any order.

``` sql
/*
tweet id
more than 15 characters in a tweet
any order
*/

SELECT
    tweet_id
FROM
    tweets
WHERE
    CHAR_LENGTH(content) > 15
;
```
<br>


51. Article Views I (1148)

Write a solution to find all the authors that viewed at least one of their own articles. Return the result table sorted by id in ascending order.

``` sql
/*
author id, no repeats
where author id is the same as viewer id
order by id ascending
*/

SELECT DISTINCT
    author_id AS id
FROM
    views
WHERE
    author_id = viewer_id
ORDER BY
    id ASC
```
<br>


50. Big Countries (595)

A country is big if:

it has an area of at least three million (i.e., 3000000 km2), or it has a population of at least twenty-five million (i.e., 25000000).
Write a solution to find the name, population, and area of the big countries. Return the result table in any order.

``` sql
/*
country name, population, area
area at least 3000000 or population at least 25000000
any order
*/

SELECT
    name,
    population,
    area
FROM
    world
WHERE
    area >= 3000000 OR
    population >= 25000000
;
```
<br>


49. Find Customer Referee (584)

Find the names of the customer that are not referred by the customer with id = 2. Return the result table in any order.

``` sql
/*
customer names
exclude those referred by id = 2
any order
*/

SELECT
    name
FROM
    customer
WHERE
    referee_id != 2 OR referee_id IS NULL
;
```
<br>


48. Recyclable and Low Fat Products (1757)

Write a solution to find the ids of products that are both low fat and recyclable. Return the result table in any order.

``` sql
/*
select products
low fat and recyclable
any order
*/

SELECT
    product_id
FROM
    products
WHERE
    low_fats = 'Y' AND recyclable = 'Y'
;
```
<br>


47. Amazon

Write a query that will calculate the number of shipments per month. The unique key for one shipment is a combination of shipment_id and sub_id. Output the year_month in format YYYY-MM and the number of shipments in that month.

``` sql
-- Count shipments grouped by year and month
SELECT
    COUNT(shipment_id) AS shipment_count,
    DATE_FORMAT(shipment_date, '%Y-%m') AS year_month
FROM
    amazon_shipment
GROUP BY
    year_month;
```
<br>


46. Find Students At Median Writing

Identify the IDs of students who scored exactly at the median for the SAT writing section.

``` sql
-- CTE #1: Assigns a sequential row number (rn) to each row ordered by sat_writing,
-- and computes the total row count (cnt) for later median calculation.
WITH ranked AS (
    SELECT
        student_id,
        sat_writing,
        ROW_NUMBER() OVER (ORDER BY sat_writing) AS rn,
        COUNT(*)     OVER ()                     AS cnt
    FROM sat_scores
),

-- CTE #2: Determines the median value from the ranked set.
-- For odd counts, selects the single middle row.
-- For even counts, selects the two middle rows and averages them.
median AS (
    SELECT
        AVG(sat_writing) AS median_value
    FROM ranked
    WHERE rn BETWEEN (cnt + 1) DIV 2  -- Lower middle row index
                AND (cnt + 2) DIV 2   -- Upper middle row index
)

-- Final query: Retrieves all student_id values where sat_writing equals the median.
SELECT
    s.student_id
FROM
    sat_scores AS s
CROSS JOIN
    median AS m
WHERE
    s.sat_writing = m.median_value;  -- Use a small tolerance if sat_writing is decimal/float
```
<br>


45. Top Ranked Songs

Find songs that have ranked in the top position. Output the track name and the number of times it ranked at the top. Sort your records by the number of times the song was in the top position in descending order.

``` sql
-- Get the count of times each track reached position 1 in the Spotify worldwide daily ranking
SELECT
    trackname,
    COUNT(*) AS times_at_number_one
FROM
    spotify_worldwide_daily_song_ranking
WHERE
    position = 1
GROUP BY
    trackname
ORDER BY
    times_at_number_one DESC;
```
<br>


44. McKinsey

You’re a consultant for a major pizza chain that will be running a promotion where all 3-topping pizzas will be sold for a fixed price, and are trying to understand the costs involved. Given a list of pizza toppings, consider all the possible 3-topping pizzas, and print out the total cost of those 3 toppings. Sort the results with the highest total cost on the top followed by pizza toppings in ascending order. Break ties by listing the ingredients in alphabetical order, starting from the first ingredient, followed by the second and third.

``` sql
SELECT
  CONCAT( a.topping_name, ',', b.topping_name, ',', c.topping_name) AS pizza,
  (a.ingredient_cost + b.ingredient_cost + c.ingredient_cost) AS total_cost
FROM
  pizza_toppings a
CROSS JOIN
  pizza_toppings b
CROSS JOIN
  pizza_toppings c
WHERE
  a.topping_name < b.topping_name AND
  b.topping_name < c.topping_name
ORDER BY
  total_cost DESC,
  pizza ASC
;
```
<br>


43. Facebook

You're provided with two tables: the advertiser table contains information about advertisers and their respective payment status, and the daily_pay table contains the current payment information for advertisers, and it only includes advertisers who have made payments.

Write a query to update the payment status of Facebook advertisers based on the information in the daily_pay table. The output should include the user ID and their current payment status, sorted by the user id.

The payment status of advertisers can be classified into the following categories:

New: Advertisers who are newly registered and have made their first payment.
Existing: Advertisers who have made payments in the past and have recently made a current payment.
Churn: Advertisers who have made payments in the past but have not made any recent payment.
Resurrect: Advertisers who have not made a recent payment but may have made a previous payment and have made a payment again recently.

``` sql
SELECT
  user_id,
  CASE
    WHEN paid IS NULL THEN 'CHURN'
    WHEN paid NOTNULL AND status IN ('NEW', 'EXISTING', 'RESURRECT') THEN 'EXISTING'
    WHEN paid NOTNULL AND status = 'CHURN' THEN 'RESURRECT'
    WHEN paid NOTNULL AND status IS NULL THEN 'NEW'
  END AS new_status
FROM
  advertiser
FULL OUTER JOIN
  daily_pay USING(user_id)
ORDER BY
  user_id ASC
;
```

42. Google

Google's marketing team is making a Superbowl commercial and needs a simple statistic to put on their TV ad: the median number of searches a person made last year. However, at Google scale, querying the 2 trillion searches is too costly. Luckily, you have access to the summary table which tells you the number of searches made last year and how many Google users fall into that bucket. Write a query to report the median of searches made by a user. Round the median to one decimal point.

Method 1: Efficient for large datasets
``` sql
WITH cte as(
  SELECT
    *,
    SUM(num_users) OVER(ORDER BY searches) as cumsum_users,
    SUM(num_users) OVER() as tot_users
  FROM search_frequency
)

SELECT
  ROUND(AVG(searches), 1) as median
FROM
  cte
WHERE
  tot_users <= cumsum_users * 2 AND
  tot_users >= (cumsum_users - num_users) * 2
;
```

Method 2: Using GENERATE_SERIES and PERCENTILE_CONT
``` sql
WITH searches_expanded AS (
  SELECT
    searches
  FROM
    search_frequency
  GROUP BY
    searches,
    GENERATE_SERIES(1, num_users)
)

SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY searches) AS median
FROM searches_expanded
;
```


<br>


41. AirBNB

Find the average number of bathrooms and bedrooms for each city’s property types. Output the result along with the city name and the property type.

``` sql
-- Calculate average number of bathrooms and bedrooms by city and property type
SELECT
    city,
    property_type,
    AVG(bathrooms) AS avg_bathrooms,
    AVG(bedrooms) AS avg_bedrooms
FROM
    airbnb_search_details
GROUP BY
    city,
    property_type;
```
<br>


40. Apple

Count the number of user events performed by MacBookPro users. Output the result along with the event name. Sort the result based on the event count in the descending order.

``` sql
-- Count events by name for MacBook Pro devices, sorted by most frequent
SELECT
    event_name,
    COUNT(*) AS total
FROM
    playbook_events
WHERE
    device = 'macbook pro'
GROUP BY
    event_name
ORDER BY
    total DESC;
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


32. Shopify: Order Details

Find order details made by Jill and Eva.
Consider the Jill and Eva as first names of customers.
Output the order date, details and cost along with the first name.
Order records based on the customer id in ascending order.

``` sql
-- Get orders for customers named Jill or Eva
SELECT
    o.order_date,
    o.order_details,
    o.total_order_cost,
    c.first_name
FROM
    orders AS o
INNER JOIN
    customers AS c
    ON o.cust_id = c.id
WHERE
    c.first_name IN ('Jill', 'Eva')
ORDER BY
    o.cust_id;

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


30. Microsoft

Find the number of employees working in the Admin department that joined in April or later, in any year.

``` sql
-- Count all Admin workers who joined after March
SELECT
    COUNT(*)
FROM
    worker
WHERE
    MONTH(joining_date) > 3
    AND department = 'Admin';
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


26. Amazon

Find the job titles of the employees with the highest salary. If multiple employees have the same highest salary, include the job titles for all such employees.

``` sql
-- Get the worker titles for workers with the highest salary
SELECT
    t.worker_title
FROM
    worker w
JOIN
    title t
    ON w.worker_id = t.worker_ref_id
WHERE
    w.salary = (
        SELECT MAX(salary)
        FROM worker
    );
```
<br>


25. Meta/Facebook

Meta/Facebook has developed a new programing language called Hack.To measure the popularity of Hack they ran a survey with their employees. The survey included data on previous programing familiarity as well as the number of years of experience, age, gender and most importantly satisfaction with Hack. Due to an error location data was not collected, but your supervisor demands a report showing average popularity of Hack by office location. Luckily the user IDs of employees completing the surveys were stored.
Based on the above, find the average popularity of the Hack per office location.
Output the location along with the average popularity.

``` sql
-- Average hack survey popularity by employee location
SELECT  
    e.location,
    AVG(h.popularity) AS avg_popularity
FROM
    facebook_employees e
JOIN
    facebook_hack_survey h ON e.id = h.employee_id
GROUP BY
    e.location;
```
<br>


24. Titanic Survivors and Non-Survivors

Make a report showing the number of survivors and non-survivors by passenger class. Output the number of survivors and non-survivors by each class.

``` sql
/* 
   Cross-tab survival outcomes by passenger class.
   Each CASE maps pclass values into separate columns via conditional aggregation.
*/
SELECT
    survived,
    SUM(CASE WHEN pclass = 1 THEN 1 END) AS class1_count,
    SUM(CASE WHEN pclass = 2 THEN 1 END) AS class2_count,
    SUM(CASE WHEN pclass = 3 THEN 1 END) AS class3_count
FROM titanic
GROUP BY survived;
```
<br>


23. Apple

Find the details of each customer regardless of whether the customer made an order. Output the customer's first name, last name, and the city along with the order details. Sort records based on the customer's first name and the order details in ascending order.

``` sql
-- Get all customers and their orders (include customers with no orders)
SELECT
    c.first_name,
    c.last_name,
    c.city,
    o.order_details
FROM
    customers c
LEFT JOIN
    orders o
    ON o.cust_id = c.id
ORDER BY
    c.first_name,
    o.order_details;
```
<br>


22. FAANG

Imagine you're an HR analyst at a tech company tasked with analyzing employee salaries. Your manager is keen on understanding the pay distribution and asks you to determine the second highest salary among all employees. It's possible that multiple employees may share the same second highest salary. In case of duplicate, display the salary only once.

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

Assume you are given the table below on Uber transactions made by users. Write a query to obtain the third transaction of every user. Output the user id, spend and transaction date.

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

UnitedHealth Group (UHG) has a program called Advocate4Me, which allows policy holders (or, members) to call an advocate and receive support for their health care needs – whether that's claims and benefits support, drug coverage, pre- and post-authorisation, medical records, emergency assistance, or member portal services. Write a query to find how many UHG policy holders made three, or more calls, assuming each call is identified by the case_id column.

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


19. Bikes last used

Find the last time each bike was in use. Output both the bike number and the date-timestamp of the bike's last use (i.e., the date-time the bike was returned). Order the results by bikes that were most recently used.

``` sql
-- Get the latest end_time for each bike_number
SELECT
    bike_number,
    MAX(end_time) AS latest_end_time
FROM
    dc_bikeshare_q1_2012
GROUP BY
    bike_number
ORDER BY
    latest_end_time DESC;
```
<br>


18. CVS Health

CVS Health is analyzing its pharmacy sales data, and how well different products are selling in the market. Each drug is exclusively manufactured by a single manufacturer. Write a query to identify the manufacturers associated with the drugs that resulted in losses for CVS Health and calculate the total amount of losses incurred. Output the manufacturer's name, the number of drugs associated with losses, and the total losses in absolute value. Display the results sorted in descending order with the highest losses displayed at the top.

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

CVS Health is trying to better understand its pharmacy sales, and how well different products are selling. Each drug can only be produced by one manufacturer. Write a query to find the top 3 most profitable drugs sold, and how much profit they made. Assume that there are no ties in the profits. Display the result from the highest to the lowest total profit.

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

Your team at JPMorgan Chase is preparing to launch a new credit card, and to gain some insights, you're analyzing how many credit cards were issued each month. Write a query that outputs the name of each credit card and the difference in the number of issued cards between the month with the highest issuance cards and the lowest issuance. Arrange the results based on the largest disparity.

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


14. Lyft

Find all Lyft drivers who earn either equal to or less than 30k USD or equal to or more than 70k USD. Output all details related to retrieved records.

``` sql
-- Select Lyft drivers with salary <= 30,000 or >= 70,000
SELECT
    *
FROM
    lyft_drivers
WHERE
    yearly_salary <= 30000
    OR yearly_salary >= 70000;
```
<br>


13. TikTok

Assume you're given tables with information about TikTok user sign-ups and confirmations through email and text. New users on TikTok sign up using their email addresses, and upon sign-up, each user receives a text message confirmation to activate their account. Write a query to display the user IDs of those who did not confirm their sign-up on the first day, but confirmed on the second day.

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

Assume you have an events table on Facebook app analytics. Write a query to calculate the click-through rate (CTR) for the app in 2022 and round the results to 2 decimal places. Definition and note: Percentage of click-through rate (CTR) = 100.0 * Number of clicks / Number of impressions. To avoid integer division, multiply the CTR by 100.0, not 100.

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

Companies often perform salary analyses to ensure fair compensation practices. One useful analysis is to check if there are any employees earning more than their direct managers. As a HR Analyst, you're asked to identify all employees who earn more than their direct managers. The result should include the employee's ID and name.

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
<br>


9. Robinhood

Assume you're given the tables containing completed trade orders and user details in a Robinhood trading system. Write a query to retrieve the top three cities that have the highest number of completed trade orders listed in descending order. Output the city name and the corresponding number of completed trade orders.

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

Assume you're given a table containing job postings from various companies on the LinkedIn platform. Write a query to retrieve the count of companies that have posted duplicate job listings. Definition: Duplicate job listings are defined as two job listings within the same company that share identical titles and descriptions.

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

Write a query to identify the top 2 Power Users who sent the highest number of messages on Microsoft Teams in August 2022. Display the IDs of these 2 users along with the total number of messages they sent. Output the results in descending order based on the count of the messages. Assumption: No two users have sent the same number of messages in August 2022.

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
<br>


5. NY Times

Assume you're given the table on user viewership categorised by device type where the three types are laptop, tablet, and phone. Write a query that calculates the total viewership for laptops and mobile devices where mobile is defined as the sum of tablet and phone viewership. Output the total viewership for laptops as laptop_reviews and the total viewership for mobile devices as mobile_views.

``` sql
SELECT 
  SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views, 
  SUM(CASE WHEN device_type IN ('tablet', 'phone') THEN 1 ELSE 0 END) AS mobile_views 
FROM 
  viewership;
```
<br>


4.  Tesla

Tesla is investigating production bottlenecks and they need your help to extract the relevant data. Write a query to determine which parts have begun the assembly process but are not yet finished. <br>

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

Assume you're given two tables containing data about Facebook Pages and their respective likes (as in "Like a Facebook Page"). Write a query to return the IDs of the Facebook pages that have zero likes. The output should be sorted in ascending order based on the page IDs.

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


2. Average Salaries

Compare each employee's salary with the average salary of the corresponding department.
Output the department, first name, and salary of employees along with the average salary of that department.

``` sql
SELECT
    department,
    first_name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS avg_dept
FROM
    employee;
```
<br>


1. Histogram of Tweets

Assume you're given a table Twitter tweet data, write a query to obtain a histogram of tweets posted per user in 2022. Output the tweet count per user as the bucket and the number of Twitter users who fall into that bucket. In other words, group the users by the number of tweets they posted in 2022 and count the number of users in each group.

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
