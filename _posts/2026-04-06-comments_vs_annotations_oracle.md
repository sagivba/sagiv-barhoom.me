---
layout: post
title: "COMMENT ON TABLE, COMMENT ON COLUMN, and ANNOTATIONS"
author: "Sagiv Barhoom"
date: 2026-04-06
categories: ORACLE
background: '/img/posts/randomnace.jpg'
---

# COMMENT ON TABLE, COMMENT ON COLUMN, and ANNOTATIONS

This is the first of two posts on this topic, in the context of Oracle Database. `ANNOTATIONS` are a relatively new feature and are available starting with Oracle Database 19c Release Update 19.28. 
In this post, I will focus on the basics: 
what the difference is between `COMMENT ON TABLE`, `COMMENT ON COLUMN`, and `ANNOTATIONS`, and when to use each one.
In the next post, I will go deeper into `ANNOTATIONS`, show how they look in practice, and where they provide more than a regular comment.

In the post on `users_demo`, I used a simple table for demonstration purposes. 
I will use the same table here to explain, in a practical way, the difference between `COMMENT ON TABLE`, `COMMENT ON COLUMN`, and `ANNOTATIONS`.

At first glance, these three mechanisms may look similar, but in practice, they serve different purposes.

## COMMENT ON TABLE

`COMMENT ON TABLE` is used to add a description to a table.

Usually this is a short description that explains what the table represents, what its role is, or how it should be understood when looking at the schema. This is documentation meant mainly for humans.

So if someone opens the schema and wants to understand what `users_demo` is, a table comment is a natural place to put that explanation.

## COMMENT ON COLUMN

`COMMENT ON COLUMN` does the same thing, but at the column level.

Here, the goal is to explain the meaning of a specific column, especially when the column name is not clear enough on its own. This can be a short description of the content, an explanation of expected values, or a business distinction that is not obvious from the name alone.

For example, if there is a column called `status`, a comment can be used to briefly explain which values appear there and what they mean in general.

Again, the main goal here is human-oriented documentation.

## ANNOTATIONS

`ANNOTATIONS` are meant for a different use case.

Instead of free text that someone reads to understand the schema, an annotation stores more structured metadata. 
This information does not have to be only for humans. 
It can also be consumed by code, an internal tool, a UI, or even AI and LLM-based systems.

In other words, a comment explains. An annotation describes metadata.

## When a comment is enough

In most cases, a comment is enough when all you need is to explain the schema to a human reader.

This fits cases such as:

- a short table description
- an explanation of a column meaning
- clarification of unclear names
- documentation meant for readability and maintenance

If no application needs to read this information and act on it, a comment is usually the simple and correct solution.

## When an annotation is better

An annotation is better when the information is not just a description, but application metadata.

For example, if you want to indicate that a column:

- should be displayed in a specific way in the UI
- is used for filters
- is considered a sensitive field
- requires special handling in internal tooling or code

In these cases, free-text comments are less suitable. Structured metadata is usually a better fit.

## A short example

As mentioned, I will use `users_demo` from the earlier post [Generate random data in a table](https://www.sagiv-barhoom.me/oracle/2023/09/12/generate_random_data_table.html).

### Table structure

```sql
CREATE TABLE users_demo
(
    some_id       INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username      VARCHAR2(40),
    user_type     INT,
    user_level    INT,
    status        VARCHAR2(10),
    create_date   DATE
);
```

### Table comment

In a table comment, I would put a short description of what the table represents and in what context it is used.

So this is a good place to explain that `users_demo` is a demo table, but not the place to document every column, list the values of specific fields, or describe application logic.

```sql
COMMENT ON TABLE users_demo IS 'Demo table for generated users';
```

### Column comment

```sql
COMMENT ON COLUMN users_demo.status IS 'User status: A=active, B=blocked, D=deleted';
```

In this example, the comment on `status` gives a short explanation of the values, and that is fine as long as the list is small and simple.

If there is already a defined code list, a lookup table, or a need for some other system to consume the information, it is usually better to keep the comment short and store the detailed meaning elsewhere.

If, for example, you want to say that `status` is used for filters, for UI presentation, or for code-level behavior, an annotation is usually a better fit.

### A short annotation example on the table

```sql
ALTER TABLE users_demo
  ANNOTATIONS (
    ADD business_context 'Demo table used for user-related test data'
  );
```

Here, this is no longer just free text for a human reader. This is metadata with a name and a value.

An annotation can also describe business context, including the relationship between tables, for example that one table is the master of another, or that a table belongs to a certain business domain. Still, it is better to treat this as metadata and not as a replacement for foreign keys, constraints, or the data model itself.

## The simple distinction

`COMMENT` is meant mainly for human documentation.

`ANNOTATIONS` are meant more for structured application metadata, sometimes including metadata that code, a UI, or AI-based tooling can consume.

In the next post, I will go a bit deeper into annotations.

