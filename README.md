# Library System Management
## Project Overview

**Project Title**: Retail Sales Analysis

**Database**: library_db

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives
1. **Set up the Library Management System Database**: Create and populate the database with tables.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
<img width="800" height="861" alt="Screenshot 2026-03-22 073504" src="https://github.com/user-attachments/assets/9f25410b-f6ea-466b-a5fe-62fd54bafffe" />

1. **Database Creation**: Created a database named library_db.
2. **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE TABLE branch (
branch_id VARCHAR (50) PRIMARY KEY,
manager_id VARCHAR(50),
branch_address VARCHAR (100),
contact_no VARCHAR(50));

CREATE TABLE employees (
emp_id VARCHAR (50) PRIMARY KEY,
emp_name VARCHAR (100),
emp_position VARCHAR (50),
salary INT,
branch_id VARCHAR(50));

CREATE TABLE books (
isbn VARCHAR (50) PRIMARY KEY,
book_title VARCHAR(100),
category VARCHAR (50),
rental_price FLOAT,
book_status VARCHAR (15),
author VARCHAR (30),
publisher VARCHAR (30));

CREATE TABLE members (
member_id VARCHAR(20) PRIMARY KEY,
member_name VARCHAR(40),
member_address VARCHAR(50),
reg_date DATE);

CREATE TABLE issued_status (
issued_id VARCHAR(20) PRIMARY KEY,
issued_member_id VARCHAR(20),
issued_book_name VARCHAR(100),
issued_date DATE,
issued_book_isbn VARCHAR(100),
issued_emp_id VARCHAR(50));

CREATE TABLE return_status (
return_id VARCHAR(25) PRIMARY KEY,
issued_id VARCHAR (20),
return_book_name VARCHAR(75),
return_date DATE,
return_book_isbn VARCHAR (40));
```
### 2. CRUD Operations
1. **Create**: Inserted sample records into the books table.

2. **Read**: Retrieved and displayed data from various tables.

3. **Update**: Updated records in the employees table.

4. **Delete**: Removed records from the members table as needed.

   

#### Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books VALUES (
'978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
#### Task 2: Update an Existing Member's Address

```sql
UPDATE members 
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

#### Task 3: Delete a Record from the Issued Status Table -- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
SELECT * FROM issued_status;
```

#### Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

#### Task 5: List Members Who Have Issued More Than One Book -- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT issued_emp_id ,
COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1;
```

### 3. CTAS (Create Table As Select)

#### Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt

```sql
CREATE TABLE book_cnts -- Creating book counts table 
AS
SELECT b.isbn, b.book_title,
 COUNT(ist.issued_id) AS no_issued
 FROM books as b
JOIN 
issued_status as ist
ON ist.issued_book_isbn = b.isbn 
GROUP BY 1, 2;
SELECT * FROM book_cnts;
```

### 4. Data Analysis And Findings

#### Task 7. Retrieve All Books in a Specific Category
```sql
SELECT * FROM books
WHERE category = 'Classic';
```

#### Task 8: Find Total Rental Income by Category
```sql
SELECT b.category, SUM(b.rental_price), COUNT(*)
FROM books as b
JOIN 
issued_status as ist
ON ist.issued_book_isbn = b.isbn 
GROUP BY 1;
```

#### Task 9: List Members Who Registered in the Last 180 Days
```sql
SELECT * FROM members
WHERE reg_date >= CURDATE() - INTERVAL 180 DAY;
```

#### Task 10: List Employees with Their Branch Manager's Name and their branch details
```sql
SELECT 
e1.*, b.branch_address, e2.emp_name as manager
 FROM employees as e1
JOIN 
branch as b
ON b.branch_id = e1.branch_id
JOIN 
employees AS e2
ON b.manager_id = e2.emp_id;
```

#### Task 11: Create a Table of Books with Rental Price Above a Certain Threshold USD 7
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7;
SELECT * FROM expensive_books;
```

#### Task 12: Retrieve the List of Books Not Yet Returned
```sql
SELECT * FROM issued_status as ist
LEFT JOIN return_status as rs
ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL;
```

#### Task 13: Identify Members with Overdue Books
 Write a query to identify members who have overdue books (assume a 30-day return period). 
 Display the member's_id, member's name, book title, issue date, and days overdue.
 ```sql
SELECT ist.issued_member_id,m.member_name,bk.book_title, ist.issued_date, rs.return_date,
CURDATE() - ist.issued_date AS Overdue_Days
FROM issued_status as ist
JOIN members as m
   ON m.member_id = ist.issued_member_id -- JOIN 1 (Memeber id == Issued Member id)
JOIN books as bk
   ON bk.isbn = ist.issued_book_isbn -- JOIN 2 (Book ISBN == Issued Book ISBN)
LEFT JOIN return_status as rs
   ON rs.issued_id = ist.issued_id -- JOIN 3 (Return Table Issued Id == Issued Id)
