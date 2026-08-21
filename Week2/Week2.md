# Week 2 – DBMS SQL Plus

## Employee and Department Tables

### 1. Create Employee Table with Constraints

```sql
CREATE TABLE employee (
    EmpNo INT PRIMARY KEY,
    EmpName VARCHAR(50) NOT NULL,
    Gender CHAR(1) NOT NULL CHECK (Gender IN ('M','F')),
    Salary DECIMAL(10,2) NOT NULL,
    Address VARCHAR(100) NOT NULL,
    DNo INT
);
```

### 2. Create Department Table

```sql
CREATE TABLE department (
    DeptNo INT PRIMARY KEY,
    DeptName VARCHAR(20) UNIQUE,
    Location VARCHAR(20)
);
```

### 3. Add Foreign Key to Employee

```sql
ALTER TABLE Employee
ADD FOREIGN KEY (DNo) REFERENCES Department(DeptNo);
```

### 4. Insert Employee and Department Records

```sql
INSERT INTO Department VALUES (1, 'HR', 'Bangalore');
INSERT INTO Department VALUES (2, 'Sales', 'Mumbai');
INSERT INTO Department VALUES (3, 'IT', 'Chennai');

INSERT INTO Employee VALUES (101, 'Advait', 'M', 30000, 'Bangalore', 1);
INSERT INTO Employee VALUES (102, 'Arnav', 'F', 35000, 'Mumbai', 2);
INSERT INTO Employee VALUES (103, 'Achyut', 'M', 40000, 'Chennai', 3);
```

### 5. Insert Records Violating Constraints

```sql
INSERT INTO Department VALUES (1, 'Finance', 'Delhi');
INSERT INTO Department VALUES (4, 'HR', 'Delhi');

INSERT INTO Employee VALUES (104, 'Rahul', 'X', 30000, 'Pune', 1);
INSERT INTO Employee VALUES (105, 'Amit', 'M', 35000, 'Delhi', 10);
```

### 6. Modify/Delete Records Violating Constraints

```sql
UPDATE Department
SET DeptNo = 5
WHERE DeptNo = 1;

DELETE FROM Department
WHERE DeptNo = 1;
```

### 7. Add ON DELETE CASCADE to Foreign Key

```sql
ALTER TABLE Employee
DROP CONSTRAINT SYS_C00279127;

ALTER TABLE Employee
ADD CONSTRAINT FK_Employee_Department
FOREIGN KEY (DNo)
REFERENCES Department(DeptNo)
ON DELETE CASCADE;
```

### 8. Set Default Salary to 10000

```sql
ALTER TABLE Employee
MODIFY Salary DEFAULT 10000;
```

## University Database

### 9. List All Students with Their Department Names

```sql
SELECT name AS name, dept_name AS dept
FROM student;
```

### 10. List All Instructors in CSE Department

```sql
SELECT *
FROM instructor
WHERE dept_name = 'Comp. Sci.';
```

### 11. Find CSE Courses with 3 Credits

```sql
SELECT *
FROM course
WHERE dept_name = 'Comp. Sci.'
AND credits = 3;
```

### 12. Display Courses Registered by a Student

```sql
SELECT c.course_id, c.title
FROM course c
JOIN takes t ON c.course_id = t.course_id
WHERE t.ID = '12345';
```

### 13. List Instructors with Salary Between 40000 and 90000

```sql
SELECT *
FROM instructor
WHERE salary BETWEEN 40000 AND 90000;
```

### 14. Find Instructors Who Have Never Taught a Course

```sql
SELECT i.ID
FROM instructor i
WHERE i.ID NOT IN (SELECT ID FROM teaches);
```

### 15. Find Students Who Attended Classes in Room 303

```sql
SELECT s.name, c.title, sec.year
FROM student s
JOIN takes t ON s.ID = t.ID
JOIN section sec
ON t.course_id = sec.course_id
AND t.sec_id = sec.sec_id
AND t.semester = sec.semester
AND t.year = sec.year
JOIN course c ON sec.course_id = c.course_id
WHERE sec.room_number = '303';
```

## Rename and Tuple Variables

### 16. Students Who Opted for Courses in 2015

```sql
SELECT s.name, c.course_id, c.title AS c_name
FROM student s, takes t, course c
WHERE s.ID = t.ID
AND t.course_id = c.course_id
AND t.year = 2015;
```

### 17. Instructors with Salary Greater Than at Least One CSE Instructor

```sql
SELECT i.name, i.salary AS inst_salary
FROM instructor i
WHERE i.salary > ANY (
    SELECT salary
    FROM instructor
    WHERE dept_name = 'Comp. Sci.'
);
```

### 18. Find Instructors Whose Department Contains 'ch'

```sql
SELECT name
FROM instructor
WHERE dept_name LIKE '%ch%';
```

## Built-in Functions

### 19. Display Student Names and Their Length

```sql
SELECT name, LENGTH(name) AS name_length
FROM student;
```

### 20. Display 3 Characters from the 3rd Position of Department Names

```sql
SELECT dept_name, SUBSTR(dept_name, 3, 3) AS characterser
FROM department;
```

### 21. Display Instructor Names in Upper Case

```sql
SELECT UPPER(name) AS instructor_name
FROM instructor;
```

### 22. Replace NULL Salary with 0

```sql
SELECT name, COALESCE(salary, 0) AS salary
FROM instructor;
```

### 23. Display Salary and Salary/3 Rounded to Nearest Hundred

```sql
SELECT salary, ROUND(salary/3, -2) AS rounded_salary
FROM instructor;
```

## Date of Birth

### Add DOB Column and Insert Values

```sql
ALTER TABLE Employee ADD DOB DATE;

UPDATE Employee
SET DOB = TO_DATE('15-JAN-2004','DD-MON-YYYY')
WHERE EmpNo = 101;

UPDATE Employee
SET DOB = TO_DATE('20-MAR-2003','DD-MON-YYYY')
WHERE EmpNo = 102;

UPDATE Employee
SET DOB = TO_DATE('10-JUL-2002','DD-MON-YYYY')
WHERE EmpNo = 103;

INSERT INTO Employee
VALUES (104, 'Rahul', 'M', 30000, 'Pune', 1,
TO_DATE('05-DEC-2001','DD-MON-YYYY'));
```

### 24. Display Employee Birth Dates

#### DD-MON-YYYY

```sql
SELECT EmpName, TO_CHAR(DOB, 'DD-MON-YYYY') AS Birth_Date
FROM Employee;
```

#### DD-MON-YY

```sql
SELECT EmpName, TO_CHAR(DOB, 'DD-MON-YY') AS Birth_Date
FROM Employee;
```

#### DD-MM-YY

```sql
SELECT EmpName, TO_CHAR(DOB, 'DD-MM-YY') AS Birth_Date
FROM Employee;
```

### 25. Display Employee Birth Year

#### YEAR

```sql
SELECT EmpName, TO_CHAR(DOB, 'YEAR') AS Birth_Year
FROM Employee;
```

#### Year

```sql
SELECT EmpName, TO_CHAR(DOB, 'Year') AS Birth_Year
FROM Employee;
```

#### year

```sql
SELECT EmpName, TO_CHAR(DOB, 'year') AS Birth_Year
FROM Employee;
```
