---
name: database-sql
description: SQL database design, PostgreSQL/MySQL optimization, indexing strategies, query performance tuning, and migration best practices
tags: [sql, postgresql, mysql, database, optimization]
author: Antigravity Team
version: 1.0.0
---

# SQL Database Skill

Master relational database design and optimization.

## Schema Design

### Normalization
```sql
-- Users table (3NF)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Profiles (1-to-1)
CREATE TABLE profiles (
    user_id INT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    avatar_url TEXT
);

-- Posts (1-to-many)
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    published_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Tags (many-to-many)
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
    post_id INT REFERENCES posts(id) ON DELETE CASCADE,
    tag_id INT REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);
```

## Indexing Strategies

```sql
-- B-tree index (default, equality & range)
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at DESC);

-- Partial index (filtered)
CREATE INDEX idx_posts_published ON posts(published_at) 
WHERE published_at IS NOT NULL;

-- Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- GIN index for JSONB
CREATE INDEX idx_posts_metadata ON posts USING GIN(metadata);

-- Full-text search
CREATE INDEX idx_posts_search ON posts 
USING GIN(to_tsvector('english', title || ' ' || content));
```

## Query Optimization

### EXPLAIN ANALYZE
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.email, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id
ORDER BY post_count DESC
LIMIT 10;
```

### Common Optimizations
```sql
-- Use EXISTS instead of IN for subqueries
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM posts p WHERE p.user_id = u.id
);

-- Use LIMIT with ORDER BY
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;

-- Batch inserts
INSERT INTO posts (user_id, title, content) VALUES
(1, 'Title 1', 'Content 1'),
(1, 'Title 2', 'Content 2'),
(2, 'Title 3', 'Content 3');

-- UPSERT (INSERT ON CONFLICT)
INSERT INTO users (email, name) VALUES ('test@email.com', 'Test')
ON CONFLICT (email) 
DO UPDATE SET name = EXCLUDED.name, updated_at = NOW();
```

## Window Functions

```sql
-- Ranking
SELECT 
    user_id,
    title,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn,
    RANK() OVER (ORDER BY view_count DESC) as popularity_rank
FROM posts;

-- Running totals
SELECT 
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) as running_total,
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7d
FROM daily_sales;
```

## Transactions & Locking

```sql
-- Transaction with savepoint
BEGIN;
SAVEPOINT before_update;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Check constraint violation
DO $$ BEGIN
    IF (SELECT balance FROM accounts WHERE id = 1) < 0 THEN
        ROLLBACK TO before_update;
    END IF;
END $$;

COMMIT;

-- Row-level locking
SELECT * FROM products WHERE id = 1 FOR UPDATE;
SELECT * FROM products WHERE id = 1 FOR UPDATE SKIP LOCKED;
```

## Migrations

```sql
-- Add column with default (PostgreSQL 11+)
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';

-- Rename safely
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Add index concurrently (no lock)
CREATE INDEX CONCURRENTLY idx_posts_title ON posts(title);
```

## Performance Monitoring

```sql
-- Slow queries
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Table sizes
SELECT relname, pg_size_pretty(pg_total_relation_size(relid))
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Index usage
SELECT indexrelname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE idx_scan = 0;  -- Unused indexes
```
