# Week 1 – DBMS SQL Plus

## Employee Table

### 1. Create Employee Table

```sql
CREATE TABLE EMPLOYEE (
    emp_no NUMBER(10),
    emp_name VARCHAR2(10),
    emp_address VARCHAR2(20)
);
```

### 2. Insert Employee Information

```sql
INSERT INTO EMPLOYEE(emp_no, emp_name, emp_address)
VALUES (5, 'MINTER', 'BIHAR');
```

### 3. Display Names of All Employees

```sql
SELECT EMP_NAME FROM EMPLOYEE;
```

### 4. Display Employees from Manipal

```sql
SELECT * FROM EMPLOYEE
WHERE EMP_ADDRESS = 'MANIPAL';
```

### 5. Add Salary Column

```sql
ALTER TABLE EMPLOYEE
ADD (salary NUMBER(5));
```

### 6. Assign Salary to Employees

```sql
UPDATE EMPLOYEE
SET SALARY = 10000
WHERE EMP_NAME = 'ADVAIT';
```

### 7. View Table Structure

```sql
DESCRIBE EMPLOYEE;
```

### 8. Delete Employees from Mangalore

```sql
DELETE FROM EMPLOYEE
WHERE EMP_ADDRESS = 'MANGALORE';
```

### 9. Rename Employee Table

```sql
RENAME EMPLOYEE TO EMPLOYEE1;
```

### 10. Drop the Table

```sql
DROP TABLE EMPLOYEE1;
```

## Tools Used

* Oracle SQL Plus
* SQL
* Oracle Database
