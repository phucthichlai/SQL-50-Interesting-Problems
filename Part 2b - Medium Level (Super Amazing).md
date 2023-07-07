# SQL 50 Interesting Problems (part 2b - SUPER AMAZING 🔥)

## 6. Product Price at a Given Date  💵 📆 💵

  ### Level: Medium 🟡

Table: Products

| Column Name   | Type    |
|---------------|---------|
| product_id    | int     |
| new_price     | int     |
| change_date   | date    |

(product_id, change_date) is the primary key of this table.
Each row of this table indicates that the price of some product was changed to a new price at some date.
 
Write an SQL query to find the prices of all products on 2019-08-16. Assume the price of all products before any change is 10.

Return the result table in any order.

Input: 
Products table:

| product_id | new_price | change_date |
|------------|-----------|-------------|
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |

Output: 

| product_id | price |
|------------|-------|
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |


## Solution: 🥇

- Intuition

+ Break the list into 2 scenarios: products with min of change_date <= 2016-08-16; and products with min of change_date > 2016-08-16.
+ In case 1, we find the nearest change date by using MAX of change_date then GROUP BY.
+ In case 2, we find the MIN of change_date, BUT with a little trick because MIN of change_date will go wrong if the product belongs to case 1, not from 2019-08-16. Thus, I use HAVING to filter after aggregating the MIN.

- Code

```
select product_id, new_price as price from products
where (product_id, change_date) in (
  select product_id, max(change_date) from products
  where change_date <= '2019-08-16'
  group by product_id
)
union
select product_id, 10 from products
where (product_id, change_date) in (
  select product_id, min(change_date) from products
  group by product_id
  having min(change_date) > '2019-08-16'
)
```

## 7. The last person to fit in the bus <warning: super amazing 🔥> 

### Level: Medium 🟡

Table: Queue

| Column Name | Type    |
|-------------|---------|
| person_id   | int     |
| person_name | varchar |
| weight      | int     |
| turn        | int     |

person_id is the primary key column for this table.
This table has the information about all people waiting for a bus.
The person_id and turn columns will contain all numbers from 1 to n, where n is the number of rows in the table.
turn determines the order of which the people will board the bus, where turn=1 denotes the first person to board and turn=n denotes the last person to board.
weight is the weight of the person in kilograms.
 
There is a queue of people waiting to board a bus. However, the bus has a weight limit of 1000 kilograms, so there may be some people who cannot board.

Write an SQL query to find the person_name of the last person that can fit on the bus without exceeding the weight limit. The test cases are generated such that the first person does not exceed the weight limit.

Input: 
Queue table:

| person_id | person_name | weight | turn |
|-----------|-------------|--------|------|
| 5         | Alice       | 250    | 1    |
| 4         | Bob         | 175    | 5    |
| 3         | Alex        | 350    | 2    |
| 6         | John Cena   | 400    | 3    |
| 1         | Winston     | 500    | 6    |
| 2         | Marie       | 200    | 4    |

Output: 

| person_name |
|-------------|
| John Cena   |

## Solution: 🚀

- Intuition:

+ We are familiar with self-join on queue1.turn = queue2.turn, but what if we join on queue1.turn >= queue2.turn?
We will then create a combination of rows like:
(q1.turn = 1; q2.turn = 1),
(q1.turn = 2; q2.turn = 1),(q1.turn = 2; q2.turn = 2),
(q1.turn = 3; q2.turn = 1),(q1.turn = 3; q2.turn = 2),(q1.turn = 3; q2.turn = 3)
...

When we GROUP BY q1.turn, we will have the cumulative total weights of 1 turn, 2 turns, 3 turns,...

- Code:

```
select q1.person_name
from queue q1 join queue q2 on q1.turn >= q2.turn
group by q1.turn
having sum(q2.weight) <= 1000
order by sum(q2.weight) desc
limit 1
```

## 8. Salary Categories 💲 <warning: it's not easy as it looks 🙂> 

### Level: Medium 🟡

Table: Accounts

| Column Name | Type |
|-------------|------|
| account_id  | int  |
| income      | int  |

account_id is the primary key for this table.
Each row contains information about the monthly income for one bank account.
 
