# SQL 50 Interesting Problems (part 1)

## 1. Rising temperature day 🥵
   ### Level: Easy 🟢
   
Table: Weather

| Column Name   | Type    |
|---------------|---------|
| id            | int     |
| recordDate    | date    |
| temperature   | int     |

id is the primary key for this table.
This table contains information about the temperature on a certain day.
 
Write an SQL query to find all dates' Id with higher temperatures compared to its previous dates (yesterday).

Return the result table in any order.

The query result format is in the following example.

## Solution ⛈️

- Intuition

We need to put have a new column containing the previous day of the recordDate.

- Approach

Because the days are 1 day apart from the previous, we can join the table with itself on recordDate = recordDate - 1.

- Code

```
select w1.id
from weather w1 join weather w2 on
w1.recordDate = Dateadd(day, 1, w2.recordDate)
where w1.temperature > w2.temperature
```

Input: 

Weather table:

| id | recordDate | temperature |
-----|------------|-------------|
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |

Output: 
| id |
|----|
| 2  |
| 4  |


## 2. Average Time of Process per Machine 💻
   ### Level: Easy 🟢
   
Table: Activity

| Column Name    | Type    |
|----------------|---------|
| machine_id     | int     |
| process_id     | int     |
| activity_type  | enum    |
| timestamp      | float   |

The table shows the user activities for a factory website.
(machine_id, process_id, activity_type) is the primary key of this table.
machine_id is the ID of a machine.
process_id is the ID of a process running on the machine with ID machine_id.
activity_type is an ENUM of type ('start', 'end').
timestamp is a float representing the current time in seconds.
'start' means the machine starts the process at the given timestamp and 'end' means the machine ends the process at the given timestamp.
The 'start' timestamp will always be before the 'end' timestamp for every (machine_id, process_id) pair.

There is a factory website that has several machines each running the same number of processes. Write an SQL query to find the average time each machine takes to complete a process.

The time to complete a process is the 'end' timestamp minus the 'start' timestamp. The average time is calculated by the total time to complete every process on the machine divided by the number of processes that were run.

The resulting table should have the machine_id along with the average time as processing_time, which should be rounded to 3 decimal places.

Return the result table in any order.

## Solution 🤖

- Intuition

We need to put start time and end time in the same line, then take the average of (endtime - starttime)

- Approach
  
We can extract the table with only start time, then join the table with only end time. But how could we join them perfectly? Based on the attributes that make a process running unique, it is machine_id + process_id.

- Code
```
select a1.machine_id, round(avg(a2.timestamp)-avg(a1.timestamp),3) as processing_time
from activity a1 join activity a2 on
a1.machine_id = a2.machine_id and a1.process_id = a2.process_id
where a1.activity_type = 'start' and a2.activity_type = 'end'
group by a1.machine_id
```

The query result format is in the following example.

Input: 
Activity table:

| machine_id | process_id | activity_type | timestamp |
|------------|------------|---------------|-----------|
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |

Output: 

| machine_id | processing_time |
|------------|-----------------|
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |


## 3. Students and Exams 📜
   ### Level: Easy 🟢
   
Table: Students 

| Column Name   | Type    |
|---------------|---------|
| student_id    | int     |
| student_name  | varchar |

student_id is the primary key for this table.
Each row of this table contains the ID and the name of one student in the school.
 
Table: Subjects 

| Column Name  | Type    |
|--------------|---------|
| subject_name | varchar |

subject_name is the primary key for this table.
Each row of this table contains the name of one subject in the school.
 
Table: Examinations 

| Column Name  | Type    |
|--------------|---------|
| student_id   | int     |
| subject_name | varchar |

There is no primary key for this table. It may contain duplicates.
Each student from the Students table takes every course from the Subjects table.
Each row of this table indicates that a student with ID student_id attended the exam of subject_name.

Find the number of times each student attended each exam.

Return the result table ordered by student_id and subject_name.

## Solution: 📑

