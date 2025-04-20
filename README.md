# SQL Practice

<!-- Hidden text for templates

/*

98- MySQL
https://www.hackerrank.com/domains/sql

48-97 MySQL
https://leetcode.com/studyplan/top-sql-50/

1-47 PostgreSQL
https://datalemur.com/questions?category=SQL

*/

/*

*/

``` sql

```
<br>

-->




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


97. Department top three salaries

A company's executives are interested in seeing who earns the most money in each of the company's departments. A high earner in a department is an employee who has a salary in the top three unique salaries for that department. Write a solution to find the employees who are high earners in each of the departments. Return the result table in any order.

``` sql
WITH one_table AS (
    SELECT
    dense_rank() OVER (PARTITION BY departmentID ORDER BY salary DESC) AS ranked_salary,
    d.name AS Department,
    e.name AS Employee,
    salary AS Salary
FROM
    employee e
JOIN
    department d ON e.departmentId = d.id
)

SELECT
    Department,
    Employee,
    Salary
FROM
    one_table
WHERE
    ranked_salary < 4
;
```
<br>


96. Restaurant Growth

You are the restaurant owner and you want to analyze a possible expansion (there will be at least one customer every day). Compute the moving average of how much the customer paid in a seven days window (i.e., current day + 6 days before). average_amount should be rounded to two decimal places. Return the result table ordered by visited_on in ascending order.

``` sql
SELECT
    visited_on,
    amount,
    average_amount 
FROM (
    SELECT DISTINCT
        visited_on,
        SUM(amount) OVER (
            ORDER BY visited_on
            RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
            ) AS amount,
        ROUND(SUM(amount) OVER (
            ORDER BY visited_on
            RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW)
            / 7, 2) AS average_amount
    FROM Customer
    ) AS whole_totals
WHERE
    DATEDIFF(visited_on, (
        SELECT MIN(visited_on)
        FROM Customer)
        ) >= 6
;
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


93. Find users with valid e-emails

Write a solution to find the users who have valid emails. A valid e-mail has a prefix name and a domain where: <br>

The prefix name is a string that may contain letters (upper or lower case), digits, underscore '_', period '.', and/or dash '-'. <br>
The prefix name must start with a letter. <br>
The domain is '@leetcode.com'. <br>

Return the result table in any order.

``` sql
SELECT
    user_id,
    name,
    mail
FROM
    users

WHERE

    # Checks first character to be a letter
    mail REGEXP '^[a-zA-Z]' 

    # Checks if there is only one @ symbol
    AND LENGTH(mail) - LENGTH(REPLACE(mail, '@', '')) = 1

    # Checks if characters before the @ symbol are a-z, A-Z, 0-9, _, ., -
    AND SUBSTRING_INDEX(mail, '@', 1) REGEXP '^[a-zA-Z0-9_.-]+$'

    # Checks the domain at the end
    AND SUBSTRING_INDEX(mail, '@', -1) = 'leetcode.com'
    
;
```
<br>


Combined into one WHERE statement
``` sql
WHERE
    mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode[.]com$'
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


88. Friend requests II: who has the most friends

Write a solution to find the people who have the most friends and the most friends number. The test cases are generated so that only one person has the most friends.

``` sql
SELECT
    id,
    COUNT(id) AS num
FROM (
    
SELECT requester_id AS id
FROM RequestAccepted
UNION ALL
SELECT accepter_id
FROM RequestAccepted

) AS all_friends

GROUP BY
    1
ORDER BY
    2 DESC
LIMIT
    1
;
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



86. Consecutive numbers (180)

Find all numbers that appear at least three times consecutively. Return the result table in any order.

``` sql
WITH numbered_logs AS (
    SELECT
        num,
        LEAD(num, 1) OVER (ORDER BY id) AS num_lead1,
        LEAD(num, 2) OVER (ORDER BY id) AS num_lead2
    FROM
        logs
),

consecutive_nums AS (
    SELECT 
        num AS ConsecutiveNums
    FROM
        numbered_logs
    WHERE 
        num = num_lead1
        AND num = num_lead2
)

SELECT DISTINCT
    ConsecutiveNums
FROM
    consecutive_nums
;
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


83. Game Play Analysis IV (550)

Write a solution to report the fraction of players that logged in again on the day after the day they first logged in, rounded to 2 decimal places. In other words, you need to count the number of players that logged in for at least two consecutive days starting from their first login date, then divide that number by the total number of players.