Calculate the number of bank accounts of each salary category. The salary categories are:

"Low Salary": All the salaries strictly less than $20000.
"Average Salary": All the salaries in the inclusive range [$20000, $50000].
"High Salary": All the salaries strictly greater than $50000.
The result table must contain all three categories. If there are no accounts in a category, then report 0.

Return the result table in any order.

Input: 
Accounts table:

| account_id | income |
|------------|--------|
| 3          | 108939 |
| 2          | 12747  |
| 8          | 87709  |
| 6          | 91796  |

Output: 

| category       | accounts_count |
|----------------|----------------|
| Low Salary     | 1              |
| Average Salary | 0              |
| High Salary    | 3              |

## Solution: 🪙

- Intuition

+ At first, I thought it was easy, just add a CATEGORY column next to the income, then COUNT and GROUP BY 3 cateogries.
+ However, the problem happens when the test case only has high and low income (no average income). So the output table only has 2 categories.
So we need to actively create the categories.
+ Filter the income < 20000 the COUNT () , we have Low Salary accounts. Just do the same way with the others and UNION them.

- Code
```
select 'Low Salary' as category, count(*) as accounts_count from accounts
where income < 20000
union
select 'Average Salary' as category, count(*) as accounts_count from accounts
where income >= 20000 and income <= 50000
union
select 'High Salary' as category, count(*) as accounts_count from accounts
where income > 50000
```

## 9. Exchange Seat 🔃 "A beautiful problem always comes with a surprisingly beautiful solution"

### Level: Medium 🟡

Table: Seat

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| student     | varchar |

id is the primary key column for this table.
Each row of this table indicates the name and the ID of a student.
id is a continuous increment.
 
Write an SQL query to swap the seat id of every two consecutive students. If the number of students is odd, the id of the last student is not swapped.

Return the result table ordered by id in ascending order.

Input: 
Seat table:

| id | student |
|----|---------|
| 1  | Abbot   |
| 2  | Doris   |
| 3  | Emerson |
| 4  | Green   |
| 5  | Jeames  |

Output: 

| id | student |
|----|---------|
| 1  | Doris   |
| 2  | Abbot   |
| 3  | Green   |
| 4  | Emerson |
| 5  | Jeames  |

## Solution: 🔄

- Code
select row_number() over() as id, student
from seat
order by if(id % 2 = 0, id-1, id+1)

## 10. Restaurant Growth 📉 <This 💩 is tough>

### Level: Medium 🟡

Table: Customer

| Column Name   | Type    |
|---------------|---------|
| customer_id   | int     |
| name          | varchar |
| visited_on    | date    |
| amount        | int     |

(customer_id, visited_on) is the primary key for this table.
This table contains data about customer transactions in a restaurant.
visited_on is the date on which the customer with ID (customer_id) has visited the restaurant.
amount is the total paid by a customer.
 
You are the restaurant owner and you want to analyze a possible expansion (there will be at least one customer every day).

Write an SQL query to compute the moving average of how much the customer paid in a seven days window (i.e., current day + 6 days before). average_amount should be rounded to two decimal places.

Return result table ordered by visited_on in ascending order.

Input: 
Customer table:

| customer_id | name         | visited_on   | amount      |
|-------------|--------------|--------------|-------------|
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 

Output: 

| visited_on   | amount       | average_amount |
|--------------|--------------|----------------|
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |

## Solution: 🤙

- Intuition
  
+ Basically, the requirement is to filter groups of 7 consecutive days and calculate SUM.
+ We can use self-join in which the 2 dates difference ranges from 0 - 6 so we will have needed pairs to GROUP BY
+ However, the groups of only 2,3,4,5, or 6 consecutive days also meet the condition.
+ Therefore, we have to add 1 more condition: COUNT of the GROUPED dates = 7.

- Code

```
select c1.visited_on, sum(c2.amount) as amount, round(avg(c2.amount),2) as average_amount from
(select visited_on, sum(amount) as amount from customer group by visited_on) c1
join
(select visited_on, sum(amount) as amount from customer group by visited_on) c2
on
datediff(c1.visited_on, c2.visited_on) between 0 and 6
group by c1.visited_on
having count(c2.visited_on) = 7
```
