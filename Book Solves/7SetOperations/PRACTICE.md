1. Employees such that they where hired between the minimum and maximum hiring dates of their respective departments but not on those dates

```sql
-- using set operations
SELECT
	E1.EMPLOYEE_ID,
	TO_CHAR( E1.HIRE_DATE, 'DD-MON-YY' ) HIREDATE,
	E1.DEPARTMENT_ID 
FROM
	EMPLOYEES E1 
WHERE
	DEPARTMENT_ID IS NOT NULL 
MINUS
	(
	SELECT
		E2.EMPLOYEE_ID,
		TO_CHAR( E2.HIRE_DATE, 'DD-MON-YY' ) HIREDATE,
		E2.DEPARTMENT_ID 
	FROM
		EMPLOYEES E2 
	WHERE
		E2.HIRE_DATE = ( SELECT MIN( ES.HIRE_DATE ) FROM EMPLOYEES ES WHERE ES.DEPARTMENT_ID = E2.DEPARTMENT_ID ) UNION
	SELECT
		E3.EMPLOYEE_ID,
		TO_CHAR( E3.HIRE_DATE, 'DD-MON-YY' ) HIREDATE,
		E3.DEPARTMENT_ID 
	FROM
		EMPLOYEES E3 
	WHERE
		E3.HIRE_DATE = ( SELECT MAX( ES.HIRE_DATE ) FROM EMPLOYEES ES WHERE ES.DEPARTMENT_ID = E3.DEPARTMENT_ID ) 
	);

-- using exists and not exists
SELECT 
	E1.EMPLOYEE_ID,
	TO_CHAR(E1.HIRE_DATE,'DD-MON-YY') HIREDATE,
	E1.DEPARTMENT_ID
FROM
	EMPLOYEES E1
WHERE
	DEPARTMENT_ID IS NOT NULL
MINUS
(SELECT
	E2.EMPLOYEE_ID,
	TO_CHAR(E2.HIRE_DATE,'DD-MON-YY') HIREDATE,
	E2.DEPARTMENT_ID
	FROM
	EMPLOYEES E2
WHERE
	NOT EXISTS(SELECT E3.EMPLOYEE_ID,
	TO_CHAR(E3.HIRE_DATE,'DD-MON-YY') HIREDATE,
	E3.DEPARTMENT_ID
	FROM
	EMPLOYEES E3 
	WHERE
	E3.DEPARTMENT_ID = E2.DEPARTMENT_ID AND E3.HIRE_DATE > E2.HIRE_DATE
	)
	OR NOT EXISTS(
	SELECT E3.EMPLOYEE_ID,
	TO_CHAR(E3.HIRE_DATE,'DD-MON-YY') HIREDATE,
	E3.DEPARTMENT_ID
	FROM
	EMPLOYEES E3 
	WHERE
	E3.DEPARTMENT_ID = E2.DEPARTMENT_ID AND E3.HIRE_DATE < E2.HIRE_DATE
	));
```


