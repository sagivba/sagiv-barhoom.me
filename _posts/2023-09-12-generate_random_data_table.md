---
layout: post
title:  "Generate random data  in  a table"
author: "Sagiv Barhoom"
date:   2023-09-12
categories: ORACLE 
background: '/img/posts/randomnace.jpg'
---
# Generate random data  in  a table
We can create a random data table using the `CONNECT BY` clause in a SQL query. 
Here's a basic example:

```sql
--DROP TABLE users_demo;

CREATE TABLE users_demo
(
    some_id       INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username      VARCHAR2 (40),
    user_type     INT,                                     -- in the range 1-5
    user_level    INT,                                     -- in the range 1-10000
    status        VARCHAR2(10),                            -- active, blocked, deleted
    create_date   DATE
);

INSERT INTO users_demo (username, user_type, user_level,status,create_date)
        SELECT DBMS_RANDOM.string ('U', TRUNC (DBMS_RANDOM.VALUE (7, 9))) username,
               TRUNC (DBMS_RANDOM.VALUE (1, 5)) user_type,
               TRUNC (DBMS_RANDOM.VALUE (1, 10000)) user_level,
               CASE  round(dbms_random.value(1,10))
                    WHEN 1  THEN 'active'
                    WHEN 2  THEN 'active'
                    WHEN 3  THEN 'active'
                    WHEN 4  THEN 'active'
                    WHEN 5  THEN 'active' 
                    WHEN 6  THEN 'blocked' 
                    ELSE 'deleted' 
               END AS status,
               TO_DATE('2020-01-01', 'YYYY-MM-DD') + DBMS_RANDOM.value(1, 365*2) create_date
          FROM DUAL
          CONNECT BY LEVEL <= 100000;


```

1. We use the CONNECT BY LEVEL construct to generate rows from 1 to 100.
   We can change the LEVEL <= 100 condition to specify the number of rows you want in your table.

## Eamples of DBMS_RANDOM usages

1. `SELECT DBMS_RANDOM.value FROM dual;`  --> returned value in [0,1)
2. `SELECT TRUNC(DBMS_RANDOM.value(1, 101)) FROM dual;` --> random integer in [1,100]
3. `SELECT TO_DATE('2022-01-01', 'YYYY-MM-DD') + DBMS_RANDOM.value(1, 365) FROM dual;`
4. `SELECT DBMS_RANDOM.string('U', 10) FROM dual;` -->  random uppercase string of length 10
5. `SELECT * FROM my_table ORDER BY DBMS_RANDOM.value;` -->Shuffle the rows in a table randomly:
