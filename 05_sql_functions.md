### Sql functions type :

- functions take an i/p , process and return rows

1) single row funtion :
take a single value and return a single value     lowers()

2) multi-row funtion :
take multiple values and return a single value     sum()


### Nested funtions :

LEN(LOWER(LEFT("Maria",2)))    executions start from inside to outside 



## Categorization of Functions in SQL :

- Single row function are imp for data engineers and multi-row functions for Data analysts

![function_cat](./images/functions_categories.png)







--------------------------------------------------------------------------------------------------------

### sql Aggregate functions :

- multiple rows as i/p and single value as o/p

1) count(*)

count the no. of rows even if all nulls are present in a row , that row is also counted

count(col_name) here nulls will not be counted for that column

```sql
SELECT COUNT(*)               AS total_rows,
       COUNT(bonus)           AS bonus_given,
       COUNT(DISTINCT bonus)  AS distinct_bonus_values --> counts distinct bonus values in table excludes NUll counting
FROM employees;

```
count(1) is same as count(*) , no performance difference


2) sum(salary)

3) AVG(sales)
does not consider null values in sum and also does not consider count in sums division

4) MIN(score)

5) MAX(score)



