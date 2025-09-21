# EASY
## [175](https://leetcode.com/problems/combine-two-tables/description/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
select p.firstname,p.lastname,a.city,a.state 
from Person p
left join Address a 
on p.personId=a.personId
```
## [181](https://leetcode.com/problems/employees-earning-more-than-their-managers/description/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
select e1.name as Employee
from Employee e1
left join Employee e2
on e1.ManagerId=e2.id
where e1.salary>e2.salary
```
## [182](https://leetcode.com/problems/duplicate-emails/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
 select email
 from Person 
 group by email
 having count(distinct(id))>1;
```
## [183](https://leetcode.com/problems/customers-who-never-order/description/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
select c.name as Customers
from Customers c
where c.id Not in
(select c.id
from Customers c
inner join Orders o
on c.id=o.customerId)
```

## [196](https://leetcode.com/problems/delete-duplicate-emails/description/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
delete P1
from Person p1
join person p2 
on p1.id> p2.id 
And p1.email=p2.email
```

## [197](https://leetcode.com/problems/rising-temperature/description/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
with cte as(
select * ,
Date_add(recordDate,interval-1 day) as yesterdays_date,
lag(recordDate)over(order by recordDate)as prev_recordDate,
lag(temperature)over(order by recordDate)as prev_temp
from Weather )

select id
from cte
where yesterdays_date=prev_recordDate
and temperature>prev_temp
```

## [511](https://leetcode.com/problems/game-play-analysis-i/?envType=problem-list-v2&envId=database)
- **ANS:**
```sql
select player_id ,MIN(event_date) first_login
from Activity 
group by player_id
```

## [577](https://leetcode.com/problems/employee-bonus/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select e.name,b.bonus
from Employee e
left join Bonus b
on e.empId=b.empId
where b.bonus<1000
or b.bonus is null
```

## [584](https://leetcode.com/problems/find-customer-referee/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select name 
from Customer  
where referee_id<>2 
or referee_id is null
```

## [586](https://leetcode.com/problems/customer-placing-the-largest-number-of-orders/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as (select customer_number,
count(order_number) as numOrd
from Orders 
group by customer_number)
select customer_number from cte
where numOrd=(select max(numord)from cte)
```

## [595](https://leetcode.com/problems/big-countries/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select name,population ,area
from World 
where area>= 3000000  
or population>=25000000
```

## [596](https://leetcode.com/problems/classes-with-at-least-5-students/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select class
from Courses
group by class
having count(Student)>=5
```

## [607](https://leetcode.com/problems/sales-person/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select sales_id
from Orders o
left join Company c
on c.com_id=o.com_id
where c.name Like 'RED')

select name
from SalesPerson 
where sales_id not in(select Distinct
sales_id from cte)
```

## [610](https://leetcode.com/problems/triangle-judgement/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select *,CASE WHEN x+y>z and y+z>x and x+z>y THEN  'Yes' else 'No' END as triangle
FROM Triangle;
```

## [619](https://leetcode.com/problems/biggest-single-number/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql

with cte as
(select num
from MyNumbers 
group by  num 
having count(num)=1)
select case when count(*)>0 
then max(num) else null end as num
from cte
```
## [620](https://leetcode.com/problems/not-boring-movies/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select *
from Cinema 
where id%2<>0 
and description<>'boring'
order by rating desc
```

## [627](https://leetcode.com/problems/swap-salary/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
update Salary
set sex = case 
when sex='m' 
then 'f'
when sex='f'
then 'm' end
```

## [1050](https://leetcode.com/problems/actors-and-directors-who-cooperated-at-least-three-times/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select actor_id,director_id
from ActorDirector
group by actor_id,director_id
having count(timestamp)>=3
```

## [1068](https://leetcode.com/problems/product-sales-analysis-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select p.product_name,
s.year,
s.price
from Sales s
left join Product p
on s.product_id=p.product_id
```

