# FROM THE VAULT  
A collection of sql queries without any context

1. Details about the tables and columns under a specific owner
```sql
SELECT owner, table_name, column_name, data_type
FROM all_tab_columns
WHERE owner = 'HR';
```

2. Dates !!!
```sql
SELECT ROUND(TO_DATE('15-01-23','DD-MM-YY'),'MONTH') FROM DUAL;
SELECT ROUND(TO_DATE('16-01-23','DD-MM-YY'),'MONTH') FROM DUAL;
SELECT TO_CHAR(ROUND(TO_DATE('16-01-23','DD-MM-YY'),'MONTH'),'DDSPTH MONTH,YEAR') FROM DUAL;
SELECT TO_CHAR(ROUND(TO_DATE('16-01-23','DD-MM-YY'),'MONTH'),'DDSPTH MONTH,YEAR') FROM DUAL;
SELECT TO_CHAR(TO_DATE('21-7-23','DD-MM-YY'),'DdspTH Month,Year') FROM DUAL;
```

3. Number of Rows of all tables in a database schema
```sql
SELECT table_name, num_rows
FROM all_tables
WHERE owner = 'UNIVERSITY';
```

4. Show the last name and salary of all employees in taka (show thousands,hundreds)
```sql
SELECT
	LAST_NAME,
	TRUNC( SALARY / 1000, 0 ) || ' thousands ' || TRUNC( MOD( SALARY, 1000 ) / 100, 0 ) || ' hundreds ' || MOD( SALARY, 100 ) || ' taka only' "Salary In Taka" 
FROM
	EMPLOYEES;
```

