# Library Management System using SQL Project-

## Project Overview

**Project Title**: Library Management System   
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.



## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_database;
USE library_database;

CREATE TABLE books (
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(50),
    category VARCHAR(30),
    rental_price FLOAT,
    status VARCHAR(20),
    author VARCHAR(50),
    publisher VARCHAR(70)
);



CREATE TABLE branch (
    branch_id VARCHAR(20),
    manager_id VARCHAR(20),
    branch_address VARCHAR(75),
    contact_no VARCHAR(52),
    PRIMARY KEY (branch_id)
);



CREATE TABLE employees (
    emp_id VARCHAR(25) PRIMARY KEY,
    emp_name VARCHAR(35),
    position VARCHAR(30),
    salary INT,
    branch_id VARCHAR(25)
);



CREATE TABLE issued_status (
    issued_id VARCHAR(35) PRIMARY KEY,
    issued_member_id VARCHAR(35),
    issued_book_name VARCHAR(75),
    issued_date DATE,
    issued_book_isbn VARCHAR(55),
    issued_emp_id VARCHAR(35)
);



CREATE TABLE return_status (
    return_id VARCHAR(35) PRIMARY KEY,
    issued_id VARCHAR(35),
    return_date DATE
);



CREATE TABLE members (
    member_id VARCHAR(20),
    member_name VARCHAR(25),
    member_address VARCHAR(50),
    reg_date DATE,
    PRIMARY KEY (member_id)
);



CREATE TABLE issued_status (
    issued_id VARCHAR(35) PRIMARY KEY,
    issued_member_id VARCHAR(35),
    issued_book_name VARCHAR(75),
    issued_date DATE,
    issued_book_isbn VARCHAR(55),
    issued_emp_id VARCHAR(35)
);

ADD FOREIGN KEYS -

alter table issued_status
add constraint fk_members
foreign key(issued_member_id)
references members(member_id );


alter table issued_status
add constraint fk_books
foreign key(issued_book_isbn)
references books(isbn);



alter table employees
add constraint fk_branch
foreign key(branch_id)
references branch(branch_id);


alter table issued_status
add constraint fk_employees
foreign key(issued_emp_id)
references employees(emp_id);


alter table return_status
add constraint fk_issued_status
foreign key(issued_id)
references issued_status(issued_id);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT
    issued_emp_id,
    COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

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

8. **Task 8: Find Total Rental Income by Category**:

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

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE() - INTERVAL 180 day;
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

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
WHERE rental_price > 5.00;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
    isd.issued_date,
    isd.issued_member_id,
    m.member_name,
    b.book_title,
    rst.return_date,
    rst.return_id,
    DATEDIFF(CURRENT_DATE, isd.issued_date) AS overdue_days
FROM
    issued_status AS isd
        JOIN
    members AS m ON isd.issued_member_id = m.member_id
        JOIN
    books AS b ON b.isbn = isd.issued_book_isbn
        LEFT JOIN
    return_status AS rst ON isd.issued_id = rst.issued_id
WHERE
    rst.return_date IS NULL
        AND DATEDIFF(CURRENT_DATE, isd.issued_date) > 30
ORDER BY 1;
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql
DELIMITER $$

CREATE PROCEDURE add_return_records(m_return_id VARCHAR(35), m_issued_id VARCHAR(35), m_book_quality VARCHAR(15))   -- -------3 arguements pass ------------------

BEGIN
	DECLARE v_isbn varchar(50);
    DECLARE v_book_name varchar(50);                            -- ----------------declare variables that use in this procedure or function--------------------
    
    
	INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
	VALUES
    (m_return_id, m_issued_id, current_date, m_book_quality);
    
    
    SELECT issued_book_isbn,
    issued_book_name
    INTO
    v_isbn,
    v_book_name                                                   <--   -----------returning the book---------------------------------
    FROM issued_status
    WHERE issued_id = m_issued_id;
    
    
    UPDATE books
    SET status = "yes"                                             <-- -----------once the book will return, change its status to yes----------------
    WHERE isbn = v_isbn;
    
    SELECT CONCAT("Thank you for returning the book: ",v_book_name) AS message;         <-- ------------comment--------------------
END$$

DELIMITER ;

CALL add_return_records('RS104', 'IS106', 'Good');       


-- Testing FUNCTION add_return_records

issued_id = IS135
ISBN = WHERE isbn = '978-0-307-58837-1'

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');

-- calling function 
CALL add_return_records('RS148', 'IS140', 'Good');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_report AS SELECT br.manager_id,
    br.branch_id,
    COUNT(isd.issued_id) AS no_of_book_issued,
    COUNT(r.return_id) AS no_of_book_returned,
    SUM(b.rental_price) AS total_revenue FROM
    issued_status AS isd
        LEFT JOIN
    return_status AS r ON isd.issued_id = r.issued_id
        JOIN
    books AS b ON isd.issued_book_isbn = b.isbn
        JOIN
    employees AS e ON e.emp_id = isd.issued_emp_id
        JOIN
    branch AS br ON e.branch_id = br.branch_id
GROUP BY 1 , 2;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

CREATE TABLE active_members AS SELECT * FROM
    members
WHERE
    member_id IN (SELECT DISTINCT
            issued_member_id
        FROM
            issued_status
        WHERE
            issued_date >= CURRENT_DATE() - INTERVAL 60 DAY);

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
    e.emp_name, br.*, COUNT(isd.issued_id) AS no_of_books_issued
FROM
    issued_status AS isd
        JOIN
    books AS b ON isd.issued_book_isbn = b.isbn
        JOIN
    employees AS e ON isd.issued_emp_id = e.emp_id
        JOIN
    branch AS br ON br.branch_id = e.branch_id
GROUP BY 1 , 2
LIMIT 3;
```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.    

```sql
SELECT 
    rst.book_quality,
    m.member_name,
    b.book_title,
    COUNT(isd.issued_id) AS times_of_books_issued
FROM
    members AS m
        JOIN
    issued_status AS isd ON isd.issued_member_id = m.member_id
        JOIN
    books AS b ON isd.issued_book_isbn = b.isbn
        JOIN
    return_status AS rst ON isd.issued_id = rst.issued_id
WHERE
    rst.book_quality = 'Damaged'
GROUP BY 1 , 2 , 3
HAVING COUNT(isd.issued_id) > 2
ORDER BY COUNT(isd.issued_id);

```

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

DELIMITER $$
create procedure update_database (in m_issued_id varchar(35), in m_issued_member_id varchar(35), in m_issued_book_isbn varchar(55), in m_issued_emp_id varchar(35))

begin
	declare v_status varchar(35);
    
    
	select status into v_status from books
    where isbn = m_issued_book_isbn;
    
    if v_status = "yes" then
    
		insert into issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        values
        (m_issued_id, m_issued_member_id, current_date, m_issued_book_isbn, m_issued_emp_id);
        
	update books 
		set status = "no"
    where isbn = m_issued_book_isbn;
    
    select concat("book records added successfully for book isbn", m_issued_book_isbn) as message;
    
	else
		select concat("sorry, book is not avilable now", m_issued_book_isbn) as message;
        
	end if;

end $$
DELIMITER ;

call update_database ("IS157", "C108", "978-0-06-112241-5", "E104");
call update_database ("IS159", "C106", "978-0-06-112008-4", "E101");



-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL update_database('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL update_database('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'

```



## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
