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
 
  
2. Find the students who are advised by an instructor who teaches at least 
one course that the student has taken. You cannot use any GROUP 
functions. Order the output according to student name. 
 
 
 
3. Find the names, departments and total credits of students who have 
completed all the prerequisites for 'Image Processing' course of 
'Finance' department. Order the output according to student name.You 
cannot use join in the main query. You must use SUB-QUERY in 
where clause. You can use join in the sub-queries. 
  
4. Find the instructors who have taught courses at least in 5 semesters. 
You cannot use any join operations. 
 
  
 
5. Find the instructors who have taught the same course in consecutive 
years. You must use EXISTS clause. 
 