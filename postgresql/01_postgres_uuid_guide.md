# PostgreSQL UUID Generation: Solutions and Best Practices

## Overview

This guide covers the resolution of the `ERROR: function uuidv7() does not exist` error and provides comprehensive solutions for UUID generation in PostgreSQL, comparing different approaches based on version compatibility, performance, and maintainability.

---

## Problem Statement

When executing the following DDL statement:

```sql
CREATE TABLE users (
  id uuid DEFAULT uuidv7() PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);
```

PostgreSQL throws the error:

```
SQL Error [42883]: ERROR: function uuidv7() does not exist
  Hint: No function matches the given name and argument types. You might need to add explicit type casts.
```

### Root Cause

The `uuidv7()` function is not available in standard PostgreSQL distributions and requires:

- Specific PostgreSQL versions (typically 15.2+)
- Third-party extensions
- Custom implementations

---

## Solution Comparison Matrix

| Feature                | `gen_random_uuid()` | `uuid_generate_v4()` | `uuidv7()`             |
| ---------------------- | ------------------- | -------------------- | ---------------------- |
| PostgreSQL Version     | 13+                 | 8.1+                 | 15.2+ (with extension) |
| Extension Required     | ❌ No               | ✅ Yes (uuid-ossp)   | ✅ Yes                 |
| Cryptographic Security | ✅ Yes              | ✅ Yes               | ✅ Yes                 |
| Performance            | ⭐⭐⭐ High         | ⭐⭐⭐ High          | ⭐⭐⭐ High            |
| Recommended            | ✅ **Primary**      | ⚠️ Legacy Support    | 🔄 Future              |
| Sortability            | ❌ No               | ❌ No                | ✅ Yes (time-based)    |

---

## Solution 1: `gen_random_uuid()` (Recommended)

### Prerequisites

- PostgreSQL 13 or higher
- No extensions required

### Implementation

```sql
CREATE TABLE users (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);
```

### Advantages

✅ **Zero Dependencies** - Built-in function, no external extensions needed  
✅ **Superior Performance** - No extension overhead, optimal resource utilization  
✅ **Cryptographically Secure** - Uses `/dev/urandom` for random generation  
✅ **Modern Standard** - Aligns with PostgreSQL's future direction  
✅ **Simple Syntax** - Clean and straightforward implementation  
✅ **Maintenance-Free** - No extension version management required

### Usage Example

```sql
-- Insert a new user
INSERT INTO users (email) VALUES ('john.doe@example.com');

-- Verify the generated UUID
SELECT * FROM users WHERE email = 'john.doe@example.com';
```

---

## Solution 2: `uuid_generate_v4()` (Backward Compatibility)

### Prerequisites

- PostgreSQL 8.1 or higher
- `uuid-ossp` extension installed

### Implementation

```sql
-- Step 1: Create the extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Step 2: Create the table
CREATE TABLE users (
  id uuid DEFAULT uuid_generate_v4() PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);
```

### Advantages

✅ **Backward Compatible** - Works with older PostgreSQL versions (8.1+)  
✅ **RFC 4122 Compliant** - Follows standard UUID v4 specification  
✅ **Widely Adopted** - Extensively used in legacy systems  
✅ **Battle-Tested** - Proven reliability in production environments

### Considerations

⚠️ **Extension Dependency** - Requires `uuid-ossp` module management  
⚠️ **Slightly Higher Overhead** - Extension loading adds minimal latency  
⚠️ **Version Management** - Must ensure extension compatibility

### Usage Example

```sql
-- Insert a new user
INSERT INTO users (email) VALUES ('jane.smith@example.com');

-- Verify the generated UUID
SELECT * FROM users WHERE email = 'jane.smith@example.com';
```

---

## Solution 3: `uuidv7()` (Future Standard)

### Prerequisites

- PostgreSQL 15.2 or higher
- Custom extension or contrib module
- Not available by default in all distributions

### Implementation

```sql
-- Install pgcrypto or uuid-ossp extension that provides uuidv7()
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE users (
  id uuid DEFAULT uuidv7() PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);
```

### Advantages

✅ **Time-Based Sorting** - UUID values are sortable by generation time  
✅ **Better Indexing Performance** - Reduces index fragmentation  
✅ **RFC 4122 Compliant** - Follows UUID v7 specification (draft)  
✅ **Reduced Collisions** - Time-based component ensures uniqueness

