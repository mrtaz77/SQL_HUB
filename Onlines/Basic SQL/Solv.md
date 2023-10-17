# BASIC SQL : Solutions 

- [A1](#a1) 
- [A2_B2](#a2_b2) 
- [B1](#b1) 

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

## A2_B2
1. For  each  department,  find  its  name  and  address.  The  address  should  print  each 
location in the following format: STREET_ADDRESS, CITY, STATE_PROVINCE, 
POSTAL_CODE. Order the output according to department ID (low to high). 
```sql
SELECT
	DEPARTMENT_ID,
	DEPARTMENT_NAME,
	( STREET_ADDRESS || ',' || CITY || ',' || STATE_PROVINCE || ',' || POSTAL_CODE ) ADDRESS 
FROM
	DEPARTMENTS
	JOIN LOCATIONS USING ( LOCATION_ID ) 
ORDER BY
	DEPARTMENT_ID;
```

2. Show the name of the day and number of employees hired in that day. Day should be 
sorted as the following order Friday, Saturday, Sunday, Monday, Tuesday, Wednesday, 
Thursday (See Sample output). 
```sql
SELECT
	TO_CHAR( HIRE_DATE, 'DAY' ) WEEKDAY,
	COUNT( * ) EMPLOYEE_COUNT 
FROM
	EMPLOYEES 
GROUP BY
	TO_CHAR( HIRE_DATE, 'DAY' ),
	TO_CHAR( HIRE_DATE, 'D' ) 
ORDER BY
	MOD(( TO_NUMBER( TO_CHAR( HIRE_DATE, 'D' ) ) + 1 ),7);
```


3.  For each employee, find the total number of employees who earn more than him/her 
and who earn less than him/her. Print employee ID, full name (first_name, space, 
last_name), salary, number of employees who earn more than him/her and number of 
employees who earn less than him/her. Order the output according to employee ID 
(low to high). Print only those employees who are gets more salary than at least 10 
other employees and gets less salary than at most 25 other employees.  
```sql
SELECT
	E1.EMPLOYEE_ID,
	(E1.FIRST_NAME||' '|| E1.LAST_NAME) FULLNAME,
	E1.SALARY,
	COUNT(DISTINCT E2.EMPLOYEE_ID) "RICHER THAN",
	COUNT(DISTINCT E3.EMPLOYEE_ID) "POORER THAN"
FROM
	EMPLOYEES E1
	LEFT OUTER JOIN EMPLOYEES E2 ON ( E2.SALARY < E1.SALARY ) 
	LEFT OUTER JOIN EMPLOYEES E3 ON ( E3.SALARY > E1.SALARY ) 
GROUP BY
	E1.EMPLOYEE_ID,(E1.FIRST_NAME||' '|| E1.LAST_NAME),E1.SALARY
HAVING
	COUNT(DISTINCT E2.EMPLOYEE_ID) >= 10 AND
	COUNT(DISTINCT E3.EMPLOYEE_ID) <= 25
ORDER BY
	E1.EMPLOYEE_ID;
```
 
4.  Display  the  first  name  and  salary  for  all  employees.  Format  the  salary  to  be  10 
characters  long,  right  justified  with  the  ‘$’  symbol,  the  department  id  and  the 
department name (without spaces) according to the length of the department name 
(without spaces) ordered from high to low and belonging to department id in range of 
10 to 90.  
```sql
SELECT
	FIRST_NAME,
	LPAD( SALARY, 10, '$' ),
	DEPARTMENT_ID,
	REPLACE ( DEPARTMENT_NAME, ' ', '' ) NAME 
FROM
	EMPLOYEES
	JOIN DEPARTMENTS USING ( DEPARTMENT_ID ) 
WHERE
	DEPARTMENT_ID BETWEEN 10 
	AND 90 
ORDER BY
	LENGTH( REPLACE ( DEPARTMENT_NAME, ' ', '' ) ) DESC;
```
 
5.  Display the Employee_Code (Employee_Code consists of 1st letter of the first name, 
1st letter of the last name, a SPACE and the employee id; i.e. if first name is Lionel, 
last name is Messi and employee id is 10 then Employee_Code is LM 10) and the total 
salary s/he earned so far, for those employees whose experience is more than 18 years. 
You can safely assume, if an employee joins in the middle of a month s/he gets the full 
salary of that month also this month’s salary hasn’t been paid yet. Order the output 
according to employee id high to low.
(the output is dependent on the current date)
```sql
SELECT
	( SUBSTR( FIRST_NAME, 1, 1 ) || SUBSTR( LAST_NAME, 1, 1 ) || ' ' || EMPLOYEE_ID ) EMPLOYEE_CODE,
	( SALARY * MONTHS_BETWEEN( TRUNC( SYSDATE, 'MONTH' ), TRUNC( HIRE_DATE, 'MONTH' ) ) ) EARNED_SALARY 
FROM
	EMPLOYEES 
WHERE
	( SYSDATE - HIRE_DATE ) / 365 > 18 
ORDER BY
	EMPLOYEE_ID DESC;
```

## B1
1.  Show the full name (full name means including single space between first_name and 
last_name) and salary (sorted from low to high) of the employees whose length of the 
full name is more than 10 characters (without spaces). 
```sql
SELECT
	FIRST_NAME || ' ' || LAST_NAME FULLNAME,
	SALARY 
FROM
	EMPLOYEES 
WHERE
	LENGTH( CONCAT( FIRST_NAME, LAST_NAME ) ) > 10 
ORDER BY
	SALARY;
```


2.  Show the name of the month and number of employees hired in that month. Month 
should be sorted according to the appearance in the calendar. (See Sample output). 
```sql
SELECT
	TO_CHAR( TRUNC( HIRE_DATE ), 'MONTH' ) MONTH_NAME,
	COUNT( * ) TOTAL_EMPLOYEES 
FROM
	EMPLOYEES 
GROUP BY
	TO_CHAR( TRUNC( HIRE_DATE ), 'MONTH' ) 
ORDER BY
	TO_DATE( TO_CHAR( TRUNC( HIRE_DATE ), 'MONTH' ), 'MONTH' );
```


3.  For each manager who manages more than 4 employees show the manager id, number 
of  employees  and  the  average  salary  of  the  employees  (up  to  2nd  decimal  point) 
managed by him/her. Show the outputs with average salary greater than 3000. Print the 
output in the descending order of number of employees managed and if there is a tie 
then print those in ascending order of average salary.  
```sql
SELECT
	MANAGER_ID,
	COUNT( * ) TOTAL_EMPLOYEES,
	ROUND( AVG( SALARY ), 2 ) AVG_SAL 
FROM
	EMPLOYEES 
GROUP BY
	MANAGER_ID 
HAVING
	ROUND( AVG( SALARY ), 2 ) > 3000 
	AND COUNT( * ) > 4 
ORDER BY
	COUNT( * ) DESC,
	AVG_SAL ASC;
```


4.  Show  the  department  name,  number  of  employees  and  the  average  salary  of  the 
employees (up to 2nd decimal point) for each of the department. Sorted by number of 
employees from high to low.  
```sql
SELECT
	DEPARTMENT_NAME,
	COUNT( * ) TOTAL_EMPLOYEES,
	ROUND( AVG( SALARY ), 2 ) AVG_SAL 
FROM
	DEPARTMENTS
	JOIN EMPLOYEES USING ( DEPARTMENT_ID ) 
GROUP BY
	DEPARTMENT_NAME 
ORDER BY
	TOTAL_EMPLOYEES DESC;
```



5.  Display the fullname (fullname consists of 1st letter of first name, a DOT, a SPACE and the 
last name; i.e. if first name is Lionel and last name is Messi then fullname is L. Messi) and the 
yearly_income (sorted high to low) of the employees whose last name starts with 'M' or 'm'.  

Where, yearly_income is calculated as follows:  
 
      yearly_salary + yearly_salary * commision_pct 
 
Note that, there should be no null value in the output.
```sql
SELECT
	SUBSTR( FIRST_NAME, 1, 1 ) || '. ' || INITCAP( LAST_NAME ) FULLNAME,
	( SALARY * 12 ) * ( 1+NVL ( COMMISSION_PCT, 0 ) ) YEARLY_INCOME 
FROM
	EMPLOYEES 
WHERE
	LOWER( LAST_NAME ) LIKE 'm%' 
ORDER BY
	YEARLY_INCOME DESC;
```