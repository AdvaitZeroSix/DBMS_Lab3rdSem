# Week 4 – DBMS SQL Plus

## Group By

### 1. Find the Number of Students in Each Course

```sql
SELECT c.course_id, c.title,
       COUNT(t.id) AS number_of_students
FROM course c
LEFT JOIN takes t
ON c.course_id = t.course_id
GROUP BY c.course_id, c.title;
```

### 2. Find Departments Where the Average Number of Students is Greater Than 10

```sql
SELECT dept_name,
       AVG(student_count) AS average_students
FROM (
    SELECT c.course_id,
           c.dept_name,
           COUNT(t.ID) AS student_count
    FROM course c
    LEFT JOIN takes t
    ON c.course_id = t.course_id
    GROUP BY c.course_id, c.dept_name
)
GROUP BY dept_name
HAVING AVG(student_count) > 10;
```

### 3. Find the Total Number of Courses in Each Department

```sql
SELECT dept_name,
       COUNT(course_id) AS total_courses
FROM course
GROUP BY dept_name
ORDER BY dept_name;
```

### 4. Find Departments with Average Salary Greater Than 42000

```sql
SELECT dept_name,
       AVG(salary) AS average_salary
FROM instructor
GROUP BY dept_name
HAVING AVG(salary) > 42000
ORDER BY dept_name;
```

### 5. Find the Enrolment of Each Section Offered in Spring 2009

```sql
SELECT course_id, sec_id,
       COUNT(ID) AS enrolment
FROM takes
WHERE semester = 'Spring'
AND year = 2009
GROUP BY course_id, sec_id
ORDER BY course_id, sec_id;
```

## Ordering the Display of Tuples

### 6. List Courses with Prerequisite Courses in Increasing Order

```sql
SELECT course_id, prereq_id
FROM prereq
ORDER BY course_id ASC;
```

### 7. Display Instructors Sorted by Salary in Decreasing Order

```sql
SELECT *
FROM instructor
ORDER BY salary DESC;
```

## Derived Relations

### 8. Find the Maximum Total Salary Across Departments

```sql
SELECT dept_name, SUM(salary)
FROM instructor
GROUP BY dept_name;
```

### 9. Find the Average Instructor Salary of Departments Where Average Salary is Greater Than 42000

```sql
SELECT AVG(average_salary) AS average_salary
FROM (
    SELECT dept_name,
           AVG(salary) AS average_salary
    FROM instructor
    GROUP BY dept_name
    HAVING AVG(salary) > 42000
);
```

### 10. Find Sections with Maximum Enrolment in Spring 2010

```sql
SELECT course_id,
       sec_id,
       COUNT(ID) AS enrolment
FROM takes
WHERE semester = 'Spring'
AND year = 2010
GROUP BY course_id, sec_id;
```

### 11. Find Instructors Who Teach All Students Belonging to the CSE Department

```sql
SELECT i.name
FROM instructor i
WHERE NOT EXISTS (
    SELECT s.ID
    FROM student s
    WHERE s.dept_name = 'Comp. Sci.'
    AND NOT EXISTS (
        SELECT 1
        FROM teaches te
        JOIN takes t
        ON te.course_id = t.course_id
        AND te.sec_id = t.sec_id
        AND te.semester = t.semester
        AND te.year = t.year
        WHERE te.ID = i.ID
        AND t.ID = s.ID
    )
);
```

### 12. Find Average Salary of Departments with Average Salary Greater Than 50000 and More Than 5 Instructors

```sql
SELECT AVG(average_salary) AS average_salary
FROM (
    SELECT dept_name,
           AVG(salary) AS average_salary,
           COUNT(ID) AS total_instructors
    FROM instructor
    GROUP BY dept_name
    HAVING AVG(salary) > 50000
    AND COUNT(ID) > 5
);
```

## With Clause

### 13. Find All Departments with the Maximum Budget

```sql
WITH max_budget AS (
    SELECT MAX(budget) AS maximum_budget
    FROM department
)
SELECT d.dept_name,
       d.budget
FROM department d, max_budget m
WHERE d.budget = m.maximum_budget;
```

### 14. Find Departments Where Total Salary is Greater Than the Average Total Salary

```sql
WITH dept_salary AS (
    SELECT dept_name,
           SUM(salary) AS total_salary
    FROM instructor
    GROUP BY dept_name
),
average_salary AS (
    SELECT AVG(total_salary) AS avg_total_salary
    FROM dept_salary
)
SELECT d.dept_name,
       d.total_salary
FROM dept_salary d,
     average_salary a
WHERE d.total_salary > a.avg_total_salary
ORDER BY d.total_salary DESC;
```

## Update and Transaction Control

Use `ROLLBACK` and `SAVEPOINT` to undo modifications before `COMMIT`.

### 15. Transfer Students from CSE Department to IT Department

```sql
UPDATE student
SET dept_name = 'Elec. Eng.'
WHERE dept_name = 'Comp. Sci.';
```

### 16. Increase Instructor Salaries

Instructors earning over 100000 receive a 3% raise. All others receive a 5% raise.

```sql
SAVEPOINT before_salary_update;

UPDATE instructor
SET salary =
    CASE
        WHEN salary > 100000 THEN salary * 1.03
        ELSE salary * 1.05
    END;
```
