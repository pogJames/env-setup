## OLTP
1. Place Order (Transaction) 
```sql
BEGIN;
INSERT INTO orders (customer_id, order_date, total_amount) VALUES (101, NOW(), 119.97);
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES (last_insert_rowid(), 1001, 2, 29.99);
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES (last_insert_rowid(), 1002, 1, 49.99);
UPDATE products SET stock = stock - 2 WHERE product_id = 1001;
UPDATE products SET stock = stock - 1 WHERE product_id = 1002;
COMMIT;
```
2. Check Inventory
```sql
SELECT stock FROM products WHERE product_id = 1001;
```
3. Get Order History
```sql
SELECT o.order_id, o.order_date, p.name, oi.quantity, oi.unit_price
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.customer_id = 101;
```
## UAA
1. Register a New user
```sql
INSERT INTO users (email, password_hash, role_id)
VALUES ('newuser@example.com', crypt('password123', gen_salt('bf')), 1)  -- 1 = default 'user' role
RETURNING user_id, email, created_at;
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

-- Get user's role
SELECT r.name 
FROM users u
JOIN roles r ON u.role_id = r.role_id
WHERE u.user_id = 123;
```
