-- ============================================================================
-- Lab 8
-- Name: Advait Saxena
-- ID: 251090580220
-- ============================================================================


-- ============================================================================
-- 1. Salary Raise and Audit Logging
-- ============================================================================

CREATE TABLE salary_raise (
    Instructor_Id VARCHAR2(5),
    Raise_date    DATE,
    Raise_amt     NUMBER(8,2),
    FOREIGN KEY (Instructor_Id) REFERENCES instructor(ID) ON DELETE CASCADE
);

SET SERVEROUTPUT ON;

DECLARE
    v_dept_name instructor.dept_name%TYPE := 'Comp. Sci.'; 
    v_raise_amt NUMBER(8,2);
    
    CURSOR c_instructors IS
        SELECT ID, salary 
        FROM instructor 
        WHERE dept_name = v_dept_name;
BEGIN
    FOR r_inst IN c_instructors LOOP
        v_raise_amt := r_inst.salary * 0.05;
        
        UPDATE instructor
        SET salary = salary + v_raise_amt
        WHERE ID = r_inst.ID;
        
        INSERT INTO salary_raise (Instructor_Id, Raise_date, Raise_amt)
        VALUES (r_inst.ID, SYSDATE, v_raise_amt);
    END LOOP;
    
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Successfully updated salaries and logged raises for department: ' || v_dept_name);
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END;
/


-- ============================================================================
-- 2. Display Top 10 Students with Lowest Total Credits
-- ============================================================================

SET SERVEROUTPUT ON;

DECLARE
    CURSOR c_low_cred_students IS
        SELECT ID, name, dept_name, tot_cred
        FROM student
        ORDER BY tot_cred ASC
        FETCH FIRST 10 ROWS ONLY;
BEGIN
    DBMS_OUTPUT.PUT_LINE(RPAD('ID', 8) || RPAD('Name', 20) || RPAD('Dept Name', 20) || RPAD('Tot Cred', 10));
    DBMS_OUTPUT.PUT_LINE(RPAD('-', 7, '-') || ' ' || RPAD('-', 19, '-') || ' ' || RPAD('-', 19, '-') || ' ' || RPAD('-', 9, '-'));
    
    FOR r_student IN c_low_cred_students LOOP
        DBMS_OUTPUT.PUT_LINE(
            RPAD(r_student.ID, 8) || 
            RPAD(r_student.name, 20) || 
            RPAD(r_student.dept_name, 20) || 
            RPAD(TO_CHAR(r_student.tot_cred), 10)
        );
    END LOOP;
END;
/


-- ============================================================================
-- 3. Course Details and Registered Student Count
-- ============================================================================

SET SERVEROUTPUT ON;

DECLARE
    CURSOR c_course_details IS
        SELECT 
            c.course_id, 
            c.title, 
            c.dept_name, 
            c.credits, 
            NVL(i.name, 'TBA') AS instructor_name, 
            NVL(s.building, 'N/A') AS building, 
            NVL(s.room_number, 'N/A') AS room_number, 
            NVL(s.time_slot_id, 'N/A') AS time_slot_id, 
            COUNT(t.ID) AS tot_student_no
        FROM course c
        JOIN section s ON c.course_id = s.course_id
        LEFT JOIN teaches th ON s.course_id = th.course_id AND s.sec_id = th.sec_id AND s.semester = th.semester AND s.year = th.year
        LEFT JOIN instructor i ON th.ID = i.ID
        LEFT JOIN takes t ON s.course_id = t.course_id AND s.sec_id = t.sec_id AND s.semester = t.semester AND s.year = t.year
        GROUP BY c.course_id, c.title, c.dept_name, c.credits, i.name, s.building, s.room_number, s.time_slot_id;
BEGIN
    DBMS_OUTPUT.PUT_LINE(RPAD('Course ID', 10) || RPAD('Title', 28) || RPAD('Instructor', 15) || RPAD('Students', 10));
    DBMS_OUTPUT.PUT_LINE(RPAD('-', 9, '-') || '  ' || RPAD('-', 26, '-') || '  ' || RPAD('-', 13, '-') || '  ' || RPAD('-', 8, '-'));
    
    FOR r IN c_course_details LOOP
        DBMS_OUTPUT.PUT_LINE(
            RPAD(r.course_id, 10) || 
            RPAD(r.title, 28) || 
            RPAD(r.instructor_name, 15) || 
            RPAD(TO_CHAR(r.tot_student_no), 10)
        );
    END LOOP;