WHERE rs.return_date is NULL
	AND 
    (CURDATE() - ist.issued_date) > 30
ORDER BY 1;
```

#### Task 14: Update Book Status on Return
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).
```sql
-- Creating Stored Procedure
DELIMITER $$

CREATE PROCEDURE update_book_status()
BEGIN
	UPDATE books as bk
    JOIN issued_status as ist
		ON bk.isbn = ist.issued_book_isbn
    JOIN return_status as rst
		ON ist.issued_id = rst.issued_id
	SET bk.bpok_status = 'Yes';
END $$
DELIMITER ; -- Procedure Created

-- Testing the Procedure
CALL update_book_status();
```

#### Task 15: Branch Performance Report
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.
```sql
CREATE TABLE branch_reports
AS
	SELECT 
	b.branch_id, b.manager_id,
	COUNT(ist.issued_id) AS Books_issued, -- Total Books Issued
	COUNT(rst.return_id) AS Books_returned, -- Total Books Returned
	SUM(bk.rental_price) AS Total_Revenue -- Total Revenue
FROM issued_status AS ist
JOIN employees as e
	ON
	e.emp_id = ist.issued_emp_id -- JOIN 1 (Emp ID == Issued Emp ID)
JOIN branch as b
	ON
	e.branch_id = b.branch_id -- JOIN 2 (Emp.Branch ID == Br.Branch ID)
LEFT JOIN return_status as rst
	ON
	rst.issued_id = ist.issued_id -- JOIN 3 ( Return Issued Id == Issued Id)
JOIN books as bk
	ON 
    ist.issued_book_isbn = bk.isbn -- JOIN 4 ( Issued Book ISBN == Book ISBN)
GROUP BY 1,2;
	SELECT * FROM branch_reports;
```

#### Task 17: Find Employees with the Most Book Issues Processed
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.
```sql
SELECT e.emp_name, b.branch_id,b.branch_address,COUNT(ist.issued_id) AS books_issued
	FROM issued_status as ist
JOIN employees as e
	ON e.emp_id = ist.issued_emp_id
JOIN branch as b
	ON e.branch_id = b.branch_id
GROUP BY 1,2
	ORDER BY books_issued DESC
LIMIT 3;
```

#### Task 18: Identify Members Issuing High-Risk Books
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. 
Display the member name, book title, and the number of times they've issued damaged books.
```sql
ALTER TABLE books
	ADD COLUMN book_quality VARCHAR(20) DEFAULT('Good');
UPDATE books
	SET book_quality = 'Damaged'
WHERE isbn 
    IN ('978-0-06-112008-4', '978-0-06-440055-8', '978-0-14-044930-3');
SELECT * FROM books; -- Altered the table

SELECT m.member_id, bk.book_title, COUNT(*) AS damaged_book_issued
	FROM issued_status as ist
JOIN members as m
	ON ist.issued_member_id = m.member_id
JOIN books as bk
	ON ist.issued_book_isbn = bk.isbn
WHERE bk.book_quality = 'Damaged'
	GROUP BY m.member_id, m.member_name, bk.book_title
HAVING COUNT(*) > 2;
```

#### Task 19: Stored Procedure
Create a stored procedure to manage the status of books in a library system. Description: Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows: The stored procedure should take the book_id as an input parameter. The procedure should first check if the book is available (status = 'yes'). If the book is available, it should be issued, and the status in the books table should be updated to 'no'. If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql
DELIMITER $$
CREATE PROCEDURE issue_books(IN input_book_id VARCHAR(50))
BEGIN
	DECLARE book_status VARCHAR(20);
SELECT book_status INTO book_status FROM books
WHERE isbn = input_book_id;

IF book_status = 'No'
	THEN SELECT 'Book Not Found' AS message;

ELSEIF book_status = 'Yes'
	THEN UPDATE books SET book_status = 'No'
    WHERE isbn = input_book_id;
    
SELECT 'Book Issued Successfully' AS message;
	ELSE
SELECT 'Book Is Currently Not Avaiable' AS message;
END IF;
END$$
DELIMITER ; -- Procedure Created

-- Testing the Procedure
CALL issue_books();
```

### Conclusion

This project provided a comprehensive understanding of how SQL can be used to design and manage a real-world system such as a Library Management System. Through this project, I implemented database design concepts, including table creation, relationships using primary and foreign keys, and ER diagram modeling.

I developed and executed various SQL queries to extract meaningful insights, such as identifying overdue books, analyzing member activity, and evaluating employee performance. Additionally, I implemented stored procedures to automate business logic like issuing and returning books, ensuring data consistency and operational efficiency.

Overall, this project strengthened my ability to translate business requirements into structured database solutions and enhanced my skills in SQL, data modeling, and analytical thinking. It also demonstrated how data can be leveraged to support decision-making and improve system performance.
