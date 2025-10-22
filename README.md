# Treasure 
⚙️ STEP-BY-STEP LAB PLAN (for Oracle SQL+ / SQL Developer)
🧱 STEP 1: Start clean

At the top of your file (or before running anything in SQL+):

SET SERVEROUTPUT ON;


(Always do this — or you won’t see DBMS_OUTPUT text.)

If you already created something earlier:

DROP TABLE issues CASCADE CONSTRAINTS;
DROP TABLE members CASCADE CONSTRAINTS;
DROP TABLE books CASCADE CONSTRAINTS;
DROP SEQUENCE issues_seq;

📘 STEP 2: Create the tables

Paste and run:

CREATE TABLE books (
    book_id NUMBER PRIMARY KEY,
    title VARCHAR2(100),
    author VARCHAR2(100),
    available_copies NUMBER
);

CREATE TABLE members (
    member_id NUMBER PRIMARY KEY,
    name VARCHAR2(100)
);

CREATE TABLE issues (
    issue_id NUMBER PRIMARY KEY,
    book_id NUMBER REFERENCES books(book_id),
    member_id NUMBER REFERENCES members(member_id),
    issue_date DATE,
    return_date DATE
);

⚙️ STEP 3: Create the sequence
CREATE SEQUENCE issues_seq START WITH 1;

📚 STEP 4: Insert sample data (5-5 rows)
INSERT INTO books VALUES (1, 'Database Systems', 'Elmasri', 5);
INSERT INTO books VALUES (2, 'Operating Systems', 'Silberschatz', 4);
INSERT INTO books VALUES (3, 'Java Programming', 'James Gosling', 3);
INSERT INTO books VALUES (4, 'Data Structures', 'Weiss', 6);
INSERT INTO books VALUES (5, 'Discrete Mathematics', 'Rosen', 2);

INSERT INTO members VALUES (1, 'Aryan Meena');
INSERT INTO members VALUES (2, 'Ravi Kumar');
INSERT INTO members VALUES (3, 'Sita Sharma');
INSERT INTO members VALUES (4, 'Aman Verma');
INSERT INTO members VALUES (5, 'Neha Singh');

COMMIT;

🔧 STEP 5: Create one procedure (start with issue_book)

Paste this and run:

CREATE OR REPLACE PROCEDURE issue_book(p_member_id IN NUMBER, p_book_id IN NUMBER) AS
    available NUMBER;
BEGIN
    SELECT available_copies INTO available FROM books WHERE book_id = p_book_id;

    IF available > 0 THEN
        UPDATE books
        SET available_copies = available_copies - 1
        WHERE book_id = p_book_id;

        INSERT INTO issues(issue_id, book_id, member_id, issue_date, return_date)
        VALUES (issues_seq.NEXTVAL, p_book_id, p_member_id, SYSDATE, NULL);

        DBMS_OUTPUT.PUT_LINE('Book issued successfully.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Book not available.');
    END IF;
END;
/


If it says Procedure created. → ✅ good.
If it says with compilation errors → type SHOW ERRORS;.

▶️ STEP 6: Execute your procedure

Now show this to your teacher:

EXEC issue_book(1, 3);


Output:

Book issued successfully.


Then show table change:

SELECT * FROM issues;
SELECT * FROM books;


Teacher will see that available_copies decreased and one new issue record was added.

🔄 STEP 7: Create and test another procedure

Example: return_book

CREATE OR REPLACE PROCEDURE return_book(p_issue_id IN NUMBER) AS
    b_id NUMBER;
BEGIN
    SELECT book_id INTO b_id FROM issues WHERE issue_id = p_issue_id;

    UPDATE issues
    SET return_date = SYSDATE
    WHERE issue_id = p_issue_id;

    UPDATE books
    SET available_copies = available_copies + 1
    WHERE book_id = b_id;

    DBMS_OUTPUT.PUT_LINE('Book returned successfully.');
END;
/


Then run:

EXEC return_book(1);


Output:

Book returned successfully.


Then verify:

SELECT * FROM issues;
SELECT * FROM books;

🧾 STEP 8: Show other small procedures (optional)

Example:

CREATE OR REPLACE PROCEDURE book_availability(p_book_id IN NUMBER) AS
    t VARCHAR2(100);
    a NUMBER;
BEGIN
    SELECT title, available_copies INTO t, a FROM books WHERE book_id = p_book_id;
    DBMS_OUTPUT.PUT_LINE('Title: ' || t || ', Available Copies: ' || a);
END;
/


Run:

EXEC book_availability(3);
