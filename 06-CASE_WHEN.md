## Case statements

- Evaluates a list of conditions and returns a value when the first condition is met 

- syntax :

```sql

CASE 
    WHEN cond1 THEN result 1
    WHEN cond2 THEN result 2
    ...
    ELSE result          ---> if no condition is true then return this o/p (its optional btw)

END

```

- if any of the condition is evaluated as True, the below conditions wont be checked for that argument

### A single rule to follow :
- datatype after THEN and ELSE must be matching TYPE

- CASE STATEMENTS CAN BE USED IN ANY OF THE CLAUSE , SELECT , GROUP BY, HAVING , ORDER BY , JOIN ,ETC



## USE CASES :

Main purpose is to do data Transformation ( derive new col for analytics )

### 1) Categorizing Data :

- group data in categories based on some conditions 

```sql
Task : generate a report showing total sales per category 
       high if sales higher than 50
       medium if b/w 20 and 50
       low  if <=20

sort the result highest to lowest 

SELECT 
Category,
SUM(sales) as Total_sales
FROM (

SELECT 
    ORDERID,
    sales, 
    CASE
       WHEN sales > 50 THEN 'HIGH'
       WHEN sales > 20 THEN 'MEDIUM'
       ELSE 'LOW'
END AS CATEGORY
) t
GROUP BY Category
ORDER BY Total_sales;

--> Here we have used subquery, think of it like inner query executes first and forms a temporary table , and the outer query is being made on top of that inner temp table, so u can refer to inner tables columns and all in outer query

```

---------------------------------------------------------------------------------------------


### 2) MAPPING VALUES

```sql
Task : Retrieve employee details with gender displayed as full text 

select 
    emp_id,
    name,
    gender,
    CASE 
        WHEN gender = 'M' then 'male'
        WHEN gender = 'F' then 'female'
        ELSE 'Not Available'
    END gender_full
from employees;


```

---------------------------------------------------------------------------------------------


### short hand syntax of case when :

```sql 
CASE  Country                 --> only 1 col can be evealuated here 
WHEN 'germany' then 'de'
when 'india' then 'ind'
when 'australia' then 'aus'
END 

otherwise
WHEN country = 'germany' then 'de'
when country = 'india' then 'ind'
when country = 'australia' then 'aus'

```

--------------------------------------------------------------------------------------------

### 3) handling nulls :

- replace null with a specific value

```sql
Task : find avg scores of customers and treat nulls as 0 and additionally provide details such as cust_id and lastname

 
```


---------------------------------------------------------------------------------------------

### 4) conditional aggregation :

```sql

Task : count how many times each customer has made and order with sales greater than 30

```

![table_to_look](./images/table.png)


```sql
ans:

SELECT 
    customer_id
    SUM(CASE 
        WHEN sales >30 then 1
        ELSE 0
        END) TOTAL_ORDERS,
    count(*) as in_total_order_count
from orders
group by customer_id;


```



--------------------------------------------------------------------------------------------------------

### CASE STATEMENTS can be used in any of the clause below are few eg with each clause

1) CASE WHEN in select (derieve a column)

```sql
SELECT name, salary,
       CASE WHEN salary >= 100000 THEN 'High'
            WHEN salary >= 50000  THEN 'Mid'
            ELSE 'Low' END AS band
FROM employees;

the column: band will appear in o/p and string "hig , mid and low" will appear based on rows 

```


2) CASE WHEN in WHERE clause :

```sql

SELECT name, dept, salary
FROM employees
WHERE (salary > 
    CASE WHEN dept = 'IT' THEN 70000 
    ELSE 50000 
END);

Here the query will return all the rows where if the dept of person is IT then his salary must be greater than 70000 , else if the person is from other dept then even at 50000 > he can be shown in o/p

```


3) CASE WHEN IN ORDER BY clause :

```sql
SELECT name, rating
FROM employees
ORDER BY CASE rating WHEN 'A' THEN 1 WHEN 'B' THEN 2 ELSE 3 END, name;

the rows appearing for name, rating from employees will be ordered by the conditional we genrate and then by name, nested order bying

```



4) CASE WHEN in GROUP BY :


```sql
SELECT CASE WHEN age < 30 THEN 'Young'
            WHEN age < 50 THEN 'Mid-age'
            ELSE 'Senior' END AS age_group,
       COUNT(*) AS cnt
FROM employees
GROUP BY CASE WHEN age < 30 THEN 'Young'
              WHEN age < 50 THEN 'Mid-age'
              ELSE 'Senior' END;

In group by the same case when as select is usually used as in execution order GROUP BY executes before SELECT therefore cant use the alias directly in GROUP BY 


```


5) CASE WHEN IN HAVING :

```sql

SELECT dept, SUM(salary) AS total
FROM employees
GROUP BY dept
HAVING SUM(salary) > CASE WHEN dept = 'IT' THEN 200000 ELSE 100000 END;

```



6) CASE WHEN INSIDE AGGREGATES :

```sql
SELECT dept,
       SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_cnt,
       SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_cnt
FROM employees
GROUP BY dept;

```


7) CASE WHEN INSIDE WINDOW FUNCTION :

```sql

SELECT name, dept, salary,
       SUM(CASE WHEN rating = 'A' THEN salary ELSE 0 END)
           OVER (PARTITION BY dept) AS dept_A_payroll
FROM employees
ORDER BY dept, name;

```


8) CASE WHEN IN UPDATE STATEMENTS :

```sql
UPDATE employees
SET salary = CASE rating
                 WHEN 'A' THEN salary * 1.15
                 WHEN 'B' THEN salary * 1.10
                 ELSE salary * 1.03
             END;

```