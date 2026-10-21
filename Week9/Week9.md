# DBMS Lab Manual: PL/SQL

## Procedures

### 1. Procedure to Display a Message

```sql
SET SERVEROUTPUT ON;

CREATE OR REPLACE PROCEDURE good_day AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Good Day to You');
END;
/

BEGIN
    good_day;
END;
/
```

### 2. Display Instructors and Courses of a Department

```sql
CREATE OR REPLACE PROCEDURE dept_details(
    d IN VARCHAR2
) AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Instructors:');

    FOR i IN (SELECT name FROM instructor
              WHERE dept_name = d) LOOP
        DBMS_OUTPUT.PUT_LINE(i.name);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Courses:');

    FOR c IN (SELECT course_id, title FROM course
              WHERE dept_name = d) LOOP
        DBMS_OUTPUT.PUT_LINE(c.course_id || ' ' || c.title);
    END LOOP;
END;
/

BEGIN
    dept_details('Comp. Sci.');
END;
/
```

### 3. Most Popular Course in Each Department

```sql
CREATE OR REPLACE PROCEDURE course_popular(
    d IN VARCHAR2
) AS
    v_title course.title%TYPE;
    v_count NUMBER;
BEGIN
    SELECT title, cnt INTO v_title, v_count
    FROM (
        SELECT c.title, COUNT(t.ID) cnt
        FROM course c LEFT JOIN takes t
        ON c.course_id = t.course_id
        WHERE c.dept_name = d
        GROUP BY c.course_id, c.title
        ORDER BY COUNT(t.ID) DESC
    )
    WHERE ROWNUM = 1;

    DBMS_OUTPUT.PUT_LINE(
        d || ': ' || v_title || ' (' || v_count || ')'
    );
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No courses found');
END;
/

BEGIN
    FOR d IN (SELECT dept_name FROM department) LOOP
        course_popular(d.dept_name);
    END LOOP;
END;
/
```

### 4. Display Students and Courses of a Department

```sql
CREATE OR REPLACE PROCEDURE dept_students(
    d IN VARCHAR2
) AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Students:');

    FOR s IN (SELECT name FROM student
              WHERE dept_name = d) LOOP
        DBMS_OUTPUT.PUT_LINE(s.name);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Courses:');

    FOR c IN (SELECT course_id, title FROM course
              WHERE dept_name = d) LOOP
        DBMS_OUTPUT.PUT_LINE(c.course_id || ' ' || c.title);
    END LOOP;
END;
/

BEGIN
    dept_students('Comp. Sci.');
END;
/
```

## Functions

### 5. Function to Return the Square of a Number

```sql
CREATE OR REPLACE FUNCTION square_num(n NUMBER)
RETURN NUMBER AS
BEGIN
    RETURN n * n;
END;
/

BEGIN
    DBMS_OUTPUT.PUT_LINE('Square = ' || square_num(5));
END;
/
```

### 6. Highest-Paid Instructor in Each Department

```sql
CREATE OR REPLACE FUNCTION department_highest(
    d VARCHAR2
) RETURN VARCHAR2 AS
    v_name instructor.name%TYPE;
BEGIN
    SELECT name INTO v_name
    FROM (
        SELECT name FROM instructor
        WHERE dept_name = d
        ORDER BY salary DESC
    )
    WHERE ROWNUM = 1;

    RETURN v_name;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'No Instructor Found';
END;
/

BEGIN
    FOR d IN (SELECT dept_name FROM department) LOOP
        DBMS_OUTPUT.PUT_LINE(
            d.dept_name || ': ' ||
            department_highest(d.dept_name)
        );
    END LOOP;
END;
/
```

## Packages

### 7. Package with Procedure and Function

**Package Specification**

```sql
CREATE OR REPLACE PACKAGE dept_pkg AS
    PROCEDURE list_instructors(d VARCHAR2);
    FUNCTION max_salary(d VARCHAR2) RETURN NUMBER;
END dept_pkg;
/
```

**Package Body**

```sql
CREATE OR REPLACE PACKAGE BODY dept_pkg AS

    PROCEDURE list_instructors(d VARCHAR2) AS
    BEGIN
        FOR i IN (SELECT name FROM instructor
                  WHERE dept_name = d) LOOP
            DBMS_OUTPUT.PUT_LINE(i.name);
        END LOOP;
    END;

    FUNCTION max_salary(d VARCHAR2) RETURN NUMBER AS
        s NUMBER;
    BEGIN
        SELECT MAX(salary) INTO s FROM instructor
        WHERE dept_name = d;
        RETURN s;
    END;

END dept_pkg;
/
```

**Calling the Package**

```sql
BEGIN
    dept_pkg.list_instructors('Comp. Sci.');

    DBMS_OUTPUT.PUT_LINE(
        'Maximum Salary: ' ||
        dept_pkg.max_salary('Comp. Sci.')
    );
END;
/
```

## Parameter Modes

### 8. Simple Interest, Compound Interest and Total Amount

```sql
CREATE OR REPLACE PROCEDURE interest(
    p IN NUMBER,
    r IN NUMBER,
    t IN NUMBER,
    si OUT NUMBER,
    ci OUT NUMBER,
    total IN OUT NUMBER
) AS
BEGIN
    si := p * r * t / 100;
    ci := p * (POWER(1 + r/100, t) - 1);
    total := p + si;
END;
/

DECLARE
    p NUMBER := 10000;
    r NUMBER := 5;
    t NUMBER := 2;
    si NUMBER;
    ci NUMBER;
    total NUMBER;
BEGIN
    total := p;
    interest(p, r, t, si, ci, total);

    DBMS_OUTPUT.PUT_LINE('Simple Interest: ' || si);
    DBMS_OUTPUT.PUT_LINE('Compound Interest: ' || ci);
    DBMS_OUTPUT.PUT_LINE('Total Amount: ' || total);
END;
/
```

Formulas:

* Simple Interest = P × R × T / 100
* Compound Interest = P × (1 + R/100)^T − P
* Total Amount = P + SI
