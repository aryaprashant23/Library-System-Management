# Library-System-Management

# Project Overview
Project Title: Library Management System
Level: Intermediate
Database: library_db

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

# Objectives
Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

# Project Structure
## 1. Database Setup
<img width="1114" height="816" alt="image" src="https://github.com/user-attachments/assets/a6078867-d074-43aa-9c2c-7f12a8bf3339" />

```sql
USE DATABASE librarydb;

-- Create table "Branch"
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
branch_id VARCHAR(10) PRIMARY KEY,
manager_id VARCHAR(10),
branch_address VARCHAR(100),
contach_no VARCHAR(10)
);

ALTER TABLE branch
ALTER COLUMN contact_no TYPE VARCHAR(50);

-- Create table "Employees"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
emp_id VARCHAR(10) PRIMARY KEY,
emp_name VARCHAR(50),
position VARCHAR(50),
salary VARCHAR(50),
branch_id VARCHAR(25)
);

-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
isbn VARCHAR(50) PRIMARY KEY,
book_title VARCHAR(50),
category VARCHAR(50),
status VARCHAR(50),
aut VARCHAR(25)
);
ALTER TABLE books
RENAME COLUMN aut TO author
ALTER TABLE books
ALTER COLUMN book_title TYPE VARCHAR(100); 

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
members_id VARCHAR(10) PRIMARY KEY,
members_name VARCHAR(50),
branch_address VARCHAR(100),
reg_date VARCHAR(50)
);
ALTER TABLE members
RENAME COLUMN branch_address TO member_address;
SELECT*FROM members;

-- Create table "IssuedStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
issued_id VARCHAR(10) PRIMARY KEY,
issued_member_id VARCHAR(50), -- FK
issued_book_name VARCHAR(100),
issued_date DATE,
issued_book_isbn VARCHAR(50), -- FK
issued_emp_id VARCHAR(50) -- FK
);

-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
return_id VARCHAR(10) PRIMARY KEY,
issued_id VARCHAR(50),
return_book_name VARCHAR(100),
return_date DATE,
return_book_isbn VARCHAR(50),
issued_emp_id VARCHAR(50)
);
```


## CRUD Operations
-- Create: Inserted sample records into the books table.

-- Read: Retrieved and displayed data from various tables.

-- Update: Updated records in the employees table.

-- Delete: Removed records from the members table as needed.


-- Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
  ```sql
   INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```

-- Task 2: Update an Existing Member's Address
```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

-- Task 3: Delete a Record from the Issued Status Table -- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

-- Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```

-- Task 5: List Members Who Have Issued More Than One Book -- Objective: Use GROUP BY to find members who have issued more than one book.
```sql
SELECT
    issued_emp_id,
    COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1
```

-- Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**
```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist
JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```

### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

Task 8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
    b.category,
    SUM(b.rental_price),
    COUNT(*)
FROM 
issued_status as ist
JOIN
books as b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1
```

Task 9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

Task 10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
FROM employees as e1
JOIN 
branch as b
ON e1.branch_id = b.branch_id    
JOIN
employees as e2
ON e2.emp_id = b.manager_id
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```
## ADV. SQL OPERATIONS:

-- Task 13: Identify Members with Returned Books:
-- Write a query to identify members who have returned the books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

-- issued_status == members == books == return_status
-- filter books which is return
-- Returned < 30 days
```sql
SELECT 
     ist.issued_member_id,
	 m.members_id,
	 bk.book_title,
	 ist.issued_date,
	 rs.return_date
FROM issued_status AS ist
JOIN
members as m
ON ist.issued_member_id = m.members_id
JOIN 
books AS bk
ON ist.issued_book_isbn = bk.isbn
JOIN
return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL;
```
-- Task 14: Branch Performance Report
-- Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports AS 
SELECT 
      br.branch_id,
	  br.manager_id,
	  COUNT(ist.issued_id) AS no_of_issued_book,
	  COUNT(rs.return_id) AS no_of_return_book,
	  SUM(bk.rental_price) AS total_revenue
FROM issued_status as ist
JOIN employees as em
ON em.emp_id = ist.issued_emp_id
JOIN branch as br
ON br.branch_id = em.branch_id
LEFT JOIN return_status as rs
ON rs.issued_id = ist.issued_id
JOIN books as bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY 1,2;
```

-- Task 15: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.
-- member issued book in last 2 months
```sql
CREATE TABLE Active_Mambers AS
SELECT * FROM members
WHERE members_id IN (
SELECT DISTINCT issued_member_id
FROM issued_status 
WHERE 
issued_date >= CURRENT_DATE - INTERVAL '2 month'
)
```
-- Task 16: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.
```sql
SELECT 
       em.emp_name,
	   br.*,
       COUNT(ist.issued_id) AS no_books_issued
FROM issued_status AS ist
JOIN employees As em
ON ist.issued_emp_id = em.emp_id
JOIN branch AS br
ON em.branch_id = br.branch_id
GROUP BY 1, 2 ORDER BY no_books_issued DESC LIMIT 3;
```
-- Task 18: Stored Procedure Objective: 
-- Create a stored procedure to manage the status of books in a library system. 
-- Description: Write a stored procedure that updates the status of a book in the library based on its issuance. 
-- The procedure should function as follows: The stored procedure should take the book_id as an input parameter. 
-- The procedure should first check if the book is available (status = 'yes'). If the book is available,
-- it should be issued, and the status in the books table should be updated to 'no'. 
-- If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.
```sql
	   CREATE PROCEDURE issue_book (
  p_issued_id VARCHAR(50),
  p_issued_member_id VARCHAR(50),
  p_issued_book_isbn VARCHAR(50),
  p_issued_emp_id VARCHAR(50)
)
LANGUAGE plpgsql
AS $$
DECLARE
  v_status VARCHAR(50);
BEGIN
  -- Get the status of the book using the provided ISBN
  SELECT status
  FROM books
  INTO v_status
  WHERE isbn = p_issued_book_isbn;

  -- Check if the book is available
  IF v_status = 'yes' THEN
    -- Insert a new record into the issued_status table
    INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
    VALUES
      (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);
      
    -- Update the book status to 'no'
    UPDATE books
    SET status = 'no'
    WHERE isbn = p_issued_book_isbn;
    
    -- Notify the user that the book was issued successfully
    RAISE NOTICE 'Book records added successfully for book isbn: %', p_issued_book_isbn;
  ELSE
    -- Notify the user that the book is unavailable
    RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
  END IF;
END;
$$;
```
