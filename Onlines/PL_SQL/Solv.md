# PL_SQL : Solutions 

- [A1](#a1) 
- [A2_B2](#a2_b2) 
- [B1](#b1) 

## A1
1. 
    - Write a PL/SQL function that takes two course IDs as arguments and returns TRUE if the first course is a prerequisite of strictly more courses than the second one and FALSE otherwise.


    
    - For all courses offered by departments in the Lambeau building, find whether they have more prerequisites than any course offered by the departments in the Bronfman building and print the information using a PL/SQL anonymous block. Order your output by the course titles as shown below. __You cannot use the INTO or WHILE keywords__.

```sql
-- function

CREATE OR REPLACE FUNCTION COMPARE_COURSE(CID1 IN VARCHAR2,CID2 IN VARCHAR2) RETURN VARCHAR2 IS 
C1NUM NUMBER;
C2NUM NUMBER;
BEGIN
SELECT COUNT(COURSE_ID) INTO C1NUM 
FROM PREREQ
WHERE PREREQ_ID = CID1;

SELECT COUNT(COURSE_ID) INTO C2NUM 
FROM PREREQ
WHERE PREREQ_ID = CID2;
IF C1NUM > C2NUM THEN 
	RETURN ('TRUE');
ELSE 
	RETURN ('FALSE');
END IF;
EXCEPTION 
	WHEN NO_DATA_FOUND THEN
		RETURN ('No rows found.');

	WHEN TOO_MANY_ROWS THEN
	RETURN( 'More than one row found.' );

	WHEN OTHERS THEN
	RETURN( 'Some unknown error occurred.');
END;
/

-- plSql block
DECLARE
BEGIN 
FOR L IN (
	SELECT COURSE_ID,TITLE,BUILDING
	FROM COURSE NATURAL JOIN DEPARTMENT
	WHERE BUILDING = 'Lambeau'
	ORDER BY TITLE
)
LOOP 
FOR B IN (
	SELECT COURSE_ID,TITLE,BUILDING
	FROM COURSE NATURAL JOIN DEPARTMENT
	WHERE BUILDING = 'Bronfman'
	ORDER BY TITLE
) 
LOOP 
IF COMPARE_COURSE(L.COURSE_ID,B.COURSE_ID) = 'TRUE' THEN 
	DBMS_OUTPUT.PUT_LINE(L.TITLE||' needs more prerequisites than '||B.TITLE);
END IF;
END LOOP;
END LOOP;
END;
/
```

2. Write a PL/SQL procedure that takes the Student ID and input and calculates
the CGPA per every semester (e.g., Fall 2003). If the CGPA chronologically
increases (let staying the same be considered an increase) in every semester,
print __CGPA is always increasing__. If the CGPA chronologically decreases in
every semester, print __CGPA is always decreasing__. Otherwise print __CGPA is following a zig-zag pattern__. You can assume the input student ID will be
correct.

For *CGPA* calculation, you can use the following functions:

```sql
CREATE OR REPLACE FUNCTION GRADE_TO_POINT(GRADE IN VARCHAR2) 
RETURN FLOAT IS 
    INVALID_LETTER_GRADE EXCEPTION;
BEGIN
-- DBMS_OUTPUT.PUT_LINE(GRADE);
IF GRADE = 'A+' THEN
    RETURN 4.00;
ELSIF GRADE = 'A ' THEN
    RETURN 3.75;
ELSIF GRADE = 'A-' THEN
    RETURN 3.50;
ELSIF GRADE = 'B+' THEN
    RETURN 3.25;
ELSIF GRADE = 'B ' THEN
    RETURN 3.00;
ELSIF GRADE = 'B-' THEN
    RETURN 2.75;
ELSIF GRADE = 'C+' THEN
    RETURN 2.50;
ELSIF GRADE = 'C ' THEN
    RETURN 2.25;
ELSIF GRADE = 'C-' THEN
    RETURN 2.15;
end if;
    RAISE INVALID_LETTER_GRADE;
end;
/

create FUNCTION FIND_CGPA(S_ID IN NUMBER) RETURN FLOAT IS
CGPA FLOAT;
BEGIN
SELECT ROUND(SUM(GP) / SUM(CREDITS), 4) INTO CGPA
FROM
(SELECT MAX(GRADE_TO_POINT(GRADE) * CREDITS) AS GP, CREDITS
FROM TAKES INNER JOIN COURSE C2 on TAKES.COURSE_ID = C2.COURSE_ID WHERE ID = S_ID
GROUP BY ID, C2.COURSE_ID, CREDITS);
-- DBMS_OUTPUT.PUT_LINE('CGPA OF ID ' || S_ID || ' IS ' || CGPA);
RETURN CGPA;
end;
/
```

At first , modify the find_cgpa function to calculate the cgpa of a student in a particular semester of an year.

```sql
CREATE OR REPLACE FUNCTION FIND_CGPA_IN_SEMESTER_OF_YEAR(S_ID IN NUMBER,SEM IN VARCHAR2,Y IN NUMBER) RETURN FLOAT IS 
	CGPA FLOAT;
BEGIN
SELECT 
	ROUND(SUM(GP) / SUM(CREDITS), 4) INTO CGPA 
FROM
(
	SELECT 
		MAX(GRADE_TO_POINT(GRADE) * CREDITS) AS GP, 
		CREDITS 
	FROM 
		TAKES 
	INNER JOIN COURSE C2 
	ON TAKES.COURSE_ID = C2.COURSE_ID 
WHERE ID = S_ID
AND TAKES.SEMESTER = SEM AND TAKES.YEAR = Y
GROUP BY ID, C2.COURSE_ID, CREDITS);
-- DBMS_OUTPUT.PUT_LINE('CGPA OF ID ' || S_ID || ' IS ' || CGPA); 
RETURN CGPA;
END;
/
```
The procedure : 
```sql
CREATE OR REPLACE PROCEDURE CGPA_TREND(SID IN VARCHAR2) IS 
		PREV_CGPA NUMBER(10,4);
		CURR_CGPA NUMBER(10,4);
		NUM NUMBER(10);
		FLAG NUMBER(10);
BEGIN 
		NUM := 0;
		FLAG := 1;
FOR R IN (
SELECT DISTINCT
	SEMESTER,YEAR
FROM 
	TAKES 
WHERE
	ID = SID 
ORDER BY
	YEAR,SEMESTER DESC
)
LOOP
-- 	DBMS_OUTPUT.PUT_LINE(SID||' '||R.SEMESTER||' '||R.YEAR||' '||FIND_CGPA_IN_SEMESTER_OF_YEAR(SID,R.SEMESTER,R.YEAR));
	IF NUM = 0 THEN
		PREV_CGPA :=  FIND_CGPA_IN_SEMESTER_OF_YEAR(SID,R.SEMESTER,R.YEAR);
	ELSIF NUM = 1 THEN 
		CURR_CGPA :=  FIND_CGPA_IN_SEMESTER_OF_YEAR(SID,R.SEMESTER,R.YEAR);
		IF CURR_CGPA >= PREV_CGPA THEN 
			FLAG := 1;
		ELSE 
			FLAG := 2;
		END IF;
		PREV_CGPA := CURR_CGPA;
	ELSE 
		CURR_CGPA :=  FIND_CGPA_IN_SEMESTER_OF_YEAR(SID,R.SEMESTER,R.YEAR);
		IF CURR_CGPA > PREV_CGPA AND FLAG = 2 THEN 
			FLAG := 3;
			EXIT;
		ELSIF CURR_CGPA < PREV_CGPA AND FLAG = 1 THEN  
			FLAG := 3;
			EXIT;
		END IF;
		PREV_CGPA := CURR_CGPA;
	END IF;
	NUM := NUM + 1 ;
END LOOP;
IF FLAG = 1 THEN
	DBMS_OUTPUT.PUT_LINE('CGPA is always increasing');
ELSIF FLAG = 2 THEN 
	DBMS_OUTPUT.PUT_LINE('CGPA is always decreasing');
ELSE 
	DBMS_OUTPUT.PUT_LINE('CGPA is following a zig-zag pattern');
END IF;
END;
/
```


## A2_B2    
1. 
    * Write a PL/SQL function that takes student ID as input and returns TRUE
if that student has attended courses of more than ten departments and
FALSE otherwise.
    * Write a PL/SQL anonymous block that loops over, in descending order of
the student ID, all students who have completed exactly 125 credits. For
each student, using the function you have just written, print whether they
have enrolled in more than ten departments or not. __You cannot declare
any variable in the anonymous block__. 

```sql
-- function
CREATE OR REPLACE FUNCTION CHK_MORE_THAN_10_DEPT(SID VARCHAR2)RETURN VARCHAR2 IS 
CNUM NUMBER;
BEGIN
SELECT COUNT(DISTINCT C.DEPT_NAME) INTO CNUM
FROM TAKES TK NATURAL JOIN COURSE C 
WHERE TK.ID = SID
GROUP BY TK.ID;
IF CNUM > 10 THEN 
	RETURN('TRUE');
ELSE 
	RETURN('FALSE');
END IF;
EXCEPTION 
	WHEN NO_DATA_FOUND THEN
		RETURN ('No rows found.');

	WHEN TOO_MANY_ROWS THEN
	RETURN( 'More than one row found.' );

	WHEN OTHERS THEN
	RETURN( 'Some unknown error occurred.');
END;
/

-- plSql block
BEGIN
FOR R IN (SELECT ID,TOT_CRED
FROM STUDENT
WHERE TOT_CRED = 125
ORDER BY TO_NUMBER(ID) DESC)
LOOP
	IF CHK_MORE_THAN_10_DEPT(R.ID) = 'TRUE' THEN  
		DBMS_OUTPUT.PUT_LINE('Student ID: '||R.ID||' has enrolled in more than ten departments.');
	ELSE 
		DBMS_OUTPUT.PUT_LINE('Student ID: '||R.ID||' has not enrolled in more than ten departments.');
		END IF;
END LOOP;
END;
/
```

2.  Write a PL/SQL stored procedure that takes in Student ID as input and emits the CGPA as output. The CGPA calculation is based on the following formula: $$CGPA = \frac{\sum_{\text{course}}(point\_grade \times credit)}{\sum_{\text{course}}(credit)} $$ Feel free to create additional PL/SQL functions or procedures for your convenience, if needed. Please note that a student may take the same course more than once. In that case, only consider the best grade the student has obtained in that course. For the letter grade and point grade equivalence, use the following table:

| Letter Grade | Point Grade |
|--------------|-------------|
| A+           | 4.00        |
| A            | 3.75        |
| A-           | 3.50        |
| B+           | 3.25        |
| B            | 3.00        |
| B-           | 2.75        |
| C+           | 2.50        |
| C            | 2.25        |
| C-           | 2.15        | 

```sql
CREATE OR REPLACE FUNCTION LETTER_TO_POINT(LET IN VARCHAR2)RETURN DECIMAL IS 
BEGIN 
	IF LET = 'A+' THEN
		RETURN 4.00;
	ELSIF LET = 'A ' THEN 
		RETURN 3.75;
	ELSIF LET = 'A-' THEN 
		RETURN 3.5;
	ELSIF LET = 'B+' THEN 
		RETURN 3.25;
	ELSIF LET = 'B ' THEN 
		RETURN 3.0;
	ELSIF LET = 'B-' THEN 
		RETURN 2.75;
	ELSIF LET = 'C+' THEN 
		RETURN 2.5;
	ELSIF LET = 'C ' THEN 
		RETURN 2.25;
	ELSIF LET = 'C-' THEN 
		RETURN 2.15;
	ELSE 
		RETURN 0;
	END IF;
END;
/

-- required procedure
CREATE OR REPLACE PROCEDURE MY_CGPA(SID IN VARCHAR2,CGPA OUT NUMBER) IS 
TOT_GP NUMBER(10,4);
TOT_CRED NUMBER(10,4);
BEGIN
TOT_GP := 0;
TOT_CRED := 0;
FOR R IN (
SELECT 
	COURSE_ID,
	(SELECT CREDITS FROM COURSE WHERE COURSE.COURSE_ID = "TAKES".COURSE_ID) CREDITS,
	MAX(LETTER_TO_POINT(GRADE)) GRADE 
FROM 
	"TAKES"
WHERE 
	ID = SID
GROUP BY
	COURSE_ID
ORDER BY
	COURSE_ID
)
LOOP 
TOT_GP := TOT_GP + R.CREDITS * R.GRADE;
TOT_CRED := TOT_CRED + R.CREDITS ;
END LOOP;
CGPA := ROUND(TOT_GP / TOT_CRED,2);
EXCEPTION 
	WHEN NO_DATA_FOUND THEN
		CGPA := -1;
	WHEN TOO_MANY_ROWS THEN
		CGPA := -2;
	WHEN OTHERS THEN
		CGPA := -3;
END;
/

-- plSql block for output
DECLARE 
CG NUMBER(10,4);
BEGIN
CG := 0;
FOR R IN  
(SELECT ID,NAME
FROM STUDENT
ORDER BY TO_NUMBER(ID)
FETCH FIRST 20 ROWS ONLY)
LOOP 
	MY_CGPA(R.ID,CG) ;
	DBMS_OUTPUT.PUT_LINE(R.ID||' '||R.NAME||' '||CG);
END LOOP;
END;
/

-- sql of intended output 
-- for verification
SELECT 
	ID,
	ROUND(SUM(CREDITS*GRADE)/SUM(CREDITS),2) CGPA
FROM
(
	SELECT 
		ID,
		(SELECT CREDITS FROM COURSE WHERE COURSE.COURSE_ID = "TAKES".COURSE_ID) CREDITS,
		MAX(LETTER_TO_POINT(GRADE)) GRADE 
	FROM 
		"TAKES"
	GROUP BY
		ID,COURSE_ID
)
GROUP BY
	ID
ORDER BY
	TO_NUMBER(ID);
```

## B1

1. 
    * Write a PL/SQL function that takes student ID as input and returns (+13)
if the student has enrolled in strictly more courses in the fall semesters
than in the spring semesters, (-7) if the student has enrolled in strictly less
courses in the fall semesters than in the spring semesters and 0 otherwise.
    * Write a PL/SQL anonymous block that loops over, in ascending order of
the student ID, all students who are advised by the instructor with ID 90643. For each student, print whether they have more courses in spring
semesters or otherwise. __You cannot use the INTO keyword in the
anonymous block. In addition, in each iteration of the loop, you
have to call your function exactly once.__

```sql
-- function
CREATE OR REPLACE FUNCTION COURSE_COUNT(SID IN VARCHAR2) RETURN NUMBER IS 
	ANS NUMBER;
	SC NUMBER;
	FC NUMBER;
BEGIN 
	SELECT COUNT(*) INTO FC FROM TAKES WHERE SEMESTER = 'Fall' AND ID = SID;
	SELECT COUNT(*) INTO SC FROM TAKES WHERE SEMESTER = 'Spring' AND ID = SID;
	IF FC > SC THEN 
		ANS := 13;
	ELSIF FC < SC THEN 
		ANS := -7;
	ELSE 
		ANS := 0;
	END IF;
	RETURN ANS;
END;
/ 

-- plSqlBlock 
DECLARE 
    VAR NUMBER ;
    BEGIN 
    FOR R IN (SELECT S_ID FROM ADVISOR WHERE I_ID = 90643 ORDER BY TO_NUMBER(S_ID))
    LOOP
    VAR := COURSE_COUNT(R.S_ID);
		IF VAR = 13 THEN 
		    DBMS_OUTPUT.PUT_LINE('Student ID: '||R.S_ID||' has more courses in fall semesters.');
		ELSIF VAR = -7 THEN 
		    DBMS_OUTPUT.PUT_LINE('Student ID: '||R.S_ID||' has more courses in spring semesters.');
		ELSE 
		    DBMS_OUTPUT.PUT_LINE('Student ID: '||R.S_ID||' has the same number of courses in spring and fall semesters.');
		END IF;
    END LOOP;
END;
/

```

2. Let SC be the average number of credits that an instructor teaches per year.
Write a PL/SQL procedure that increases the salary of each instructor by SC%
and prints that information in the following order. However, if the previous
salary is greater than $125000, there will be no increment. __You cannot join
three or more tables together. You cannot write more than two SELECT statements. You can declare only one variable.__ _N.B. : If you (or your IDE) commit, the salary figures will
vary each time you run the procedure_

```sql
-- procedure
CREATE OR REPLACE PROCEDURE INCR 
IS 
	NEWSAL INSTRUCTOR.SALARY%TYPE;
BEGIN
		FOR R IN (
			SELECT
				I.ID,
				I.NAME,
				I.SALARY,
				C.SC 
			FROM
				INSTRUCTOR I,
				(
				SELECT
					TC.ID ID,
					AVG( C.CREDITS ) SC 
				FROM
					TEACHES TC,
					COURSE C 
				WHERE
					TC.COURSE_ID = C.COURSE_ID 
				GROUP BY
					TC.ID 
			) C 
		WHERE
			I.ID = C.ID 
		ORDER BY
			TO_NUMBER( I.ID ) 
		)
		LOOP
			UPDATE INSTRUCTOR 
			SET SALARY = R.SALARY * (1 + (R.SC/100)) 
			WHERE ID = R.ID  AND SALARY <= 125000;
			SELECT SALARY INTO NEWSAL 
			FROM INSTRUCTOR
			WHERE ID = R.ID;
			DBMS_OUTPUT.PUT_LINE ( 'Salary of ' || R.NAME|| ' has increased from ' || ROUND( R.SALARY, 2 ) || ' to ' || ROUND( NEWSAL, 2 ) );
		END LOOP;
        -- Comment out the following rollback to make changes permanent
		ROLLBACK;
END;
/

-- plSql Block for testing
DECLARE
BEGIN 
    INCR();
END;
/
```


## 19_A1_B2

```sql
DECLARE
	COUNTER NUMBER;
	NEWSAL DECIMAL(10,2);
BEGIN
	COUNTER := 0;
	FOR R IN (
	SELECT DISTINCT
E.EMPLOYEE_ID,E.SALARY,ROUND(MONTHS_BETWEEN(SYSDATE,E.HIRE_DATE)/12,0) YEARS,E.DEPARTMENT_ID,E.LAST_NAME
FROM EMPLOYEES E , EMPLOYEES M 
WHERE E.EMPLOYEE_ID = M.MANAGER_ID AND ROUND(MONTHS_BETWEEN(SYSDATE,E.HIRE_DATE)/12,0) >=19
ORDER BY E.EMPLOYEE_ID
	)
LOOP 
	UPDATE EMPLOYEES
	SET SALARY = R.SALARY * 1.15
	WHERE EMPLOYEE_ID = R.EMPLOYEE_ID;
	SELECT SALARY INTO NEWSAL 
	FROM EMPLOYEES WHERE EMPLOYEE_ID = R.EMPLOYEE_ID;
DBMS_OUTPUT.PUT_LINE(R.LAST_NAME||' '||R.DEPARTMENT_ID||' '||R.SALARY||' '||ROUND(NEWSAL,2));
COUNTER := COUNTER + 1;
END LOOP;
DBMS_OUTPUT.PUT_LINE(COUNTER);
ROLLBACK;
END;
/

SELECT DISTINCT
E.EMPLOYEE_ID,E.SALARY,ROUND(MONTHS_BETWEEN(SYSDATE,E.HIRE_DATE)/12,0) YEARS
FROM EMPLOYEES E , EMPLOYEES M 
WHERE E.EMPLOYEE_ID = M.MANAGER_ID AND ROUND(MONTHS_BETWEEN(SYSDATE,E.HIRE_DATE)/12,0) >=19
ORDER BY E.EMPLOYEE_ID;

CREATE OR REPLACE PROCEDURE INTERCHANGE_SALARY (EID1 IN VARCHAR2,EID2 IN VARCHAR2) IS
OLDSAL1 DECIMAL(10,2); 
OLDSAL2 DECIMAL(10,2); 
NEWSAL1 DECIMAL(10,2);
NEWSAL2 DECIMAL(10,2);
BEGIN 
	SELECT SALARY INTO OLDSAL1 FROM EMPLOYEES WHERE EMPLOYEE_ID = EID1;
	SELECT SALARY INTO OLDSAL2 FROM EMPLOYEES WHERE EMPLOYEE_ID = EID2;
	UPDATE EMPLOYEES 
	SET SALARY = OLDSAL2 
	WHERE EMPLOYEE_ID = EID1;
	UPDATE EMPLOYEES
	SET SALARY = OLDSAL1 
	WHERE EMPLOYEE_ID = EID2;
	SELECT SALARY INTO NEWSAL1 FROM EMPLOYEES WHERE EMPLOYEE_ID = EID1;
	SELECT SALARY INTO NEWSAL2 FROM EMPLOYEES WHERE EMPLOYEE_ID = EID2;
	DBMS_OUTPUT.PUT_LINE(EID1||' '||OLDSAL1||' '||NEWSAL1);
	DBMS_OUTPUT.PUT_LINE(EID2||' '||OLDSAL2||' '||NEWSAL2);
	ROLLBACK;
EXCEPTION 
	WHEN NO_DATA_FOUND THEN
	DBMS_OUTPUT.PUT_LINE ( 'No employee found.' );
	
	WHEN TOO_MANY_ROWS THEN
	DBMS_OUTPUT.PUT_LINE ( 'More than one employee found.' );
	
	WHEN OTHERS THEN
	DBMS_OUTPUT.PUT_LINE ( 'Some unknown error occurred.' );
END;
/

DECLARE 
BEGIN
	INTERCHANGE_SALARY(100,120);
	INTERCHANGE_SALARY(120,100);
	INTERCHANGE_SALARY(1000,120);
END;
/

CREATE OR REPLACE FUNCTION ACCDEPT
RETURN NUMBER IS 
NUM NUMBER;
BEGIN
NUM := 0;
FOR R IN (
SELECT 
	EMPLOYEE_ID,ROUND(MONTHS_BETWEEN(SYSDATE, HIRE_DATE)/12,2) YEARS 
FROM EMPLOYEES 
WHERE 
DEPARTMENT_ID = (SELECT DEPARTMENT_ID FROM DEPARTMENTS WHERE DEPARTMENT_NAME = 'Accounting') AND 
ROUND(MONTHS_BETWEEN(SYSDATE, HIRE_DATE)/12,2) > 20
)
LOOP 
DBMS_OUTPUT.PUT_LINE(R.EMPLOYEE_ID||' '||R.YEARS);
NUM := NUM + 1;
END LOOP;
RETURN NUM;
END;
/



SELECT 
	EMPLOYEE_ID,ROUND(MONTHS_BETWEEN(SYSDATE, HIRE_DATE)/12,2) YEARS 
FROM EMPLOYEES 
WHERE 
DEPARTMENT_ID = (SELECT DEPARTMENT_ID FROM DEPARTMENTS WHERE DEPARTMENT_NAME = 'Accounting') AND 
ROUND(MONTHS_BETWEEN(SYSDATE, HIRE_DATE)/12,2) > 20;

DECLARE 
BEGIN
	DBMS_OUTPUT.PUT_LINE(ACCDEPT);
END;
/

```

## 19_B1

```sql
CREATE OR REPLACE PROCEDURE MANRANK(N IN NUMBER,M IN DECIMAL,M_ID IN VARCHAR2,R OUT NUMBER) 
IS
	RANK NUMBER;
BEGIN
	RANK := 1;
	FOR L IN (
	SELECT 
	MAN.EMPLOYEE_ID,
	MAN.FIRST_NAME || ' ' || MAN.LAST_NAME FULLNAME,
	ROUND(AVG(E.SALARY),2) AVGSAL 
	FROM EMPLOYEES MAN JOIN EMPLOYEES E 
	ON MAN.EMPLOYEE_ID = E.MANAGER_ID
	GROUP BY MAN.FIRST_NAME || ' ' || MAN.LAST_NAME,MAN.EMPLOYEE_ID
	HAVING ROUND(AVG(E.SALARY),2) < M AND COUNT(E.EMPLOYEE_ID) > N
	ORDER BY AVGSAL
	)
	LOOP 
	IF L.EMPLOYEE_ID = M_ID THEN 
		R := RANK;
	END IF;
	DBMS_OUTPUT.PUT_LINE(L.FULLNAME || ' RANK : ' || RANK);
	RANK := RANK + 1;
	END	LOOP;
EXCEPTION 
	WHEN NO_DATA_FOUND THEN
	DBMS_OUTPUT.PUT_LINE ( 'No rows found.' );
	WHEN OTHERS THEN 
	DBMS_OUTPUT.PUT_LINE ( 'UNKNOWN ERROR' );
END;
/

DECLARE 
R NUMBER;
BEGIN
	MANRANK(1,20000,100,R);
	DBMS_OUTPUT.PUT_LINE(R);
END;
/

CREATE OR REPLACE FUNCTION RANKEMP(EID IN VARCHAR2,M IN VARCHAR2) RETURN NUMBER IS 
	RANK NUMBER; 
	R NUMBER;
BEGIN
	RANK := -1;
	R := 1;
	FOR K IN (
	SELECT EMPLOYEE_ID 
	FROM EMPLOYEES 
	WHERE MANAGER_ID = M
	ORDER BY SALARY DESC
	)
	LOOP
	IF K.EMPLOYEE_ID = EID THEN 
		RANK := R;
	END IF;
	R := R+1;
	END LOOP;
	IF RANK = -1 THEN 
		DBMS_OUTPUT.PUT_LINE('EMPLOYEE NOT UNDER GIVEN MANAGER');
	END IF;
		RETURN RANK;
END;
/


SELECT EMPLOYEE_ID,MANAGER_ID,RANKEMP(EMPLOYEE_ID,MANAGER_ID) RANK
FROM EMPLOYEES
WHERE MANAGER_ID IS NOT NULL
ORDER BY MANAGER_ID,RANK;
```

