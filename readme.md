# PLSQL-Collection-Records-Nursery-Admission

This project demonstrates a PL/SQL solution for evaluating nursery school admission eligibility based on a child's age and health status. It leverages Oracle Database features such as **Nested Tables** to store multiple symptoms and a **Stored Procedure** to apply business rules.

---

## Project Structure & Code Explanation

The following sections detail the code execution steps shown in the project screenshots.

---

### #1. Database Schema Setup

**What this represents**: Defining custom data types and the main storage table for children's health records.

Before storing data, we create a collection type to manage multiple symptoms per child.

```sql
-- Create a collection type to store a list of symptoms
CREATE OR REPLACE TYPE t_symptoms AS TABLE OF VARCHAR2(40);
/

-- Create the children table
-- The 'current_symptoms' column uses the custom 't_symptoms' type (Nested Table)
CREATE TABLE children (
    child_id NUMBER,
    child_name VARCHAR2(50),
    health_status VARCHAR2(20),
    current_symptoms t_symptoms
) NESTED TABLE current_symptoms STORE AS symptoms_table;
/
```

---

### #2. Data Insertion

**What this represents**: Populating the database with test children, including both healthy and sick cases.

```sql
-- Insert sample records
INSERT INTO children VALUES (01, 'Regis', 'healthy', t_symptoms());
INSERT INTO children VALUES (02, 'Ikaze', 'sick', t_symptoms('fever', 'cough'));
INSERT INTO children VALUES (03, 'Chris', 'healthy', t_symptoms());
INSERT INTO children VALUES (04, 'Ivan', 'sick', t_symptoms('runny nose'));
INSERT INTO children VALUES (05, 'Gisa', 'sick', t_symptoms('stomach ache'));
```

---

### #3. Stored Procedure: `check_nursery_admission`

**What this represents**: Core logic to determine if a child may attend nursery based on **minimum age (3 years)** and **health status**.

```sql
CREATE OR REPLACE PROCEDURE check_nursery_admission(
    p_child_id IN NUMBER,
    p_age IN NUMBER
) IS
    c_min_age CONSTANT NUMBER := 3;
    v_child children%ROWTYPE;
BEGIN
    SELECT * INTO v_child FROM children WHERE child_id = p_child_id;

    IF p_age < c_min_age THEN
        DBMS_OUTPUT.PUT_LINE(v_child.child_name || ' is too young for nursery school.');
    ELSIF v_child.health_status = 'sick' THEN
        DBMS_OUTPUT.PUT_LINE(v_child.child_name || ' cannot attend school today, they are unwell.');
        FOR i IN 1..v_child.current_symptoms.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE('- ' || v_child.current_symptoms(i));
        END LOOP;
    ELSE
        DBMS_OUTPUT.PUT_LINE('Success: ' || v_child.child_name || ' is allowed to attend nursery school today!');
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Child with ID ' || p_child_id || ' not found.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An unexpected error occurred.');
END;
/
```

---

### #4. Testing & Results

> **Note**: Run `SET SERVEROUTPUT ON` before testing to see output.

#### View all records:
```sql
SELECT * FROM children;
```

---

#### **Test Case 1: Too Young (Regis, age 2)**
```sql
BEGIN check_nursery_admission(1, 2); END;
```
**Output**:  
`Regis is too young for nursery school.`

---

#### **Test Case 2: Sick Child (Ikaze, age 4)**
```sql
BEGIN check_nursery_admission(2, 4); END;
```
**Output**:  
```
Ikaze cannot attend school today, they are unwell.
- fever
- cough
```

---

#### **Test Case 3: Eligible Child (Chris, age 4)**
```sql
BEGIN check_nursery_admission(3, 4); END;
```
**Output**:  
`Success: Chris is allowed to attend nursery school today!`

---

## About

This project demonstrates a PL/SQL solution for managing nursery school admission eligibility using Oracle collections and procedural logic.

---