## [1075](https://leetcode.com/problems/project-employees-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select p.project_id ,
round(avg(e.experience_years),2)as average_years
from Project p
left join Employee e
on e.employee_id=p.employee_id
group by p.project_id
```

## [1084](https://leetcode.com/problems/sales-analysis-iii/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
SELECT p.product_id, p.product_name
FROM product p
WHERE p.product_id in(SELECT product_id FROM sales WHERE sale_date BETWEEN '2019-01-01' AND '2019-03-31') and
p.product_id NOT IN (SELECT product_id FROM sales WHERE sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31')
```

## [1141](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select activity_date as day,
count(distinct user_id) as active_users
from Activity 
where activity_date between date_add('2019-07-27',interval -29 day) and
'2019-07-27'
group by activity_date
```

## [1148](https://leetcode.com/problems/article-views-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select distinct author_id as id
from Views 
where author_id=viewer_id
order by author_id asc
```

## [1179](https://leetcode.com/problems/reformat-department-table/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
SELECT id,
    MAX(CASE WHEN month = 'Jan' THEN revenue END) AS Jan_Revenue,
    MAX(CASE WHEN month = 'Feb' THEN revenue END) AS Feb_Revenue,
    MAX(CASE WHEN month = 'Mar' THEN revenue END) AS Mar_Revenue,
    MAX(CASE WHEN month = 'Apr' THEN revenue END) AS Apr_Revenue,
    MAX(CASE WHEN month = 'May' THEN revenue END) AS May_Revenue,
    MAX(CASE WHEN month = 'Jun' THEN revenue END) AS Jun_Revenue,
    MAX(CASE WHEN month = 'Jul' THEN revenue END) AS Jul_Revenue,
    MAX(CASE WHEN month = 'Aug' THEN revenue END) AS Aug_Revenue,
    MAX(CASE WHEN month = 'Sep' THEN revenue END) AS Sep_Revenue,
    MAX(CASE WHEN month = 'Oct' THEN revenue END) AS Oct_Revenue,
    MAX(CASE WHEN month = 'Nov' THEN revenue END) AS Nov_Revenue,
    MAX(CASE WHEN month = 'Dec' THEN revenue END) AS Dec_Revenue
FROM Department
GROUP BY id;
```

## [1211](https://leetcode.com/problems/queries-quality-and-percentage/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select query_name,
rating/position as ratio,
case when rating<3 then  1
else 0 end as quality_binary
from Queries)

select query_name,
round(avg(ratio),2) as quality,
round((sum(quality_binary)/count(*))*100,2) as poor_query_percentage
from cte group by query_name
```

## [1251](https://leetcode.com/problems/average-selling-price/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as (
select price,units,p.product_id as product_id,case when units is null then 0 else units end as newunits 
from prices p left join UnitsSold u 
on p.product_id=u.product_id 
and (purchase_date between start_date and end_date)
)

select product_id,round(sum(newunits*price)/if(sum(newunits)=0,1,sum(newunits)),2) as average_price from
cte group by product_id;
```

## [1280](https://leetcode.com/problems/students-and-examinations/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as 
(select *
from Students 
cross join Subjects),

cte2 as(select student_id, 
subject_name,count(subject_name) as count
from Examinations 
group by student_id,subject_name)

select cte.student_id,cte.student_name,
cte.subject_name,
case when count is not null then count else 0 end as attended_exams
from cte
left  join cte2
on cte.student_id=cte2.student_id
and cte.subject_name=cte2.subject_name
order by cte.student_id,cte.subject_name
```

## [1327](https://leetcode.com/problems/list-the-products-ordered-in-a-period/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select product_id ,sum(unit) as unit
from Orders 
where year(order_date)=2020
and month(order_date)=2
group by product_id
)

select Product_name, cte.unit
from Products p
join cte
on p.product_id=cte.product_id
where unit>=100
```

## [1378](https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select u.unique_id,e.name
from Employees e
left join EmployeeUNI u
on e.id=u.id
```

