
# Database Schema with SQL Migrations

## Create Schools

```sql
CREATE TABLE schools (
 id UUID PRIMARY KEY,
 name TEXT NOT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Create Users

```sql
CREATE TABLE users (
 id UUID PRIMARY KEY,
 school_id UUID REFERENCES schools(id),
 email TEXT UNIQUE,
 password_hash TEXT
);
```
