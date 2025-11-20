# PostgreSQL DateTime Naming Conventions - Real-World Examples & Comprehensive Tables

## Table of Contents

1. [Master Naming Convention Tables](#master-tables)
2. [Audit Trail Examples](#audit-trail)
3. [User Management System](#user-management)
4. [E-Commerce Platform](#ecommerce)
5. [Content Publishing Platform](#content-publishing)
6. [Payment Processing System](#payment-system)
7. [Job Scheduler System](#job-scheduler)
8. [Notification System](#notification-system)
9. [Social Media Platform](#social-media)
10. [SaaS Subscription System](#saas-subscription)
11. [Healthcare/Appointment System](#healthcare)
12. [Event Management System](#event-management)

---

## 1. Master Naming Convention Tables {#master-tables}

### Comprehensive DateTime Naming Convention Reference

#### Table 1: All DateTime Column Categories

| Category            | Pattern                   | Example                          | Data Type     | Default             | Immutable | Nullable | When to Use                                |
| ------------------- | ------------------------- | -------------------------------- | ------------- | ------------------- | --------- | -------- | ------------------------------------------ |
| **Audit Trail**     | `created_at`              | `users.created_at`               | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | Record creation timestamp (MANDATORY)      |
| **Audit Trail**     | `updated_at`              | `users.updated_at`               | `timestamptz` | `CURRENT_TIMESTAMP` | ❌ No     | ❌ No    | Last modification timestamp (MANDATORY)    |
| **Audit Trail**     | `deleted_at`              | `users.deleted_at`               | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | Soft delete marker (NULL = active)         |
| **Lifecycle**       | `activated_at`            | `accounts.activated_at`          | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When entity became active                  |
| **Lifecycle**       | `deactivated_at`          | `accounts.deactivated_at`        | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When entity became inactive                |
| **Lifecycle**       | `suspended_at`            | `users.suspended_at`             | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When entity was suspended                  |
| **Lifecycle**       | `resumed_at`              | `users.resumed_at`               | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When suspended entity was resumed          |
| **Lifecycle**       | `archived_at`             | `documents.archived_at`          | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When entity was archived                   |
| **Business Event**  | `{action}_at`             | `orders.submitted_at`            | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When user submitted something              |
| **Business Event**  | `{action}_at`             | `orders.confirmed_at`            | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When order was confirmed                   |
| **Business Event**  | `{action}_at`             | `orders.approved_at`             | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When request was approved                  |
| **Business Event**  | `{action}_at`             | `orders.rejected_at`             | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When request was rejected                  |
| **Business Event**  | `{action}_at`             | `orders.cancelled_at`            | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When order was cancelled                   |
| **Business Event**  | `{action}_at`             | `orders.completed_at`            | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When task/order completed                  |
| **Timeline Event**  | `{action}_at`             | `events.started_at`              | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When event/process started                 |
| **Timeline Event**  | `{action}_at`             | `events.ended_at`                | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When event/process ended                   |
| **Communication**   | `{type}_sent_at`          | `emails.sent_at`                 | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ✅ Yes   | When message was sent                      |
| **Communication**   | `{type}_delivered_at`     | `emails.delivered_at`            | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When message was delivered                 |
| **Communication**   | `{type}_opened_at`        | `emails.opened_at`               | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When message was opened                    |
| **Communication**   | `{type}_clicked_at`       | `emails.clicked_at`              | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When link in message clicked               |
| **Communication**   | `{type}_read_at`          | `notifications.read_at`          | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When notification was read                 |
| **Publishing**      | `published_at`            | `articles.published_at`          | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When content was published                 |
| **Publishing**      | `unpublished_at`          | `articles.unpublished_at`        | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When content was removed                   |
| **Publishing**      | `drafted_at`              | `articles.drafted_at`            | `timestamptz` | `CURRENT_TIMESTAMP` | ❌ No     | ❌ No    | When draft was saved                       |
| **Publishing**      | `scheduled_publish_at`    | `articles.scheduled_publish_at`  | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When to auto-publish                       |
| **Publishing**      | `expires_at`              | `coupons.expires_at`             | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When content/offer expires                 |
| **Verification**    | `verified_at`             | `emails.verified_at`             | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When email was verified                    |
| **Verification**    | `re_verified_at`          | `emails.re_verified_at`          | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When re-verification done                  |
| **Verification**    | `verification_expires_at` | `emails.verification_expires_at` | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When verification expires                  |
| **Scheduling**      | `scheduled_for`           | `tasks.scheduled_for`            | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When task is scheduled (EXCEPTION to \_at) |
| **Scheduling**      | `scheduled_at`            | `tasks.scheduled_at`             | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When scheduling was recorded               |
| **Scheduling**      | `due_at`                  | `invoices.due_at`                | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When something is due                      |
| **Scheduling**      | `deadline_at`             | `projects.deadline_at`           | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | Final deadline                             |
| **Integration**     | `synced_at`               | `sync_logs.synced_at`            | `timestamptz` | `CURRENT_TIMESTAMP` | ❌ No     | ❌ No    | When last sync occurred                    |
| **Integration**     | `imported_at`             | `imports.imported_at`            | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When data was imported                     |
| **Integration**     | `exported_at`             | `exports.exported_at`            | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When data was exported                     |
| **Integration**     | `migrated_at`             | `migration_log.migrated_at`      | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When data was migrated                     |
| **Integration**     | `backed_up_at`            | `backup_log.backed_up_at`        | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When backup was created                    |
| **Validity Period** | `effective_from`          | `prices.effective_from`          | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When data becomes effective                |
| **Validity Period** | `effective_to`            | `prices.effective_to`            | `timestamptz` | `'9999-12-31'`      | ❌ No     | ❌ No    | When data expires                          |
| **Validity Period** | `valid_from`              | `licenses.valid_from`            | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When license valid from                    |
| **Validity Period** | `valid_until`             | `licenses.valid_until`           | `timestamptz` | ⚠️ User             | ❌ No     | ❌ No    | When license valid until                   |
| **Process**         | `{action}_at`             | `payments.received_at`           | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When payment received                      |
| **Process**         | `{action}_at`             | `refunds.issued_at`              | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | When refund issued                         |
| **Process**         | `{action}_at`             | `shipments.shipped_at`           | `timestamptz` | `NULL`              | ✅ Yes    | ✅ Yes   | When shipment occurred                     |
| **Process**         | `{action}_at`             | `shipments.delivered_at`         | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | When delivery occurred                     |

#### Legend:

- ✅ = True / Recommended
- ❌ = False / Not Recommended
- ⚠️ = User Provided (not system default)

---

## 2. Audit Trail Examples {#audit-trail}

### Table 2.1: Audit Trail Naming Convention

| Column Name  | Purpose                     | Data Type     | Default             | Immutable | Nullable | Use Case                   |
| ------------ | --------------------------- | ------------- | ------------------- | --------- | -------- | -------------------------- |
| `created_at` | Record creation timestamp   | `timestamptz` | `CURRENT_TIMESTAMP` | ✅ Yes    | ❌ No    | All tables - mandatory     |
| `updated_at` | Last modification timestamp | `timestamptz` | `CURRENT_TIMESTAMP` | ❌ No     | ❌ No    | All tables - mandatory     |
| `deleted_at` | Soft delete marker          | `timestamptz` | `NULL`              | ❌ No     | ✅ Yes   | Soft delete implementation |
| `created_by` | User who created record     | `varchar(50)` | `current_user`      | ✅ Yes    | ❌ No    | Audit trail                |
| `created_ip` | IP address of creator       | `inet`        | `NULL`              | ✅ Yes    | ✅ Yes   | Security audit             |
| `updated_by` | User who modified record    | `varchar(50)` | `current_user`      | ❌ No     | ❌ No    | Change tracking            |
| `updated_ip` | IP address of last updater  | `inet`        | `NULL`              | ❌ No     | ✅ Yes   | Security audit             |

### Example: User Table with Complete Audit Trail

```sql
-- ============================================================================
-- Users Table with Complete Audit Trail
-- ============================================================================

CREATE TABLE users (
    -- Primary Key
    id bigserial PRIMARY KEY,

    -- Business Data
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    password_hash varchar(255) NOT NULL,

    -- Audit Columns (MANDATORY)
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Extended Audit Information
    created_by varchar(50) NOT NULL DEFAULT current_user,
    created_ip inet DEFAULT NULL,
    updated_by varchar(50) NOT NULL DEFAULT current_user,
    updated_ip inet DEFAULT NULL,

    -- Constraints
    CHECK (deleted_at IS NULL OR deleted_at >= created_at),
    CHECK (updated_at >= created_at)
);

-- Indexes
CREATE INDEX idx_users_created_at ON users(created_at DESC);
CREATE INDEX idx_users_updated_at ON users(updated_at DESC);
CREATE INDEX idx_users_deleted_at ON users(deleted_at) WHERE deleted_at IS NOT NULL;
CREATE INDEX idx_users_created_by ON users(created_by, created_at DESC);
CREATE INDEX idx_users_updated_by ON users(updated_by, updated_at DESC);

-- Trigger for auto-update
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Usage Examples:
-- SELECT * FROM users WHERE deleted_at IS NULL;  -- Active users
-- SELECT * FROM users WHERE deleted_at IS NOT NULL ORDER BY deleted_at DESC;  -- Deleted users
-- SELECT * FROM users WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '30 days';  -- New users
-- SELECT * FROM users WHERE created_by = 'admin';  -- Created by admin
```

---

## 3. User Management System {#user-management}

### Table 3.1: User Management DateTime Naming Convention

| Column Name                  | Purpose              | Pattern      | Data Type     | When to Use              | Example Query                                            |
| ---------------------------- | -------------------- | ------------ | ------------- | ------------------------ | -------------------------------------------------------- |
| `created_at`                 | Account creation     | Audit Trail  | `timestamptz` | All users                | `WHERE created_at > NOW() - INTERVAL '30 days'`          |
| `updated_at`                 | Last profile update  | Audit Trail  | `timestamptz` | All users                | `WHERE updated_at > NOW() - INTERVAL '1 day'`            |
| `deleted_at`                 | Soft delete          | Audit Trail  | `timestamptz` | User deactivation        | `WHERE deleted_at IS NULL`                               |
| `email_verified_at`          | Email verification   | Verification | `timestamptz` | After email confirmation | `WHERE email_verified_at IS NOT NULL`                    |
| `phone_verified_at`          | Phone verification   | Verification | `timestamptz` | After SMS confirmation   | `WHERE phone_verified_at IS NOT NULL`                    |
| `activated_at`               | Account activation   | Lifecycle    | `timestamptz` | Full account activation  | `WHERE activated_at IS NOT NULL`                         |
| `suspended_at`               | Account suspension   | Lifecycle    | `timestamptz` | Temporary ban            | `WHERE suspended_at IS NOT NULL`                         |
| `last_login_at`              | Last login time      | Process      | `timestamptz` | Track activity           | `WHERE last_login_at > NOW() - INTERVAL '90 days'`       |
| `password_changed_at`        | Last password change | Process      | `timestamptz` | Security audit           | `WHERE password_changed_at < NOW() - INTERVAL '90 days'` |
| `two_factor_enabled_at`      | 2FA activation       | Process      | `timestamptz` | Security feature         | `WHERE two_factor_enabled_at IS NOT NULL`                |
| `terms_accepted_at`          | Terms acceptance     | Verification | `timestamptz` | Legal compliance         | `WHERE terms_accepted_at IS NULL`                        |
| `privacy_policy_accepted_at` | Privacy acceptance   | Verification | `timestamptz` | Legal compliance         | `WHERE privacy_policy_accepted_at IS NULL`               |

### Complete User Management Table

```sql
-- ============================================================================
-- User Management System - Complete Example
-- ============================================================================

CREATE TABLE users (
    -- Primary Key
    id bigserial PRIMARY KEY,

    -- Basic Information
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    phone_number varchar(20),
    password_hash varchar(255) NOT NULL,
    first_name varchar(100),
    last_name varchar(100),

    -- Account Status
    status varchar(20) NOT NULL DEFAULT 'pending_verification',

    -- Email & Phone Verification
    email_verified_at timestamptz DEFAULT NULL,
    email_verification_token varchar(255),
    email_verification_expires_at timestamptz DEFAULT NULL,

    phone_verified_at timestamptz DEFAULT NULL,
    phone_verification_code varchar(10),
    phone_verification_expires_at timestamptz DEFAULT NULL,

    -- Account Lifecycle
    activated_at timestamptz DEFAULT NULL,
    suspended_at timestamptz DEFAULT NULL,
    suspended_reason text DEFAULT NULL,

    -- Activity Tracking
    last_login_at timestamptz DEFAULT NULL,
    last_activity_at timestamptz DEFAULT NULL,

    -- Security
    password_changed_at timestamptz DEFAULT NULL,
    two_factor_enabled_at timestamptz DEFAULT NULL,
    two_factor_phone varchar(20),

    -- Legal Compliance
    terms_accepted_at timestamptz DEFAULT NULL,
    privacy_policy_accepted_at timestamptz DEFAULT NULL,
    marketing_consent_at timestamptz DEFAULT NULL,

    -- Audit Columns
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('pending_verification', 'active', 'suspended', 'deactivated')),
    CHECK (email_verified_at IS NULL OR email_verified_at >= created_at),
    CHECK (phone_verified_at IS NULL OR phone_verified_at >= created_at),
    CHECK (activated_at IS NULL OR activated_at >= COALESCE(email_verified_at, created_at)),
    CHECK (suspended_at IS NULL OR suspended_at >= activated_at),
    CHECK (deleted_at IS NULL OR deleted_at >= created_at),
    CHECK (password_changed_at IS NULL OR password_changed_at >= created_at),
    CHECK (last_login_at IS NULL OR last_login_at >= created_at)
);

-- Indexes
CREATE INDEX idx_users_created_at ON users(created_at DESC);
CREATE INDEX idx_users_email_verified_at ON users(email_verified_at) WHERE email_verified_at IS NOT NULL;
CREATE INDEX idx_users_activated_at ON users(activated_at DESC) WHERE activated_at IS NOT NULL;
CREATE INDEX idx_users_suspended_at ON users(suspended_at DESC) WHERE suspended_at IS NOT NULL;
CREATE INDEX idx_users_last_login_at ON users(last_login_at DESC) WHERE last_login_at IS NOT NULL;
CREATE INDEX idx_users_deleted_at ON users(deleted_at) WHERE deleted_at IS NOT NULL;
CREATE INDEX idx_users_active ON users(id) WHERE deleted_at IS NULL AND status = 'active';

-- Trigger
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Useful Queries:

-- Active users
SELECT id, username, email FROM users WHERE deleted_at IS NULL AND status = 'active';

-- Users with verified email
SELECT id, username, email FROM users WHERE email_verified_at IS NOT NULL;

-- Suspended users
SELECT id, username, email, suspended_at, suspended_reason
FROM users
WHERE suspended_at IS NOT NULL AND deleted_at IS NULL
ORDER BY suspended_at DESC;

-- Users never logged in (sign up but no login)
SELECT id, username, email, created_at
FROM users
WHERE last_login_at IS NULL AND email_verified_at IS NOT NULL
ORDER BY created_at DESC;

-- Users inactive for more than 90 days
SELECT id, username, email, last_login_at
FROM users
WHERE last_login_at < CURRENT_TIMESTAMP - INTERVAL '90 days'
ORDER BY last_login_at DESC;

-- Users who haven't changed password in 6 months
SELECT id, username, email, password_changed_at
FROM users
WHERE password_changed_at < CURRENT_TIMESTAMP - INTERVAL '6 months'
ORDER BY password_changed_at DESC;

-- Users with 2FA enabled
SELECT id, username, email, two_factor_enabled_at
FROM users
WHERE two_factor_enabled_at IS NOT NULL
ORDER BY two_factor_enabled_at DESC;

-- Recently deleted users (last 30 days)
SELECT id, username, email, deleted_at
FROM users
WHERE deleted_at IS NOT NULL
    AND deleted_at > CURRENT_TIMESTAMP - INTERVAL '30 days'
ORDER BY deleted_at DESC;
```

---

## 4. E-Commerce Platform {#ecommerce}

### Table 4.1: E-Commerce DateTime Naming Convention

| Column Name           | Table    | Purpose              | Pattern         | When to Use      | Example                                       |
| --------------------- | -------- | -------------------- | --------------- | ---------------- | --------------------------------------------- |
| `placed_at`           | orders   | Order placement time | Business Event  | Always           | `WHERE placed_at > NOW() - INTERVAL '7 days'` |
| `confirmed_at`        | orders   | Order confirmation   | Business Event  | After payment    | `WHERE confirmed_at IS NOT NULL`              |
| `shipped_at`          | orders   | Shipment dispatch    | Business Event  | After packing    | `WHERE shipped_at IS NOT NULL`                |
| `delivered_at`        | orders   | Delivery completion  | Business Event  | After arrival    | `WHERE delivered_at IS NOT NULL`              |
| `returned_at`         | orders   | Return initiation    | Business Event  | Customer returns | `WHERE returned_at IS NOT NULL`               |
| `cancelled_at`        | orders   | Order cancellation   | Business Event  | User cancels     | `WHERE cancelled_at IS NOT NULL`              |
| `payment_received_at` | payments | Payment confirmation | Process         | After payment    | `WHERE payment_received_at IS NOT NULL`       |
| `refund_issued_at`    | refunds  | Refund processing    | Process         | During return    | `WHERE refund_issued_at IS NOT NULL`          |
| `review_submitted_at` | reviews  | Review posting       | Business Event  | After purchase   | `WHERE review_submitted_at IS NOT NULL`       |
| `expires_at`          | coupons  | Coupon expiration    | Validity Period | Promotion end    | `WHERE expires_at > CURRENT_TIMESTAMP`        |

### Complete E-Commerce Order System

```sql
-- ============================================================================
-- E-Commerce Platform - Complete Order System
-- ============================================================================

-- Orders Table
CREATE TABLE orders (
    -- Primary Key
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    order_number varchar(50) NOT NULL UNIQUE,

    -- Order Timeline
    placed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    confirmed_at timestamptz DEFAULT NULL,
    paid_at timestamptz DEFAULT NULL,
    packed_at timestamptz DEFAULT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    cancelled_at timestamptz DEFAULT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'pending',
    cancellation_reason text,

    -- Financial Data
    total_amount numeric(12, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',
    tax_amount numeric(10, 2) DEFAULT 0,
    shipping_cost numeric(10, 2) DEFAULT 0,
    discount_amount numeric(10, 2) DEFAULT 0,

    -- Shipping
    shipping_address_id bigint,
    estimated_delivery_at timestamptz DEFAULT NULL,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('pending', 'confirmed', 'paid', 'packed', 'shipped', 'delivered', 'cancelled', 'refunded')),
    CHECK (total_amount > 0),
    CHECK (confirmed_at IS NULL OR confirmed_at >= placed_at),
    CHECK (paid_at IS NULL OR paid_at >= confirmed_at),
    CHECK (packed_at IS NULL OR packed_at >= paid_at),
    CHECK (shipped_at IS NULL OR shipped_at >= packed_at),
    CHECK (delivered_at IS NULL OR delivered_at >= shipped_at),
    CHECK (cancelled_at IS NULL OR cancelled_at >= placed_at),
    CHECK (estimated_delivery_at IS NULL OR estimated_delivery_at > shipped_at)
);

-- Payments Table
CREATE TABLE payments (
    -- Primary Key
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id),

    -- Payment Details
    payment_method varchar(50) NOT NULL,
    amount numeric(12, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',
    transaction_id varchar(100) UNIQUE,

    -- Payment Timeline
    initiated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    received_at timestamptz DEFAULT NULL,
    failed_at timestamptz DEFAULT NULL,
    refunded_at timestamptz DEFAULT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'pending',
    failure_reason text,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending', 'processing', 'received', 'failed', 'refunded')),
    CHECK (amount > 0),
    CHECK (received_at IS NULL OR received_at >= initiated_at),
    CHECK (failed_at IS NULL OR failed_at >= initiated_at),
    CHECK (refunded_at IS NULL OR refunded_at >= received_at)
);

-- Shipments Table
CREATE TABLE shipments (
    -- Primary Key
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id),

    -- Shipment Details
    carrier varchar(50) NOT NULL,
    tracking_number varchar(100),

    -- Timeline
    prepared_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    shipped_at timestamptz NOT NULL,
    in_transit_at timestamptz DEFAULT NULL,
    out_for_delivery_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    failed_delivery_at timestamptz DEFAULT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'prepared',
    delivery_attempts integer NOT NULL DEFAULT 0,
    last_status_update_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('prepared', 'shipped', 'in_transit', 'out_for_delivery', 'delivered', 'failed_delivery')),
    CHECK (shipped_at >= prepared_at),
    CHECK (in_transit_at IS NULL OR in_transit_at >= shipped_at),
    CHECK (delivered_at IS NULL OR delivered_at >= in_transit_at),
    CHECK (failed_delivery_at IS NULL OR failed_delivery_at >= shipped_at),
    CHECK (delivery_attempts >= 0)
);

-- Refunds Table
CREATE TABLE refunds (
    -- Primary Key
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id),
    payment_id bigint NOT NULL REFERENCES payments(id),

    -- Refund Details
    amount numeric(12, 2) NOT NULL,
    reason varchar(255) NOT NULL,

    -- Timeline
    requested_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    approved_at timestamptz DEFAULT NULL,
    rejected_at timestamptz DEFAULT NULL,
    processed_at timestamptz DEFAULT NULL,
    issued_at timestamptz DEFAULT NULL,
    received_at timestamptz DEFAULT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'requested',

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('requested', 'approved', 'rejected', 'processing', 'issued', 'received')),
    CHECK (amount > 0),
    CHECK (approved_at IS NULL OR approved_at >= requested_at),
    CHECK (rejected_at IS NULL OR rejected_at >= requested_at),
    CHECK (processed_at IS NULL OR processed_at >= approved_at),
    CHECK (issued_at IS NULL OR issued_at >= processed_at),
    CHECK (received_at IS NULL OR received_at >= issued_at)
);

-- Product Reviews Table
CREATE TABLE product_reviews (
    -- Primary Key
    id bigserial PRIMARY KEY,
    product_id bigint NOT NULL REFERENCES products(id),
    order_id bigint NOT NULL REFERENCES orders(id),
    user_id bigint NOT NULL REFERENCES users(id),

    -- Review Data
    rating integer NOT NULL,
    comment text,

    -- Timeline
    submitted_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    verified_at timestamptz DEFAULT NULL,
    approved_at timestamptz DEFAULT NULL,
    rejected_at timestamptz DEFAULT NULL,
    published_at timestamptz DEFAULT NULL,
    unpublished_at timestamptz DEFAULT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'pending_verification',

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (rating BETWEEN 1 AND 5),
    CHECK (status IN ('pending_verification', 'pending_approval', 'approved', 'rejected', 'published')),
    CHECK (verified_at IS NULL OR verified_at >= submitted_at),
    CHECK (approved_at IS NULL OR approved_at >= verified_at),
    CHECK (rejected_at IS NULL OR rejected_at >= submitted_at),
    CHECK (published_at IS NULL OR published_at >= approved_at)
);

-- Coupons Table
CREATE TABLE coupons (
    -- Primary Key
    id bigserial PRIMARY KEY,
    code varchar(50) NOT NULL UNIQUE,

    -- Coupon Details
    discount_percent numeric(5, 2) DEFAULT NULL,
    discount_amount numeric(10, 2) DEFAULT NULL,
    max_uses integer DEFAULT NULL,
    current_uses integer NOT NULL DEFAULT 0,

    -- Validity Period
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    activated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at timestamptz NOT NULL,

    -- Status
    is_active boolean NOT NULL DEFAULT true,
    deactivated_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK ((discount_percent IS NOT NULL OR discount_amount IS NOT NULL)),
    CHECK (discount_percent IS NULL OR discount_percent > 0),
    CHECK (discount_amount IS NULL OR discount_amount > 0),
    CHECK (max_uses IS NULL OR max_uses > 0),
    CHECK (current_uses >= 0),
    CHECK (expires_at > created_at),
    CHECK (activated_at >= created_at),
    CHECK (deactivated_at IS NULL OR deactivated_at >= activated_at)
);

-- Indexes
CREATE INDEX idx_orders_placed_at ON orders(placed_at DESC);
CREATE INDEX idx_orders_user_id_placed_at ON orders(user_id, placed_at DESC);
CREATE INDEX idx_orders_status_placed_at ON orders(status, placed_at DESC);
CREATE INDEX idx_orders_shipped_at ON orders(shipped_at DESC) WHERE shipped_at IS NOT NULL;
CREATE INDEX idx_orders_delivered_at ON orders(delivered_at DESC) WHERE delivered_at IS NOT NULL;

CREATE INDEX idx_payments_received_at ON payments(received_at DESC) WHERE received_at IS NOT NULL;
CREATE INDEX idx_payments_order_id ON payments(order_id, initiated_at DESC);

CREATE INDEX idx_shipments_shipped_at ON shipments(shipped_at DESC);
CREATE INDEX idx_shipments_delivered_at ON shipments(delivered_at DESC) WHERE delivered_at IS NOT NULL;

CREATE INDEX idx_refunds_issued_at ON refunds(issued_at DESC) WHERE issued_at IS NOT NULL;

CREATE INDEX idx_coupons_expires_at ON coupons(expires_at DESC);
CREATE INDEX idx_coupons_active ON coupons(expires_at) WHERE is_active = true AND expires_at > CURRENT_TIMESTAMP;

-- Triggers
CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_payments_updated_at BEFORE UPDATE ON payments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_shipments_updated_at BEFORE UPDATE ON shipments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Useful Queries:

-- Orders placed in the last 7 days
SELECT id, order_number, placed_at, status
FROM orders
WHERE placed_at > CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY placed_at DESC;

-- Pending orders (not yet confirmed after 24 hours)
SELECT id, order_number, placed_at,
    AGE(CURRENT_TIMESTAMP, placed_at) as pending_duration
FROM orders
WHERE status = 'pending'
    AND placed_at < CURRENT_TIMESTAMP - INTERVAL '24 hours'
ORDER BY placed_at ASC;

-- Orders shipped but not delivered (in transit)
SELECT id, order_number, shipped_at,
    AGE(CURRENT_TIMESTAMP, shipped_at) as in_transit_duration
FROM orders
WHERE shipped_at IS NOT NULL AND delivered_at IS NULL
ORDER BY shipped_at ASC;

-- Failed refunds (requested but not processed after 7 days)
SELECT id, amount, requested_at,
    AGE(CURRENT_TIMESTAMP, requested_at) as waiting_duration
FROM refunds
WHERE status IN ('requested', 'approved')
    AND requested_at < CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY requested_at ASC;

-- Active coupons about to expire (within 7 days)
SELECT code, discount_percent, discount_amount, expires_at
FROM coupons
WHERE is_active = true
    AND expires_at <= CURRENT_TIMESTAMP + INTERVAL '7 days'
    AND expires_at > CURRENT_TIMESTAMP
ORDER BY expires_at ASC;

-- Revenue analysis by day
SELECT DATE_TRUNC('day', placed_at)::date as order_date,
    COUNT(*) as total_orders,
    SUM(total_amount) as total_revenue
FROM orders
WHERE delivered_at IS NOT NULL
GROUP BY DATE_TRUNC('day', placed_at)
ORDER BY order_date DESC;
```

---

## 5. Content Publishing Platform {#content-publishing}

### Table 5.1: Content Publishing DateTime Naming Convention

| Column Name            | Purpose                | Pattern         | Status Workflow | When to Use        | Example                                            |
| ---------------------- | ---------------------- | --------------- | --------------- | ------------------ | -------------------------------------------------- |
| `created_at`           | Content creation       | Audit Trail     | Draft           | All content        | Always                                             |
| `draft_saved_at`       | Draft save time        | Publishing      | Draft           | Every auto-save    | `WHERE draft_saved_at > NOW() - INTERVAL '1 hour'` |
| `submitted_at`         | Submit for review      | Business Event  | Submitted       | After user submits | `WHERE submitted_at IS NOT NULL`                   |
| `reviewed_at`          | Reviewer review time   | Business Event  | In Review       | After review       | `WHERE reviewed_at IS NOT NULL`                    |
| `approved_at`          | Editor approval        | Business Event  | Approved        | After approval     | `WHERE approved_at IS NOT NULL`                    |
| `scheduled_publish_at` | Scheduled publish time | Scheduling      | Scheduled       | Future publication | `WHERE scheduled_publish_at > NOW()`               |
| `published_at`         | Content publication    | Publishing      | Published       | After publish      | `WHERE published_at IS NOT NULL`                   |
| `unpublished_at`       | Content removal        | Publishing      | Unpublished     | After unpublish    | `WHERE unpublished_at IS NOT NULL`                 |
| `valid_from`           | Validity start         | Validity Period | All             | Content period     | `WHERE valid_from <= NOW()`                        |
| `valid_until`          | Validity end           | Validity Period | All             | Content period     | `WHERE valid_until > NOW()`                        |

### Complete Content Publishing System

```sql
-- ============================================================================
-- Content Publishing Platform
-- ============================================================================

CREATE TABLE articles (
    -- Primary Key
    id bigserial PRIMARY KEY,
    author_id bigint NOT NULL REFERENCES users(id),

    -- Basic Information
    title varchar(255) NOT NULL,
    slug varchar(255) NOT NULL UNIQUE,
    content text NOT NULL,
    summary text,
    category_id bigint REFERENCES categories(id),

    -- Status
    status varchar(20) NOT NULL DEFAULT 'draft',

    -- Draft Management
    draft_saved_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    draft_version_number integer NOT NULL DEFAULT 1,

    -- Submission & Review Workflow
    submitted_at timestamptz DEFAULT NULL,
    submitted_by varchar(50),

    reviewed_at timestamptz DEFAULT NULL,
    reviewed_by varchar(50),
    review_comments text,

    approved_at timestamptz DEFAULT NULL,
    approved_by varchar(50),

    rejected_at timestamptz DEFAULT NULL,
    rejection_reason text,

    -- Publishing
    scheduled_publish_at timestamptz DEFAULT NULL,
    published_at timestamptz DEFAULT NULL,
    published_version integer,

    unpublished_at timestamptz DEFAULT NULL,
    unpublished_reason text,

    -- Content Validity Period
    valid_from timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    valid_until timestamptz NOT NULL DEFAULT '9999-12-31'::timestamptz,

    -- SEO & Metadata
    seo_title varchar(60),
    seo_description varchar(160),
    tags text[],
    featured_image_url text,
    featured_image_updated_at timestamptz DEFAULT NULL,

    -- Statistics
    view_count integer NOT NULL DEFAULT 0,
    last_viewed_at timestamptz DEFAULT NULL,
    comment_count integer NOT NULL DEFAULT 0,
    share_count integer NOT NULL DEFAULT 0,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('draft', 'submitted', 'reviewing', 'approved', 'scheduled', 'published', 'archived')),
    CHECK (draft_saved_at >= created_at),
    CHECK (submitted_at IS NULL OR submitted_at >= draft_saved_at),
    CHECK (reviewed_at IS NULL OR reviewed_at >= submitted_at),
    CHECK (approved_at IS NULL OR approved_at >= reviewed_at),
    CHECK (rejected_at IS NULL OR rejected_at >= submitted_at),
    CHECK (scheduled_publish_at IS NULL OR scheduled_publish_at > CURRENT_TIMESTAMP),
    CHECK (published_at IS NULL OR published_at >= COALESCE(scheduled_publish_at, approved_at)),
    CHECK (unpublished_at IS NULL OR unpublished_at > published_at),
    CHECK (valid_until > valid_from),
    CHECK (published_at IS NULL OR published_at >= valid_from)
);

-- Article Comments Table
CREATE TABLE article_comments (
    -- Primary Key
    id bigserial PRIMARY KEY,
    article_id bigint NOT NULL REFERENCES articles(id) ON DELETE CASCADE,
    author_id bigint NOT NULL REFERENCES users(id),
    parent_comment_id bigint REFERENCES article_comments(id) ON DELETE CASCADE,

    -- Content
    comment_text text NOT NULL,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'pending_moderation',

    -- Timeline
    submitted_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    approved_at timestamptz DEFAULT NULL,
    rejected_at timestamptz DEFAULT NULL,
    published_at timestamptz DEFAULT NULL,
    edited_at timestamptz DEFAULT NULL,
    deleted_at timestamptz DEFAULT NULL,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending_moderation', 'approved', 'rejected', 'published', 'deleted')),
    CHECK (approved_at IS NULL OR approved_at >= submitted_at),
    CHECK (rejected_at IS NULL OR rejected_at >= submitted_at),
    CHECK (published_at IS NULL OR published_at >= approved_at),
    CHECK (edited_at IS NULL OR edited_at >= published_at),
    CHECK (deleted_at IS NULL OR deleted_at >= published_at)
);

-- Scheduled Publishing Queue
CREATE TABLE publishing_schedule (
    -- Primary Key
    id bigserial PRIMARY KEY,
    article_id bigint NOT NULL REFERENCES articles(id) ON DELETE CASCADE,

    -- Schedule
    scheduled_for timestamptz NOT NULL,

    -- Execution
    executed_at timestamptz DEFAULT NULL,
    failed_at timestamptz DEFAULT NULL,
    failure_reason text,

    -- Status
    status varchar(20) NOT NULL DEFAULT 'pending',

    -- Retry
    retry_count integer NOT NULL DEFAULT 0,
    max_retries integer NOT NULL DEFAULT 3,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending', 'executing', 'executed', 'failed')),
    CHECK (scheduled_for > created_at),
    CHECK (executed_at IS NULL OR executed_at >= scheduled_for),
    CHECK (failed_at IS NULL OR failed_at >= scheduled_for),
    CHECK (retry_count <= max_retries)
);

-- Indexes
CREATE INDEX idx_articles_published_at ON articles(published_at DESC) WHERE published_at IS NOT NULL;
CREATE INDEX idx_articles_author_published_at ON articles(author_id, published_at DESC);
CREATE INDEX idx_articles_status ON articles(status, published_at DESC);
CREATE INDEX idx_articles_scheduled_publish_at ON articles(scheduled_publish_at) WHERE scheduled_publish_at IS NOT NULL AND published_at IS NULL;
CREATE INDEX idx_articles_valid_period ON articles(valid_from, valid_until) WHERE valid_from <= CURRENT_TIMESTAMP AND valid_until > CURRENT_TIMESTAMP;

CREATE INDEX idx_comments_article_published_at ON article_comments(article_id, published_at DESC) WHERE published_at IS NOT NULL;

CREATE INDEX idx_publishing_schedule_pending ON publishing_schedule(scheduled_for ASC) WHERE status = 'pending';

-- Triggers
CREATE TRIGGER update_articles_updated_at BEFORE UPDATE ON articles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_comments_updated_at BEFORE UPDATE ON article_comments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Useful Queries:

-- Draft articles saved in the last hour
SELECT id, title, author_id, draft_saved_at
FROM articles
WHERE status = 'draft'
    AND draft_saved_at > CURRENT_TIMESTAMP - INTERVAL '1 hour'
ORDER BY draft_saved_at DESC;

-- Articles pending review
SELECT id, title, author_id, submitted_at
FROM articles
WHERE status IN ('submitted', 'reviewing')
ORDER BY submitted_at ASC;

-- Articles scheduled for future publication
SELECT id, title, scheduled_publish_at
FROM articles
WHERE status = 'scheduled'
    AND scheduled_publish_at > CURRENT_TIMESTAMP
ORDER BY scheduled_publish_at ASC;

-- Published articles (currently valid)
SELECT id, title, published_at, view_count, last_viewed_at
FROM articles
WHERE published_at IS NOT NULL
    AND valid_from <= CURRENT_TIMESTAMP
    AND valid_until > CURRENT_TIMESTAMP
ORDER BY published_at DESC;

-- Articles not published after 7 days of approval
SELECT id, title, approved_at,
    AGE(CURRENT_TIMESTAMP, approved_at) as approved_duration
FROM articles
WHERE status = 'approved'
    AND approved_at < CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY approved_at ASC;

-- Comments pending moderation
SELECT id, article_id, author_id, submitted_at
FROM article_comments
WHERE status = 'pending_moderation'
ORDER BY submitted_at ASC;

-- Recently unpublished articles
SELECT id, title, published_at, unpublished_at, unpublished_reason
FROM articles
WHERE unpublished_at IS NOT NULL
    AND unpublished_at > CURRENT_TIMESTAMP - INTERVAL '30 days'
ORDER BY unpublished_at DESC;
```

---

## 6. Payment Processing System {#payment-system}

### Table 6.1: Payment DateTime Naming Convention

| Column Name               | Purpose               | Pattern         | Stage      | When to Use            | Example                                          |
| ------------------------- | --------------------- | --------------- | ---------- | ---------------------- | ------------------------------------------------ |
| `initiated_at`            | Payment start         | Process         | Initiated  | When user initiates    | `WHERE initiated_at > NOW() - INTERVAL '1 hour'` |
| `processing_started_at`   | Payment processing    | Process         | Processing | Payment gateway calls  | `WHERE processing_started_at IS NOT NULL`        |
| `processing_completed_at` | Processing finish     | Process         | Processing | After gateway response | `WHERE processing_completed_at IS NOT NULL`      |
| `authorized_at`           | Payment authorization | Process         | Authorized | After auth approved    | `WHERE authorized_at IS NOT NULL`                |
| `captured_at`             | Amount capture        | Process         | Captured   | After amount reserved  | `WHERE captured_at IS NOT NULL`                  |
| `received_at`             | Payment received      | Process         | Received   | Confirmed receipt      | `WHERE received_at IS NOT NULL`                  |
| `settled_at`              | Settlement            | Process         | Settled    | Bank transfer          | `WHERE settled_at IS NOT NULL`                   |
| `failed_at`               | Payment failure       | Process         | Failed     | Payment declined       | `WHERE failed_at IS NOT NULL`                    |
| `refund_issued_at`        | Refund issuance       | Process         | Refunded   | Refund processed       | `WHERE refund_issued_at IS NOT NULL`             |
| `expires_at`              | Card/Token expiry     | Validity Period | N/A        | Token/card validity    | `WHERE expires_at > NOW()`                       |

### Complete Payment Processing System

```sql
-- ============================================================================
-- Payment Processing System
-- ============================================================================

CREATE TABLE payment_methods (
    -- Primary Key
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),

    -- Payment Method Details
    type varchar(20) NOT NULL,  -- 'credit_card', 'bank_account', 'digital_wallet'
    is_primary boolean NOT NULL DEFAULT false,

    -- Card Information (for credit cards)
    card_last_four varchar(4),
    card_brand varchar(20),  -- 'visa', 'mastercard', 'amex'
    card_holder_name varchar(255),
    card_expiry_month integer,
    card_expiry_year integer,

    -- Bank Account (for bank transfers)
    bank_name varchar(100),
    account_number_last_four varchar(4),

    -- Token
    payment_token varchar(255),

    -- Validity & Status
    activated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deactivated_at timestamptz DEFAULT NULL,
    expires_at timestamptz NOT NULL,  -- Card expiry or token expiry
    last_used_at timestamptz DEFAULT NULL,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (type IN ('credit_card', 'bank_account', 'digital_wallet')),
    CHECK (expires_at > created_at),
    CHECK (deactivated_at IS NULL OR deactivated_at > activated_at),
    CHECK (last_used_at IS NULL OR last_used_at >= activated_at)
);

-- Payments Table
CREATE TABLE payments (
    -- Primary Key
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    order_id bigint REFERENCES orders(id),
    payment_method_id bigint REFERENCES payment_methods(id),

    -- Payment Details
    amount numeric(12, 2) NOT NULL,
    currency varchar(3) NOT NULL DEFAULT 'USD',
    transaction_id varchar(100) UNIQUE,

    -- Payment Status
    status varchar(20) NOT NULL DEFAULT 'pending',

    -- Payment Timeline
    initiated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    processing_started_at timestamptz DEFAULT NULL,
    processing_completed_at timestamptz DEFAULT NULL,
    authorized_at timestamptz DEFAULT NULL,
    captured_at timestamptz DEFAULT NULL,
    received_at timestamptz DEFAULT NULL,
    settled_at timestamptz DEFAULT NULL,
    failed_at timestamptz DEFAULT NULL,

    -- Error Information
    failure_code varchar(50),
    failure_reason text,
    failure_attempts integer NOT NULL DEFAULT 0,
    last_failure_at timestamptz DEFAULT NULL,
    next_retry_at timestamptz DEFAULT NULL,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending', 'processing', 'authorized', 'captured', 'received', 'settled', 'failed', 'expired')),
    CHECK (amount > 0),
    CHECK (processing_started_at IS NULL OR processing_started_at >= initiated_at),
    CHECK (processing_completed_at IS NULL OR processing_completed_at >= processing_started_at),
    CHECK (authorized_at IS NULL OR authorized_at >= processing_completed_at),
    CHECK (captured_at IS NULL OR captured_at >= authorized_at),
    CHECK (received_at IS NULL OR received_at >= captured_at),
    CHECK (settled_at IS NULL OR settled_at >= received_at),
    CHECK (failed_at IS NULL OR failed_at >= initiated_at),
    CHECK (failure_attempts >= 0),
    CHECK (last_failure_at IS NULL OR last_failure_at >= initiated_at)
);

-- Refunds Table
CREATE TABLE refunds (
    -- Primary Key
    id bigserial PRIMARY KEY,
    payment_id bigint NOT NULL REFERENCES payments(id),

    -- Refund Details
    amount numeric(12, 2) NOT NULL,
    reason varchar(255) NOT NULL,

    -- Refund Status
    status varchar(20) NOT NULL DEFAULT 'requested',

    -- Refund Timeline
    requested_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    approved_at timestamptz DEFAULT NULL,
    approved_by varchar(50),
    rejected_at timestamptz DEFAULT NULL,
    rejection_reason text,
    processing_started_at timestamptz DEFAULT NULL,
    issued_at timestamptz DEFAULT NULL,
    received_at timestamptz DEFAULT NULL,

    -- Refund Details
    refund_transaction_id varchar(100),

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('requested', 'approved', 'rejected', 'processing', 'issued', 'received')),
    CHECK (amount > 0),
    CHECK (approved_at IS NULL OR approved_at >= requested_at),
    CHECK (rejected_at IS NULL OR rejected_at >= requested_at),
    CHECK (processing_started_at IS NULL OR processing_started_at >= approved_at),
    CHECK (issued_at IS NULL OR issued_at >= processing_started_at),
    CHECK (received_at IS NULL OR received_at >= issued_at)
);

-- Payment Disputes
CREATE TABLE payment_disputes (
    -- Primary Key
    id bigserial PRIMARY KEY,
    payment_id bigint NOT NULL REFERENCES payments(id),

    -- Dispute Details
    dispute_code varchar(50) NOT NULL,
    reason varchar(255) NOT NULL,
    description text,

    -- Dispute Status
    status varchar(20) NOT NULL DEFAULT 'open',

    -- Timeline
    reported_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    acknowledged_at timestamptz DEFAULT NULL,
    under_investigation_at timestamptz DEFAULT NULL,
    resolved_at timestamptz DEFAULT NULL,
    closed_at timestamptz DEFAULT NULL,

    -- Resolution
    resolution_notes text,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('open', 'acknowledged', 'under_investigation', 'resolved', 'closed')),
    CHECK (acknowledged_at IS NULL OR acknowledged_at >= reported_at),
    CHECK (under_investigation_at IS NULL OR under_investigation_at >= acknowledged_at),
    CHECK (resolved_at IS NULL OR resolved_at >= under_investigation_at),
    CHECK (closed_at IS NULL OR closed_at >= resolved_at)
);

-- Indexes
CREATE INDEX idx_payments_initiated_at ON payments(initiated_at DESC);
CREATE INDEX idx_payments_user_id_initiated_at ON payments(user_id, initiated_at DESC);
CREATE INDEX idx_payments_status ON payments(status, initiated_at DESC);
CREATE INDEX idx_payments_received_at ON payments(received_at DESC) WHERE received_at IS NOT NULL;
CREATE INDEX idx_payments_settled_at ON payments(settled_at DESC) WHERE settled_at IS NOT NULL;
CREATE INDEX idx_payments_failed_at ON payments(failed_at DESC) WHERE failed_at IS NOT NULL;

CREATE INDEX idx_refunds_issued_at ON refunds(issued_at DESC) WHERE issued_at IS NOT NULL;
CREATE INDEX idx_refunds_status ON refunds(status, requested_at DESC);

CREATE INDEX idx_payment_methods_user_activated ON payment_methods(user_id, activated_at DESC) WHERE deactivated_at IS NULL;
CREATE INDEX idx_payment_methods_expires_at ON payment_methods(expires_at) WHERE deactivated_at IS NULL;

-- Triggers
CREATE TRIGGER update_payments_updated_at BEFORE UPDATE ON payments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_refunds_updated_at BEFORE UPDATE ON refunds
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Useful Queries:

-- Pending payments (not yet received after 1 hour)
SELECT id, user_id, amount, initiated_at,
    AGE(CURRENT_TIMESTAMP, initiated_at) as pending_duration
FROM payments
WHERE status IN ('pending', 'processing', 'authorized', 'captured')
    AND initiated_at < CURRENT_TIMESTAMP - INTERVAL '1 hour'
ORDER BY initiated_at ASC;

-- Failed payments (retry eligible)
SELECT id, user_id, amount, failed_at, failure_reason, failure_attempts
FROM payments
WHERE status = 'failed'
    AND failure_attempts < 3
    AND (next_retry_at IS NULL OR next_retry_at <= CURRENT_TIMESTAMP)
ORDER BY failed_at ASC;

-- Recently settled payments
SELECT id, user_id, amount, currency, settled_at
FROM payments
WHERE settled_at IS NOT NULL
    AND settled_at > CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY settled_at DESC;

-- Pending refunds (requested but not processed after 3 days)
SELECT id, payment_id, amount, requested_at,
    AGE(CURRENT_TIMESTAMP, requested_at) as waiting_duration
FROM refunds
WHERE status IN ('requested', 'approved', 'processing')
    AND requested_at < CURRENT_TIMESTAMP - INTERVAL '3 days'
ORDER BY requested_at ASC;

-- Open disputes
SELECT id, payment_id, dispute_code, reason, reported_at, status
FROM payment_disputes
WHERE status IN ('open', 'acknowledged', 'under_investigation')
ORDER BY reported_at ASC;

-- Expiring payment methods (within 30 days)
SELECT id, user_id, card_brand, expires_at
FROM payment_methods
WHERE deactivated_at IS NULL
    AND expires_at <= CURRENT_TIMESTAMP + INTERVAL '30 days'
    AND expires_at > CURRENT_TIMESTAMP
ORDER BY expires_at ASC;

-- Payment trends by day
SELECT DATE_TRUNC('day', received_at)::date as payment_date,
    COUNT(*) as total_payments,
    SUM(amount) as total_amount,
    COUNT(DISTINCT user_id) as unique_customers
FROM payments
WHERE received_at IS NOT NULL
GROUP BY DATE_TRUNC('day', received_at)
ORDER BY payment_date DESC;
```

---

## 7. Job Scheduler System {#job-scheduler}

### Table 7.1: Job Scheduler DateTime Naming Convention

| Column Name             | Purpose            | Pattern         | Stage      | When to Use      | Example                                   |
| ----------------------- | ------------------ | --------------- | ---------- | ---------------- | ----------------------------------------- |
| `scheduled_for`         | Job scheduled time | Scheduling      | Scheduled  | Future execution | `WHERE scheduled_for > NOW()`             |
| `queued_at`             | Added to queue     | Process         | Queued     | Job enqueued     | `WHERE queued_at IS NOT NULL`             |
| `started_at`            | Execution start    | Process         | Running    | Job started      | `WHERE started_at IS NOT NULL`            |
| `processing_started_at` | Processing begins  | Process         | Processing | Start processing | `WHERE processing_started_at IS NOT NULL` |
| `completed_at`          | Execution finish   | Process         | Completed  | Job finished     | `WHERE completed_at IS NOT NULL`          |
| `failed_at`             | Job failure        | Process         | Failed     | Job error        | `WHERE failed_at IS NOT NULL`             |
| `next_retry_at`         | Retry schedule     | Scheduling      | Retry      | Schedule retry   | `WHERE next_retry_at <= NOW()`            |
| `retry_until`           | Retry deadline     | Validity Period | Retry      | Stop retrying    | `WHERE retry_until > NOW()`               |
| `created_at`            | Job creation       | Audit Trail     | Created    | Always           | Always                                    |
| `updated_at`            | Last update        | Audit Trail     | Updated    | Always           | Always                                    |

### Complete Job Scheduler System

```sql
-- ============================================================================
-- Job Scheduler System
-- ============================================================================

CREATE TABLE jobs (
    -- Primary Key
    id bigserial PRIMARY KEY,

    -- Job Definition
    job_type varchar(100) NOT NULL,
    job_name varchar(255) NOT NULL,
    job_description text,

    -- Scheduling
    scheduled_for timestamptz NOT NULL,
    run_immediately boolean NOT NULL DEFAULT false,

    -- Retry Configuration
    max_retries integer NOT NULL DEFAULT 3,
    retry_count integer NOT NULL DEFAULT 0,
    retry_delay_seconds integer NOT NULL DEFAULT 300,  -- 5 minutes
    next_retry_at timestamptz DEFAULT NULL,
    retry_until timestamptz DEFAULT NULL,

    -- Job Status
    status varchar(20) NOT NULL DEFAULT 'pending',

    -- Execution Timeline
    queued_at timestamptz DEFAULT NULL,
    started_at timestamptz DEFAULT NULL,
    processing_started_at timestamptz DEFAULT NULL,
    processing_completed_at timestamptz DEFAULT NULL,
    completed_at timestamptz DEFAULT NULL,
    failed_at timestamptz DEFAULT NULL,

    -- Job Data
    payload jsonb,
    result jsonb,
    error_message text,
    error_stack_trace text,

    -- Execution Context
    executed_by varchar(50),
    worker_id varchar(100),

    -- Performance
    execution_duration_ms integer,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('pending', 'queued', 'processing', 'completed', 'failed', 'cancelled')),
    CHECK (retry_count <= max_retries),
    CHECK (scheduled_for >= created_at),
    CHECK (queued_at IS NULL OR queued_at >= created_at),
    CHECK (started_at IS NULL OR started_at >= queued_at),
    CHECK (completed_at IS NULL OR completed_at >= started_at),
    CHECK (failed_at IS NULL OR failed_at >= started_at),
    CHECK (next_retry_at IS NULL OR next_retry_at > failed_at),
    CHECK (retry_until IS NULL OR retry_until > failed_at),
    CHECK (execution_duration_ms IS NULL OR execution_duration_ms > 0)
);

-- Job Execution Logs
CREATE TABLE job_execution_logs (
    -- Primary Key
    id bigserial PRIMARY KEY,
    job_id bigint NOT NULL REFERENCES jobs(id) ON DELETE CASCADE,

    -- Log Details
    log_level varchar(20) NOT NULL,  -- 'info', 'warning', 'error', 'debug'
    log_message text NOT NULL,

    -- Timestamp
    logged_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Context
    execution_step varchar(100),
    context_data jsonb,

    -- Constraints
    CHECK (log_level IN ('debug', 'info', 'warning', 'error'))
);
```
