## OLTP
1. Create a New Order
```sql
BEGIN TRANSACTION;

-- Insert order header
INSERT INTO orders (customer_id, order_date, status, total_amount)
VALUES (101, CURRENT_TIMESTAMP, 'Pending', 109.97);

-- Insert items using the new order_id
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES 
    (last_insert_rowid(), 1001, 2, 29.99),
    (last_insert_rowid(), 1002, 1, 49.99);

-- Deduct stock
UPDATE products SET stock = stock - 2 WHERE product_id = 1001;
UPDATE products SET stock = stock - 1 WHERE product_id = 1002;

COMMIT;
```
2. Update Order Status
```sql
UPDATE orders 
SET status = 'Shipped', shipped_date = CURRENT_TIMESTAMP
WHERE order_id = 5001;
```
3. Get Order Details with items
```sql
SELECT 
    o.order_id, o.order_date, o.status, o.total_amount,
    p.name AS product_name, oi.quantity, oi.unit_price
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_id = 5001;
```
4. Check Product Stock Before Sale
```sql
SELECT stock FROM products WHERE product_id = 1001 FOR UPDATE;  -- Locks row for safe update
```
## UAA
1. Register a New user
```sql
INSERT INTO users (email, password_hash, role_id, is_active)
VALUES ('newuser@example.com', crypt('password123', gen_salt('bf')), 1, True)
```
2. Authenticate User on Login
```sql
SELECT user_id, email, role_id
FROM users
WHERE email = 'user@example.com'
  AND password_hash = crypt('entered_password', password_hash)
  AND is_active = true;
```
3. Create a Session/Token After Login
```sql
INSERT INTO sessions (user_id, token, expires_at)
VALUES (123, 'jwt-or-random-token-abc123', NOW() + INTERVAL '7 days')
RETURNING session_id, token, expires_at;
```
4. Validate Session/Token
```sql
SELECT u.user_id, u.email, r.name AS role
FROM sessions s
JOIN users u ON s.user_id = u.user_id
JOIN roles r ON u.role_id = r.role_id
WHERE s.token = 'jwt-or-random-token-abc123'
  AND s.expires_at > NOW()
  AND u.is_active = true;
```
5. Logout/Invalidate Session
```sql
DELETE FROM sessions
WHERE token = 'jwt-or-random-token-abc123';
-- Or mark as expired:
UPDATE sessions SET expires_at = NOW() WHERE token = '...';
-- Cleanup expired sessions
DELETE FROM sessions WHERE expires_at < NOW();
```
6. Check User Role/Permission
```sql
-- Is user an admin?
SELECT EXISTS (
    SELECT 1 FROM users u
    JOIN roles r ON u.role_id = r.role_id
    WHERE u.user_id = 123 AND r.name = 'admin'
);
```
## CRM
1. Pipeline Overview
2. Revenue by Salesperson
3. Customer Activity History
4. Update Deal Stage
5. Deals Closing This Month
## CMS
1. Latest Published Content
```sql
SELECT title, slug, published_at
FROM posts
WHERE status = 'Published'
ORDER BY published_at DESC
LIMIT 10;
```
2. Content with Category/Author
```sql
SELECT 
    p.title,
    p.slug,
    c.name AS category,
    a.name AS author
FROM posts p
JOIN categories c ON p.category_id = c.category_id
JOIN authors a ON p.author_id = a.author_id
WHERE p.slug = 'best-gadgets-2025';
```
3. Content by Tag
```sql
SELECT p.title, p.slug
FROM posts p
JOIN post_tags pt ON p.post_id = pt.post_id
JOIN tags t ON pt.tag_id = t.tag_id
WHERE t.name = 'gadgets'
ORDER BY p.published_at DESC;
```
4. Search Content
```sql
SELECT title, slug, content
FROM posts
WHERE title ILIKE '%python%' 
   OR content ILIKE '%python%'
ORDER BY published_at DESC;
```
## FRA
1. Record a Transaction
```sql
BEGIN TRANSACTION;

INSERT INTO transactions (date, description)
VALUES ('2025-12-30', 'Customer payment for invoice #9001');

-- Debit Cash (increase asset), Credit Revenue (increase income)
INSERT INTO journal_entries (transaction_id, account_id, debit, credit)
VALUES 
    (last_insert_rowid(), 101, 5000.00, NULL),  -- 101 = Cash
    (last_insert_rowid(), 401, NULL, 5000.00);  -- 401 = Revenue

COMMIT;
```
2. Account Balance
```sql
SELECT 
    a.name,
    SUM(COALESCE(je.debit, 0)) - SUM(COALESCE(je.credit, 0)) AS balance
FROM accounts a
LEFT JOIN journal_entries je ON a.account_id = je.account_id
GROUP BY a.account_id, a.name
ORDER BY a.name;
```
## Configurations & Settings Storage
1. Get All Settings for a Tenant
```sql
SELECT key, value
FROM settings
WHERE tenant_id = 501;  -- Current tenant
```
2. Get Specific Setting
```sql
SELECT value
FROM settings
WHERE tenant_id = 501
  AND key = 'timezone';
```
3. Check Feature Flag
```sql
SELECT enabled
FROM feature_flags
WHERE tenant_id = 501
  AND feature_name = 'advanced_analytics';
```
4. Update or Insert Setting
```sql
-- PostgreSQL UPSERT
INSERT INTO settings (tenant_id, key, value)
VALUES (501, 'currency', 'EUR')
ON CONFLICT (tenant_id, key) DO UPDATE SET value = EXCLUDED.value;
```
5. Enable/Disable Feature for Tenant
```sql
INSERT INTO feature_flags (tenant_id, feature_name, enabled)
VALUES (502, 'api_access', true)
ON CONFLICT (tenant_id, feature_name) DO UPDATE SET enabled = EXCLUDED.enabled;
```
