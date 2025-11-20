# PostgreSQL DateTime Columns - Naming Rules & Conventions Guide

## Table of Contents

1. [Core Rules for DateTime Columns](#core-rules)
2. [Comprehensive Naming Conventions](#naming-conventions)
3. [Column Design Rules](#column-design-rules)
4. [Constraint Rules](#constraint-rules)
5. [Trigger Patterns](#trigger-patterns)
6. [Index Strategies](#index-strategies)
7. [Testing Guidelines](#testing-guidelines)
8. [Common Patterns & Templates](#patterns)
9. [Anti-Patterns to Avoid](#anti-patterns)
10. [Practical Examples by Use Case](#use-cases)

---

## 1. Core Rules for DateTime Columns {#core-rules}

### Rule 1: Always Use `timestamptz` for Production Tables

**Definition:** `timestamptz` (timestamp with time zone) is the ONLY approved data type for storing datetime information in production databases.

**Why:**

- Stores all data in UTC internally
- Automatically handles timezone conversions
- Prevents data loss during timezone transitions
- Supports DST (Daylight Saving Time) correctly

**Implementation:**

```sql
-- ✅ CORRECT
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- ❌ WRONG
created_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
created_at timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP
```

### Rule 2: Mandatory Audit Columns for All Tables

**Definition:** Every table must have `created_at` and `updated_at` columns for data governance and audit trail.

**Implementation:**

```sql
-- Required for ALL tables
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- Exception: Log/Archive tables may have only created_at
-- Exception: Immutable tables may skip updated_at
```

### Rule 3: Database-Level UTC Timezone Enforcement

**Definition:** The database must be initialized with UTC timezone and explicitly enforced for all users.

**Implementation:**

```sql
-- During database creation
CREATE DATABASE production_db
  WITH OWNER postgres
  ENCODING 'UTF8'
  LC_COLLATE 'en_US.UTF-8'
  LC_CTYPE 'en_US.UTF-8'
  TEMPLATE template0;

ALTER DATABASE production_db SET timezone = 'UTC';

-- For existing databases
ALTER DATABASE production_db SET timezone = 'UTC';

-- For all users
ALTER USER app_user SET timezone = 'UTC';
ALTER USER report_user SET timezone = 'UTC';

-- Verify
SHOW timezone;  -- Must return 'UTC'
```

### Rule 4: Use CURRENT_TIMESTAMP for Default Values

**Definition:** Always use `CURRENT_TIMESTAMP` (or `NOW()`) as the default value for datetime columns, never hardcoded values.

**Implementation:**

```sql
-- ✅ CORRECT
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- ❌ WRONG
created_at timestamptz NOT NULL DEFAULT '2025-11-20'::timestamptz
created_at timestamptz NOT NULL DEFAULT now()  -- Evaluated once at function definition

-- ✅ Also acceptable
created_at timestamptz NOT NULL DEFAULT NOW()
```

### Rule 5: Never Store Local Time, Always Use UTC

**Definition:** All datetime values stored in the database must be in UTC. Conversion to local time happens ONLY on retrieval.

**Implementation:**

```sql
-- ✅ CORRECT: Insert with UTC
INSERT INTO events (started_at, created_at)
VALUES (
    '2025-11-20 10:30:00+00:00'::timestamptz,  -- UTC
    CURRENT_TIMESTAMP
);

-- ✅ CORRECT: Application converts to UTC before insert
-- Python: datetime.now(timezone.utc)
-- Java: Instant.now()
-- JavaScript: new Date().toISOString()

-- ❌ WRONG: Storing local time without timezone
INSERT INTO events (started_at)
VALUES ('2025-11-20 10:30:00');  -- Ambiguous, no timezone info
```

### Rule 6: Index All Frequently Queried DateTime Columns

**Definition:** Any datetime column used in WHERE clauses, JOINs, or ORDER BY must have an index.

**Implementation:**

```sql
-- Rule: At least these indexes are MANDATORY
CREATE INDEX idx_table_created_at ON table_name(created_at DESC);

-- For tables with status/state columns
CREATE INDEX idx_table_status_created_at ON table_name(status, created_at DESC);

-- For soft-delete tables
CREATE INDEX idx_table_deleted_at ON table_name(deleted_at) WHERE deleted_at IS NOT NULL;

-- For range queries
CREATE INDEX idx_table_date_range ON table_name(created_at DESC, updated_at DESC);
```

---

## 2. Comprehensive Naming Conventions {#naming-conventions}

### 2.1 Core Naming Rules for DateTime Columns

**Rule 1: Suffix Convention - Always Use `_at` for DateTime Columns**

All datetime columns MUST end with `_at` to immediately indicate the column contains a timestamp value.

```sql
-- ✅ CORRECT: All use _at suffix
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
deleted_at timestamptz DEFAULT NULL
started_at timestamptz NOT NULL
ended_at timestamptz NOT NULL
scheduled_for timestamptz  -- Exception: For scheduled events
published_at timestamptz
verified_at timestamptz
expired_at timestamptz

-- ❌ WRONG: Missing _at suffix
creation_date timestamp          -- Use created_at
modified_date timestamp          -- Use updated_at
deletion_time timestamp          -- Use deleted_at
start_time timestamp             -- Use started_at
end_time timestamp               -- Use ended_at
publish_date timestamp           -- Use published_at
```

**Rule 2: Use Meaningful Action Verbs Before `_at`**

The text before `_at` should clearly indicate WHAT happened at that time.

```sql
-- ✅ CORRECT: Clear action verbs
created_at      -- Record was created
updated_at      -- Record was updated
deleted_at      -- Record was deleted
started_at      -- Process/event started
ended_at        -- Process/event ended
activated_at    -- Entity became active
deactivated_at  -- Entity became inactive
published_at    -- Content was published
archived_at     -- Record was archived
verified_at     -- Verification occurred
confirmed_at    -- Confirmation happened
shipped_at      -- Shipment occurred
delivered_at    -- Delivery occurred
cancelled_at    -- Cancellation happened
rejected_at     -- Rejection occurred
approved_at     -- Approval happened
submitted_at    -- Submission occurred
processed_at    -- Processing completed
executed_at     -- Execution occurred
synced_at       -- Synchronization occurred
migrated_at     -- Migration completed
backed_up_at    -- Backup occurred
restored_at     -- Restoration occurred
suspended_at    -- Suspension occurred
resumed_at      -- Resumption occurred

-- ❌ WRONG: Ambiguous or unclear verbs
timestamp       -- Too generic
ts              -- Too abbreviated
modified       -- Missing _at
last_change    -- Wrong format
time_field     -- Too vague
dt              -- Too abbreviated
change_at      -- Unclear what changed
update_at      -- Missing 'd'
```

**Rule 3: Prefix for Context/Entity Relationship**

For columns related to a specific entity or process, use the entity name as prefix.

```sql
-- ✅ CORRECT: Entity prefix
order_placed_at         -- Order was placed
order_confirmed_at      -- Order was confirmed
order_shipped_at        -- Order was shipped
order_delivered_at      -- Order was delivered

payment_received_at     -- Payment was received
payment_processed_at    -- Payment was processed
payment_refunded_at     -- Refund was issued

user_created_at         -- User was created
user_activated_at       -- User was activated
user_verified_at        -- User was verified

email_sent_at           -- Email was sent
email_verified_at       -- Email was verified
email_bounced_at        -- Email bounced

notification_sent_at    -- Notification was sent
notification_read_at    -- Notification was read

subscription_started_at -- Subscription started
subscription_ended_at   -- Subscription ended
subscription_renewed_at -- Subscription renewed

-- ❌ WRONG: Missing context prefix
placed_at               -- Placed what? (unclear in some contexts)
confirmed                -- Confirmed what? (missing _at)
shipped_time            -- Missing _at suffix
payment_date            -- Should be payment_received_at
user_time               -- Unclear
email_date              -- Should be email_sent_at
```

**Rule 4: Status-Based Timestamp Naming**

For columns tracking when certain statuses are reached, use status name with `_at`.

```sql
-- ✅ CORRECT: Status-based naming
-- Order processing workflow
order_pending_at        -- When status changed to pending
order_processing_at     -- When status changed to processing
order_completed_at      -- When status changed to completed
order_cancelled_at      -- When status changed to cancelled
order_failed_at         -- When status changed to failed

-- User account workflow
account_pending_at      -- When account is pending
account_active_at       -- When account is active
account_suspended_at    -- When account is suspended
account_closed_at       -- When account is closed

-- Document workflow
document_draft_at       -- When document is in draft
document_review_at      -- When document is in review
document_published_at   -- When document is published
document_archived_at    -- When document is archived

-- ❌ WRONG: Ambiguous status tracking
status_changed_at       -- Which status?
state_updated_at        -- Which state?
is_active_since         -- Unclear format
was_deleted             -- Missing _at
```

### 2.2 Standard DateTime Column Names by Category

#### Audit Trail Columns (Mandatory for All Tables)

```sql
-- These 3 columns MUST exist in every production table

created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
-- Purpose: System-generated timestamp when record is created
-- Properties: NOT NULL, automatic default, immutable after insert
-- Usage: SELECT * FROM users WHERE created_at > '2025-11-01'::date

updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
-- Purpose: Automatically updated when record is modified
-- Properties: NOT NULL, automatic default, updated by trigger
-- Usage: SELECT * FROM users WHERE updated_at > CURRENT_TIMESTAMP - INTERVAL '1 hour'

deleted_at timestamptz DEFAULT NULL
-- Purpose: Soft delete marker
-- Properties: NULL for active, timestamp for deleted
-- Usage: SELECT * FROM users WHERE deleted_at IS NULL
```

#### Lifecycle Timestamp Columns

```sql
-- For tracking process/entity lifecycle

activated_at timestamptz DEFAULT NULL
-- Purpose: When entity became active/enabled
-- Example: User account activation, Feature enablement
-- Properties: NULL until activated, then immutable

deactivated_at timestamptz DEFAULT NULL
-- Purpose: When entity became inactive/disabled
-- Example: User account deactivation, Feature disablement

suspended_at timestamptz DEFAULT NULL
-- Purpose: When entity was temporarily suspended
-- Example: User suspension, Service suspension

resumed_at timestamptz DEFAULT NULL
-- Purpose: When suspended entity was resumed
-- Example: Resume suspended service

archived_at timestamptz DEFAULT NULL
-- Purpose: When entity was archived
-- Example: Old records, completed projects

restored_at timestamptz DEFAULT NULL
-- Purpose: When archived entity was restored
-- Example: Restore archived record
```

#### Business Event Timestamp Columns

```sql
-- For tracking business-critical events

submitted_at timestamptz DEFAULT NULL
-- Purpose: When content/request was submitted
-- Example: Form submission, Request submission, Job submission

approved_at timestamptz DEFAULT NULL
-- Purpose: When content/request was approved
-- Example: Approval of request, Approval of content

rejected_at timestamptz DEFAULT NULL
-- Purpose: When content/request was rejected
-- Example: Rejection of request, Rejection of proposal

confirmed_at timestamptz DEFAULT NULL
-- Purpose: When order/booking was confirmed
-- Example: Order confirmation, Reservation confirmation

cancelled_at timestamptz DEFAULT NULL
-- Purpose: When order/booking was cancelled
-- Example: Order cancellation, Event cancellation

completed_at timestamptz DEFAULT NULL
-- Purpose: When task/order was completed
-- Example: Task completion, Order completion, Project completion

started_at timestamptz NOT NULL
-- Purpose: When event/task started
-- Example: Event start time, Task start time, Shift start time

ended_at timestamptz NOT NULL
-- Purpose: When event/task ended
-- Example: Event end time, Task end time, Shift end time
```

#### Communication/Notification Timestamp Columns

```sql
-- For tracking communication events

sent_at timestamptz DEFAULT NULL
-- Purpose: When message/notification/email was sent
-- Example: Email sent, SMS sent, Notification sent

received_at timestamptz DEFAULT NULL
-- Purpose: When message was received/read
-- Example: Email read, SMS received, Notification read

delivered_at timestamptz DEFAULT NULL
-- Purpose: When message was delivered
-- Example: Email delivered, SMS delivered, Package delivered

opened_at timestamptz DEFAULT NULL
-- Purpose: When message/email was opened
-- Example: Email opened, Attachment opened

clicked_at timestamptz DEFAULT NULL
-- Purpose: When link in message was clicked
-- Example: Email link clicked, Notification link clicked

replied_at timestamptz DEFAULT NULL
-- Purpose: When reply was sent
-- Example: Email replied, Comment replied
```

#### Publishing/Content Timestamp Columns

```sql
-- For content management

published_at timestamptz DEFAULT NULL
-- Purpose: When content was published/went live
-- Example: Article publication, Post publication, Release publication

scheduled_publish_at timestamptz DEFAULT NULL
-- Purpose: When content is scheduled to be published
-- Example: Scheduled post, Scheduled release

unpublished_at timestamptz DEFAULT NULL
-- Purpose: When content was unpublished/removed
-- Example: Article unpublished, Post removed

drafted_at timestamptz DEFAULT NULL
-- Purpose: When content was saved as draft
-- Example: Draft saved, Version saved

reviewed_at timestamptz DEFAULT NULL
-- Purpose: When content was reviewed
-- Example: Code review, Content review, Document review

expires_at timestamptz DEFAULT NULL
-- Purpose: When content/offer expires
-- Example: Coupon expiry, Promotion expiry, License expiry
```

#### Verification/Validation Timestamp Columns

```sql
-- For tracking verification/validation events

verified_at timestamptz DEFAULT NULL
-- Purpose: When entity was verified
-- Example: Email verified, Phone verified, Account verified, Document verified

validation_expires_at timestamptz DEFAULT NULL
-- Purpose: When validation/verification expires
-- Example: Verification link expiry, License renewal date

re_verified_at timestamptz DEFAULT NULL
-- Purpose: When entity was re-verified
-- Example: Re-verification of email, License renewal

failed_verification_at timestamptz DEFAULT NULL
-- Purpose: When verification failed
-- Example: Verification attempt failed
```

#### Scheduling/Planning Timestamp Columns

```sql
-- For scheduled operations

scheduled_for timestamptz NOT NULL
-- Purpose: When something is scheduled to occur
-- Example: Task scheduled, Maintenance scheduled, Appointment scheduled
-- Note: This is the EXCEPTION to _at naming convention

scheduled_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
-- Purpose: When scheduling was recorded
-- Example: When the schedule was created (different from scheduled_for)

due_at timestamptz NOT NULL
-- Purpose: When something is due
-- Example: Task due date, Invoice due date, Payment due date

deadline_at timestamptz NOT NULL
-- Purpose: Final deadline
-- Example: Project deadline, Submission deadline

reminder_at timestamptz DEFAULT NULL
-- Purpose: When reminder should be sent
-- Example: Reminder set for

maintenance_scheduled_for timestamptz DEFAULT NULL
-- Purpose: When maintenance is scheduled
-- Example: Server maintenance scheduled

next_review_at timestamptz DEFAULT NULL
-- Purpose: When next review is scheduled
-- Example: Next performance review, Next audit scheduled
```

#### Integration/Sync Timestamp Columns

```sql
-- For tracking integrations and data sync

synced_at timestamptz DEFAULT NULL
-- Purpose: When data was last synchronized
-- Example: Last sync with external system

imported_at timestamptz DEFAULT NULL
-- Purpose: When data was imported
-- Example: Data import from external source

exported_at timestamptz DEFAULT NULL
-- Purpose: When data was exported
-- Example: Data export to external system

migrated_at timestamptz DEFAULT NULL
-- Purpose: When data was migrated
-- Example: Data migration from old system

backed_up_at timestamptz DEFAULT NULL
-- Purpose: When data was backed up
-- Example: Database backup time

restored_at timestamptz DEFAULT NULL
-- Purpose: When data was restored from backup
-- Example: Restore from backup

last_modified_externally_at timestamptz DEFAULT NULL
-- Purpose: When external system last modified the data
-- Example: Last modification by API, Last sync source update
```

#### Validity Period Timestamp Columns

```sql
-- For temporal/time-series data

effective_from timestamptz NOT NULL
-- Purpose: When data becomes effective/valid
-- Example: Price effective from, Policy effective from

effective_to timestamptz NOT NULL DEFAULT '9999-12-31'::timestamptz
-- Purpose: When data is no longer effective/valid
-- Example: Price effective until, Policy expires

valid_from timestamptz NOT NULL
-- Purpose: When data becomes valid
-- Example: License valid from, Ticket valid from

valid_until timestamptz NOT NULL
-- Purpose: When data is no longer valid
-- Example: License valid until, Ticket valid until

starts_at timestamptz NOT NULL
-- Purpose: When period starts
-- Example: Promotional period start, Subscription period start

ends_at timestamptz NOT NULL
-- Purpose: When period ends
-- Example: Promotional period end, Subscription period end
```

### 2.3 Naming Rules by Data Type Context

#### For Audit/Immutable Timestamps

```sql
-- These columns should NEVER be updated after creation
-- Naming: Standard audit names

created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
created_by varchar(50) NOT NULL DEFAULT current_user
created_ip inet DEFAULT NULL

-- Additional immutable context
created_from_ip inet DEFAULT NULL
created_in_timezone varchar(50) DEFAULT 'UTC'
created_by_user_agent text DEFAULT NULL
```

#### For Mutable/Status-Based Timestamps

```sql
-- These columns change based on business logic
-- Naming: Action-based with clear state

submitted_at timestamptz DEFAULT NULL  -- Can be set once
approved_at timestamptz DEFAULT NULL   -- Can be set once
rejected_at timestamptz DEFAULT NULL   -- Can be set once
cancelled_at timestamptz DEFAULT NULL  -- Can be set once

-- Multiple changes possible
last_reviewed_at timestamptz DEFAULT NULL
last_updated_by_admin_at timestamptz DEFAULT NULL
```

#### For Audit Trail/History Tables

```sql
-- For tables that record historical changes

change_timestamp timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
action_timestamp timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
recorded_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
logged_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
audit_timestamp timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- Note: In audit tables, _at suffix still applies
```

### 2.4 Composite Column Naming

#### For Related DateTime Columns

```sql
-- When multiple related timestamps exist, use consistent prefixes

-- Email workflow
email_sent_at timestamptz DEFAULT NULL
email_delivered_at timestamptz DEFAULT NULL
email_opened_at timestamptz DEFAULT NULL
email_clicked_at timestamptz DEFAULT NULL

-- Order workflow
order_placed_at timestamptz NOT NULL
order_confirmed_at timestamptz DEFAULT NULL
order_shipped_at timestamptz DEFAULT NULL
order_delivered_at timestamptz DEFAULT NULL

-- Document workflow
document_created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
document_submitted_at timestamptz DEFAULT NULL
document_reviewed_at timestamptz DEFAULT NULL
document_approved_at timestamptz DEFAULT NULL
document_published_at timestamptz DEFAULT NULL
document_archived_at timestamptz DEFAULT NULL
```

#### For Paired Validity Periods

```sql
-- Always use consistent naming for period pairs

effective_from timestamptz NOT NULL
effective_to timestamptz NOT NULL

-- OR

valid_from timestamptz NOT NULL
valid_to timestamptz NOT NULL

-- OR

starts_at timestamptz NOT NULL
ends_at timestamptz NOT NULL

-- OR for scheduled operations

scheduled_for timestamptz NOT NULL
scheduled_until timestamptz DEFAULT NULL

-- ❌ WRONG: Inconsistent naming
effective_from timestamptz NOT NULL
valid_to timestamptz NOT NULL  -- Inconsistent with 'from'

started_at timestamptz NOT NULL
end_time timestamptz NOT NULL  -- Inconsistent suffix
```

### 2.5 Naming Rules for Special Cases

#### For Soft Delete Pattern

```sql
-- Always use this exact name for soft deletes
deleted_at timestamptz DEFAULT NULL

-- Optional: Also track who deleted it
deleted_by varchar(50) DEFAULT NULL
deleted_reason text DEFAULT NULL

-- Queries
SELECT * FROM users WHERE deleted_at IS NULL;
SELECT * FROM users WHERE deleted_at IS NOT NULL;
```

#### For Unique Timestamps (One-Time Events)

```sql
-- For events that should happen only once

first_login_at timestamptz DEFAULT NULL
first_purchase_at timestamptz DEFAULT NULL
first_review_at timestamptz DEFAULT NULL

-- Can be updated: False
-- Once set: Should not change unless business allows
```

#### For Latest/Last Timestamps (Can Be Updated)

```sql
-- For tracking most recent occurrence

last_login_at timestamptz DEFAULT NULL      -- Can be updated on each login
last_activity_at timestamptz DEFAULT NULL   -- Updated on each activity
last_payment_at timestamptz DEFAULT NULL    -- Updated on each payment
last_email_sent_at timestamptz DEFAULT NULL -- Updated on each send
last_modified_at timestamptz DEFAULT NULL   -- Same as updated_at
```

#### For User-Provided vs System-Generated Timestamps

```sql
-- User provides the timestamp (in their timezone, but store as UTC)
started_at timestamptz NOT NULL             -- User specifies when it starts
ended_at timestamptz NOT NULL               -- User specifies when it ends
scheduled_for timestamptz NOT NULL          -- User specifies when to schedule

-- System generates the timestamp (always CURRENT_TIMESTAMP)
created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
recorded_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP

-- Mixed: User sets, system tracks
requested_at timestamptz NOT NULL           -- When user made request
processed_at timestamptz DEFAULT NULL       -- When system processed (NULL if not yet)
completed_at timestamptz DEFAULT NULL       -- When system completed (NULL if not yet)
```

### 2.6 Naming Conventions Summary Table

| Category         | Pattern                                  | Example                                   | Immutable        | Nullable        |
| ---------------- | ---------------------------------------- | ----------------------------------------- | ---------------- | --------------- |
| Audit Trail      | `created_at`, `updated_at`, `deleted_at` | `created_at`                              | Yes (created_at) | No (created_at) |
| Lifecycle Events | `{action}_at`                            | `activated_at`, `archived_at`             | No               | Yes             |
| Business Events  | `{entity}_{action}_at`                   | `order_shipped_at`, `payment_received_at` | No               | Yes             |
| Communication    | `{type}_{action}_at`                     | `email_sent_at`, `notification_read_at`   | No               | Yes             |
| Publishing       | `{action}_at`                            | `published_at`, `expires_at`              | No               | Yes             |
| Verification     | `verified_at`                            | `email_verified_at`, `verified_at`        | No               | Yes             |
| Scheduling       | `scheduled_for`                          | `scheduled_for`, `due_at`                 | No               | Yes             |
| Integration      | `{action}_at`                            | `synced_at`, `imported_at`                | No               | Yes             |
| Validity Period  | `{start}_from`/`{end}_to`                | `effective_from`, `effective_to`          | No               | No              |
| Event Timeline   | `{action}_at`                            | `started_at`, `ended_at`                  | No               | No/Yes          |

---

## 3. Column Design Rules {#column-design-rules}

### Rule 1: Audit Columns Are Always NOT NULL with Defaults

```sql
-- ✅ CORRECT: Audit columns with defaults
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    username varchar(50) NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL
);

-- ❌ WRONG: No default or allowing NULL
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    created_at timestamptz,  -- No default, can be NULL
    updated_at timestamptz   -- No default, can be NULL
);
```

### Rule 2: Event Timestamps (NOT NULL, User or System Provided)

```sql
-- For system-provided events
CREATE TABLE audit_log (
    id bigserial PRIMARY KEY,
    event_type varchar(50) NOT NULL,
    occurred_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    recorded_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- For user-provided events
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    order_number varchar(50) NOT NULL UNIQUE,
    placed_at timestamptz NOT NULL,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Rule 3: Optional State Timestamps (Nullable)

```sql
-- For optional state changes
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    status varchar(20) NOT NULL DEFAULT 'pending',
    placed_at timestamptz NOT NULL,
    confirmed_at timestamptz DEFAULT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    cancelled_at timestamptz DEFAULT NULL
);
```

### Rule 4: Validity Period Columns (Effective Date Range)

```sql
-- For temporal data (slowly changing dimensions)
CREATE TABLE product_prices (
    id bigserial PRIMARY KEY,
    product_id bigint NOT NULL,
    price numeric(10, 2) NOT NULL,
    effective_from timestamptz NOT NULL,
    effective_to timestamptz NOT NULL DEFAULT '9999-12-31'::timestamptz,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK (effective_to > effective_from)
);
```

### Rule 5: Soft Delete Pattern

```sql
-- For tables that require data retention
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL
);
```

---

## 4. Constraint Rules {#constraint-rules}

### Rule 1: Temporal Logic Constraints

```sql
-- Rule: Event end must be after event start
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    title varchar(255) NOT NULL,
    started_at timestamptz NOT NULL,
    ended_at timestamptz NOT NULL,
    CHECK (ended_at > started_at)
);

-- Rule: Confirmation cannot happen before creation
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    placed_at timestamptz NOT NULL,
    confirmed_at timestamptz DEFAULT NULL,
    CHECK (confirmed_at IS NULL OR confirmed_at >= placed_at)
);

-- Rule: Deletion cannot happen before creation
CREATE TABLE products (
    id bigserial PRIMARY KEY,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);
```

### Rule 2: Timeline Sequence Constraints

```sql
-- For multi-step processes, ensure chronological order
CREATE TABLE shipments (
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL,
    placed_at timestamptz NOT NULL,
    confirmed_at timestamptz,
    shipped_at timestamptz,
    delivered_at timestamptz,

    CHECK (placed_at <= confirmed_at),
    CHECK (confirmed_at <= shipped_at),
    CHECK (shipped_at <= delivered_at),
    CHECK (delivered_at > placed_at)
);
```

### Rule 3: Validity Period Constraints

```sql
-- For effective date ranges
CREATE TABLE contract_terms (
    id bigserial PRIMARY KEY,
    contract_id bigint NOT NULL,
    effective_from timestamptz NOT NULL,
    effective_to timestamptz NOT NULL,

    CHECK (effective_to > effective_from),
    CHECK (effective_to <= '9999-12-31'::timestamptz)
);
```

### Rule 4: Status-DateTime Consistency

```sql
-- Ensure status and timestamps are consistent
CREATE TABLE job_executions (
    id bigserial PRIMARY KEY,
    job_name varchar(100) NOT NULL,
    status varchar(20) NOT NULL,
    started_at timestamptz,
    completed_at timestamptz,
    error_message text,

    CHECK (status IN ('pending', 'running', 'completed', 'failed')),
    CHECK (
        CASE
            WHEN status = 'pending' THEN started_at IS NULL AND completed_at IS NULL
            WHEN status = 'running' THEN started_at IS NOT NULL AND completed_at IS NULL
            WHEN status IN ('completed', 'failed') THEN started_at IS NOT NULL AND completed_at IS NOT NULL
        END
    )
);
```

---

## 5. Trigger Patterns {#trigger-patterns}

### Pattern 1: Auto-Update Timestamp

```sql
-- Reusable function for all tables
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Apply to each table
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_products_updated_at
    BEFORE UPDATE ON products
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Pattern 2: Prevent Updated_at from Going Backward

```sql
-- Ensure updated_at never decreases
CREATE OR REPLACE FUNCTION validate_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.updated_at < OLD.updated_at THEN
        RAISE EXCEPTION 'updated_at cannot be set to an earlier time';
    END IF;
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql IMMUTABLE;

CREATE TRIGGER validate_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION validate_updated_at_column();
```

### Pattern 3: Audit Log Trigger

```sql
-- Function to log all changes
CREATE OR REPLACE FUNCTION log_record_change()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (
        table_name,
        record_id,
        action,
        old_values,
        new_values,
        changed_by,
        changed_at
    ) VALUES (
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' THEN to_jsonb(OLD) ELSE NULL END,
        CASE WHEN TG_OP = 'DELETE' THEN NULL ELSE to_jsonb(NEW) END,
        current_user,
        CURRENT_TIMESTAMP
    );

    RETURN CASE WHEN TG_OP = 'DELETE' THEN OLD ELSE NEW END;
END;
$$ LANGUAGE plpgsql;

-- Create trigger for each important table
CREATE TRIGGER log_users_changes
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION log_record_change();

CREATE TRIGGER log_orders_changes
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION log_record_change();
```

### Pattern 4: Check DateTime Constraints

```sql
-- Enforce temporal constraints in trigger
CREATE OR REPLACE FUNCTION check_order_timeline()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.confirmed_at IS NOT NULL AND NEW.confirmed_at < NEW.placed_at THEN
        RAISE EXCEPTION 'confirmed_at cannot be before placed_at';
    END IF;

    IF NEW.shipped_at IS NOT NULL AND NEW.shipped_at < COALESCE(NEW.confirmed_at, NEW.placed_at) THEN
        RAISE EXCEPTION 'shipped_at must be after confirmed_at';
    END IF;

    IF NEW.delivered_at IS NOT NULL AND NEW.delivered_at < NEW.shipped_at THEN
        RAISE EXCEPTION 'delivered_at must be after shipped_at';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_order_timeline_insert_update
    BEFORE INSERT OR UPDATE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION check_order_timeline();
```

---

## 6. Index Strategies {#index-strategies}

### Rule 1: Mandatory Indexes

```sql
-- EVERY table must have these indexes at minimum:

-- 1. Descending index on created_at for recent records first
CREATE INDEX idx_table_created_at_desc ON table_name(created_at DESC);

-- 2. For tables with deleted_at, index soft-deleted records
CREATE INDEX idx_table_deleted_at_is_not_null
    ON table_name(deleted_at)
    WHERE deleted_at IS NOT NULL;

-- 3. For tables queried by status, combine with created_at
CREATE INDEX idx_table_status_created_at
    ON table_name(status, created_at DESC);
```

### Rule 2: Composite Indexes for Common Queries

```sql
-- For user-focused queries
CREATE INDEX idx_orders_user_id_placed_at
    ON orders(user_id, placed_at DESC);

-- For date range queries combined with status
CREATE INDEX idx_orders_placed_at_status
    ON orders(placed_at DESC, status);

-- For temporal range queries
CREATE INDEX idx_prices_effective_from_to
    ON product_prices(effective_from, effective_to);
```

### Rule 3: Partial Indexes for Filtered Queries

```sql
-- For active records only
CREATE INDEX idx_users_active_created_at
    ON users(created_at DESC)
    WHERE deleted_at IS NULL;

-- For pending items
CREATE INDEX idx_orders_pending_placed_at
    ON orders(placed_at DESC)
    WHERE status = 'pending';

-- For future scheduled items
CREATE INDEX idx_events_future_scheduled
    ON events(scheduled_for)
    WHERE scheduled_for > CURRENT_TIMESTAMP;
```

### Rule 4: Index Performance Considerations

```sql
-- ✅ Use DESC for recent records first
CREATE INDEX idx_created_at_desc ON table_name(created_at DESC);

-- Avoid: No index on large range queries
-- SELECT * FROM logs WHERE created_at > '2025-01-01'  -- Should have index

-- Use DATE_TRUNC only if indexed properly
CREATE INDEX idx_created_date ON table_name(DATE_TRUNC('day', created_at));

-- For BETWEEN queries
CREATE INDEX idx_date_range ON table_name(created_at)
WHERE created_at >= CURRENT_DATE - INTERVAL '1 year';
```

---

## 7. Testing Guidelines {#testing-guidelines}

### Test Rule 1: Timezone Consistency

```sql
-- Test 1: Verify all tables store in UTC
SELECT
    schemaname,
    tablename,
    tableowner
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;

-- Test 2: Insert in different timezones and verify UTC storage
SET timezone = 'UTC';
INSERT INTO test_table (created_at) VALUES (CURRENT_TIMESTAMP)
RETURNING created_at;

SET timezone = 'America/New_York';
INSERT INTO test_table (created_at) VALUES (CURRENT_TIMESTAMP)
RETURNING created_at;

SET timezone = 'Asia/Bangkok';
INSERT INTO test_table (created_at) VALUES (CURRENT_TIMESTAMP)
RETURNING created_at;

-- All should show same UTC times despite different session timezones
SET timezone = 'UTC';
SELECT created_at FROM test_table ORDER BY id;
```

### Test Rule 2: Constraint Validation

```sql
-- Test timeline constraints
BEGIN TRANSACTION;

-- This should FAIL: ended_at before started_at
INSERT INTO events (title, started_at, ended_at)
VALUES ('Bad Event', '2025-11-21 10:00:00+00:00', '2025-11-21 09:00:00+00:00');
-- Expected: ERROR violates check constraint

-- This should SUCCEED: ended_at after started_at
INSERT INTO events (title, started_at, ended_at)
VALUES ('Good Event', '2025-11-21 09:00:00+00:00', '2025-11-21 10:00:00+00:00');
-- Expected: INSERT 0 1

ROLLBACK;
```

### Test Rule 3: Trigger Execution

```sql
-- Test that updated_at trigger works
BEGIN TRANSACTION;

INSERT INTO users (username, email, password_hash)
VALUES ('test_user', 'test@example.com', 'hash')
RETURNING id, username, created_at, updated_at;

-- Verify created_at = updated_at initially
SELECT created_at, updated_at, created_at = updated_at as are_equal
FROM users
WHERE username = 'test_user';

-- Update the record
UPDATE users SET email = 'new@example.com' WHERE username = 'test_user';

-- Verify updated_at changed but created_at didn't
SELECT
    created_at,
    updated_at,
    created_at < updated_at as updated_timestamp_changed
FROM users
WHERE username = 'test_user';

ROLLBACK;
```

### Test Rule 4: Index Usage

```sql
-- Verify indexes are used for datetime queries
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '7 days'
ORDER BY created_at DESC;

-- Should show: Index Scan using idx_orders_created_at_desc

-- Test performance with and without index
EXPLAIN (ANALYZE, BUFFERS)
SELECT COUNT(*) FROM large_table
WHERE created_at >= '2025-11-01'::timestamptz
  AND created_at < '2025-12-01'::timestamptz;
```

### Test Rule 5: Soft Delete Logic

```sql
-- Test soft delete functionality
BEGIN TRANSACTION;

INSERT INTO users (username, email, password_hash)
VALUES ('user_to_delete', 'user@example.com', 'hash')
RETURNING id;

-- Soft delete
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE username = 'user_to_delete'
RETURNING id, deleted_at;

-- Query should only return active users
SELECT COUNT(*) as active_users FROM users WHERE deleted_at IS NULL;

-- Query should only return deleted users
SELECT COUNT(*) as deleted_users FROM users WHERE deleted_at IS NOT NULL;

ROLLBACK;
```

---

## 8. Common Patterns & Templates {#patterns}

### Pattern: Standard Table with Audit Columns

```sql
CREATE TABLE standard_table (
    -- Primary Key
    id bigserial PRIMARY KEY,

    -- Business Columns
    name varchar(255) NOT NULL,
    description text,

    -- Status Column
    status varchar(20) NOT NULL DEFAULT 'active',

    -- Audit Columns (MANDATORY)
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    UNIQUE(name),
    CHECK (status IN ('active', 'inactive')),
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);

-- Indexes (MANDATORY)
CREATE INDEX idx_standard_table_created_at
    ON standard_table(created_at DESC);
CREATE INDEX idx_standard_table_deleted_at
    ON standard_table(deleted_at)
    WHERE deleted_at IS NOT NULL;
CREATE INDEX idx_standard_table_status_created_at
    ON standard_table(status, created_at DESC);

-- Trigger (MANDATORY)
CREATE TRIGGER update_standard_table_updated_at
    BEFORE UPDATE ON standard_table
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Pattern: Event/Order Timeline Table

```sql
CREATE TABLE order_timeline (
    -- Primary Key
    id bigserial PRIMARY KEY,
    order_id bigint NOT NULL REFERENCES orders(id),

    -- Timeline Columns
    placed_at timestamptz NOT NULL,
    confirmed_at timestamptz DEFAULT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    cancelled_at timestamptz DEFAULT NULL,

    -- Audit Columns
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints (Chronological Order)
    CHECK (placed_at IS NOT NULL),
    CHECK (confirmed_at IS NULL OR confirmed_at >= placed_at),
    CHECK (shipped_at IS NULL OR shipped_at >= confirmed_at),
    CHECK (delivered_at IS NULL OR delivered_at >= shipped_at),
    CHECK (cancelled_at IS NULL OR cancelled_at >= placed_at)
);

-- Indexes
CREATE INDEX idx_order_timeline_placed_at
    ON order_timeline(placed_at DESC);
CREATE INDEX idx_order_timeline_order_id_placed_at
    ON order_timeline(order_id, placed_at DESC);
```

### Pattern: Temporal Data (SCD Type 2)

```sql
CREATE TABLE product_prices_history (
    -- Primary Key
    id bigserial PRIMARY KEY,
    product_id bigint NOT NULL REFERENCES products(id),

    -- Pricing Data
    price numeric(10, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',

    -- Validity Period
    effective_from timestamptz NOT NULL,
    effective_to timestamptz NOT NULL DEFAULT '9999-12-31'::timestamptz,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by varchar(50) NOT NULL,

    -- Constraints
    CHECK (effective_to > effective_from),
    CHECK (price > 0)
);

-- Indexes
CREATE INDEX idx_prices_product_id_effective_from
    ON product_prices_history(product_id, effective_from DESC);
CREATE INDEX idx_prices_effective_from_to
    ON product_prices_history(effective_from, effective_to);
CREATE INDEX idx_prices_product_current
    ON product_prices_history(product_id)
    WHERE effective_to = '9999-12-31'::timestamptz;
```

### Pattern: Audit Log Table

```sql
CREATE TABLE audit_log (
    -- Primary Key
    id bigserial PRIMARY KEY,

    -- Audit Information
    table_name varchar(100) NOT NULL,
    record_id bigint NOT NULL,
    action varchar(10) NOT NULL,

    -- Data Changes
    old_values jsonb,
    new_values jsonb,

    -- User Information
    changed_by varchar(50) NOT NULL,
    changed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ip_address inet,
    user_agent text,

    -- Constraints
    CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
    CHECK (
        CASE
            WHEN action = 'INSERT' THEN old_values IS NULL AND new_values IS NOT NULL
            WHEN action = 'UPDATE' THEN old_values IS NOT NULL AND new_values IS NOT NULL
            WHEN action = 'DELETE' THEN old_values IS NOT NULL AND new_values IS NULL
        END
    )
);

-- Indexes (READ OPTIMIZED)
CREATE INDEX idx_audit_log_changed_at
    ON audit_log(changed_at DESC);
CREATE INDEX idx_audit_log_table_record
    ON audit_log(table_name, record_id, changed_at DESC);
CREATE INDEX idx_audit_log_changed_by
    ON audit_log(changed_by, changed_at DESC);
```

---

## 9. Anti-Patterns to Avoid {#anti-patterns}

### Anti-Pattern 1: Using `timestamp` Instead of `timestamptz`

```sql
-- ❌ WRONG: No timezone information
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    event_time timestamp NOT NULL
);

-- Problem:
-- - Can't properly handle DST transitions
-- - Ambiguous during clock changes
-- - No timezone context
-- - Data becomes meaningless across regions

-- ✅ CORRECT
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    event_time timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Anti-Pattern 2: Multiple Timestamp Columns with Unclear Purpose

```sql
-- ❌ WRONG: Redundant and confusing
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    created_date DATE,
    created_time TIME,
    creation_timestamp TIMESTAMP,
    insert_time TIMESTAMP,
    recorded_at TIMESTAMPTZ,
    modified_date DATE
);

-- Problem: Which one should be used? Maintenance nightmare.

-- ✅ CORRECT
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Anti-Pattern 3: Storing Timestamps as Strings

```sql
-- ❌ WRONG: Text representation of time
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    event_date VARCHAR(50),  -- "November 20, 2025"
    event_time VARCHAR(50)   -- "10:30 AM"
);

-- Problem:
-- - Can't query by date ranges efficiently
-- - No timezone information
-- - Vulnerable to parsing errors
-- - Takes more storage

-- ✅ CORRECT
CREATE TABLE events (
    id bigserial PRIMARY KEY,
    event_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Anti-Pattern 4: Hardcoded Dates in Queries

```sql
-- ❌ WRONG: Hardcoded date without timezone
SELECT * FROM orders
WHERE placed_at > '2025-11-20'
ORDER BY placed_at DESC;

-- Problem: Ambiguous date, depends on timezone

-- ✅ CORRECT
SELECT * FROM orders
WHERE placed_at > '2025-11-20'::timestamptz AT TIME ZONE 'UTC'
ORDER BY placed_at DESC;

-- ✅ BETTER: Use parameterized queries
SELECT * FROM orders
WHERE placed_at > CURRENT_TIMESTAMP - INTERVAL '30 days'
ORDER BY placed_at DESC;
```

### Anti-Pattern 5: Neglecting Indexes

```sql
-- ❌ WRONG: No indexes on frequently queried datetime columns
CREATE TABLE logs (
    id bigserial PRIMARY KEY,
    message text,
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- This query will do a full table scan
SELECT * FROM logs
WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '1 day';

-- ✅ CORRECT: Add appropriate indexes
CREATE INDEX idx_logs_created_at ON logs(created_at DESC);

-- Now the query is efficient
EXPLAIN ANALYZE
SELECT * FROM logs
WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '1 day';
-- Index Scan using idx_logs_created_at
```

### Anti-Pattern 6: Forgetting Soft Delete Filter

```sql
-- ❌ WRONG: Queries return deleted records
SELECT COUNT(*) FROM users;
SELECT * FROM users WHERE email = 'user@example.com';

-- Problem: Returns both active and deleted users

-- ✅ CORRECT: Always filter soft-deleted records
SELECT COUNT(*) FROM users WHERE deleted_at IS NULL;
SELECT * FROM users WHERE email = 'user@example.com' AND deleted_at IS NULL;

-- ✅ BEST: Create a view
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

SELECT * FROM active_users WHERE email = 'user@example.com';
```

### Anti-Pattern 7: Not Validating Timeline Logic

```sql
-- ❌ WRONG: No constraint validation
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    placed_at timestamptz,
    shipped_at timestamptz,
    delivered_at timestamptz
    -- No validation!
);

-- Problem: Can insert impossible sequences
INSERT INTO orders (placed_at, shipped_at, delivered_at)
VALUES ('2025-11-21', '2025-11-20', '2025-11-22');  -- Shipped before placed!

-- ✅ CORRECT: Add constraints
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    placed_at timestamptz NOT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    CHECK (shipped_at IS NULL OR shipped_at >= placed_at),
    CHECK (delivered_at IS NULL OR delivered_at >= shipped_at)
);
```

---

## 10. Practical Examples by Use Case {#use-cases}

### Use Case 1: User Management System

```sql
CREATE TABLE users (
    id bigserial PRIMARY KEY,
    username varchar(50) NOT NULL UNIQUE,
    email varchar(100) NOT NULL UNIQUE,
    password_hash varchar(255) NOT NULL,

    -- Status tracking
    status varchar(20) NOT NULL DEFAULT 'pending_verification',
    email_verified_at timestamptz DEFAULT NULL,
    activated_at timestamptz DEFAULT NULL,
    last_login_at timestamptz DEFAULT NULL,

    -- Audit columns
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('pending_verification', 'active', 'suspended', 'deactivated')),
    CHECK (email_verified_at IS NULL OR email_verified_at >= created_at),
    CHECK (activated_at IS NULL OR activated_at >= COALESCE(email_verified_at, created_at)),
    CHECK (deleted_at IS NULL OR deleted_at >= created_at)
);

CREATE INDEX idx_users_created_at ON users(created_at DESC);
CREATE INDEX idx_users_email_verified_at ON users(email_verified_at) WHERE email_verified_at IS NOT NULL;
CREATE INDEX idx_users_last_login_at ON users(last_login_at DESC) WHERE last_login_at IS NOT NULL;
CREATE INDEX idx_users_deleted_at ON users(deleted_at) WHERE deleted_at IS NOT NULL;

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Use Case 2: E-Commerce Orders

```sql
CREATE TABLE orders (
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    order_number varchar(50) NOT NULL UNIQUE,

    -- Status and timeline
    status varchar(20) NOT NULL DEFAULT 'pending',
    placed_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    confirmed_at timestamptz DEFAULT NULL,
    payment_received_at timestamptz DEFAULT NULL,
    packed_at timestamptz DEFAULT NULL,
    shipped_at timestamptz DEFAULT NULL,
    delivered_at timestamptz DEFAULT NULL,
    returned_at timestamptz DEFAULT NULL,
    cancelled_at timestamptz DEFAULT NULL,

    -- Order data
    total_amount numeric(12, 2) NOT NULL,
    currency varchar(3) DEFAULT 'USD',

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending', 'confirmed', 'paid', 'packed', 'shipped', 'delivered', 'returned', 'cancelled')),
    CHECK (total_amount > 0),
    CHECK (confirmed_at IS NULL OR confirmed_at >= placed_at),
    CHECK (payment_received_at IS NULL OR payment_received_at >= confirmed_at),
    CHECK (packed_at IS NULL OR packed_at >= payment_received_at),
    CHECK (shipped_at IS NULL OR shipped_at >= packed_at),
    CHECK (delivered_at IS NULL OR delivered_at >= shipped_at),
    CHECK (returned_at IS NULL OR returned_at > delivered_at),
    CHECK (cancelled_at IS NULL OR cancelled_at >= placed_at)
);

CREATE INDEX idx_orders_user_id_placed_at ON orders(user_id, placed_at DESC);
CREATE INDEX idx_orders_placed_at ON orders(placed_at DESC);
CREATE INDEX idx_orders_status_placed_at ON orders(status, placed_at DESC);
CREATE INDEX idx_orders_shipped_at ON orders(shipped_at DESC) WHERE shipped_at IS NOT NULL;

CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Use Case 3: Content Publishing

```sql
CREATE TABLE articles (
    id bigserial PRIMARY KEY,
    author_id bigint NOT NULL REFERENCES users(id),
    title varchar(255) NOT NULL,
    slug varchar(255) NOT NULL UNIQUE,
    content text NOT NULL,

    -- Publishing workflow
    status varchar(20) NOT NULL DEFAULT 'draft',
    draft_saved_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    submitted_at timestamptz DEFAULT NULL,
    reviewed_at timestamptz DEFAULT NULL,
    published_at timestamptz DEFAULT NULL,
    scheduled_publish_at timestamptz DEFAULT NULL,
    unpublished_at timestamptz DEFAULT NULL,

    -- Validity period
    valid_from timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    valid_until timestamptz DEFAULT '9999-12-31'::timestamptz,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamptz DEFAULT NULL,

    -- Constraints
    CHECK (status IN ('draft', 'submitted', 'reviewing', 'published', 'scheduled', 'archived')),
    CHECK (submitted_at IS NULL OR submitted_at >= draft_saved_at),
    CHECK (reviewed_at IS NULL OR reviewed_at >= submitted_at),
    CHECK (published_at IS NULL OR published_at >= COALESCE(reviewed_at, submitted_at)),
    CHECK (scheduled_publish_at IS NULL OR scheduled_publish_at > CURRENT_TIMESTAMP),
    CHECK (unpublished_at IS NULL OR unpublished_at > published_at),
    CHECK (valid_from < valid_until)
);

CREATE INDEX idx_articles_published_at ON articles(published_at DESC) WHERE published_at IS NOT NULL;
CREATE INDEX idx_articles_author_published_at ON articles(author_id, published_at DESC);
CREATE INDEX idx_articles_valid_from_until ON articles(valid_from, valid_until);
CREATE INDEX idx_articles_status ON articles(status, published_at DESC);

CREATE TRIGGER update_articles_updated_at BEFORE UPDATE ON articles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Use Case 4: Job Scheduler

```sql
CREATE TABLE scheduled_jobs (
    id bigserial PRIMARY KEY,
    job_name varchar(100) NOT NULL,
    job_type varchar(50) NOT NULL,

    -- Scheduling
    scheduled_for timestamptz NOT NULL,
    retry_until timestamptz DEFAULT NULL,

    -- Execution timeline
    started_at timestamptz DEFAULT NULL,
    completed_at timestamptz DEFAULT NULL,
    failed_at timestamptz DEFAULT NULL,

    -- Result
    status varchar(20) NOT NULL DEFAULT 'pending',
    result jsonb DEFAULT NULL,
    error_message text DEFAULT NULL,
    retry_count integer NOT NULL DEFAULT 0,

    -- Audit
    created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Constraints
    CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
    CHECK (started_at IS NULL OR started_at >= scheduled_for),
    CHECK (completed_at IS NULL OR (completed_at > started_at AND status = 'completed')),
    CHECK (failed_at IS NULL OR (failed_at >= started_at AND status = 'failed')),
    CHECK (retry_until IS NULL OR retry_until > scheduled_for),
    CHECK (retry_count >= 0)
);

CREATE INDEX idx_scheduled_jobs_scheduled_for ON scheduled_jobs(scheduled_for ASC);
CREATE INDEX idx_scheduled_jobs_status_scheduled_for ON scheduled_jobs(status, scheduled_for);
CREATE INDEX idx_scheduled_jobs_started_at ON scheduled_jobs(started_at DESC) WHERE started_at IS NOT NULL;
CREATE INDEX idx_scheduled_jobs_pending ON scheduled_jobs(scheduled_for) WHERE status = 'pending';

CREATE TRIGGER update_scheduled_jobs_updated_at BEFORE UPDATE ON scheduled_jobs
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

---

## Summary: The 10 Golden Rules

1. **Use `timestamptz` for everything** - No exceptions
2. **Add `created_at` and `updated_at` to every table** - Non-negotiable
3. **Set database timezone to UTC** - During initialization
4. **Use `CURRENT_TIMESTAMP` as defaults** - Never hardcoded values
5. **Index all datetime columns** - Critical for performance
6. **Add temporal constraints** - Validate business logic
7. **Use triggers for auto-updates** - Keep `updated_at` synchronized
8. **Filter soft-deleted records** - Never forget the `WHERE deleted_at IS NULL`
9. **Test across timezones** - Ensure consistency
10. **Follow strict naming conventions** - Use `_at` suffix, meaningful action verbs, and clear prefixes

These rules ensure data integrity, consistency, and maintainability across your entire database system.