## [1407](https://leetcode.com/problems/top-travellers/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select distinct u.name,
case when r.distance is not null then sum(r.distance)
over(partition by  r.user_id order by r.user_id)
else 0 end as travelled_distance
from Users u 
left join Rides r
on u.id=r.user_id
order by travelled_distance desc, u.name
```

## [1484](https://leetcode.com/problems/group-sold-products-by-the-date/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select sell_date,
count(distinct product) as num_sold,
group_concat(distinct product order by product )as products
from Activities 
group by sell_date
order by sell_date
```
## [1517](https://leetcode.com/problems/find-users-with-valid-e-mails/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
SELECT *
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode\\.com$' COLLATE utf8mb4_bin
```

## [1527](https://leetcode.com/problems/patients-with-a-condition/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
SELECT * 
FROM patients 
WHERE conditions REGEXP '(^DIAB1|(.*\\sDIAB1))'
```

## [1581](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/description/?envType=problem-list-v2&envId=database)
- **ANS**
``` sql
select v.customer_id,
count(v.visit_id) as count_no_trans
from Visits v
left join Transactions t
on v.visit_id=t.visit_id
where t.transaction_id is null
group by v.customer_id
```

## [1587](https://leetcode.com/problems/bank-account-summary-ii/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select u.name, sum(t.amount) as balance
from Users u
inner join Transactions t
on u.account=t.account
group by u.name
having sum(t.amount)>10000
```

## [1633](https://leetcode.com/problems/percentage-of-users-attended-a-contest/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select contest_id, 
round(count(distinct user_id)/
(select count(user_id)from Users)*100,2) as percentage
from Register
group by contest_id
order by percentage desc,contest_id 
```

## [1661](https://leetcode.com/problems/average-time-of-process-per-machine/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select a1.machine_id,
round(avg(a2.timestamp - a1.timestamp),3) as processing_time
from Activity a1 join Activity a2 on  a1.machine_id = a2.machine_id AND a1.process_id = a2.process_id AND a1.activity_type = 'start' AND a2.activity_type = 'end' group by a1.machine_id
```

## [1667](https://leetcode.com/problems/fix-names-in-a-table/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select user_id, 
concat(upper(substring(name,1,1)),lower(substring(name,2,length(name)-1))) as name
FROM Users
order by user_id
```

## [1683](https://leetcode.com/problems/invalid-tweets/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select tweet_id
from Tweets
where length(content)>15
```

## [1693](https://leetcode.com/problems/daily-leads-and-partners/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select date_id,
make_name,
count(distinct lead_id)as unique_leads,
count(distinct partner_id)as unique_partners
from DailySales
group by date_id,make_name
```

## [1729](https://leetcode.com/problems/find-followers-count/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select distinct user_id,
count(follower_id) as followers_count
from Followers 
group by user_id
order by user_id
```

## [1731](https://leetcode.com/problems/the-number-of-employees-which-report-to-each-employee/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select reports_to,
count(employee_id) as reports_count,
round(avg(age),0) as average_age
from Employees 
where reports_to is not null
group by reports_to)

select c.reports_to as employee_id,
e.name,
c.reports_count,
average_age
from cte c
left join Employees e
on c.reports_to = e.employee_id
order by employee_id
```

## [1741](https://leetcode.com/problems/find-total-time-spent-by-each-employee/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select event_day as day,emp_id,
sum(out_time-in_time) as total_time
from Employees 
group by event_day,emp_id
```

## [1757](https://leetcode.com/problems/recyclable-and-low-fat-products/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select product_id
from Products
where low_fats='Y' and recyclable='Y'
```

## [1789](https://leetcode.com/problems/primary-department-for-each-employee/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select employee_id, 
case when count(department_id)=1
then department_id
when count(department_id)>1 then 
sum((primary_flag='Y')*department_id) end as department_id
from Employee 
group by employee_id
```

## [1795](https://leetcode.com/problems/rearrange-products-table/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select product_id,
'store1' as store,
store1 as price
from Products
where store1 is not null
union
select product_id,
'store2' as store,
store2 as price
from Products
where store2 is not null
union
select product_id,
'store3' as store,
store3 as price
from Products
where store3 is not null
```

