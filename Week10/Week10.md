# Week 10: Database Triggers

## Row Triggers

### 1. Log Changes Made to the Takes Table

```sql
CREATE TABLE log_change_Takes (
    time_of_change TIMESTAMP,
    ID VARCHAR2(5),
    courseid VARCHAR2(8),
    sec_id VARCHAR2(8),
    semester VARCHAR2(6),
    year NUMBER,
    grade VARCHAR2(2)
);

CREATE OR REPLACE TRIGGER takes_log
AFTER INSERT OR UPDATE OR DELETE ON takes
FOR EACH ROW
BEGIN
    INSERT INTO log_change_Takes
    VALUES (
        SYSTIMESTAMP,
        :OLD.ID, :OLD.course_id, :OLD.sec_id,
        :OLD.semester, :OLD.year, :OLD.grade
    );
END;
/
```

### 2. Store Old Instructor Values When Salary Is Updated

```sql
CREATE TABLE Old_Data_Instructor (
    ID VARCHAR2(5),
    name VARCHAR2(20),
    dept_name VARCHAR2(20),
    salary NUMBER(8,2)
);

CREATE OR REPLACE TRIGGER instructor_log
BEFORE UPDATE OF salary ON instructor
FOR EACH ROW
BEGIN
    INSERT INTO Old_Data_Instructor
    VALUES (:OLD.ID, :OLD.name, :OLD.dept_name, :OLD.salary);
END;
/
```

## Database Triggers

### 3. Validate Instructor Name and Salary

```sql
CREATE OR REPLACE TRIGGER validate_instructor
BEFORE INSERT OR UPDATE ON instructor
FOR EACH ROW
DECLARE
    v_budget department.budget%TYPE;
BEGIN
    IF NOT REGEXP_LIKE(:NEW.name, '^[A-Za-z ]+$') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Invalid name');
    END IF;

    IF :NEW.salary <= 0 THEN
        RAISE_APPLICATION_ERROR(-20002, 'Invalid salary');
    END IF;

    SELECT budget INTO v_budget
    FROM department
    WHERE dept_name = :NEW.dept_name;

    IF :NEW.salary > v_budget THEN
        RAISE_APPLICATION_ERROR(-20003, 'Salary exceeds budget');
    END IF;
END;
/
```

### 4. Audit Deleted or Updated Client Records

```sql
CREATE TABLE auditclient (
    client_no VARCHAR2(10),
    name VARCHAR2(30),
    bal_due NUMBER(10,2),
    operation VARCHAR2(10),
    userid VARCHAR2(30),
    opdate DATE
);

CREATE OR REPLACE TRIGGER client_audit
BEFORE UPDATE OR DELETE ON Client_master
FOR EACH ROW
BEGIN
    INSERT INTO auditclient
    VALUES (
        :OLD.client_no, :OLD.name, :OLD.bal_due,
        CASE WHEN UPDATING THEN 'UPDATE' ELSE 'DELETE' END,
        USER, SYSDATE
    );
END;
/
```

## Instead Of Triggers

### 5. Create Advisor_Student View and INSTEAD OF DELETE Trigger

```sql
CREATE OR REPLACE VIEW Advisor_Student AS
SELECT s.ID AS student_id, s.name AS student_name,
       i.ID AS instructor_id, i.name AS instructor_name
FROM student s
JOIN advisor a ON s.ID = a.s_ID
JOIN instructor i ON a.i_ID = i.ID;
```

```sql
CREATE OR REPLACE TRIGGER delete_advisor
INSTEAD OF DELETE ON Advisor_Student
FOR EACH ROW
BEGIN
    DELETE FROM advisor
    WHERE s_ID = :OLD.student_id
      AND i_ID = :OLD.instructor_id;
END;
/
```
