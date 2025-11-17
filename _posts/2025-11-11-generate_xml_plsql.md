---
layout: post
title:  "Generating XML from Tables Using PL/SQL"
author: "Sagiv Barhoom"
date:   2025-11-11
categories: ORACLE 
background: '/img/posts/pivot.jpg'
---

# Generating XML from Tables Using PL/SQL

In this post, we will demonstrate how to generate a structured XML document from Oracle master details tables using PL/SQL. 

## Example Scenario

We will use two Oracle data dictionary views:

- `ALL_USERS` (master): provides information about all database users
- `ALL_OBJECTS` (details): contains information about all objects accessible to the current user

We want to build an XML where each `<User>` element contains its respective `<Objects>`.

### Generate XML from System Tables

```sql
DECLARE
    l_xml XMLTYPE;
BEGIN
    SELECT XMLSERIALIZE(DOCUMENT
        XMLELEMENT("Users",
            XMLAGG(
                XMLELEMENT("User",
                    XMLFOREST(u.username AS "Username"),
                    XMLELEMENT("Objects",
                        XMLAGG(
                            XMLELEMENT("Object",
                                XMLFOREST(o.object_name AS "Name", o.object_type AS "Type")
                            )
                        )
                    )
                )
            )
        ) AS CLOB
    )
    INTO l_xml
    FROM all_users u
    JOIN all_objects o ON u.username = o.owner
    WHERE u.username = 'SYS'
      AND o.object_name LIKE 'DBA%'
      AND ROWNUM <= 3
    GROUP BY u.username;

    -- Print XMLTYPE content in chunks using CLOB
    DECLARE
        l_clob CLOB := l_xml.getClobVal();
        l_offset NUMBER := 1;
        l_chunk  VARCHAR2(4000);
    BEGIN
        WHILE l_offset <= DBMS_LOB.getlength(l_clob) LOOP
            l_chunk := DBMS_LOB.SUBSTR(l_clob, 4000, l_offset);
            DBMS_OUTPUT.PUT_LINE(l_chunk);
            l_offset := l_offset + 4000;
        END LOOP;
    END;
END;
```

### Output XML Example

```xml
<Users>
  <User>
    <Username>SYS</Username>
    <Objects>
      <Object>
        <Name>DBA_2PC_NEIGHBORS</Name>
        <Type>VIEW</Type>
      </Object>
      <Object>
        <Name>DBA_ACCHK_EVENTS</Name>
        <Type>VIEW</Type>
      </Object>
      <Object>
        <Name>DBA_2PC_PENDING</Name>
        <Type>VIEW</Type>
      </Object>
    </Objects>
  </User>
</Users>
```
### Explanation of XML Functions Used

#### `XMLSERIALIZE(DOCUMENT ... AS CLOB)`
Converts the XML structure (of type XMLTYPE) into a CLOB for output or storage.
The DOCUMENT keyword indicates the result is a full XML document, not just a fragment.

#### `XMLELEMENT("Users", ...)`
Creates the root <Users> element that wraps the entire XML content.

#### `XMLAGG(...)`
Aggregates multiple XML fragments into one.
It's used to combine multiple <User> or <Object> elements into their respective parent containers.

#### `XMLELEMENT("User", ...)`
Constructs a <User> element for each user. This acts as a container for user details and nested objects.

#### `XMLFOREST(u.username AS "Username")`
Generates one or more XML elements from column values.
Here, it creates a <Username> element from the u.username field.

#### `XMLELEMENT("Objects", ...)`
Wraps all <Object> elements related to a single user inside an <Objects> container.

#### `XMLELEMENT("Object", ...)`
Creates a single <Object> element for each object that belongs to the user.

#### `XMLFOREST(o.object_name AS "Name", o.object_type AS "Type")`
Produces <Name> and <Type> child elements inside each <Object> element, populated from the corresponding columns.


## Tips

- This example uses a `ROWNUM` filter to limit output for readability.
- For large XMLs, consider writing the CLOB to a file using `UTL_FILE`.

## Debugging and Common Pitfalls