## [1873](https://leetcode.com/problems/calculate-special-bonus/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select employee_id,
case when employee_id%2<>0 
and name not like 'M%' then salary else 0 end as bonus
from Employees 
order by employee_id
```

## [1890](https://leetcode.com/problems/the-latest-login-in-2020/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select DISTINCT(user_id) ,
first_value(time_stamp) over(partition by user_id order by time_stamp desc) as last_stamp
from Logins
where year(time_stamp)='2020'
```

## [1965](https://leetcode.com/problems/employees-with-missing-information/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select e.employee_id
from Employees e
left join Salaries s
on e.employee_id=s.employee_id
where s.salary is null
union
select s.employee_id
from Salaries s
left join Employees e
on s.employee_id=e.employee_id
where e.name is null
order by employee_id
```
## [1978](https://leetcode.com/problems/employees-whose-manager-left-the-company/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select employee_id,manager_id
from Employees 
where salary<30000)
select employee_id
from cte 
where manager_id
not in(select employee_id from Employees)
order by employee_id
```

## [2356](https://leetcode.com/problems/number-of-unique-subjects-taught-by-each-teacher/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select teacher_id, 
count(distinct subject_id) as cnt
from Teacher
group by teacher_id
```

## [3436](https://leetcode.com/problems/find-valid-emails/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select *
from Users
where regexp_like(email,'^[a-zA-Z0-9_]*@[a-zA-Z]*[.]com$')
order by User_id
```

## [3465](https://leetcode.com/problems/find-products-with-valid-serial-numbers/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
SELECT * FROM products
WHERE REGEXP_LIKE(description, "\\bSN[0-9]{4}-[0-9]{4}\\b",'c')
ORDER BY product_id;
```

## [3570](https://leetcode.com/problems/find-books-with-no-available-copies/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
WITH CTE AS (
    SELECT 
        book_id, 
        COUNT(record_id) AS current_borrowers
    FROM borrowing_records
    WHERE return_date IS NULL
    GROUP BY book_id
)
SELECT 
    l.book_id, 
    l.title, 
    l.author, 
    l.genre, 
    l.publication_year, 
    c.current_borrowers
FROM CTE c
JOIN library_books l
    ON c.book_id = l.book_id
WHERE c.current_borrowers = l.total_copies
ORDER BY 
    c.current_borrowers DESC, 
    l.title;
```


# Med.

## [179](https://leetcode.com/problems/second-highest-salary/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
'''select max(salary) as SecondHighestSalary from Employee where salary < (select max(salary) from Employee)'''
SELECT
   MAX(a.Salary) as SecondHighestSalary
  FROM Employee a
  JOIN Employee b
    ON a.Salary < b.Salary
```

## [177](https://leetcode.com/problems/nth-highest-salary/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
    with cte as(
    select salary,Dense_rank()over(order by salary desc) as rnk
    from Employee)
    select distinct ifnull(salary,null)
    from cte
    where rnk=N
  );
END
```

## [178](https://leetcode.com/problems/rank-scores/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select score,
Dense_rank()over(order by Score desc) as 'rank'
from Scores
order by score desc
```

## [180](https://leetcode.com/problems/consecutive-numbers/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select *,
lead(num,1) over() as next_1,
lead(num,2) over() as next_2
from Logs)

select distinct num as ConsecutiveNums
from cte
where num=next_1
and num=next_2
```

## [184](https://leetcode.com/problems/department-highest-salary/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as 
(select e.name as Employee,
e.salary ,d.name as department,
max(e.salary) over(partition by e.departmentId) as max_salary
from Employee e
left join Department d
on e.departmentId=d.id
)

