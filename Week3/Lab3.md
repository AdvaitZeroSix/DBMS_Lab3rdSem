# Lab 3 – SQL Queries on UNIVERSITY Database

## Set Operations

### 1. Courses that ran in Fall 2009 or Spring 2010 — UNION

```sql
SELECT course_id
FROM section
WHERE semester = 'Fall' AND year = 2009

UNION

SELECT course_id
FROM section
WHERE semester = 'Spring' AND year = 2010;
UNION ALL — To retain duplicates
SELECT course_id
FROM section
WHERE semester = 'Fall' AND year = 2009

UNION ALL

SELECT course_id
FROM section
WHERE semester = 'Spring' AND year = 2010;
2. Courses that ran in Fall 2009 and Spring 2010 — INTERSECT
SELECT course_id
FROM section
WHERE semester = 'Fall' AND year = 2009

INTERSECT

SELECT course_id
FROM section
WHERE semester = 'Spring' AND year = 2010;
3. Courses that ran in Fall 2009 but not in Spring 2010 — MINUS
SELECT course_id
FROM section
WHERE semester = 'Fall' AND year = 2009

MINUS

SELECT course_id
FROM section
WHERE semester = 'Spring' AND year = 2010;
Subqueries
4. Course names for which no student registered
SELECT title
FROM course
WHERE course_id NOT IN (
    SELECT course_id
    FROM takes
);
Safer version using NOT EXISTS
SELECT title
FROM course c
WHERE NOT EXISTS (
    SELECT 1
    FROM takes t
    WHERE t.course_id = c.course_id
);
5. Courses offered in both Fall 2009 and Spring 2010 — IN
SELECT course_id
FROM section
WHERE semester = 'Fall'
  AND year = 2009
  AND course_id IN (
      SELECT course_id
      FROM section
      WHERE semester = 'Spring'
        AND year = 2010
  );
6. Total number of students who have taken courses taught by instructor 10101
SELECT COUNT(DISTINCT t.ID) AS total_students
FROM takes t
JOIN teaches te
  ON t.course_id = te.course_id
 AND t.sec_id = te.sec_id
 AND t.semester = te.semester
 AND t.year = te.year
WHERE te.ID = '10101';
7. Courses offered in Fall 2009 but not in Spring 2010 — NOT IN
SELECT course_id
FROM section
WHERE semester = 'Fall'
  AND year = 2009
  AND course_id NOT IN (
      SELECT course_id
      FROM section
      WHERE semester = 'Spring'
        AND year = 2010
  );
8. Students whose name is the same as an instructor's name
SELECT name
FROM student
WHERE name IN (
    SELECT name
    FROM instructor
);
Set Comparison — SOME / ALL
9. Instructors whose salary is greater than some Biology instructor
SELECT name
FROM instructor
WHERE salary > SOME (
    SELECT salary
    FROM instructor
    WHERE dept_name = 'Biology'
);
10. Instructors whose salary is greater than all Biology instructors
SELECT name
FROM instructor
WHERE salary > ALL (
    SELECT salary
    FROM instructor
    WHERE dept_name = 'Biology'
);
11. Departments having the highest average salary
SELECT dept_name
FROM instructor
GROUP BY dept_name
HAVING AVG(salary) >= ALL (
    SELECT AVG(salary)
    FROM instructor
    GROUP BY dept_name
);
12. Departments whose budget is less than the average salary of all instructors
SELECT dept_name
FROM department
WHERE budget < (
    SELECT AVG(salary)
    FROM instructor
);
Test for Empty Relations — EXISTS / NOT EXISTS
13. Courses taught in both Fall 2009 and Spring 2010 — EXISTS
SELECT DISTINCT s1.course_id
FROM section s1
WHERE s1.semester = 'Fall'
  AND s1.year = 2009
  AND EXISTS (
      SELECT 1
      FROM section s2
      WHERE s2.course_id = s1.course_id
        AND s2.semester = 'Spring'
        AND s2.year = 2010
  );
14. Students who have taken all courses offered by Biology
SELECT s.ID, s.name
FROM student s
WHERE NOT EXISTS (
    SELECT c.course_id
    FROM course c
    WHERE c.dept_name = 'Biology'
      AND NOT EXISTS (
          SELECT 1
          FROM takes t
          WHERE t.ID = s.ID
            AND t.course_id = c.course_id
      )
);
Test for Absence of Duplicate Tuples
15. Courses offered at most once in 2009
SELECT course_id
FROM section
WHERE year = 2009
GROUP BY course_id
HAVING COUNT(*) <= 1;
16. Students who have taken at least two courses offered by the CSE department

In the supplied database, the department name is Comp. Sci..

SELECT s.ID, s.name
FROM student s
WHERE (
    SELECT COUNT(DISTINCT t.course_id)
    FROM takes t
    JOIN course c
      ON t.course_id = c.course_id
    WHERE t.ID = s.ID
      AND c.dept_name = 'Comp. Sci.'
) >= 2;
Subqueries in the FROM Clause
17. Average instructor salary for departments whose average salary is greater than 42000
SELECT dept_name, avg_salary
FROM (
    SELECT dept_name, AVG(salary) AS avg_salary
    FROM instructor
    GROUP BY dept_name
)
WHERE avg_salary > 42000;
Views
18. Create all_courses view for Physics courses offered in Fall 2009
CREATE VIEW all_courses AS
SELECT c.course_id,
       c.title,
       s.sec_id,
       s.semester,
       s.year,
       s.building,
       s.room_number
FROM section s
JOIN course c
  ON s.course_id = c.course_id
WHERE c.dept_name = 'Physics'
  AND s.semester = 'Fall'
  AND s.year = 2009;
19. Select all courses from all_courses
SELECT *
FROM all_courses;
20. Create department_total_salary view
CREATE VIEW department_total_salary AS
SELECT dept_name,
       SUM(salary) AS total_salary
FROM instructor
GROUP BY dept_name;
Display the contents of the view
SELECT *
FROM department_total_salary;