``` sql
SELECT
    ROUND(
        COUNT(DISTINCT player_id) / 
        (SELECT COUNT(DISTINCT player_id) FROM Activity), 2)
    AS fraction
FROM
    activity
WHERE
    (player_id, DATE_SUB(event_date, INTERVAL 1 DAY)) IN (
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


75. Biggest single number (619)

A single number is a number that appeared only once in the MyNumbers table. Find the largest single number. If there is no single number, report null.


*Subquery that returns a single value as an expression in the SELECT clause*

``` sql
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

*From clause subquery*

``` sql
SELECT
    MAX(num) AS num
FROM (
    SELECT
        num
    FROM
        mynumbers
    GROUP BY
        num
    HAVING
        COUNT(num) = 1
) AS unique_numbers
;
```
<br>



74. Exchange seats (626)

Write a solution to swap the seat id of every two consecutive students. If the number of students is odd, the id of the last student is not swapped. Return the result table ordered by id in ascending order.

``` sql
SELECT
    id,
    CASE
        WHEN 
            id % 2 = 1 
            AND LEAD(student) OVER (ORDER BY id) IS NOT NULL 
            THEN LEAD(student) OVER (ORDER BY id)
        WHEN 
            id % 2 = 0 
            THEN LAG(student) OVER (ORDER BY id)
        ELSE student
    END AS student
FROM
    seat
ORDER BY
    id ASC
;
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


72. Product Sales Analysis III

Write a solution to select the product id, year, quantity, and price for the first year of every product sold. Return the resulting table in any order.

``` sql
WITH ordered_by_years AS (
    SELECT
        rank() OVER (PARTITION BY product_id ORDER BY year ASC) AS rank_by_year,
        product_id,
        year,
        quantity,
        price
    FROM
        sales
)

SELECT
    product_id,
    year AS first_year,
    quantity,
    price
FROM
    ordered_by_years
WHERE
    rank_by_year = 1
;
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


64. Confirmation Rate (1934)

The confirmation rate of a user is the number of 'confirmed' messages divided by the total number of requested confirmation messages. The confirmation rate of a user that did not request any confirmation messages is 0. Round the confirmation rate to two decimal places. Write a solution to find the confirmation rate of each user. Return the result table in any order.

``` sql
/*
What are the names of the table or tables used?
Signups
Confirmations

What are the columns and data types in the tables?
signups
user_id, time_stamp
int, datetime

confirmations
user_id, time_stamp, action
int, datetime, ENUM

What values exist in the action column?
confirmed, timeout

Do you have the Entity relationship diagram available?
no

First I'll focus on finding the confirmation rate using the confirmations table.
In a CTE named user confirmations, I will return two columns.
First column is user id. Second column is the confirmation rate of a user which is the sum of columns with confirmed divided by count star, alias confirmation rate.
I will group by user id.

Then in a query below, I will use the signups table.
I will left join it with the user confirmations table on user id.
I will return two columns, user id from the signups table (alias s) and confirmation rate from user confirmations table.
I will use coalesce on the confirmation rate to replace null values with 0.
*/

WITH user_confirmations AS (
    SELECT
        user_id,
        ROUND(SUM(action = 'confirmed') / COUNT(*), 2) AS rate
    FROM
        confirmations
    GROUP BY
        user_id
)

SELECT
    s.user_id,
    COALESCE(rate, 0) AS confirmation_rate
FROM
    signups s
LEFT JOIN
    user_confirmations ON s.user_id = user_confirmations.user_id
;
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

Amazon Web Services (AWS) is powered by fleets of servers. Senior management has requested data-driven solutions to optimize server usage. Write a query that calculates the total time that the fleet of servers was running. The output should be in units of full days.

Each server might start and stop several times.
The total time in which the server fleet is running can be calculated as the sum of each server's uptime.


``` sql
WITH session_info AS (
  SELECT
    server_id,
    session_status,
    status_time AS start_time,
    LEAD(status_time) OVER(PARTITION BY server_id ORDER BY status_time) AS end_time
  FROM
    server_utilization
  ORDER BY
    server_id ASC,
    status_time ASC
)

SELECT
  FLOOR(EXTRACT(EPOCH FROM SUM(end_time - start_time)) / 86400) AS total_uptime_days
FROM
  session_info
WHERE
  session_status = 'start'
;
```
<br>

Alternative version (no CTE) that works by negating the start date
``` sql
SELECT
  SUM(CASE
    WHEN session_status = 'start'
    THEN -(EXTRACT(DAY from status_time))
    ELSE EXTRACT(DAY from status_time) END
  ) as total_uptime_days