### Considerations

⚠️ **Version Dependent** - Requires PostgreSQL 15.2+  
⚠️ **Extension Management** - Must manually install and maintain  
⚠️ **Limited Availability** - Not built-in across all PostgreSQL distributions  
⚠️ **Migration Path** - Adoption is still in adoption phase

---

## Quick Decision Tree

```
┌─────────────────────────────────────────────────────────┐
│ What is your PostgreSQL version?                        │
└─────────────────────────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
       < 13.0          13.0 - 15.1      >= 15.2
           │               │               │
           ▼               ▼               ▼
   uuid_generate_v4()  gen_random_uuid()  Consider
   (with extension)    (RECOMMENDED)      uuidv7()

   • Legacy systems   • New projects     • Time-based
   • Extended support • Best practice    • Future
   • RFC 4122 v4      • No extension     • Advanced
```

---

## Version Check Query

Determine your PostgreSQL version before implementation:

```sql
-- Check PostgreSQL version
SELECT version();

-- Alternative method
SELECT current_setting('server_version_num')::int / 10000 as major_version;

-- Check if uuid-ossp extension is available
SELECT * FROM pg_available_extensions WHERE name = 'uuid-ossp';

-- Check if gen_random_uuid() is available
SELECT proname FROM pg_proc WHERE proname = 'gen_random_uuid';
```

---

## Production Recommendations

### For Modern Applications (PostgreSQL 13+)

```sql
-- Recommended approach: Zero dependencies
CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- Create indexes for optimal query performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### For Legacy Applications (PostgreSQL < 13)

```sql
-- Legacy approach with extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- Create indexes for optimal query performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### For Future-Proofing (PostgreSQL 15.2+)

```sql
-- Advanced approach with time-based sortable UUIDs
CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT uuidv7(),
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- Create indexes optimized for time-based UUIDs
CREATE INDEX idx_users_id_time ON users(id);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

---

## Performance Benchmarking

### Insertion Performance

```sql
-- Benchmark: 100,000 inserts with gen_random_uuid()
DO $$
DECLARE
  i INTEGER;
  start_time TIMESTAMP;
BEGIN
  start_time := CLOCK_TIMESTAMP();

  FOR i IN 1..100000 LOOP
    INSERT INTO users (email) VALUES ('user' || i || '@example.com');
  END LOOP;

  RAISE NOTICE 'Completed in %', CLOCK_TIMESTAMP() - start_time;
END $$;
```

### Expected Results

- `gen_random_uuid()`: ~50-100ms per 1,000 inserts
- `uuid_generate_v4()`: ~60-120ms per 1,000 inserts
- Performance differences are negligible in production

---

## Common Issues and Solutions

### Issue 1: Extension Not Found

```
ERROR: extension "uuid-ossp" does not exist
```

**Solution:**

```sql
-- Install the extension system-wide (requires superuser)
CREATE EXTENSION uuid-ossp;

-- Verify installation
SELECT * FROM pg_extension WHERE extname = 'uuid-ossp';
```

### Issue 2: Function Not Recognized

```
ERROR: function gen_random_uuid() does not exist
```

**Solution:**

```sql
-- Check PostgreSQL version
SELECT version();

-- If version < 13, use uuid_generate_v4() instead
-- Or upgrade PostgreSQL to 13+
```

### Issue 3: UUID Collision (Extremely Rare)

```sql
-- Add UNIQUE constraint as defense-in-depth
CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid() UNIQUE,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);
```

---

## Best Practices Summary

1. **Use `gen_random_uuid()`** for PostgreSQL 13+ (recommended)
2. **Upgrade PostgreSQL** if still on versions < 13
3. **Avoid manual UUID generation** at application level
4. **Index UUID columns** appropriately for query patterns
5. **Monitor extension versions** in legacy systems
6. **Document UUID strategy** in system architecture
7. **Test UUID collision scenarios** in non-production environments
8. **Plan migration path** for future upgrades

---

## Reference Documentation

- [PostgreSQL UUID Type](https://www.postgresql.org/docs/current/datatype-uuid.html)
- [RFC 4122: UUID Standard](https://tools.ietf.org/html/rfc4122)
- [uuid-ossp Extension](https://www.postgresql.org/docs/current/uuid-ossp.html)
- [PostgreSQL Cryptographic Functions](https://www.postgresql.org/docs/current/pgcrypto.html)
