## Select queries and clauses :

### Clauses :
select
distinct
top
from
join
where
group by 
having
order by


#### 1 and 2) select and from clause :

```sql
select * from table;  all column and rows shown from table


select 
    name,
    email_id      no , after last column name 
from employees; 


```
------------------------------------------------------------------------------------------------

### 3) WHERE clause :

- just for filtering based on CONDITIONS
- Lot of conditions can be added with where : 
- Common conditions:

  - AND, OR, NOT: WHERE dept_id = 2 AND salary > 56000 → Chloe
  - IN: WHERE dept_id IN (1, 3) → Asha, Ben, Elena
  - BETWEEN: WHERE salary BETWEEN 50000 AND 60000  ( BOTH INCLUSIVE )→ Chloe, Dev, Elena
  - LIKE: WHERE name LIKE 'A%' → Asha (% = any characters, _ = one character)
  - IS NULL: WHERE dept_id IS NULL → Farid. Note that = NULL never works, because NULL isn't equal to anything.


```sql
SELECT * FROM CUSTOMERS
WHERE SCORE != 0;


SELECT 
    name,
    country
from customers
where country='GERMANY';


(time pass shit NOT operator)
Task : select all the students with score not less than 500 (that means greater than or equal to 500)

SELECT *
from students
where score >=500;    this is aam zindagi

SELECT *
from students
where NOT score < 500;   this is mentos zindagi (dont use this !!)
it just exculudes matching rows  

```
-------------------------------------------------------------------------------------------------

### 4) ORDER BY :  

- this is used for sorting rows based on 1 or more columns we provide


```sql
SELECT 
    name,
    email,
    score
from students
ORDER BY SCORE DESC;   --> highest to lowest score


nested sorting - sorting based on more than 1 column

SELECT
    name,
    email,
    country,
    score
from students
ORDER BY country ASC, score DESC;  --> the rows will be sorted by country first (alphabetically) and then withing the sorted groups the rows will be shuffled based on score the highest one will appear first

order of columns in ORDER BY clause is very important, because its nested ordering

- nested sorting makes sense when u have duplicate data and want rich sorting for withing the sorted parts


```

------------------------------------------------------------------------------------------------------

### 5) GROUP BY: 

- group the columns into a single row , and in the o/p the columns seeing can only be of 2 type :
  - either the column is aggregate or the column is present in GROUP BY no other column

- granularity decreases

- columns selected in SELECT must be either **aggregated column or GROUP BY column**

```sql
SELECT
    country,
    SUM(score) as Total_Score
from CUSTOMERS
GROUP BY country;

IMP ::
--> if u provide 2 columns in GROUP BY, then grouping smashing of rows will happen only for unique combo of those 2 rows

task : find total score and total no. of customers for each country 

SELECT 
    score as total_score
    count(*) as total_no_customers
from students
GROUP BY country;

```


-------------------------------------------------------------------------------------------------


### 6) HAVING clause : 

- filter Aggregated data !! if u wanna use HAVING u need GROUP BY in query !!
- u can use CONDITIONS in HAVING but those must be applied only on aggregated columns ( even if its not present in SELECT its okay )

- why HAVING if we had WHERE ?
  - use WHERE when u want to filter data before aggregation, use HAVING if u want to filter data after Aggregation

```sql
Task: find the avg score for each country, considering customers with a score not equal to 0, and return only those countries with an avg score greater than 430

SELECT
country,
AVG(score) as Avg_score
from customers 
where score != 0
GROUP BY country
HAVING AVG(score)>430;


```

--------------------------------------------------------------------------------------------------


### 7) DISTINCT :  

- Removes Duplicates
- must be used immediately after SELECT

- **if there are multipe col names after DISTINCT , the unique combinations are  allowed to be shown, the duplication of unique combos are not shown**

- 

```sql
SELECT DISTINCT name , city, accessorie
from ...
where ....
group by ... ,etc

it means that :
Asha Pune Laptop
Asha Pune Phone
Asha Delhi Laptop    
Ben Pune phone

all the 3 combos for asha would come, if it was just Distinct of name only 2 rows would appear, if it was Distinct name, city then only 3 rows would have appeared !!

```


--------------------------------------------------------------------------------------------------


### 8) TOP / LIMIT :

- restrict the no. of rows returned

- this type of filtering is not based on any condition or anything, it simply look for top k rows after all ur operation and all are performed !!

```sql
Task: retrieve top 3 customers from india who spent most on subscriptions , also tell how much they spent in total

SELECT TOP 3
    id,
    SUM(spent) as total_spent
from sales
where country='india'
GROUP BY id
ORDER BY spent;   


Task : find the 2 ppl who scored lowest in exam 

SELECT TOP 2
    id,
    name
from students
ORDER BY scores ASC;

```

### 9) Join clause will be covered in some other file
--------------------------------------------------------------------------------------------

## SQL Definition Order

```sql

select    : filters columns to be shown
distinct  : filters duplicates
top       : filters result rows
from
join
where     : filters rows before aggregations
group by 
having    : filters rows after aggregation
order by

```

## SQL Execution Order

```sql

from
join
where
group by
having
select
distinct
order by
TOP/limit

```

## Aliases confusion ::

- Generic sql rule is as follows :

- we can defined aliases only in SELECT and FROM clauses  , the only thing that matters is, when referencing that alias in any clause we need to make sure that, that the clause in which we are referencing it must execute after the clause where the alias is defined !!

```sql
SELECT salary * 12 AS annual, annual / 12 AS monthly   -- error in standard SQL
FROM employees;

```

- for CTES : 