FROM server_utilization
;
```
<br>




46. Stripe

Sometimes, payment transactions are repeated by accident; it could be due to user error, API failure or a retry error that causes a credit card to be charged twice. Using the transactions table, identify any payments made at the same merchant with the same credit card for the same amount within 10 minutes of each other. Count such repeated payments.

``` sql
WITH trasaction_comparison AS (
  SELECT 
    merchant_id,
    EXTRACT(EPOCH FROM transaction_timestamp -
      LAG(transaction_timestamp) OVER(
        PARTITION BY merchant_id, credit_card_id, amount 
        ORDER BY transaction_timestamp)
    )/60 AS minute_difference
  FROM 
    transactions
)

SELECT
  COUNT(minute_difference) AS payment_count
FROM
  trasaction_comparison
WHERE
  minute_difference <= 10
;
```
<br>


45. FAANG

You work as a data analyst for a FAANG company that tracks employee salaries over time. The company wants to understand how the average salary in each department compares to the company's overall average salary each month. Write a query to compare the average salary of employees in each department to the company's average salary for March 2024. Return the comparison result as 'higher', 'lower', or 'same' for each department. Display the department ID, payment month (in MM-YYYY format), and the comparison result.


``` sql
WITH avg_salary AS (
  SELECT
    e.department_id,
    TO_CHAR(s.payment_date, 'MM-YYYY') AS pay_month,
    AVG(s.amount) AS salary
  FROM
    salary s
  LEFT JOIN
    employee e ON s.employee_id = e.employee_id
  GROUP BY
    pay_month,
    department_id
)

SELECT
  department_id,
  pay_month,
  CASE
    WHEN salary < (SELECT avg(amount) FROM salary) THEN 'lower'
    WHEN salary = (SELECT avg(amount) FROM salary) THEN 'same'
    WHEN salary > (SELECT avg(amount) FROM salary) THEN 'higher'
  END AS comparison
FROM
  avg_salary
WHERE
  pay_month = '03-2024'
;
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


41. Amazon

Amazon wants to maximize the storage capacity of its 500,000 square-foot warehouse by prioritizing a specific batch of prime items. The specific prime product batch detailed in the inventory table must be maintained. So, if the prime product batch specified in the item_category column included 1 laptop and 1 side table, that would be the base batch. We could not add another laptop without also adding a side table; they come all together as a batch set. After prioritizing the maximum number of prime batches, any remaining square footage will be utilized to stock non-prime batches, which also come in batch sets and cannot be separated into individual items. Write a query to find the maximum number of prime and non-prime batches that can be stored in the 500,000 square feet warehouse based on the following criteria:
Prioritize stocking prime batches
After accommodating prime items, allocate any remaining space to non-prime batches
Output the item_type with prime_eligible first followed by not_prime, along with the maximum number of batches that can be stocked.

Assumptions:
Products must be stocked in batches, so we want to find the largest available quantity of prime batches, and then the largest available quantity of non-prime batches.
Non-prime items must always be available in stock to meet customer demand, so the non-prime item count should never be zero.
Item count should be whole numbers (integers).

``` sql
SELECT
  COUNT(*) FILTER(WHERE item_type = 'prime_eligible') *
    FLOOR(
      500000 /
        SUM(square_footage) FILTER(WHERE item_type = 'prime_eligible')) 
          AS prime_eligible,
      
  COUNT(*) FILTER(WHERE item_type = 'not_prime') *
    FLOOR(
      (500000 -
        SUM(square_footage) FILTER(WHERE item_type = 'prime_eligible') *
          FLOOR(
            500000 /
              SUM(square_footage) FILTER(WHERE item_type = 'prime_eligible'))) /
                SUM(square_footage) FILTER(WHERE item_type = 'not_prime')) 
                  AS not_prime
FROM
  inventory
;
```
<br>


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


19. CVS Health

CVS Health wants to gain a clearer understanding of its pharmacy sales and the performance of various products. Write a query to calculate the total drug sales for each manufacturer. Round the answer to the nearest million and report your results in descending order of total sales. In case of any duplicates, sort them alphabetically by the manufacturer name. Since this data will be displayed on a dashboard viewed by business stakeholders, please format your results as follows: "$36 million".

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


14. IBM

IBM is analyzing how their employees are utilizing the Db2 database by tracking the SQL queries executed by their employees. The objective is to generate data to populate a histogram that shows the number of unique queries run by employees during the third quarter of 2023 (July to September). Additionally, it should count the number of employees who did not run any queries during this period. Display the number of unique queries as histogram categories, along with the count of employees who executed that number of unique queries.

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


2. LinkedIn

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. You want to find candidates who are proficient in Python, Tableau, and PostgreSQL. Write a query to list the candidates who possess all of the required skills for the job. Sort the output by candidate ID in ascending order.

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
