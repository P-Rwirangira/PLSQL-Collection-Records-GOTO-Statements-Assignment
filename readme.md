```markdown

\# PLSQL-Collection-Records-Nursery-Admission



This project demonstrates a PL/SQL solution for evaluating nursery school admission eligibility based on a child's age and health status. It leverages Oracle Database features such as \*\*Nested Tables\*\* to store multiple symptoms and a \*\*Stored Procedure\*\* to apply business rules.



---



\## Project Structure \& Code Explanation



The following sections detail the code execution steps shown in the project screenshots.



---



\### #1. Database Schema Setup



\*\*What this represents\*\*: Defining custom data types and the main storage table for children’s health records.



Before storing data, we create a collection type to manage multiple symptoms per child.



```sql

-- Create a collection type to store a list of symptoms

CREATE OR REPLACE TYPE t\_symptoms AS TABLE OF VARCHAR2(40);

/



-- Create the children table

-- The 'current\_symptoms' column uses the custom 't\_symptoms' type (Nested Table)

CREATE TABLE children (

&nbsp;   child\_id NUMBER,

&nbsp;   child\_name VARCHAR2(50),

&nbsp;   health\_status VARCHAR2(20),

&nbsp;   current\_symptoms t\_symptoms

) NESTED TABLE current\_symptoms STORE AS symptoms\_table;

/

```



---



\### #2. Data Insertion



\*\*What this represents\*\*: Populating the database with test children, including both healthy and sick cases.



```sql

-- Insert sample records

INSERT INTO children VALUES (01, 'Regis', 'healthy', t\_symptoms());

INSERT INTO children VALUES (02, 'Ikaze', 'sick', t\_symptoms('fever', 'cough'));

INSERT INTO children VALUES (03, 'Chris', 'healthy', t\_symptoms());

INSERT INTO children VALUES (04, 'Ivan', 'sick', t\_symptoms('runny nose'));

INSERT INTO children VALUES (05, 'Gisa', 'sick', t\_symptoms('stomach ache'));

```



---



\### #3. Stored Procedure: `check\_nursery\_admission`



\*\*What this represents\*\*: Core logic to determine if a child may attend nursery based on \*\*minimum age (3 years)\*\* and \*\*health status\*\*.



```sql

CREATE OR REPLACE PROCEDURE check\_nursery\_admission(

&nbsp;   p\_child\_id IN NUMBER,

&nbsp;   p\_age IN NUMBER

) IS

&nbsp;   c\_min\_age CONSTANT NUMBER := 3;

&nbsp;   v\_child children%ROWTYPE;

BEGIN

&nbsp;   SELECT \* INTO v\_child FROM children WHERE child\_id = p\_child\_id;



&nbsp;   IF p\_age < c\_min\_age THEN

&nbsp;       DBMS\_OUTPUT.PUT\_LINE(v\_child.child\_name || ' is too young for nursery school.');

&nbsp;   ELSIF v\_child.health\_status = 'sick' THEN

&nbsp;       DBMS\_OUTPUT.PUT\_LINE(v\_child.child\_name || ' cannot attend school today, they are unwell.');

&nbsp;       FOR i IN 1..v\_child.current\_symptoms.COUNT LOOP

&nbsp;           DBMS\_OUTPUT.PUT\_LINE('- ' || v\_child.current\_symptoms(i));

&nbsp;       END LOOP;

&nbsp;   ELSE

&nbsp;       DBMS\_OUTPUT.PUT\_LINE('Success: ' || v\_child.child\_name || ' is allowed to attend nursery school today!');

&nbsp;   END IF;



EXCEPTION

&nbsp;   WHEN NO\_DATA\_FOUND THEN

&nbsp;       DBMS\_OUTPUT.PUT\_LINE('Error: Child with ID ' || p\_child\_id || ' not found.');

&nbsp;   WHEN OTHERS THEN

&nbsp;       DBMS\_OUTPUT.PUT\_LINE('An unexpected error occurred.');

END;

/

```



---



\### #4. Testing \& Results



> \*\*Note\*\*: Run `SET SERVEROUTPUT ON` before testing to see output.



\#### View all records:

```sql

SELECT \* FROM children;

```



---



\#### \*\*Test Case 1: Too Young (Regis, age 2)\*\*

```sql

BEGIN check\_nursery\_admission(1, 2); END;

```

\*\*Output\*\*:  

`Regis is too young for nursery school.`



---



\#### \*\*Test Case 2: Sick Child (Ikaze, age 4)\*\*

```sql

BEGIN check\_nursery\_admission(2, 4); END;

```

\*\*Output\*\*:  

```

Ikaze cannot attend school today, they are unwell.

\- fever

\- cough

```



---



\#### \*\*Test Case 3: Eligible Child (Chris, age 4)\*\*

```sql

BEGIN check\_nursery\_admission(3, 4); END;

```

\*\*Output\*\*:  

`Success: Chris is allowed to attend nursery school today!`



---



\## About



This project demonstrates a PL/SQL solution for managing nursery school admission eligibility using Oracle collections and procedural logic.



---

