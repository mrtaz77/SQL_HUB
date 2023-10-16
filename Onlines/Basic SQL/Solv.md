# BASIC SQL : Solutions 

- [A1](#a1) 
- [A2](#a2) 
- [B1](#b1) 
- [B2](#b2)


## A1
1. Find the number of employees working in each city. Order the output according to the city name.
```sql
SELECT
	CITY,
	COUNT( DISTINCT EMPLOYEE_ID ) "Employee Count" 
FROM
	LOCATIONS
	JOIN DEPARTMENTS USING ( LOCATION_ID )
	JOIN EMPLOYEES USING ( DEPARTMENT_ID ) 
GROUP BY
	CITY 
ORDER BY
	CITY;
```

2. For each employee, find his/her manager’s name, department name and job title along with his/her
name and ID. Order the output according to employee ID.
```sql
SELECT
	E1.EMPLOYEE_ID,
	( E1.FIRST_NAME || ' ' || E1.LAST_NAME ) NAME,
	NVL( ( E2.FIRST_NAME || ' ' || E2.LAST_NAME ), '' ) "Manager Name",
	D.DEPARTMENT_NAME,
	J.JOB_TITLE 
FROM
	EMPLOYEES E1
	JOIN DEPARTMENTS D USING ( DEPARTMENT_ID )
	JOIN JOBS J USING ( JOB_ID )
	LEFT JOIN EMPLOYEES E2 ON E1.MANAGER_ID = E2.EMPLOYEE_ID 
ORDER BY
	E1.EMPLOYEE_ID;
```

3. For each manager, find their ID, name, salary, number of employees working under them and the
average salary of the employees. Take average up to two decimal points. Order the output
according to manager ID.

```sql
SELECT DISTINCT
	E1.MANAGER_ID "MANAGER_ID",
	(E2.FIRST_NAME || ' ' || E2.LAST_NAME ) FULLNAME,
	E2.SALARY,
	COUNT( E1.EMPLOYEE_ID ) EMPLOYEE_COUNT,
	ROUND( AVG( E1.SALARY ), 2 ) AVG_SAL 
FROM
	EMPLOYEES E1 JOIN EMPLOYEES E2 ON E1.MANAGER_ID = E2.EMPLOYEE_ID
GROUP BY
	E1.MANAGER_ID,
	(E2.FIRST_NAME || ' ' || E2.LAST_NAME ),
	E2.SALARY
ORDER BY
	E1.MANAGER_ID;
```

4. For each employee who have never switched job, find his/her ID, full name (first name + ' ' + last
name), department name, job title and annual salary (= salary * 12 * (1 + COMMISSION_PCT)).
Order the output according to employee ID.

```sql
SELECT
	E1.EMPLOYEE_ID,
	( E1.FIRST_NAME || ' ' || E1.LAST_NAME ) FULLNAME,
	DEPARTMENT_NAME,
	J.JOB_TITLE,
	SALARY * 12 * ( 1+NVL ( COMMISSION_PCT, 0 ) ) "ANNUAL SALARY" 
FROM
	EMPLOYEES E1
	JOIN DEPARTMENTS D USING ( DEPARTMENT_ID )
	JOIN JOBS J USING ( JOB_ID ) 	
	LEFT OUTER JOIN JOB_HISTORY JH ON E1.EMPLOYEE_ID = JH.EMPLOYEE_ID
WHERE
	JH.EMPLOYEE_ID IS NULL
ORDER BY
	E1.EMPLOYEE_ID;
```

5. For each manager, find the total number of employees who earn more than him/her and who earn
less than him/her. Print the manager’s ID, name, salary, number of employees who earn more than
him/her and number of employees who earn less than him/her. Order the output according to
manager ID.
```sql
SELECT
	E1.EMPLOYEE_ID "MANAGER_ID",
	( E1.FIRST_NAME || ' ' || E1.LAST_NAME ) FULLNAME,
	COUNT( DISTINCT E2.EMPLOYEE_ID ) "RICHER THAN",
	COUNT( DISTINCT E3.EMPLOYEE_ID ) "POORER THAN" 
FROM
	EMPLOYEES E1 LEFT OUTER
	JOIN EMPLOYEES E2 ON ( E2.SALARY < E1.SALARY ) LEFT OUTER
	JOIN EMPLOYEES E3 ON ( E3.SALARY > E1.SALARY ) 
	JOIN EMPLOYEES E4 ON  E1.EMPLOYEE_ID = E4.MANAGER_ID
GROUP BY
	E1.EMPLOYEE_ID,
	( E1.FIRST_NAME || ' ' || E1.LAST_NAME )
ORDER BY
	E1.EMPLOYEE_ID;
```