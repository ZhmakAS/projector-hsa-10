# projector-hsa-10 Isolation Level test;

## How to launch

Execute `docker compose up --build` and open DB client.

## PostgreSQL

### READ COMMITED

```
SHOW TRANSACTION ISOLATION LEVEL;
```

read committed;

```
CREATE TABLE users
(
id   serial primary key,
name VARCHAR(50) NOT NULL,
age  integer     NOT NULL
);
```

```
insert into users (name, age) values ('Alice', 20);
insert into users (name, age) values ('Bob', 25);
```

----> 1. Transaction:

```
BEGIN;
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

----> 2. Transaction:

```
BEGIN;
UPDATE users SET age = 21 WHERE id = 1;
-- no commit here
```

----> 1. Transaction:

```
SELECT age FROM users WHERE id = 1;
COMMIT;
```

In result, we retrieve 20 that meant that dirty read was not detected, as `read commited` isolation level prevents it;

### REPEATABLE READ

----> 1. Transaction:

```
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

----> 2. Transaction:

```
BEGIN;
UPDATE users SET age = 21 WHERE id = 1;
COMMIT;
```

----> 1. Transaction:

```
-- read again
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

It means that not-repeatable read problem was avoided;

### SERIALIZABLE

----> 1. Transaction:

```
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT name FROM users WHERE age > 17;
-- retrieves Alice and Bob
```

----> 2. Transaction:

```
BEGIN;
INSERT INTO users VALUES (3, 'Carol', 26);
COMMIT;
```


----> 1. Transaction:

```
-- read again;
SELECT name FROM users WHERE age > 17;
```

Returns also Bob and Alice without Carol, means that phantom data problem was avoided;


## MySQL

### READ UNCOMMITED

```
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT @@transaction_ISOLATION;
```

READ-UNCOMMITTED;

```
CREATE TABLE users
(
    id   bigint auto_increment primary key,
    name VARCHAR(50) NOT NULL,
    age  smallint    NOT NULL
);
```

```
insert into users (name, age) values ('Alice', 20);
insert into users (name, age) values ('Bob', 25);
```

----> 1. Transaction:

```
BEGIN;
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

----> 2. Transaction:

```
BEGIN;
UPDATE users SET age = 21 WHERE id = 1;
-- no commit here
```

----> 1. Transaction:

```
SELECT age FROM users WHERE id = 1;
COMMIT;
```

In result, we retrieve 21 that meant that dirty read was detected, as `read uncommited` isolation level not prevents it;

### READ COMMITED

```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED ;
SELECT @@transaction_ISOLATION;
```

READ-COMMITTED;


----> 1. Transaction:

```
BEGIN;
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

----> 2. Transaction:

```
BEGIN;
UPDATE users SET age = 21 WHERE id = 1;
-- no commit here
```

----> 1. Transaction:

```
SELECT age FROM users WHERE id = 1;
COMMIT;
```

In result, we retrieve 20 that meant that dirty read was not detected, as `read commited` isolation level prevents it;

### REPEATABLE READ

```
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ ;
SELECT @@transaction_ISOLATION;
```

REPEATABLE-READ;

----> 1. Transaction:

```
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

----> 2. Transaction:

```
BEGIN;
UPDATE users SET age = 21 WHERE id = 1;
COMMIT;
```

----> 1. Transaction:

```
-- read again
SELECT age FROM users WHERE id = 1;
-- retrieves 20
```

It means that not-repeatable read problem was avoided;

### SERIALIZABLE

```
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ ;
SELECT @@transaction_ISOLATION;
```

SERIALIZABLE;

----> 1. Transaction:

```
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT name FROM users WHERE age > 17;
-- retrieves Alice and Bob
```

----> 2. Transaction:

```
BEGIN;
INSERT INTO users VALUES (3, 'Carol', 26);
COMMIT;
```


----> 1. Transaction:

```
-- read again;
SELECT name FROM users WHERE age > 17;
```

Returns also Bob and Alice without Carol, means that phantom data problem was avoided;
