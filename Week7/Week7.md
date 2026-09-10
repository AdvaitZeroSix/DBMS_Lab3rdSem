# Lab 7 — PL/SQL

## Name: Advait Saxena
## Roll No: 25109580220

## 1. Write a PL/SQL block to display the GPA of given student.

```sql
CREATE TABLE StudentTable (
    RollNo NUMBER,
    GPA DECIMAL(2,1)
);

INSERT INTO StudentTable (RollNo, GPA) VALUES (1, 5.8);
INSERT INTO StudentTable (RollNo, GPA) VALUES (2, 6.5);
INSERT INTO StudentTable (RollNo, GPA) VALUES (3, 3.4);
INSERT INTO StudentTable (RollNo, GPA) VALUES (4, 7.8);
INSERT INTO StudentTable (RollNo, GPA) VALUES (5, 9.5);

DECLARE
    v_rollno StudentTable.RollNo%TYPE := &RollNo;
    v_gpa    StudentTable.GPA%TYPE;
BEGIN
    SELECT GPA
    INTO v_gpa
    FROM StudentTable
    WHERE RollNo = v_rollno;

    IF v_gpa IS NOT NULL THEN
        DBMS_OUTPUT.PUT_LINE('GPA = ' || v_gpa);
    END IF;
END;
/
```

## 2. Write a PL/SQL block to display the letter grade of given student.

Grade Criteria: 0-4: F; 4-5: E; 5-6: D; 6-7: C; 7-8: B; 8-9: A; 9-10: A+

```sql
DECLARE
    v_rollno StudentTable.RollNo%TYPE := &RollNo;
    v_gpa    StudentTable.GPA%TYPE;
    v_grade  VARCHAR2(2);
BEGIN
    SELECT GPA
    INTO v_gpa
    FROM StudentTable
    WHERE RollNo = v_rollno;

    IF v_gpa < 4 THEN
        v_grade := 'F';
    ELSIF v_gpa < 5 THEN
        v_grade := 'E';
    ELSIF v_gpa < 6 THEN
        v_grade := 'D';
    ELSIF v_gpa < 7 THEN
        v_grade := 'C';
    ELSIF v_gpa < 8 THEN
        v_grade := 'B';
    ELSIF v_gpa < 9 THEN
        v_grade := 'A';
    ELSE
        v_grade := 'A+';
    END IF;

    DBMS_OUTPUT.PUT_LINE('Roll No: ' || v_rollno);
    DBMS_OUTPUT.PUT_LINE('GPA: ' || v_gpa);
    DBMS_OUTPUT.PUT_LINE('Grade: ' || v_grade);
END;
/
```

## 3. Input the date of issue and date of return for a book. Calculate and display the fine with the appropriate message.

Late Period | Fine
--- | ---
7 days | NIL
8-15 days | Rs. 1/day
16-30 days | Rs. 2/day
After 30 days | Rs. 5/day

```sql
DECLARE
    v_issue_date  DATE := TO_DATE('&IssueDate', 'DD-MM-YYYY');
    v_return_date DATE := TO_DATE('&ReturnDate', 'DD-MM-YYYY');
    v_days        NUMBER;
    v_fine        NUMBER := 0;
BEGIN
    v_days := v_return_date - v_issue_date;

    IF v_days <= 7 THEN
        v_fine := 0;
        DBMS_OUTPUT.PUT_LINE('No fine');

    ELSIF v_days <= 15 THEN
        v_fine := (v_days - 7) * 1;
        DBMS_OUTPUT.PUT_LINE('Fine = Rs.' || v_fine);

    ELSIF v_days <= 30 THEN
        v_fine := (8 * 1) + (v_days - 15) * 2;
        DBMS_OUTPUT.PUT_LINE('Fine = Rs.' || v_fine);

    ELSE
        v_fine := (8 * 1) + (15 * 2) + (v_days - 30) * 5;
        DBMS_OUTPUT.PUT_LINE('Fine = Rs.' || v_fine);
    END IF;
END;
/
```

## 4. Write a PL/SQL block to print the letter grade of all the students (RollNo: 1 - 5).

```sql
DECLARE
    v_rollno NUMBER := 1;
    v_gpa    NUMBER;
    v_grade  VARCHAR2(2);
BEGIN
    LOOP
        SELECT GPA
        INTO v_gpa
        FROM StudentTable
        WHERE RollNo = v_rollno;

        IF v_gpa < 4 THEN
            v_grade := 'F';
        ELSIF v_gpa < 5 THEN
            v_grade := 'E';
        ELSIF v_gpa < 6 THEN
            v_grade := 'D';
        ELSIF v_gpa < 7 THEN
            v_grade := 'C';
        ELSIF v_gpa < 8 THEN
            v_grade := 'B';
        ELSIF v_gpa < 9 THEN
            v_grade := 'A';
        ELSE
            v_grade := 'A+';
        END IF;

        DBMS_OUTPUT.PUT_LINE(
            'Roll No: ' || v_rollno ||
            '  GPA: ' || v_gpa ||
            '  Grade: ' || v_grade
        );

        v_rollno := v_rollno + 1;

        EXIT WHEN v_rollno > 5;
    END LOOP;
END;
/
```

