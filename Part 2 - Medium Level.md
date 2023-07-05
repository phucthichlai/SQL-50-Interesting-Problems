# SQL 40 Interesting Problems (part 2)

## 1. Managers with at least 5 reports

  ### Level: Medium 🟡
  
Table: Employee

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| department  | varchar |
| managerId   | int     |
+-------------+---------+
id is the primary key column for this table.
Each row of this table indicates the name of an employee, their department, and the id of their manager.
If managerId is null, then the employee does not have a manager.
No employee will be the manager of themself.
 
Find the managers with at least five direct reports.

Return the result table in any order.
 
## Solution: 💼

- Intuition:

+ Self join the table with itself to show the name of managerId.
+ Use GROUP BY to show leaders' number of reports, and then HAVING to filter the aggregated data.
    
- Code:

```
select e2.name
from employee e1 join employee e2 on
e1.managerId = e2.id
group by e2.name
having count(*) >= 5
```

Input: 
Employee table:

| id  | name  | department | managerId |
|-----|-------|------------|-----------|
| 101 | John  | A          | None      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |

Output: 

| name |
|------|
| John |

## 2. Confirmation Rate of client request 🔐
  ### Level: Medium 🟡

Table: Signups

+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| time_stamp     | datetime |
+----------------+----------+
user_id is the primary key for this table.
Each row contains information about the signup time for the user with ID user_id.
 

Table: Confirmations

+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| time_stamp     | datetime |
| action         | ENUM     |
+----------------+----------+
(user_id, time_stamp) is the primary key for this table.
user_id is a foreign key with a reference to the Signups table.
action is an ENUM of the type ('confirmed', 'timeout')
Each row of this table indicates that the user with ID user_id requested a confirmation message at time_stamp and that confirmation message was either confirmed ('confirmed') or expired without confirming ('timeout').
 
The confirmation rate of a user is the number of 'confirmed' messages divided by the total number of requested confirmation messages. The confirmation rate of a user that did not request any confirmation messages is 0. Round the confirmation rate to two decimal places.

Write an SQL query to find the confirmation rate of each user.

Return the result table in any order.

## Solution: 🔑

- Intuition:
  
In Excel, we can just (COUNTIF action='confirmed'/ total (or COUNT all). Now we can use IF to do that.

- Code:

```
select s.user_id, round(avg(if (action='confirmed', 1, 0)),2) as confirmation_rate
from signups s full join confirmations c on
s.user_id = c.user_id
group by s.user_id
```

Input: 
Signups table:

| user_id | time_stamp          |
|---------|---------------------|
| 3       | 2020-03-21 10:16:13 |
| 7       | 2020-01-04 13:57:59 |
| 2       | 2020-07-29 23:09:44 |
| 6       | 2020-12-09 10:39:37 |

Confirmations table:

| user_id | time_stamp          | action    |
|---------|---------------------|-----------|
| 3       | 2021-01-06 03:30:46 | timeout   |
| 3       | 2021-07-14 14:00:00 | timeout   |
| 7       | 2021-06-12 11:57:29 | confirmed |
| 7       | 2021-06-13 12:58:28 | confirmed |
| 7       | 2021-06-14 13:59:27 | confirmed |
| 2       | 2021-01-22 00:00:00 | confirmed |
| 2       | 2021-02-28 23:59:59 | timeout   |

Output: 

| user_id | confirmation_rate |
|---------|-------------------|
| 6       | 0.00              |
| 3       | 0.00              |
| 7       | 1.00              |
| 2       | 0.50              |

## 3. Immediate food order rate 🍔
  ### Level: Medium 🟡

Table: Delivery

+-----------------------------+---------+
| Column Name                 | Type    |
+-----------------------------+---------+
| delivery_id                 | int     |
| customer_id                 | int     |
| order_date                  | date    |
| customer_pref_delivery_date | date    |
+-----------------------------+---------+
delivery_id is the primary key of this table.
The table holds information about food delivery to customers that make orders at some date and specify a preferred delivery date (on the same order date or after it).
 
If the customer's preferred delivery date is the same as the order date, then the order is called immediate; otherwise, it is called scheduled.

The first order of a customer is the order with the earliest order date that the customer made. It is guaranteed that a customer has precisely one first order.

Write an SQL query to find the percentage of immediate orders in the first orders of all customers, rounded to 2 decimal places.

## Solution: 🍹

- Intuition:

+ First, add a new column ranking the delivery order of each customer (using RANK)
+ Next, put it in a CTE, the first orders will have 1st ranking, get them and calculate the rate.

- Code:
  
```
with t1 as (
select *, rank() over (partition by customer_id order by order_date asc) as delivery_order
from delivery
)
select round(avg(case when order_date < customer_pref_delivery_date then 0 else 1 end)*100,2) as immediate_percentage
from t1
where delivery_order = 1
```

Input: 
Delivery table:

| delivery_id | customer_id | order_date | customer_pref_delivery_date |
|-------------|-------------|------------|-----------------------------|
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 2           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-12                  |
| 4           | 3           | 2019-08-24 | 2019-08-24                  |
| 5           | 3           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
| 7           | 4           | 2019-08-09 | 2019-08-09                  |

Output: 

| immediate_percentage |
|----------------------|
| 50.00                |

## 3. Game play analysis 🎮
  ### Level: Medium 🟡

Table: Activity

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
(player_id, event_date) is the primary key of this table.
This table shows the activity of players of some games.
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on someday using some device.
 
Write an SQL query to report the fraction of players that logged in again on the day after the day they first logged in, rounded to 2 decimal places. In other words, you need to count the number of players that logged in for at least two consecutive days starting from their first login date, then divide that number by the total number of players.

## Solution: 🎲

- Intuition:
+ First, add a new column of next event date (to check if the event_date and next_event_date are 2 consecutive days) by LEAD(), call it CTE t1.
+ Next, from t1, filter only the first login day of each player_id by MIN and GROUP BY, call it CTE t2
+ Now we had the distinct player_id, first event date and next event date. We can just count the player_id meeting the 2-consecutive-day condition and divded by the total player_id, or simply AVG((IF THE CONDITION IS MET THEN 1 ELSE 0))

- Code:

```
with t1 as (
    select *, lead(event_date,1) over (partition by player_id order by event_date asc) as next_event_date
    from activity
),

t2 as (
   select player_id, min(event_date) as first_login, next_event_date
   from t1
   group by player_id
)

select round(avg(if (date_add(first_login, interval 1 day) = next_event_date, 1, 0)),2) as fraction
from t2
```

Input: 
Activity table:

| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |

Output: 

| fraction  |
|-----------|
| 0.33      |