select department ,employee,salary
from cte
where salary=max_salary
```

## [550](https://leetcode.com/problems/game-play-analysis-iv/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select player_id,
min(event_date) as first_login
from Activity
group by player_id),

cte2 as(
select * , 
date_add(first_login,interval 1 day) as next_date
from cte )

select
round((select count(distinct player_id)
from Activity
where (player_id,event_date) in (select 
player_id,next_date from cte2))/(select count(distinct player_id) from Activity),2) as fraction
```
## [570](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select name from employee where id in
(select managerId from employee group by managerId
having (count(distinct id)>=5))
```

## [585](https://leetcode.com/problems/investments-in-2016/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as 
(select concat(lat,',',lon) as location
from Insurance
group by lat,lon
having count(pid)>1),
cte2 as
(select distinct i1.*
from Insurance i1
left join Insurance i2
on i1.tiv_2015=i2.tiv_2015
where i1.pid<>i2.pid
and concat(i1.lat,',',i1.lon) not in(select location from cte))

select round(sum(tiv_2016),2) as tiv_2016
from cte2
```

## [602](https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(select requester_id as id,accepter_id
from RequestAccepted
union
select accepter_id as id,requester_id
from RequestAccepted)

select id ,count(distinct accepter_id) as num
from cte 
group by id
order by  num desc
 limit 1
```

## [608](https://leetcode.com/problems/tree-node/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select id,
case when p_id is null then 'Root' 
when id not in (select p_id from tree where p_id is not null) then 'Leaf' 
else 'Inner' end as type
from Tree
order by id
```

## [626](https://leetcode.com/problems/exchange-seats/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select *, 
lead(id)over(order by id) as next,
lag(id)over(order by id) as prev
from Seat )

select case when ((id%2=1)and next is not null) then next when(id%2=0) then prev
else id end as id,student
from cte
order by id
```

## [1045](https://leetcode.com/problems/customers-who-bought-all-products/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select customer_id
from Customer
group by customer_id
having count(distinct product_key)= (select count(distinct product_key) from Product)
```

## [1070](https://leetcode.com/problems/product-sales-analysis-iii/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select * ,
rank()over(partition by product_id order by year) as rnk
from Sales)

select product_id,year as first_year ,quantity,price
from cte 
where rnk=1
```

## [1158](https://leetcode.com/problems/market-analysis-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select u.user_id,
sum(case when year(o.order_date)=2019 then 1 else 0 end) as orders_in_2019
from Users u 
left join Orders o
on u.user_id= o.buyer_id
group by u.user_id)

select c.user_id as buyer_id , u.join_date,
c.orders_in_2019
from cte c 
left join Users u
on c.user_id=u.user_id
```

## [1164](https://leetcode.com/problems/product-price-at-a-given-date/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select * ,rank() over(partition by product_id order by  change_date desc) as r
from Products 
where change_date <='2019-08-16')

select product_id,new_price as price
from cte
where r=1
union
select product_id, 10 as price
from products
where product_id not in (select product_id from cte)
```

## [1174](https://leetcode.com/problems/immediate-food-delivery-ii/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select *,rank()over(partition by customer_id order by order_date) as order_no,case when order_date=customer_pref_delivery_date then 'immediate' else 'scheduled' end as orders
from delivery)

select round(sum(case when orders='immediate' then 1 else 0 end)/count(*)*100,2)as immediate_percentage
from cte 
where order_no=1
```

## [1193](https://leetcode.com/problems/monthly-transactions-i/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select date_format(trans_date,'%Y-%m')as month,country,
count(id)as trans_count,
sum(case when state='approved' then 1 else 0 end) as approved_count,
sum(amount) as trans_total_amount,
sum(case when state='approved' then amount else 0 end)as approved_total_amount
from transactions
group by date_format(trans_date,'%Y-%m'),country
```

## [1204](https://leetcode.com/problems/last-person-to-fit-in-the-bus/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select * ,sum(weight)over(order by turn) as total_weight
from Queue)

select person_name
from cte 
where total_weight<=1000
order by total_weight desc
limit 1
```

## [1321](https://leetcode.com/problems/restaurant-growth/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as (
select visited_on,sum(amount) as total_amount
from customer 
group by visited_on),
cte2 as(
select visited_on,
sum(total_amount)over(order by visited_on rows between 6 PRECEDING and current row)as amount,round(avg(total_amount)over(order by visited_on rows between 6 PRECEDING and current row),2) as average_amount
from cte )

