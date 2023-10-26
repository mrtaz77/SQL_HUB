1. Display a list of all instructors, showing each instructor’s ID and the number of sections taught. Make sure to show the number of sections as 0 for instructors who have not taught any section. __Your query should use an
outer join, and should not use subqueries.__

```sql
SELECT
	I.ID,
	I.NAME,
	COUNT(DISTINCT TC.SEC_ID) NUM
FROM
	INSTRUCTOR I LEFT OUTER JOIN TEACHES TC
ON I.ID=TC.ID
GROUP BY
	I.ID,
	I.NAME
ORDER BY
	TO_NUMBER(ID);
```

2. Display the list of all departments, with the total number of instructors in each department, without using subqueries. Make sure to show departments that have no instructors, and list those departments with an instruc-
tor count of zero.
```sql
SELECT
	DEPT_NAME,
	COUNT(ID) NUM 
FROM
	DEPARTMENT LEFT JOIN INSTRUCTOR USING(DEPT_NAME)
GROUP BY
	DEPT_NAME
ORDER BY
	DEPT_NAME;
```

3. Display the list of all course sections offered in Spring 2008, along with the ID and name of each instructor teaching the section. If a section has more than one instructor, that section should appear as many times in the result as it has instructors. If a section does not have any instructor,
it should still appear in the result with the instructor name set to "-
".
```sql
SELECT DISTINCT
	S.SEC_ID,
	S.COURSE_ID,
	C.TITLE COURSE_TITLE,
	NVL(T.ID,'-') ID,
	NVL(I.NAME,'-') NAME
FROM
	SECTION S
	LEFT JOIN COURSE C ON S.COURSE_ID = C.COURSE_ID 
	LEFT OUTER JOIN 
	TEACHES T ON T.COURSE_ID = S.COURSE_ID
	LEFT OUTER JOIN 
	INSTRUCTOR I ON T.ID = I.ID
WHERE
	UPPER( S.SEMESTER ) = 'SPRING' 
	AND S.YEAR = '2008'
ORDER BY
	S.SEC_ID,TO_NUMBER(S.COURSE_ID);
```

4. Display the list of students who have taken at least one course in 'Comp Sci.' department
```sql
SELECT DISTINCT
	S.ID,
	S.NAME 
FROM
	STUDENT S 
WHERE
	EXISTS ( SELECT COURSE_ID FROM TAKES WHERE ID = S.ID AND COURSE_ID IN ( SELECT COURSE_ID FROM COURSE WHERE DEPT_NAME = 'Comp. Sci.' ) ) 
ORDER BY
	TO_NUMBER(S.ID);
```

5. List the id and name of students whose advisor is in physics department. 
```sql
SELECT
	ID,
	NAME 
FROM
	STUDENT 
WHERE
	ID IN ( SELECT S_ID FROM ADVISOR WHERE I_ID IN ( SELECT ID FROM INSTRUCTOR WHERE UPPER( DEPT_NAME ) = 'PHYSICS' ) ) 
ORDER BY
	NAME;
```

6. Students who have taken at least 3 courses
```sql
SELECT
	S.ID,
	S.NAME 
FROM
	STUDENT S
WHERE
	EXISTS (
	SELECT	
		TK.COURSE_ID
	FROM
		TAKES TK 
		WHERE TK.ID=S.ID
	GROUP BY
		TK.COURSE_ID
	HAVING COUNT(*) >= 3
	)
	ORDER BY
	S.NAME;
```

7. List of all students who have taken more than 3 courses more than once
```sql
SELECT
	ID,
	NAME
FROM
    STUDENT
WHERE
	(SELECT 
	    COUNT(DISTINCT COURSE_ID)
	FROM
	(
        SELECT
		    TK2.ID,
		    TK2.COURSE_ID
        FROM 
            TAKES TK2 
        WHERE
            STUDENT.ID=TK2.ID
        GROUP BY
            TK2.ID,
            TK2.COURSE_ID
        HAVING 
            COUNT(*) >= 2
    )
    ) > 3;
```