## 5. Alter StudentTable by appending an additional column LetterGrade VARCHAR2(2). Then write a PL/SQL block to update the table with letter grade of each student.

```sql
ALTER TABLE StudentTable
ADD LetterGrade VARCHAR2(2);

DECLARE
    v_rollno NUMBER := 1;
    v_gpa    NUMBER;
    v_grade  VARCHAR2(2);
BEGIN
    WHILE v_rollno <= 5 LOOP

        SELECT GPA
        INTO v_gpa
        FROM StudentTable
        WHERE RollNo = v_rollno;

        IF v_gpa < 4 THEN
            v_grade := 'F';
        ELSIF v_gpa < 5 THEN
            v_grade := 'E';
        ELSIF v_gpa < 6 THEN
            v_grade := 'D';
        ELSIF v_gpa < 7 THEN
            v_grade := 'C';
        ELSIF v_gpa < 8 THEN
            v_grade := 'B';
        ELSIF v_gpa < 9 THEN
            v_grade := 'A';
        ELSE
            v_grade := 'A+';
        END IF;

        UPDATE StudentTable
        SET LetterGrade = v_grade
        WHERE RollNo = v_rollno;

        v_rollno := v_rollno + 1;
    END LOOP;

    COMMIT;
END;
/
```

## 6. Write a PL/SQL block to find the student with max. GPA without using aggregate function.

```sql
DECLARE
    v_max_gpa  NUMBER := 0;
    v_max_roll NUMBER := 0;
BEGIN
    FOR i IN 1..5 LOOP

        DECLARE
            v_gpa NUMBER;
        BEGIN
            SELECT GPA
            INTO v_gpa
            FROM StudentTable
            WHERE RollNo = i;

            IF v_gpa > v_max_gpa THEN
                v_max_gpa := v_gpa;
                v_max_roll := i;
            END IF;
        END;

    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Student with maximum GPA:');
    DBMS_OUTPUT.PUT_LINE('Roll No = ' || v_max_roll);
    DBMS_OUTPUT.PUT_LINE('GPA = ' || v_max_gpa);
END;
/
```

## 7. Implement lab exercise 4 using GOTO.

```sql
DECLARE
    v_rollno NUMBER := 1;
    v_gpa    NUMBER;
    v_grade  VARCHAR2(2);
BEGIN

    <<START_LOOP>>

    IF v_rollno > 5 THEN
        GOTO END_PROGRAM;
    END IF;

    SELECT GPA
    INTO v_gpa
    FROM StudentTable
    WHERE RollNo = v_rollno;

    IF v_gpa < 4 THEN
        v_grade := 'F';
    ELSIF v_gpa < 5 THEN
        v_grade := 'E';
    ELSIF v_gpa < 6 THEN
        v_grade := 'D';
    ELSIF v_gpa < 7 THEN
        v_grade := 'C';
    ELSIF v_gpa < 8 THEN
        v_grade := 'B';
    ELSIF v_gpa < 9 THEN
        v_grade := 'A';
    ELSE
        v_grade := 'A+';
    END IF;

    DBMS_OUTPUT.PUT_LINE(
        'Roll No: ' || v_rollno ||
        '  GPA: ' || v_gpa ||
        '  Grade: ' || v_grade
    );

    v_rollno := v_rollno + 1;

    GOTO START_LOOP;

    <<END_PROGRAM>>
    DBMS_OUTPUT.PUT_LINE('Program completed.');

END;
/
```

## 8. Based on the University database schema, write a PL/SQL block to display the details of the Instructor whose name is supplied by the user. Use exceptions to show appropriate error message for the following cases.

a. Multiple instructors with the same name  
b. No instructor for the given name

```sql
SET SERVEROUTPUT ON;

DECLARE
    v_name    instructor.name%TYPE := '&InstructorName';
    v_id      instructor.id%TYPE;
    v_dept    instructor.dept_name%TYPE;
    v_salary  instructor.salary%TYPE;
BEGIN
    SELECT id, dept_name, salary
    INTO v_id, v_dept, v_salary
    FROM instructor
    WHERE name = v_name;

    DBMS_OUTPUT.PUT_LINE('Instructor ID   : ' || v_id);
    DBMS_OUTPUT.PUT_LINE('Instructor Name : ' || v_name);
    DBMS_OUTPUT.PUT_LINE('Department      : ' || v_dept);
    DBMS_OUTPUT.PUT_LINE('Salary          : ' || v_salary);

EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE(
            'Error: Multiple instructors found with the name ' || v_name
        );

    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE(
            'Error: No instructor found with the name ' || v_name
        );
END;
/
```