- Intuition:
  
First, to count the attended exams, we need to show all entries. Next, because all student_id and subject_name are shown for every student_id, we need to use table with all student_id (students) cross join Subjects table. Finally, full Join examinations table.

- Code:
```
select s.student_id, s.student_name, sub.subject_name, count(e.subject_name) as attended_exams
from students s cross join subjects sub
full join examinations e on
s.student_id = e.student_id and sub.subject_name = e.subject_name
group by s.student_id, s.student_name, sub.subject_name
order by 1, 2, 3
```

The result format is in the following example.

Input: 
Students table:

| student_id | student_name |
|------------|--------------|
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |

Subjects table:

| subject_name |
|--------------|
| Math         |
| Physics      |
| Programming  |

Examinations table:

| student_id | subject_name |
|------------|--------------|
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |

Output: 

| student_id | student_name | subject_name | attended_exams |
|------------|--------------|--------------|----------------|
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |


## 4. Percentage of Users Attended a Contest 🥇
   ### Level: Easy 🟢
   
Table: Users

| Column Name | Type    |
|-------------|---------|
| user_id     | int     |
| user_name   | varchar |
|-------------|---------|

user_id is the primary key for this table.
Each row of this table contains the name and the id of a user.
 

Table: Register

| Column Name | Type    |
|-------------|---------|
| contest_id  | int     |
| user_id     | int     |
|-------------|---------|

(contest_id, user_id) is the primary key for this table.
Each row of this table contains the id of a user and the contest they registered into.
 

Write an SQL query to find the percentage of the users registered in each contest rounded to two decimals.

Return the result table ordered by percentage in descending order. In case of a tie, order it by contest_id in ascending order.

The query result format is in the following example.

## Solution: 🏆

- Intuition:

The challenge here is we can't calculate the user joining each contest and the total user at the same time. So it requires us to understand the order of SQL function. We need some subqueries to calculate 2 figures seperately following the function order.

- Code:

```
select contest_id, round(count(user_id)*100/(select count(*) from users),2) as percentage
from register
group by contest_id
order by 2 desc, 1 asc
```

Example 1:

Input: 
Users table:

| user_id | user_name |
|---------|-----------|
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |


Register table:

| contest_id | user_id |
|------------|---------|
| 215        | 6       |
| 209        | 2       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 7       |
| 209        | 6       |
| 215        | 7       |
| 208        | 7       |
| 210        | 2       |
| 207        | 2       |
| 210        | 7       |


Output:


| contest_id | percentage |
|------------|------------|
| 208        | 100.0      |
| 209        | 100.0      |
| 210        | 100.0      |
| 215        | 66.67      |
| 207        | 33.33      |

## 5. Queries Quality and Percentage 🐶
   ### Level: Easy 🟢
   
Table: Queries

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| query_name  | varchar |
| result      | varchar |
| position    | int     |
| rating      | int     |
+-------------+---------+
There is no primary key for this table, it may have duplicate rows.
This table contains information collected from some queries on a database.
The position column has a value from 1 to 500.
The rating column has a value from 1 to 5. Query with rating less than 3 is a poor query.
 

We define query quality as:
The average of the ratio between query rating and its position.
We also define poor query percentage as:
The percentage of all queries with rating less than 3.

Write an SQL query to find each query_name, the quality and poor_query_percentage.
Both quality and poor_query_percentage should be rounded to 2 decimal places.
Return the result table in any order.

## Solution: 😻

- Code:

```
select query_name, round(avg(rating/position),2) as quality, round(avg(case when rating < 3 then 1 else 0 end)*100,2) as poor_query_percentage
from queries
group by query_name
```

Input: 
Queries table:

| query_name | result            | position | rating |
|------------|-------------------|----------|--------|
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |

Output: 

| query_name | quality | poor_query_percentage |
|------------|---------|-----------------------|
| Dog        | 2.50    | 33.33                 |
| Cat        | 0.66    | 33.33                 |