select *
from cte2
where visited_on>=(select visited_on from cte2 order by visited_on limit 1)+6
order by visited_on
```

## [1341](https://leetcode.com/problems/movie-rating/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select mr.*,u.name as name ,m.title as title
from MovieRating mr
left join users u 
on mr.user_id=u.user_id
left join movies m
on mr.movie_id=m.movie_id)

(select name as results
from cte 
group by name
order by count(*) desc,name limit 1)
union all
(select title
from cte 
where date_format(created_at,'%Y-%m')='2020-02'
group by title
order by avg(rating) desc,title  limit 1)
```

## [1393](https://leetcode.com/problems/capital-gainloss/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select stock_name,
sum(case when operation='buy'then price*-1 else price end) as capital_gain_loss
from Stocks 
group by stock_name
```

## [1907](https://leetcode.com/problems/count-salary-categories/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select 'Low Salary' as category, sum(income<20000) as accounts_count from accounts
union
select 'Average Salary' as category, sum(income between 20000 and 50000) as accounts_count from accounts
union
select 'High Salary' as category, sum(income> 50000) as accounts_count from accounts
```

## [1934](https://leetcode.com/problems/confirmation-rate/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select s.user_id, round(avg(if(c.action="confirmed",1,0)),2) as confirmation_rate
from Signups as s left join Confirmations as c on s.user_id= c.user_id group by user_id;
```

## [3220](https://leetcode.com/problems/odd-and-even-transactions/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select transaction_date,
sum(case when amount%2=1 then amount else 0 end)as odd_sum,
sum(case when amount%2=0 then amount else 0 end )as even_sum
from transactions 
group by transaction_date
order by transaction_date
```

## [3421](https://leetcode.com/problems/find-students-who-improved/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
WITH cte AS (
    SELECT 
        student_id,
        subject,
        FIRST_VALUE(score) OVER (
            PARTITION BY student_id, subject 
            ORDER BY exam_date
        ) AS first_score,
        FIRST_VALUE(score) OVER (
            PARTITION BY student_id, subject 
            ORDER BY exam_date DESC
        ) AS latest_score
    FROM scores 
)
SELECT * 
FROM cte 
GROUP BY student_id, subject 
HAVING first_score < latest_score;
```

## [3475](https://leetcode.com/problems/dna-pattern-recognition/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select *,
case when dna_sequence regexp "^ATG" then 1  else 0 end as has_start,
case when dna_sequence regexp'TAA$|TAG$|TGA$' then 1 else 0 end as has_stop ,
case when dna_sequence regexp 'ATAT' then 1 else 0 end as has_atat,
case when dna_sequence regexp 'GGG' then 1 else 0 end as has_ggg
from samples
```

## [3564](https://leetcode.com/problems/seasonal-sales-analysis/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select s.*,p.category,
case when month(s.sale_date)in(12,1,2) then "Winter"
when month(s.sale_date)in(3,4,5) then "Spring"
when month(s.sale_date)in(6,7,8) then "Summer"
else "Fall" end as season
from sales s
left join products p
on s.product_id=p.product_id),

cte2 as(
select season,category, sum(quantity) as total_quantity,
sum(quantity*price)as total_revenue
from cte 
group by season,category),

cte3 as(
select * ,
row_number()over(partition by season order  by total_quantity desc
,total_revenue desc ) as rnk from cte2)

select season,category,total_quantity,total_revenue
from cte3
where rnk=1
order by season;
```

## [3580](https://leetcode.com/problems/find-consistently-improving-employees/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as
(select *, row_number()over(partition by employee_id order by review_date desc)as rnk
from  performance_reviews as p),

cte2 as(
select employee_id,
sum(case when rnk=1 then rating end)as r1,
sum(case when rnk=2 then rating end)as r2,
sum(case when rnk=3 then rating end) as r3
from cte
where rnk<=3
group by employee_id)

select c1.employee_id,e.name,
(c1.r1-c1.r3) as improvement_score
from cte2  c1
left join employees e
on c1.employee_id=e.employee_id
where c1.r1 is not null and c1.r2 is not null and c1.r3 is not null
and c1.r1>c1.r2 and c1.r2>c1.r3
order by improvement_score desc,name
```

