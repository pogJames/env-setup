# RELATIONAL DATABASE
### Why:
1. Data Duplication and Inconsistency
2. Lack of Relationships and Integrity
3. Scalability and Querying Issues
### How:
1. Structured Data with Tables
> Data is organized into tables (relations) with rows (records/tuples) and columns (attributes/fields)
2. Relationships via Keys
> Tables are linked using primary keys and foreign keys
3. SQL for Querying
> Standardized query language (Structured Query Language)
4. ACID Properties
> Atomicity (all or nothing), Consistency (rules enforced), Isolation (concurrent safety), Durability (data survives crashes)
### Use Cases
1. Transactional Systems: 
- record sales, transfers, listings (E-commerce orders, Banking transfers, Booking systems, Inventory management)\
- core tables: customers, products, orders, order_items
2. User Authentication & Authorization:
- manage users, roles, sessions (Web apps, Mobile apps, SaaS platforms)\
- core tables: roles, users, sessions
3. Content Management & Catalogs:
- structure products, customers, items (Online stores, Blogs, Product Catalogs)
- core tables: categories, posts, tags
4. Customer Relationship Management:
- track leads, contacts, interactions (Salesforce, support ticketing, sales pipeline)\
- core tables: companies, contacts, deals, activities
5. Financial Records & Accounting:
- manage invoices, payments, ledgers (Billing systems, payroll, expense tracking)
- core tables: clients, invoices, invoice_items
6. Configuration & Settings Storage:
- store app settings, feature flags, user preferences (SaaS configuration, multi-tenant apps)
- core tables: tenants, settings, feature_flags
7. Logging & Auditing:
- record user actions, system events for compliance/debugging (Audit logs, Access logs)
- audit_logs
### Core Knowledge:
1. Data Modelling Basics
   - Entities & Attributes (Customer -> ID, Name, Address)
   - Relationships (One-to-Many, Many-to-Many, One-to-One)
   - Normalization:
     1. 1NF = atomic values, rows and columns are unique
     2. 2NF = every non-prime attribute must depend on the entire primary key, not part of it
     3. 3NF = non-prime attributes must not depend on other non-prime attributes
2. SQL Fundamentals
   - DDL (Data Definition Language): Create/Alter tables
   - DML (Data Manipulation): Insert/Update/Delete/Select
   - DQL (Queries): JOIN (INNER, LEFT), GROUP BY, HAVING, ORDER BY, indexes for speed
3. Keys & Constraints
   - Primary Key: Unique, non-null identifier
   - Foreign Key: Links tables, references primary keys
   - Constraints: UNIQUE, NOT NULL, etc.
5. Transactions & ACID
   - Atomicity
   - Consistency
   - Isolation
   - Durability

Here are 10 progressive practice questions to help you master writing correct SQL commands. They start simple and build up to more complex scenarios involving multiple tables, joins, constraints, and transactions. Assume a relational database for a small online bookstore.
Schema (Use this for all questions)
```sql
SQLCREATE TABLE authors (
    author_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    country TEXT
);

CREATE TABLE books (
    book_id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER,
    price REAL NOT NULL,
    publication_year INTEGER,
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);

CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE NOT NULL,
    total_amount REAL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INTEGER,
    book_id INTEGER,
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, book_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id)
);
```
Practice Questions

1. Basic INSERT
- Insert a new author named "Jane Austen" from "United Kingdom".
2. INSERT with Foreign Key
- Insert a book titled "Pride and Prejudice" written by Jane Austen (use her author_id), priced at 12.99, published in 1813.
3. SELECT with WHERE
- List all books published after 1900, showing title, price, and publication year, ordered by year descending.
4. JOIN Two Tables
- Show book titles along with their author's name (not author_id).
5. JOIN Three Tables
- List all order items for a specific customer (by name, e.g., "John Doe"), showing book title, quantity, and order date.
6. Aggregate Query
- Find the total revenue (sum of quantity × price) per book. Show book title and total revenue, ordered by revenue descending.
7. UPDATE
- Increase the price of all books published before 1900 by 10%.
8. DELETE with Constraint Safety
- Delete an author who has no books (write a safe DELETE that won’t violate foreign key).
9. Transaction with Atomicity
- Write a transaction that:
  - Creates a new customer "Alice Smith" with email "alice@example.com"
  - Creates a new order for Alice today with total_amount 45.97.
  - Adds two order items: 2 copies of one book and 1 copy of another.
- Ensure the whole thing succeeds or fails together.
10. Complex Query with GROUP BY and HAVING
- Find customers who have placed orders totaling more than $100 across all their orders. Show customer name and total spent.

ANSWERS

```sql
INSERT INTO authors (name, country) 
VALUES ('Jane Austen', 'United Kingdom');
```
```sql
INSERT INTO books (title, author_id, price, publication_year)
VALUES ('Pride and Prejudice', 
        (SELECT author_id FROM authors WHERE name = 'Jane Austen'), 
        12.99, 
        1813);
```
```sql
SELECT title, price, publication_year 
FROM books 
WHERE publication_year > 1900 
ORDER BY publication_year DESC;
```
```sql
SELECT b.title, a.name
FROM books b
INNER JOIN authors a ON b.author_id = a.author_id;
```
```sql
SELECT b.title, oi.quantity, o.order_date
FROM customers c
INNER JOIN orders o USING (customer_id)
INNER JOIN order_items oi USING (order_id)
INNER JOIN books b USING (book_id)
WHERE c.name = 'John Doe';
```
```sql
SELECT b.title, SUM(oi.quantity * b.price) AS TotalRevenue
FROM books b
LEFT JOIN order_items oi USING (book_id)
GROUP BY b.book_id, b.title
ORDER BY TotalRevenue DESC;
```
```sql
UPDATE books
SET price = price * 1.1
WHERE publication_year < 1900;
```
```sql
DELETE FROM authors 
WHERE author_id NOT IN (SELECT author_id FROM books WHERE author_id IS NOT NULL);
```
```sql
BEGIN TRANSACTION;

INSERT INTO customers (name, email) 
VALUES ('Alice Smith', 'alice@example.com');

INSERT INTO orders (customer_id, order_date, total_amount)
VALUES (
    (SELECT customer_id FROM customers WHERE name = 'Alice Smith'),
    '2025-12-30',
    45.97
);

INSERT INTO order_items (order_id, book_id, quantity)
VALUES (last_insert_rowid(), (SELECT book_id FROM books WHERE title = 'Pride and Prejudice'), 2);

INSERT INTO order_items (order_id, book_id, quantity)
VALUES (last_insert_rowid(), (SELECT book_id FROM books WHERE title = 'Modern Python Cookbook'), 1);

COMMIT;
```
```sql
SELECT c.name, SUM(o.total_amount) AS TotalSpent
FROM customers c
INNER JOIN orders o USING (customer_id)
GROUP BY c.customer_id, c.name
HAVING SUM(o.total_amount) > 100
ORDER BY TotalSpent DESC;
```
