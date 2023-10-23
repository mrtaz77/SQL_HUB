# PL_SQL : Solutions 

- [A1](#a1) 
- [A2_B2](#a2_b2) 
- [B1](#b1) 

## A1
1. 
    - Write a PL/SQL function that takes two course IDs as arguments and returns TRUE if the first course is a prerequisite of strictly more courses than the second one and FALSE otherwise.


    
    - For all courses offered by departments in the Lambeau building, find whether they have more prerequisites than any course offered by the departments in the Bronfman building and print the information using a PL/SQL anonymous block. Order your output by the course titles as shown below. __You cannot use the INTO or WHILE keywords__.



2. Write a PL/SQL procedure that takes the Student ID and input and calculates
the CGPA per every semester (e.g., Fall 2003). If the CGPA chronologically
increases (let staying the same be considered an increase) in every semester,
print CGPA is always increasing. If the CGPA chronologically decreases in
every semester, print CGPA is always decreasing. Otherwise print CGPA is
following a zig-zag pattern. You can assume the input student ID will be
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
ORDER BY
	ID,COURSE_ID)
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