## [3586](https://leetcode.com/problems/find-covid-recovery-patients/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select * ,row_number()over(partition by patient_id,result order by test_date)as rnk1
from covid_tests),

cte2 as(
select c1.patient_id,row_number()over(partition by c1.patient_id order by c2.test_date)as rnk2,datediff(c2.test_date,c1.test_date) as recovery_time
from cte as c1
left join cte as c2
on c1.patient_id=c2.patient_id
and c1.test_date<c2.test_date
where c1.result='positive' and c1.rnk1=1 and c2.result='negative')

select c.patient_id,p.patient_name,p.age,c.recovery_time
from cte2 c
left join patients p
on c.patient_id =p.patient_id
where c.rnk2=1
order by c.recovery_time,patient_name
```
## [3601](https://leetcode.com/problems/find-drivers-with-improved-fuel-efficiency/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select * ,distance_km/fuel_consumed as fuel_efficiency
from trips),

cte2 as(
select c.driver_id,d.driver_name,avg(case when month(trip_date) between 1 and 6 then fuel_efficiency else null end)as first_half_avg,
avg(case when month(trip_date) between 7 and 12 then fuel_efficiency else null end) as second_half_avg
from cte c
left join drivers d
on c.driver_id=d.driver_id
group by driver_id)


select driver_id,driver_name,
round(first_half_avg,2)as first_half_avg ,
round(second_half_avg,2)as second_half_avg,
round(second_half_avg-first_half_avg,2)as efficiency_improvement
from cte2
where second_half_avg>first_half_avg
order by efficiency_improvement desc,driver_name 
```

## [3611](https://leetcode.com/problems/find-overbooked-employees/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(
select employee_id,
week(meeting_date,1) as week_monday,
sum(duration_hours),
case when (sum(duration_hours)/40)>0.5 then "yes" else "no"end as meeting_heavy
from meetings
group by employee_id,week(meeting_date,1))


select c.employee_id, 
e.employee_name , 
e.department,
count(*) as meeting_heavy_weeks
from cte as c
left join employees as e
on c.employee_id=e.employee_id
where c.meeting_heavy="yes"
group by c.employee_id,e.employee_name,e.department
having count(*)>1
order by count(*) desc,e.employee_name
```

# Hard

## [185](https://leetcode.com/problems/department-top-three-salaries/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with cte as(select 
e.name as Employee,
e.salary as Salary,
dense_rank()over(partition by d.id order by Salary desc) as rnk,
d.name as Department
from Employee e
join Department d
on d.id=e.departmentId)


select Department,Employee,Salary
from cte 
where rnk < 4 
```

## [262](https://leetcode.com/problems/trips-and-users/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
select request_at as Day, round(sum(status<>'completed')/count(*),2)as 'Cancellation Rate'
from trips t
left join users u1
on t.client_id=u1.users_id
left join users u2
on t.driver_id=u2.users_id
where u1.banned='No' and u2.banned='No 'and 
t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
group by day
order by day
```

## [3482](https://leetcode.com/problems/analyze-organization-hierarchy/description/?envType=problem-list-v2&envId=database)
- **ANS**
```sql
with recursive cte1 as(
select employee_id,employee_name , manager_id ,salary ,1 as level from employees where manager_id is null
union all 
select e.employee_id,e.employee_name, e.manager_id ,e.salary ,c.level+1 as level from employees as e 
inner join cte1 as c 
on c.employee_id=e.manager_id),
cte2 as(
select employee_id as manager_id,
employee_id as sub_id,salary
from employees
union all
select c.manager_id,e.employee_id,e.salary
from employees e
join cte2 as c
on e.manager_id= c.sub_id),
cte3 as(
select manager_id,count(*)-1 as teamsize,
sum(salary)as budget
from cte2
group by manager_id)

select c.employee_id ,c.employee_name,c.level,c3.teamsize as team_size,c3.budget as budget
from cte1 c
left join cte3 c3
on c.employee_id=c3.manager_id
order by c.level , budget desc, c.employee_name
```