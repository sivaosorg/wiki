# PostgreSQL DateTime and Timezone Management: A Comprehensive Guide

## Table of Contents

1. [Database Setup and Timezone Configuration](#database-setup)
2. [User Creation and Privileges](#user-privileges)
3. [DateTime Data Types](#datetime-types)
4. [Table Design with DateTime Columns](#table-design)
5. [DDL Scripts and Best Practices](#ddl-scripts)
6. [Real-World Examples](#examples)
7. [Troubleshooting and Tips](#tips)

---

## 1. Database Setup and Timezone Configuration {#database-setup}

### Understanding PostgreSQL Timezone

PostgreSQL stores datetime values using the `timestamp` data type, which can be timezone-aware (`timestamp with time zone`) or timezone-naive (`timestamp without time zone`). The recommended approach for production systems is to always use `timestamp with time zone` and store data in UTC.

### Check Current Timezone Configuration

```sql
-- Check the server timezone setting
SHOW timezone;

-- Check the default timezone for a session
SHOW timezone;

-- List all available timezones
SELECT * FROM pg_timezone_names;

-- Get more detailed timezone info
SELECT name, abbrev, utc_offset, is_dst
FROM pg_timezone_names
ORDER BY utc_offset;
```

### Set Timezone at Database Level

```sql
-- Connect to PostgreSQL as a superuser
psql -U postgres -d postgres

-- Set the timezone for the entire cluster (postgresql.conf)
-- Edit postgresql.conf file:
-- timezone = 'UTC'

-- Or set it using SQL (requires server restart for cluster-wide change)
ALTER SYSTEM SET timezone = 'UTC';

-- Apply the change
SELECT pg_reload_conf();

-- Verify the change
SHOW timezone;
```

### Set Timezone at Session Level

```sql
-- Set timezone for current session only
SET timezone = 'UTC';

-- Set timezone for a specific user
ALTER USER username SET timezone = 'UTC';

-- Set timezone for a specific database
ALTER DATABASE database_name SET timezone = 'UTC';

-- Verify session timezone
SHOW timezone;
```

### Example: Complete Database Initialization with UTC Timezone

```sql
-- Connect as superuser
psql -U postgres -d postgres -h localhost

-- Create the database with UTF-8 encoding and UTC timezone
CREATE DATABASE production_db
  WITH OWNER postgres
  ENCODING 'UTF8'
  LC_COLLATE 'en_US.UTF-8'
  LC_CTYPE 'en_US.UTF-8'
  TEMPLATE template0;

-- Connect to the new database
\c production_db

-- Set timezone to UTC for the database
ALTER DATABASE production_db SET timezone = 'UTC';

-- Verify the setting
SHOW timezone;
```

---

## 2. User Creation and Privileges {#user-privileges}

### Create Database Users with Proper Privileges

```sql
-- Create a read-write user for application
CREATE USER app_user WITH PASSWORD 'secure_password_here';

-- Set timezone for the user
ALTER USER app_user SET timezone = 'UTC';

-- Create a read-only user for reporting
CREATE USER report_user WITH PASSWORD 'secure_password_here';
ALTER USER report_user SET timezone = 'UTC';

-- Create a migration user for schema changes
CREATE USER migration_user WITH SUPERUSER PASSWORD 'secure_password_here';
ALTER USER migration_user SET timezone = 'UTC';
```

### Grant Privileges

```sql
-- Grant privileges on database
GRANT CONNECT ON DATABASE production_db TO app_user;
GRANT CONNECT ON DATABASE production_db TO report_user;

-- Grant schema privileges
GRANT USAGE ON SCHEMA public TO app_user;
GRANT USAGE ON SCHEMA public TO report_user;

-- Grant table privileges (after tables are created)
-- For read-write user
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- For read-only user
GRANT SELECT ON ALL TABLES IN SCHEMA public TO report_user;

-- Set default privileges for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO app_user;
```

### Verify User Configuration

```sql
-- List all users and their settings
\du+

-- Check timezone setting for a specific user
ALTER USER app_user SET timezone = 'UTC';
SELECT * FROM pg_user_mapping WHERE usename = 'app_user';

-- List user privileges
SELECT grantee, privilege_type
FROM information_schema.table_privilege_usage
WHERE table_name = 'your_table_name';
```

---

## 3. DateTime Data Types {#datetime-types}

### Available DateTime Data Types

| Data Type                  | Size     | Description                                             | Use Case                    |
| -------------------------- | -------- | ------------------------------------------------------- | --------------------------- |
| `timestamp`                | 8 bytes  | Without timezone (alias: `timestamp without time zone`) | Local time only, no TZ info |
| `timestamp with time zone` | 8 bytes  | With timezone info (alias: `timestamptz`)               | Recommended for UTC storage |
| `date`                     | 4 bytes  | Date only (no time)                                     | Birth dates, event dates    |
| `time`                     | 8 bytes  | Time only (no date)                                     | Business hours, shift times |
| `time with time zone`      | 12 bytes | Time with timezone                                      | Rarely used                 |
| `interval`                 | 16 bytes | Time duration                                           | Differences, durations      |

### Recommended Column Definitions

```sql
-- RECOMMENDED: Use timestamp with time zone (stored as UTC)
created_at timestamp with time zone NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at timestamp with time zone NOT NULL DEFAULT CURRENT_TIMESTAMP

-- OR shorter syntax
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- For date-only columns
birth_date date

-- For time-only columns (with timezone)
business_open_time time with time zone

-- For duration/interval
processing_duration interval
```

### Important Notes on Storage

1. **PostgreSQL always stores `timestamptz` as UTC internally** - the timezone info is only used for display
2. **When you insert a timestamp with TZ, it's converted to UTC for storage**
3. **When you retrieve a timestamp, it's converted to the session timezone**

---

## 4. Table Design with DateTime Columns {#table-design}

### DateTime Columns Design Patterns

#### Pattern 1: Basic Audit Timestamps (Recommended)

```sql
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

#### Pattern 2: Soft Delete with Timestamp

```sql
CREATE TABLE products (
    id bigserial PRIMARY KEY,
    name varchar(255) NOT NULL,
    sku varchar(50) NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);
```

#### Pattern 3: Event with Start and End Times

```sql
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    event_name varchar(255) NOT NULL,
    description text,
    started_at timestamptz NOT NULL,
    ended_at timestamptz NOT NULL,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK (ended_at > started_at)
);
```

#### Pattern 4: Version History with Effective Dates

```sql
CREATE TABLE product_prices (
    id bigserial PRIMARY KEY,
    product_id bigint NOT NULL,
    price numeric(10, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',
    effective_from timestamptz NOT NULL,
    effective_to timestamptz DEFAULT '9999-12-31'::timestamptz,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by varchar(50) NOT NULL,
    CHECK (effective_to > effective_from),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

### Column Naming Conventions

```sql
-- Standard naming patterns:
created_at      -- When the record was created
updated_at      -- When the record was last modified
deleted_at      -- Soft delete marker (NULL if not deleted)
started_at      -- When an event started
ended_at        -- When an event ended
scheduled_for   -- When something is scheduled
published_at    -- When content was published
due_at          -- When something is due
expires_at      -- When something expires
processed_at    -- When processing completed
```

---

## 5. DDL Scripts and Best Practices {#ddl-scripts}

### Complete Example: E-Commerce Database

```sql
-- ============================================================================
-- Database Setup
-- ============================================================================

-- Create database with UTF-8 encoding and UTC timezone
CREATE DATABASE ecommerce_db
  WITH OWNER postgres
  ENCODING 'UTF8'
  LC_COLLATE 'en_US.UTF-8'
  LC_CTYPE 'en_US.UTF-8'
  TEMPLATE template0;

-- Set database timezone
ALTER DATABASE ecommerce_db SET timezone = 'UTC';

-- Connect to the database
\c ecommerce_db

-- ============================================================================
-- Create Users and Grant Privileges
-- ============================================================================

-- Create application user
CREATE USER ecom_app WITH PASSWORD 'your_secure_password';
ALTER USER ecom_app SET timezone = 'UTC';

-- Create read-only user for analytics
CREATE USER ecom_analyst WITH PASSWORD 'your_secure_password';
ALTER USER ecom_analyst SET timezone = 'UTC';

-- Grant privileges
GRANT CONNECT ON DATABASE ecommerce_db TO ecom_app;
GRANT CONNECT ON DATABASE ecommerce_db TO ecom_analyst;

GRANT USAGE ON SCHEMA public TO ecom_app;
GRANT USAGE ON SCHEMA public TO ecom_analyst;

-- ============================================================================
-- Create Tables
-- ============================================================================

-- Users table
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    first_name varchar(100),
    last_name varchar(100),
    password_hash varchar(255) NOT NULL,
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);

-- Categories table
CREATE TABLE categories (
    id bigserial PRIMARY KEY,
    name varchar(100) NOT NULL UNIQUE,
    description text,
    parent_category_id bigint REFERENCES categories(id) ON DELETE SET NULL,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Products table with pricing history
CREATE TABLE products (
    id bigserial PRIMARY KEY,
    category_id bigint NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
    sku varchar(50) NOT NULL UNIQUE,
    name varchar(255) NOT NULL,
    description text,
    current_price numeric(10, 2) NOT NULL,
    stock_quantity integer NOT NULL DEFAULT 0,
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,
    CHECK (current_price > 0),
    CHECK (stock_quantity >= 0),
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);

-- Price history for audit trail
CREATE TABLE product_price_history (
    id bigserial PRIMARY KEY,
    product_id bigint NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    old_price numeric(10, 2) NOT NULL,
    new_price numeric(10, 2) NOT NULL,
    changed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    changed_by varchar(50) NOT NULL,
    reason varchar(255)
);

-- Orders table
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    order_number varchar(50) NOT NULL UNIQUE,
    status varchar(20) NOT NULL DEFAULT 'pending',
    total_amount numeric(12, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',
    placed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    confirmed_at timestamptz DEFAULT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    cancelled_at timestamptz DEFAULT NULL,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK (total_amount > 0),
    CHECK (cancelled_at IS NULL OR cancelled_at >= placed_at),
    CHECK (shipped_at IS NULL OR shipped_at >= confirmed_at),
    CHECK (delivered_at IS NULL OR delivered_at >= shipped_at)
);

-- Order items table
CREATE TABLE order_items (
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id bigint NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity integer NOT NULL DEFAULT 1,
    unit_price numeric(10, 2) NOT NULL,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK (quantity > 0),
    CHECK (unit_price > 0)
);

-- Audit log for tracking changes
CREATE TABLE audit_log (
    id bigserial PRIMARY KEY,
    table_name varchar(100) NOT NULL,
    record_id bigint NOT NULL,
    action varchar(10) NOT NULL,
    old_values jsonb,
    new_values jsonb,
    changed_by varchar(50) NOT NULL,
    changed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ip_address inet,
    CHECK (action IN ('INSERT', 'UPDATE', 'DELETE'))
);

-- ============================================================================
-- Create Indexes for DateTime Columns
-- ============================================================================

CREATE INDEX idx_users_created_at ON users(created_at DESC);
CREATE INDEX idx_users_deleted_at ON users(deleted_at) WHERE deleted_at IS NOT NULL;
CREATE INDEX idx_orders_placed_at ON orders(placed_at DESC);
CREATE INDEX idx_orders_status_placed_at ON orders(status, placed_at DESC);
CREATE INDEX idx_orders_user_id_placed_at ON orders(user_id, placed_at DESC);
CREATE INDEX idx_product_price_history_product_id_changed_at
    ON product_price_history(product_id, changed_at DESC);
CREATE INDEX idx_audit_log_changed_at ON audit_log(changed_at DESC);
CREATE INDEX idx_audit_log_table_name_record_id ON audit_log(table_name, record_id);

-- ============================================================================
-- Grant Table Privileges
-- ============================================================================

GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO ecom_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO ecom_app;

GRANT SELECT ON ALL TABLES IN SCHEMA public TO ecom_analyst;

-- Set default privileges for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO ecom_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO ecom_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO ecom_analyst;

-- ============================================================================
-- Create Triggers for updated_at Columns
-- ============================================================================

-- Create a reusable function for updating updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Add triggers to tables
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_categories_updated_at BEFORE UPDATE ON categories
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_products_updated_at BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================================
-- Verify Setup
-- ============================================================================

-- Check timezone
SHOW timezone;

-- List tables
\dt

-- List users and their timezone settings
\du+
```

---

## 6. Real-World Examples {#examples}

### Example 1: Insert Data and Verify UTC Storage

```sql
-- Set session timezone to different values and observe behavior
SET timezone = 'UTC';
SELECT now()::timestamptz;  -- 2025-11-20 06:45:53+00:00

-- Insert a user
INSERT INTO users (username, email, first_name, last_name, password_hash)
VALUES ('john_doe', 'john@example.com', 'John', 'Doe', 'hashed_password')
RETURNING id, username, created_at;

-- Change session timezone and insert another user
SET timezone = 'America/New_York';
INSERT INTO users (username, email, first_name, last_name, password_hash)
VALUES ('jane_smith', 'jane@example.com', 'Jane', 'Smith', 'hashed_password')
RETURNING id, username, created_at;

-- Query and see all timestamps in current session timezone
SELECT id, username, email, created_at FROM users;

-- Switch to UTC and see timestamps in UTC
SET timezone = 'UTC';
SELECT id, username, email, created_at FROM users;

-- Switch to Asia/Bangkok and see timestamps in that timezone
SET timezone = 'Asia/Bangkok';
SELECT id, username, email, created_at FROM users;
```

### Example 2: Creating an Order with Status Timeline

```sql
-- Insert an order
INSERT INTO orders (user_id, order_number, status, total_amount)
VALUES (1, 'ORD-2025-00001', 'pending', 150.00)
RETURNING id, order_number, status, placed_at;

-- Get the order ID (assume it's 1)
-- Simulate order confirmation after 5 minutes
UPDATE orders
SET status = 'confirmed', confirmed_at = CURRENT_TIMESTAMP
WHERE id = 1
RETURNING order_number, status, placed_at, confirmed_at;

-- Simulate shipping after some time
UPDATE orders
SET status = 'shipped', shipped_at = CURRENT_TIMESTAMP
WHERE id = 1
RETURNING order_number, status, placed_at, shipped_at;

-- Query the complete timeline
SELECT
    order_number,
    status,
    placed_at,
    confirmed_at,
    shipped_at,
    delivered_at,
    EXTRACT(EPOCH FROM (confirmed_at - placed_at)) / 60 as minutes_to_confirm,
    EXTRACT(EPOCH FROM (shipped_at - confirmed_at)) / 60 as minutes_to_ship
FROM orders
WHERE id = 1;
```

### Example 3: Query Orders Within a Date Range

```sql
-- Orders placed today
SELECT
    id,
    order_number,
    placed_at,
    status
FROM orders
WHERE placed_at::date = CURRENT_DATE
ORDER BY placed_at DESC;

-- Orders placed in the last 7 days
SELECT
    id,
    order_number,
    placed_at,
    status
FROM orders
WHERE placed_at >= CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY placed_at DESC;

-- Orders placed between two specific dates
SELECT
    id,
    order_number,
    placed_at,
    status
FROM orders
WHERE placed_at BETWEEN
    '2025-11-01'::timestamptz AT TIME ZONE 'UTC'
    AND '2025-11-30'::timestamptz AT TIME ZONE 'UTC'
ORDER BY placed_at DESC;

-- Orders that are still pending after 24 hours
SELECT
    id,
    order_number,
    placed_at,
    AGE(CURRENT_TIMESTAMP, placed_at) as time_pending
FROM orders
WHERE status = 'pending'
    AND placed_at < CURRENT_TIMESTAMP - INTERVAL '24 hours'
ORDER BY placed_at ASC;
```

### Example 4: Price History Tracking

```sql
-- Insert a product
INSERT INTO products (category_id, sku, name, current_price, stock_quantity)
VALUES (1, 'PROD-001', 'Laptop', 999.99, 50)
RETURNING id, name, current_price, created_at;

-- Record initial price in history
INSERT INTO product_price_history (product_id, old_price, new_price, changed_by, reason)
VALUES (1, 0.00, 999.99, 'admin', 'Initial price')
RETURNING product_id, old_price, new_price, changed_at;

-- Update price
UPDATE products
SET current_price = 899.99, updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- Record price change in history
INSERT INTO product_price_history (product_id, old_price, new_price, changed_by, reason)
VALUES (1, 999.99, 899.99, 'admin', 'Black Friday promotion');

-- Query price history timeline
SELECT
    id,
    product_id,
    old_price,
    new_price,
    (new_price - old_price) as price_change,
    ROUND(((new_price - old_price) / old_price * 100)::numeric, 2) as percent_change,
    changed_at,
    changed_by,
    reason
FROM product_price_history
WHERE product_id = 1
ORDER BY changed_at DESC;
```

### Example 5: Soft Delete with Timestamp

```sql
-- Create a user
INSERT INTO users (username, email, password_hash)
VALUES ('inactive_user', 'inactive@example.com', 'hashed_password')
RETURNING id, username, deleted_at;

-- Mark user as deleted (soft delete)
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE username = 'inactive_user'
RETURNING id, username, deleted_at;

-- Query only active users
SELECT id, username, email, created_at
FROM users
WHERE deleted_at IS NULL
ORDER BY created_at DESC;

-- Query deleted users with deletion timestamp
SELECT id, username, email, deleted_at
FROM users
WHERE deleted_at IS NOT NULL
ORDER BY deleted_at DESC;

-- Query users deleted in the last 30 days
SELECT id, username, email, deleted_at
FROM users
WHERE deleted_at IS NOT NULL
    AND deleted_at >= CURRENT_TIMESTAMP - INTERVAL '30 days'
ORDER BY deleted_at DESC;

-- Permanently delete users marked as deleted for more than 90 days
DELETE FROM users
WHERE deleted_at IS NOT NULL
    AND deleted_at < CURRENT_TIMESTAMP - INTERVAL '90 days';
```

### Example 6: Audit Trail with JSON

```sql
-- Create a function to log changes
CREATE OR REPLACE FUNCTION log_order_change()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, record_id, action, old_values, new_values, changed_by)
    VALUES (
        TG_TABLE_NAME,
        NEW.id,
        TG_OP,
        to_jsonb(OLD),
        to_jsonb(NEW),
        current_user
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger for order changes
CREATE TRIGGER log_order_changes
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION log_order_change();

-- Now when you update an order, it's logged
UPDATE orders SET status = 'shipped' WHERE id = 1;

-- Query the audit log
SELECT
    id,
    table_name,
    record_id,
    action,
    new_values->'status' as status,
    new_values->'shipped_at' as shipped_at,
    changed_at,
    changed_by
FROM audit_log
WHERE table_name = 'orders'
ORDER BY changed_at DESC;

-- Query specific field changes
SELECT
    id,
    record_id,
    action,
    old_values->>'status' as old_status,
    new_values->>'status' as new_status,
    changed_at
FROM audit_log
WHERE table_name = 'orders'
    AND new_values->>'status' IS NOT NULL
ORDER BY changed_at DESC;
```

---

## 7. Troubleshooting and Tips {#tips}

### Common Issues and Solutions

#### Issue 1: Timezone Mismatch Between Application and Database

```sql
-- Problem: Your application sends '2025-11-20 10:00:00' (local time)
-- but it's stored as a different UTC time

-- Solution 1: Always send timezone info from application
-- Wrong: INSERT INTO events (started_at) VALUES ('2025-11-20 10:00:00')
-- Correct: INSERT INTO events (started_at) VALUES ('2025-11-20 10:00:00+05:30')

-- Solution 2: Convert to UTC before sending
-- In your application, convert to UTC:
-- Timestamp.now(ZoneId.of("UTC"))  // Java
-- datetime.now(timezone.utc)        // Python
-- new Date().toISOString()          // JavaScript

-- Solution 3: Set application timezone explicitly
-- In connection string or application config:
-- jdbc:postgresql://host:5432/db?serverTimezone=UTC
-- set timezone = 'UTC' in app config
```

#### Issue 2: Comparing Timestamps from Different Timezones

```sql
-- Problem: Comparing timestamps that appear different due to timezone display

-- Solution: PostgreSQL handles this automatically for timestamptz
-- Both of these are the same instant in time:
SELECT
    '2025-11-20 06:45:53+00:00'::timestamptz as utc_time,
    '2025-11-20 12:15:53+05:30'::timestamptz as ist_time,
    '2025-11-20 06:45:53+00:00'::timestamptz = '2025-11-20 12:15:53+05:30'::timestamptz as are_equal;

-- Output: are_equal = true
```

#### Issue 3: Daylight Saving Time (DST) Handling

```sql
-- Problem: Timestamps during DST transitions can be ambiguous

-- Solution: Always use timestamptz (timestamp with time zone)
-- PostgreSQL handles DST transitions correctly

-- Example: EST to EDT transition
SELECT
    '2025-03-09 01:30:00-05:00'::timestamptz as before_dst,
    '2025-03-09 03:30:00-04:00'::timestamptz as after_dst;

-- PostgreSQL automatically handles the offset change
```

#### Issue 4: Storing Business Hours (Time without Date)

```sql
-- Problem: Storing business hours like "9:00 AM" for multiple days

-- Solution: Use time with time zone or store as interval
-- Option 1: Use time with time zone
ALTER TABLE stores ADD COLUMN business_open_time time with time zone;
ALTER TABLE stores ADD COLUMN business_close_time time with time zone;

-- Option 2: Store as interval (duration from midnight)
ALTER TABLE stores ADD COLUMN business_open_time interval;
ALTER TABLE stores ADD COLUMN business_close_time interval;

INSERT INTO stores (name, business_open_time, business_close_time)
VALUES ('Store A', '09:00:00'::time, '18:00:00'::time);

-- Check if store is open now
SELECT name
FROM stores
WHERE CAST(CURRENT_TIME AS time) BETWEEN business_open_time AND business_close_time;
```

### Best Practices Checklist

```sql
-- ✅ DO:

-- 1. Always use timestamptz for audit columns
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- 2. Set database timezone to UTC
ALTER DATABASE your_db SET timezone = 'UTC';

-- 3. Store all timestamps in UTC internally
-- (PostgreSQL does this automatically with timestamptz)

-- 4. Use CURRENT_TIMESTAMP or NOW() for server timestamps
DEFAULT CURRENT_TIMESTAMP

-- 5. Add CHECK constraints for logical validation
CHECK (deleted_at IS NULL OR deleted_at >= created_at)
CHECK (ended_at > started_at)

-- 6. Index frequently queried timestamp columns
CREATE INDEX idx_created_at ON table_name(created_at DESC);

-- 7. Document timezone expectations in schema comments
COMMENT ON TABLE orders IS 'All timestamps are stored in UTC (timestamptz)';

-- 8. Use triggers to automatically update updated_at
CREATE TRIGGER update_products_updated_at BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- 9. Query with explicit timezone handling
SELECT * FROM orders
WHERE placed_at AT TIME ZONE 'America/New_York' > '2025-11-20'::date;

-- 10. Test thoroughly with multiple timezones
SET timezone = 'America/New_York';
SET timezone = 'Asia/Bangkok';
SET timezone = 'Europe/London';

-- ❌ DON'T:

-- 1. Use timestamp WITHOUT time zone for audit columns
-- BAD: created_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP

-- 2. Store timestamps in application code
-- BAD: new_timestamp = datetime.now()

-- 3. Compare timestamps from different timezones without conversion
-- BAD: timestamp > '2025-11-20 10:00:00'

-- 4. Use time without time zone for business hours across regions
-- BAD: business_hours time

-- 5. Forget to validate timestamp logic with CHECK constraints
-- BAD: No validation between related timestamps

-- 6. Ignore indexes on timestamp columns
-- BAD: SELECT * FROM logs WHERE created_at > ... (without index)

-- 7. Mix local time and UTC in the same column
-- BAD: Some rows in UTC, some in local time

-- 8. Assume application and database timezones match
-- BAD: Not explicitly setting timezone in both places

-- 9. Store human-readable timestamps
-- BAD: created_at varchar(50) with values like "Nov 20, 2025"

-- 10. Forget to set timezone for new users
-- BAD: Creating users without: ALTER USER user SET timezone = 'UTC';
```

### Performance Tips for DateTime Queries

```sql
-- 1. Use timestamp range queries efficiently
-- This uses the index on created_at
EXPLAIN ANALYZE
SELECT COUNT(*) FROM orders
WHERE created_at >= '2025-11-01'::timestamptz
  AND created_at < '2025-12-01'::timestamptz;

-- 2. Partition large tables by date range
CREATE TABLE orders_2025_11 PARTITION OF orders
    FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');

-- 3. Archive old data efficiently
CREATE TABLE orders_archive AS
SELECT * FROM orders
WHERE created_at < CURRENT_TIMESTAMP - INTERVAL '1 year'
WITH NO DATA;

-- 4. Use DATE_TRUNC for grouping by time periods
SELECT
    DATE_TRUNC('day', created_at) as day,
    COUNT(*) as order_count
FROM orders
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY day DESC;

-- 5. Calculate time differences efficiently
SELECT
    order_number,
    EXTRACT(EPOCH FROM (updated_at - created_at)) as processing_seconds
FROM orders
WHERE status = 'completed'
ORDER BY processing_seconds DESC;

-- 6. Use partial indexes for filtered queries
CREATE INDEX idx_pending_orders_created_at
    ON orders(created_at DESC)
    WHERE status = 'pending';

-- 7. Avoid CAST operations in WHERE clauses for performance
-- BAD: WHERE created_at::date = CURRENT_DATE
-- GOOD: WHERE created_at >= CURRENT_DATE::timestamptz
--       AND created_at < (CURRENT_DATE + INTERVAL '1 day')::timestamptz

-- 8. Use prepared statements for repeated queries
PREPARE get_recent_orders AS
SELECT * FROM orders
WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY created_at DESC
LIMIT $1;

EXECUTE get_recent_orders(10);
```

### Useful DateTime Functions Reference

```sql
-- Current date/time functions
CURRENT_TIMESTAMP                     -- Current timestamp with tz
CURRENT_DATE                          -- Current date
CURRENT_TIME                          -- Current time with tz
CLOCK_TIMESTAMP()                     -- Current timestamp (high precision)
NOW()                                 -- Alias for CURRENT_TIMESTAMP

-- Date/Time arithmetic
CURRENT_TIMESTAMP + INTERVAL '1 day'  -- Add 1 day
CURRENT_TIMESTAMP - INTERVAL '1 hour' -- Subtract 1 hour
age(timestamp1, timestamp2)           -- Duration between timestamps

-- Date/Time extraction
DATE_TRUNC('day', timestamp_col)      -- Truncate to day
EXTRACT(YEAR FROM timestamp_col)      -- Extract year
EXTRACT(MONTH FROM timestamp_col)     -- Extract month
EXTRACT(DAY FROM timestamp_col)       -- Extract day
EXTRACT(EPOCH FROM interval_col)      -- Convert interval to seconds

-- Date/Time conversion
timestamp_col::date                   -- Cast to date
timestamp_col::time                   -- Cast to time
timestamp_col AT TIME ZONE 'UTC'      -- Convert to specific timezone
TO_TIMESTAMP(unix_timestamp)          -- Convert from Unix timestamp
TO_CHAR(timestamp, format)            -- Format as string

-- Examples:
SELECT
    CURRENT_TIMESTAMP as now,
    CURRENT_TIMESTAMP::date as today,
    CURRENT_TIMESTAMP::time as current_time,
    DATE_TRUNC('month', CURRENT_TIMESTAMP) as month_start,
    EXTRACT(EPOCH FROM CURRENT_TIMESTAMP) as unix_timestamp,
    AGE(CURRENT_TIMESTAMP, created_at) as age_in_interval
FROM users
LIMIT 1;
```

---

## Summary

PostgreSQL provides robust support for handling datetime and timezone data. Key takeaways:

1. **Always use `timestamptz` (timestamp with time zone)** for audit and event timestamps
2. **Set database timezone to UTC** at the database level during initialization
3. **Store all timestamps in UTC** - PostgreSQL handles this automatically with `timestamptz`
4. **Use `CURRENT_TIMESTAMP`** for server-generated timestamps
5. **Add CHECK constraints** to validate logical relationships between timestamps
6. **Create indexes** on frequently queried timestamp columns
7. **Use triggers** to automatically maintain `updated_at` columns
8. **Document timezone expectations** in your schema
9. **Test with multiple timezones** to ensure correctness
10. **Convert to application timezone only for display**, not storage

This approach ensures data consistency, prevents timezone-related bugs, and makes your database more maintainable across global operations.
