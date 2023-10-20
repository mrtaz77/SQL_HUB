# ADVANCED SQL : Solutions 

- [A1](#a1) 
- [A2_B2](#a2_b2) 
- [B1](#b1)

## A1
1. For each instructor, print their ID, name, department name, and the number of
other instructors who have at least one building in common where they teach,
along with the number of such building(s), and sort by the number of instructors.
You cannot use JOIN in the main query.
2. Find the advisors who advise the highest number of students along with the total
credits they oer. You must write at least one subquery to do this.
3. Retrieve the students who have taken more non-departmental courses than
departmental ones. You cannot use JOIN in the main query.
4. For each department, find out the average number of credit hours spent by its
students for its departmental courses. The average should be calculated by
dividing the total number of credits (departmental courses only) taken by the
students of that department divided by the number of students who have taken
departmental courses. You cannot use JOIN in the main query.
5. Retrieve the students who have taken at least two courses where one is covered by their 
advisors and another is oered by some other teacher of their own department. You cannot use JOIN 
or GROUP_BY in the main query.

## A2_B2
1. Find the courses that have the highest enrollment count in each semester. Order the
output according to year and semester.

```sql

```


2. Find the distinct courses taught by instructors with at least 100 students receiving a
grade of *A+*. You cannot use join in the main query. You must use SUB-QUERY in
the where clause. You can use join in the sub-queries.
3. Find the distinct course IDs and titles that appear in either the "Comp. Sci."
department or have a prerequisite in the "Math" department. You must use set
operations. You cannot use any join operations. Order the output according to the
department name.
4. Find the courses that have conflicting time slots (overlapping schedules).
5. Find the instructors who have taught courses with enrollment counts higher than the
average enrollment count of all courses in each semester. Your column names are
expected to match the provided output’s column names.


## B1
1. Combine the list of courses that have been taken by students in either 
the "Comp. Sci." or "Math" departments, along with the list of courses 
that have not been taken by any student in the "Elec. Eng." 
department. Order the output according to course title. You cannot 
use any join operations. 

```sql
SELECT
    TITLE COURSE_TITLE 
FROM
    COURSE 
WHERE
    COURSE_ID IN ( SELECT COURSE_ID FROM TAKES WHERE ID IN ( SELECT ID FROM STUDENT WHERE DEPT_NAME IN ( 'Comp. Sci.', 'Math' ) ) ) UNION
SELECT
    TITLE COURSE_TITLE 
FROM
    COURSE 
WHERE
    COURSE_ID NOT IN ( SELECT COURSE_ID FROM TAKES WHERE ID IN ( SELECT ID FROM STUDENT WHERE DEPT_NAME = 'Elec. Eng.' ) );
```

2. Find the students who are advised by an instructor who teaches at least 
one course that the student has taken. You cannot use any GROUP 
functions. Order the output according to student name. 

```sql
SELECT
	NAME STUDENT_NAME 
FROM
	STUDENT 
WHERE
	ID IN ( SELECT S_ID FROM ADVISOR ) 
	AND EXISTS (
	SELECT
		COURSE_ID 
	FROM
		TAKES 
	WHERE
		TAKES.ID = STUDENT.ID 
		AND TAKES.COURSE_ID IN ( SELECT COURSE_ID FROM TEACHES WHERE ID = ( ( SELECT I_ID FROM ADVISOR WHERE S_ID = STUDENT.ID ) ) ) 
	) 
ORDER BY
	NAME;
```

3. Find the names, departments and total credits of students who have 
completed all the prerequisites for 'Image Processing' course of 
'Finance' department. Order the output according to student name.You 
cannot use join in the main query. You must use SUB-QUERY in 
where clause. You can use join in the sub-queries. 

```sql
SELECT
	NAME STUDENT_NAME,
	DEPT_NAME,
	TOT_CRED TOTAL_CRED 
FROM
	STUDENT 
WHERE
	ID IN (
	SELECT
		ID 
	FROM
		TAKES 
	WHERE
		COURSE_ID = ( SELECT PREREQ_ID FROM PREREQ WHERE COURSE_ID = ( SELECT COURSE_ID FROM COURSE WHERE TITLE = 'Image Processing' AND DEPT_NAME = 'Finance' ) ) 
	) 
ORDER BY
	NAME;
```

4. Find the instructors who have taught courses at least in 5 semesters. 
You cannot use any join operations. 

```sql
SELECT
	ID,
	NAME INSTRUCTOR_NAME 
FROM
	INSTRUCTOR 
WHERE
	( SELECT COUNT( DISTINCT CONCAT( SEMESTER, YEAR ) ) FROM TEACHES WHERE TEACHES.ID = INSTRUCTOR.ID ) >= 5 
ORDER BY
	NAME;
```

5. Find the instructors who have taught the same course in consecutive 
years. You must use EXISTS clause. 

```sql
SELECT
	T3.NAME INSTRUCTOR_NAME,
	D.COURSE_TITLE,
	D.YEAR 
FROM
	INSTRUCTOR T3,
	(
	SELECT
		T1.ID,
		T1.COURSE_ID,
		( SELECT TITLE FROM COURSE WHERE COURSE.COURSE_ID = T1.COURSE_ID ) COURSE_TITLE,
		T1.YEAR 
	FROM
		TEACHES T1 
	WHERE
		EXISTS ( SELECT T2.COURSE_ID FROM TEACHES T2 WHERE T2.COURSE_ID = T1.COURSE_ID AND T2.YEAR = T1.YEAR + 1 ) 
	) D 
WHERE
	D.ID = T3.ID 
	ORDER BY
		D.YEAR;
```