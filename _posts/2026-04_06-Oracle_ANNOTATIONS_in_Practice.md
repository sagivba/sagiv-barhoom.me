---
layout: post
title: "Oracle ANNOTATIONS in Practice"
author: "Sagiv Barhoom"
date: 2026-04-06
categories: ORACLE
background: '/img/posts/randomnace.jpg'
---

# Working with ANNOTATIONS in Oracle Database

In the previous post, I focused on the difference between `COMMENT ON TABLE`, `COMMENT ON COLUMN`, and `ANNOTATIONS`. This time, I will focus only on `ANNOTATIONS`: how to define them, how to change them, and what kinds of metadata they are useful for in practice.

This post is about Oracle Database, where `ANNOTATIONS` are available starting with 19.28.

The examples here continue with `users_demo` and add a `users_roles` table, so I can show how annotations can be used at both the table and column level, and also to describe business context between tables.

## Why use ANNOTATIONS at all

`ANNOTATIONS` are meant for application metadata.

That means not just human-readable documentation like a comment, but more structured information that can be consumed by code, a UI, an internal tool, or an automated process.

In practice, this metadata can describe things such as:

- how to display a field
- whether a field is used for filters
- whether a field is sensitive
- the role of a table in the model
- whether a column gets its values from a closed set

## Example structure

```sql
CREATE TABLE users_demo
(
    user_id        INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username       VARCHAR2(40),
    status         VARCHAR2(10),
    create_date    DATE
)
ANNOTATIONS (
    ADD business_role 'master'
);

CREATE TABLE users_roles
(
    user_id        INT NOT NULL,
    role_type      VARCHAR2(10) NOT NULL,
    role_code      VARCHAR2(30) NOT NULL,
    assigned_date  DATE,
    CONSTRAINT fk_users_roles_user
        FOREIGN KEY (user_id) REFERENCES users_demo(user_id)
);
```

In this example, `users_demo` is the master table, and `users_roles` is a table associated with the user.

Here the annotation is defined directly in `CREATE TABLE`, but of course annotations can also be added later with `ALTER TABLE`.

## A first short example

```sql
ALTER TABLE users_demo
  ANNOTATIONS (
    ADD business_role 'master'
  );

ALTER TABLE users_roles
  ANNOTATIONS (
    ADD business_role 'details',
    ADD parent_table 'USERS_DEMO'
  );

ALTER TABLE users_roles
  MODIFY role_type ANNOTATIONS (
    ADD business_meaning 'Determines which role set is used'
  );

ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    ADD business_meaning 'Application role code'
  );
```

You can see three different uses here:

- an annotation on a table to describe its role in the model
- an annotation on a table to describe business context
- an annotation on a column to describe application meaning

It is important to remember that an annotation can describe the relationship, but it does not replace the model itself. The actual relationship should still be defined through foreign keys, constraints, and clear naming.

## ADD in practice

`ADD` is the operation that adds an annotation. What actually changes in practice is the annotation name and its value.

So there is no official list of “common ADDs”. What you really see are recurring annotation patterns.

### Annotation as a flag without a value

Sometimes the presence of the annotation is enough:

```sql
ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    ADD ui_hidden
  );
```

### Display label

If you want UI metadata:

```sql
ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    ADD display_label 'Role code'
  );
```

### Business meaning

If you want to describe what the column represents:

```sql
ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    ADD business_meaning 'Application role assigned to a user'
  );
```

### Business context on a table

```sql
ALTER TABLE users_roles
  ANNOTATIONS (
    ADD business_role 'details',
    ADD parent_table 'USERS_DEMO'
  );
```

### Closed value set

If the values are not defined through an FK but still come from a closed set, an annotation can describe that. This is especially useful when the source of the values depends on another column, for example when `role_type` determines which lookup table supplies the values for `role_code`:

```sql
ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    ADD value_set 'USER_ROLE_CODES',
    ADD value_source 'TABLE:lookup_by_role_type',
    ADD closed_set
  );
```

This is useful when another system needs to understand that the column comes from a closed set, even if the database does not enforce that through one direct FK.

If the rule also needs to be enforced, an annotation does not replace a lookup table, a foreign key, or a constraint.

## Change and remove

```sql
ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    REPLACE display_label 'User role'
  );

ALTER TABLE users_roles
  MODIFY role_code ANNOTATIONS (
    DROP ui_hidden
  );
```

This is one of the practical differences compared to a comment: once you work with structured metadata, it becomes easier to manage it consistently.

## How to inspect what was defined

Once annotations are added, they can of course be accessed through the data dictionary.

I will cover that part separately in a follow-up post, including tools that display comments and annotations in a more convenient way.

## When this is actually useful

`ANNOTATIONS` become useful when the same metadata has more than one consumer.

For example:

- a UI that reads metadata for display
- an internal process that builds filters
- code that classifies sensitive fields
- an AI or LLM-based tool that tries to understand a schema more accurately

In these cases, an annotation can be a good place to keep metadata in one consistent location instead of spreading the same information across code, documentation, or configuration.

## What not to do with ANNOTATIONS

It is better not to use annotations as a replacement for the data model.

They do not replace:

- foreign keys
- constraints
- lookup tables
- clear table and column names

They sit on top of the model, not instead of it.

## The practical distinction

If the goal is simply to explain something to a human reader, a comment is usually enough.

If you want structured application metadata that people, code, and tools can consume, an annotation is usually the better choice.

