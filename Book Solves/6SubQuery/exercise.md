# 6 

- [6.1](#exercise-61)
- [6.2](#exercise-62)

## Exercise 6.1
5. Find the last names and salaries of those employees whose salary is within ± 5k of the average salary of SALES department.

```sql
SELECT
	E1.FIRST_NAME || ' ' || E1.LAST_NAME FULLNAME,
	E1.SALARY
FROM
	EMPLOYEES E1 
WHERE
	SALARY BETWEEN (
        SELECT 
            AVG(SALARY)-5000 
        FROM 
            EMPLOYEES 
            JOIN DEPARTMENTS ON EMPLOYEES.DEPARTMENT_ID = DEPARTMENTS.DEPARTMENT_ID 
        WHERE LOWER(DEPARTMENT_NAME) = 'sales'
    ) AND
	(
        SELECT 
            AVG(SALARY) + 5000 
        FROM 
            EMPLOYEES JOIN DEPARTMENTS ON EMPLOYEES.DEPARTMENT_ID=DEPARTMENTS.DEPARTMENT_ID 
        WHERE LOWER(DEPARTMENT_NAME) = 'sales'
    );
```

## Exercise 6.2
1. Find those employees whose salary is higher than at least three other employees. Print last names and salary of each employee. You cannot use join in the main query. Use sub-query in WHERE clause only. You can use join in the sub-queries.

```sql
SELECT
	E1.LAST_NAME,E1.SALARY
FROM
	EMPLOYEES E1
WHERE
	(SELECT COUNT(E2.EMPLOYEE_ID) FROM EMPLOYEES E2 WHERE E1.SALARY > E2.SALARY) >= 3;
```


2. Find those departments whose average salary is greater than the minimum salary of all other departments. Print department names. Use sub-query. You can use join in the sub-queries.

```sql
SELECT
	E1.LAST_NAME,E1.SALARY
FROM
	EMPLOYEES E1
WHERE
	(SELECT COUNT(E2.EMPLOYEE_ID) FROM EMPLOYEES E2 WHERE E1.SALARY > E2.SALARY) >= 3;
```


3. Find those department names which have the highest number of employees in service. Print department names. Use sub-query. You can use join in the sub-queries.

```sql
SELECT
	( SELECT D.DEPARTMENT_NAME FROM DEPARTMENTS D WHERE E1.DEPARTMENT_ID = D.DEPARTMENT_ID ) DEPARTMENT_NAME 
FROM
	EMPLOYEES E1 
WHERE
	E1.DEPARTMENT_ID IS NOT NULL 
	AND (
	SELECT
		COUNT( E2.EMPLOYEE_ID ) 
	FROM
		EMPLOYEES E2 
	WHERE
		E2.DEPARTMENT_ID = E1.DEPARTMENT_ID 
		) =  (
	SELECT
		MAX(COUNT( E3.EMPLOYEE_ID ))
	FROM
		EMPLOYEES E3 
	WHERE
		DEPARTMENT_ID IS NOT NULL
	GROUP BY
		E3.DEPARTMENT_ID 
	) 
GROUP BY
	E1.DEPARTMENT_ID 
ORDER BY
	E1.DEPARTMENT_ID;
```

4. Find those employees who worked in more than one department in the company. Print employee last names. You cannot use join in the main query. Use sub-query. You can use join in the sub-queries.

```sql
SELECT
	E1.LAST_NAME
FROM
	EMPLOYEES E1 
WHERE
	E1.EMPLOYEE_ID IN ( SELECT J.EMPLOYEE_ID FROM JOB_HISTORY J WHERE J.EMPLOYEE_ID = E1.EMPLOYEE_ID AND J.DEPARTMENT_ID <> E1.DEPARTMENT_ID );
```


5. For each employee, find the minimum and maximum salary of his/her department. Print employee last name, minimum salary, and maximum salary. Do not use sub-query in WHERE clause. Use sub-query in FROM clause.

```sql
SELECT
	E1.LAST_NAME,
	MAX_SALARY,
	MIN_SALARY
FROM 
	EMPLOYEES E1 ,
	(
	SELECT 
		DEPARTMENT_ID ,
		MIN(SALARY) MIN_SALARY,
		MAX(SALARY) MAX_SALARY
	FROM 
		EMPLOYEES
	WHERE	
		DEPARTMENT_ID IS NOT NULL
	GROUP BY
		DEPARTMENT_ID
	) D 
WHERE
	E1.DEPARTMENT_ID = D.DEPARTMENT_ID
```


6. For each job type, find the employee who gets the highest salary. Print job title and last name
of the employee. Assume that there is one and only one such employee for every job type.

```sql
SELECT 
	E1.LAST_NAME ,
	(SELECT JOB_TITLE FROM JOBS WHERE JOB_ID = E1.JOB_ID) JOB_TITLE 
FROM 
	EMPLOYEES E1
WHERE 
	E1.SALARY >= ALL (
	SELECT 
		SALARY
	FROM
		EMPLOYEES 
	WHERE 
		JOB_ID = E1.JOB_ID
	)
ORDER BY 
	SALARY DESC;
```

