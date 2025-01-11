# Library Management System using SQL Project

## Project Overview

**Project Title**: Library Management System  
**Database**: `library_mgmt`

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
use library_mgmt;

drop table if exists branch;
create table branch
(branch_id varchar(10) primary key,
manager_id varchar(10),
branch_address varchar(55),
contact_no varchar(25)
);

drop table if exists employees;
create table employees
(emp_id varchar(10) primary key,
emp_name varchar(50),
position varchar(25),
salary int,
branch_id varchar(10)
);

drop table if exists books;
create table books
(isbn varchar(20) primary key,
book_title varchar(75),
category varchar(75),	
rental_price float,
status varchar(15),
author varchar(75),
publisher varchar(80)
);

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);

-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```


### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
select * from books;
insert into books(isbn,book_title,category,rental_price,status,author,publisher)
values
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
select * from books;
```
**Task 2: Update an Existing Member's Address**

```sql
select * from members;
update members
set member_address = '125 Main St'
where member_id = 'C101';
select * from members;
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE from issued_status
where issued_id = 'IS121';
select * from issued_status;
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
select issued_book_name from issued_status where issued_emp_id = 'E101';
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
select issued_emp_id, count(issued_id) as nbr_of_books from issued_status
group by 1
having count(issued_id) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
Create Table Book_cnts
 as 
select 
b.isbn,b.book_title, count(isb.issued_book_isbn) as no_issued
from
books as b
join
issued_status as isb
on isb.issued_book_isbn = b.isbn
group by 1,2;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
select * from books where category = 'Fiction';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select 
b.category,sum(b.rental_price) as total_price
from
books as b
join
issued_status as isb
on isb.issued_book_isbn = b.isbn
group by 1
order by 2 desc;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
select * from members
where reg_date >= date_sub(current_date,interval 180 day);
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
select e1.emp_name,e2.emp_name as manager,b.branch_id,b.branch_address
from
employees as e1
join
branch as b
on e1.branch_id = b.branch_id
join
employees as e2
on b.manager_id = e2.emp_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
Create Table books_price_greater_than_seven
as
select * from books where rental_price > 7;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select distinct i.issued_book_name
from
issued_status as i
left join
return_status as r
on i.issued_id = r.issued_id
where return_id is Null;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select i.issued_member_id,m.member_name,b.book_title,i.issued_date,datediff(current_date, i.issued_date) as overdue
from
members as m
join
issued_status as i
on m.member_id = i.issued_member_id
join
books as b
on issued_book_isbn = b.isbn
left join
return_status as r
on i.issued_id = r.issued_id
where return_date is Null and datediff(current_date, i.issued_date) > 30;
```

**Task 14: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
select e.branch_id, count(i.issued_id) as nbr_of_books_issued, count(r.return_id) as nbr_of_books_returned, sum(rental_price) as Revenue
from
issued_status as i
join
employees as e
on i.issued_emp_id = e.emp_id
join 
books as b
on b.isbn = i.issued_book_isbn
left join
return_status as r
on r.issued_id = i.issued_id
group by 1;
```

**Task 15: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql
create table active_members as
select * from members
where member_id in (select distinct issued_member_id from issued_status where issued_date >=date_sub(current_date , interval 2 month));

 create table active_members_2 as
select m.*
from
issued_status as i
join
members as m
on m.member_id = i.issued_member_id
where issued_date >= date_sub(current_date,interval 2 month);
```


**Task 16: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
with cte as(
select e.emp_name, count(issued_id) as nbr_of_books_processed, e.branch_id, dense_rank() over(order by count(issued_id) desc) as ranks
from
issued_status as i
join
employees as e
on i.issued_emp_id = e.emp_id
group by 1,3)

select emp_name, nbr_of_books_processed, branch_id from cte where ranks <= 3
```

**Task 17: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.

```sql
select member_name, issued_book_name, count(book_quality) as books_damaged from
members as m
join
issued_status as i
on m.member_id = i.issued_member_id
join
books as b
on i.issued_book_isbn = b.isbn
left join
return_status as r
on i.issued_id = r.issued_id
where book_quality = 'Damaged'
group by member_name, issued_book_name
having books_damaged > 2;
```

**Task 18: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines

```sql
Create table overdue_book_fines as 
select i.issued_member_id as Member_ID,m.member_name,count(b.book_title) as nbr_of_books_overdue,
sum(datediff(current_date, i.issued_date)*0.50) as total_fine
from
members as m
join
issued_status as i
on m.member_id = i.issued_member_id
join
books as b
on issued_book_isbn = b.isbn
left join
return_status as r
on i.issued_id = r.issued_id
where return_date is Null and datediff(current_date, i.issued_date) > 30
group by 1,2
```

## Reports

- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid data management and analysis foundation.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
   ```sh

   ```
3. **Run the Queries**: Use the SQL queries in the `Library_Management.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Adithya Anand