- **NULL values** in XMLFOREST may result in missing XML elements. Use `NVL` or `COALESCE` if you want default values.
- **XMLAGG aggregation issues** can arise if inner queries are not properly grouped. Always verify your GROUP BY clause matches the outer element's key.
- **Output too large** for `DBMS_OUTPUT` may get truncated. Typically, if the XML exceeds a few thousand characters (around 4000 bytes), you'll only see part of it in the console. For full XML, consider writing to a file using `UTL_FILE` or returning the CLOB through a function for external retrieval.
- **Special characters** (like `&`, `<`, `>`) in string values are automatically escaped in XML output, but verify rendering in your target system.
- Use `DBMS_OUTPUT.PUT_LINE(SUBSTR(...))` for controlled output display during debugging. For example:

```sql
FOR i IN 0 .. CEIL(DBMS_LOB.GETLENGTH(l_xml) / 4000) - 1 LOOP
    DBMS_OUTPUT.PUT_LINE(SUBSTR(l_xml, i * 4000 + 1, 4000));
END LOOP;
```

This approach is not needed when using `XMLTYPE`, since `getClobVal()` handles output conversion efficiently.

- If you encounter `ORA-19011: Character string buffer too small`,
  this means Oracle attempted to return more XML data than can fit into a VARCHAR2 buffer.<br/>
  To avoid this:
  - Use `XMLTYPE` instead of VARCHAR2 or raw CLOB when generating XML.
  - Use `getClobVal()` to safely extract the content for display or export.
  - Avoid direct `SUBSTR()` calls on `CLOB`; prefer `DBMS_LOB.SUBSTR()` if needed.
    These approaches ensure you handle large XML without truncation or buffer overflows.

## Converting XMLTYPE to CLOB

You can convert `XMLTYPE` to `CLOB` explicitly using the `getClobVal()` method, which is useful for outputting or storing the XML in plain text format:

```sql
DECLARE
    l_xml  XMLTYPE;
    l_clob CLOB;
BEGIN
    l_xml := XMLTYPE('<root><item>Example</item></root>');
    l_clob := l_xml.getClobVal();
    DBMS_OUTPUT.PUT_LINE(SUBSTR(l_clob, 1, 4000));
END;
```

This method is safe and avoids errors related to buffer overflow.

## Wrapping XMLTYPE inside another XML element

Sometimes you might want to wrap an existing XML fragment inside a parent tag. Use `XMLELEMENT` and pass your `XMLTYPE` as a child:

```sql
DECLARE
    l_inner XMLTYPE;
    l_wrapped XMLTYPE;
BEGIN
    l_inner := XMLTYPE('<item>Content</item>');
    l_wrapped := XMLELEMENT("Wrapper", l_inner);
    DBMS_OUTPUT.PUT_LINE(l_wrapped.getClobVal());
END;
```

This would produce:

```xml
<Wrapper>
  <item>Content</item>
</Wrapper>
```

### Wrapping XMLTYPE with Attributes using XMLATTRIBUTES

If you need to wrap and include attributes, you can use `XMLELEMENT` together with `XMLATTRIBUTES` to construct elements that include metadata:

```sql
DECLARE
    l_inner XMLTYPE;
    l_wrapped XMLTYPE;
BEGIN
    l_inner := XMLTYPE('<property name="propA" value="123"/>');
    l_wrapped := XMLELEMENT("Properties",
        XMLATTRIBUTES('456' AS "groupId", 'true' AS "enabled"),
        l_inner
    );
    DBMS_OUTPUT.PUT_LINE(l_wrapped.getClobVal());
END;
```

Produces:

```xml
<Properties groupId="456" enabled="true">
  <property name="propA" value="123"/>
</Properties>
```

## Oracle Documentation Links

- [Oracle XML DB Developer's Guide 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/adxdb/index.html)
- [Using XML in SQL and PL/SQL](https://docs.oracle.com/en/database/oracle/oracle-database/19/adxdb/using-xmltype.html)
- [DBMS_XMLGEN Package](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_XMLGEN.html)
- [XMLSERIALIZE Function](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/XMLSERIALIZE.html)