END;
/


-- ============================================================================
-- 4. Deregister Students with Less Than 30 Credits from Course CS-101
-- ============================================================================

SET SERVEROUTPUT ON;

BEGIN
    DELETE FROM takes
    WHERE course_id = 'CS-101'
      AND ID IN (
          SELECT ID 
          FROM student 
          WHERE tot_cred < 30
      );
    
    COMMIT;
END;
/


-- ============================================================================
-- 5. Grade Mapping based on GPA
-- ============================================================================

SET SERVEROUTPUT ON;

DECLARE
    CURSOR c_students IS
        SELECT ROLLNO, GPA
        FROM StudentTable;
    v_grade VARCHAR2(2);
BEGIN
    FOR r IN c_students LOOP
        IF r.GPA >= 9.0 THEN
            v_grade := 'A+';
        ELSIF r.GPA >= 8.0 THEN
            v_grade := 'A';
        ELSIF r.GPA >= 7.0 THEN
            v_grade := 'B';
        ELSIF r.GPA >= 6.0 THEN
            v_grade := 'C';
        ELSIF r.GPA >= 5.0 THEN
            v_grade := 'D';
        ELSE
            v_grade := 'F';
        END IF;

        UPDATE StudentTable
        SET LETTERGRADE = v_grade
        WHERE ROLLNO = r.ROLLNO;
    END LOOP;
    COMMIT;
END;
/


-- ============================================================================
-- 6. List Instructors Teaching a Specified Course
-- ============================================================================

SET SERVEROUTPUT ON;

DECLARE
    v_course_id VARCHAR2(8) := 'CS-101';
    
    CURSOR c_instructors IS
        SELECT DISTINCT i.ID, i.name, i.dept_name
        FROM instructor i
        JOIN teaches t ON i.ID = t.ID
        WHERE t.course_id = v_course_id;
BEGIN
    DBMS_OUTPUT.PUT_LINE(RPAD('Instructor ID', 15) || RPAD('Name', 20) || RPAD('Department', 20));
    DBMS_OUTPUT.PUT_LINE(RPAD('-', 14, '-') || ' ' || RPAD('-', 19, '-') || ' ' || RPAD('-', 19, '-'));
    
    FOR r IN c_instructors LOOP
        DBMS_OUTPUT.PUT_LINE(
            RPAD(r.ID, 15) || 
            RPAD(r.name, 20) || 
            RPAD(r.dept_name, 20)
        );
    END LOOP;
END;
/


-- ============================================================================
-- 7. List Students Registered for Courses Taught by Their Advisor
-- ============================================================================

SET SERVEROUTPUT ON;

DECLARE
    CURSOR c_advisor_students IS
        SELECT DISTINCT s.ID AS student_id, s.name AS student_name, c.course_id, c.title
        FROM student s
        JOIN advisor a ON s.ID = a.s_ID
        JOIN takes tk ON s.ID = tk.ID
        JOIN teaches th ON a.i_ID = th.ID 
            AND tk.course_id = th.course_id 
            AND tk.sec_id = th.sec_id 
            AND tk.semester = th.semester 
            AND tk.year = th.year
        JOIN course c ON tk.course_id = c.course_id;
BEGIN
    DBMS_OUTPUT.PUT_LINE(RPAD('Student ID', 15) || RPAD('Student Name', 20) || RPAD('Course ID', 12) || RPAD('Title', 30));
    DBMS_OUTPUT.PUT_LINE(RPAD('-', 14, '-') || ' ' || RPAD('-', 19, '-') || ' ' || RPAD('-', 11, '-') || ' ' || RPAD('-', 29, '-'));
    
    FOR r IN c_advisor_students LOOP
        DBMS_OUTPUT.PUT_LINE(
            RPAD(r.student_id, 15) || 
            RPAD(r.student_name, 20) || 
            RPAD(r.course_id, 12) || 
            RPAD(r.title, 30)
        );
    END LOOP;
END;
/
