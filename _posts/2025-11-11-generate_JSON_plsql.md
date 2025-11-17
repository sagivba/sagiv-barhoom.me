---
layout: post
title:  "Generating JSON from Tables Using PL/SQL"
author: "Sagiv Barhoom"
date:   2025-11-11
categories: ORACLE 
background: '/img/posts/json.jpg'
---

# Generating JSON from Tables Using PL/SQL

In this post, we will demonstrate how to generate a structured JSON document from Oracle master–detail tables using PL/SQL.

## Example Scenario

We will use the same Oracle data dictionary views:

- `ALL_USERS` (master): provides information about all database users  
- `ALL_OBJECTS` (details): contains information about all objects accessible to the current user  

We want to build a JSON where each `User` element contains its respective `Objects` array.

### Generate JSON from System Tables

```sql
DECLARE
    l_json CLOB;
BEGIN
    SELECT JSON_OBJECT(
        'Users' VALUE JSON_ARRAYAGG(
            JSON_OBJECT(
                'Username' VALUE u.username,
                'Objects' VALUE JSON_ARRAYAGG(
                    JSON_OBJECT(
                        'Name' VALUE o.object_name,
                        'Type' VALUE o.object_type
                    )
                )
            )
        )
    RETURNING CLOB PRETTY)
    INTO l_json
    FROM all_users u
    JOIN all_objects o ON u.username = o.owner
    WHERE u.username = 'SYS'
      AND o.object_name LIKE 'DBA%'
      AND ROWNUM <= 3
    GROUP BY u.username;

    -- Print JSON in chunks
    DECLARE
        l_offset NUMBER := 1;
        l_chunk  VARCHAR2(4000);
    BEGIN
        WHILE l_offset <= DBMS_LOB.getlength(l_json) LOOP
            l_chunk := DBMS_LOB.SUBSTR(l_json, 4000, l_offset);
            DBMS_OUTPUT.PUT_LINE(l_chunk);
            l_offset := l_offset + 4000;
        END LOOP;
    END;
END;
```

### Output JSON Example

```json
{
  "Users": [
    {
      "Username": "SYS",
      "Objects": [
        {
          "Name": "DBA_2PC_NEIGHBORS",
          "Type": "VIEW"
        },
        {
          "Name": "DBA_ACCHK_EVENTS",
          "Type": "VIEW"
        },
        {
          "Name": "DBA_2PC_PENDING",
          "Type": "VIEW"
        }
      ]
    }
  ]
}
```

### Explanation of JSON Functions Used

#### `JSON_OBJECT`
Creates a JSON object. Each key–value pair is generated using the syntax `'key' VALUE expression`.

#### `JSON_ARRAYAGG`
Aggregates multiple rows into a JSON array.  
Used here to collect all users and all objects under each user.

#### `RETURNING CLOB PRETTY`
Returns the generated JSON as a formatted CLOB (pretty-printed).

#### `DBMS_LOB.SUBSTR`
Reads the JSON output in chunks to avoid buffer size limitations when using `DBMS_OUTPUT`.

### Converting JSON to CLOB

You can directly generate JSON as a CLOB in Oracle by using the `RETURNING CLOB` clause.  
Example:

```sql
DECLARE
    l_json CLOB;
BEGIN
    SELECT JSON_OBJECT(
               'status' VALUE 'ok',
               'timestamp' VALUE SYSTIMESTAMP
           )
    INTO l_json
    FROM dual;

    DBMS_OUTPUT.PUT_LINE(SUBSTR(l_json, 1, 4000));
END;
```

This approach ensures you can safely handle large JSON outputs and write them to files if needed.

### Wrapping JSON Objects with Attributes

You can nest JSON objects or arrays inside others easily:

```sql
DECLARE
    l_inner   CLOB;
    l_wrapped CLOB;
BEGIN
    SELECT JSON_OBJECT(
               'property' VALUE JSON_OBJECT('name' VALUE 'propA', 'value' VALUE 123)
           ) 
    INTO l_inner
    FROM dual;

    SELECT JSON_OBJECT(
               'Properties' VALUE JSON_QUERY(l_inner, '$'),
               'Metadata' VALUE JSON_OBJECT('groupId' VALUE 456, 'enabled' VALUE 'true')
           )
    INTO l_wrapped
    FROM dual;

    DBMS_OUTPUT.PUT_LINE(SUBSTR(l_wrapped, 1, 4000));
END;
```

Produces:

```json
{
  "Properties": {
    "property": {
      "name": "propA",
      "value": 123
    }
  },
  "Metadata": {
    "groupId": 456,
    "enabled": true
  }
}
```

### Tips

- Use `PRETTY` for readable JSON or omit it for compact output.  
- Large JSON data should be written with `UTL_FILE` instead of `DBMS_OUTPUT`.  
- Avoid returning `VARCHAR2` for large JSON strings — always use `CLOB`.  
- Use `NVL` or `COALESCE` to replace `NULL` values where needed.  
- `JSON_OBJECTAGG` can be used for key-value maps instead of arrays when needed.

### References

- [Oracle Database JSON Developer’s Guide 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/adjsn/index.html)  
- [JSON_OBJECT Function](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/JSON_OBJECT.html)  
- [JSON_ARRAYAGG Function](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/JSON_ARRAYAGG.html)  
- [Working with CLOBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_LOB.html)
