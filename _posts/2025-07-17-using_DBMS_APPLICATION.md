---   
layout: post
title: "Tracking Session Activity with DBMS_APPLICATION_INFO"
subtitle: "Collecting info using DBMS_APPLICATION_INFO for ASH reprot"
date: 2025-07-17
background: '/img/posts/engine.jpg'
categories: Oracle
---  

# Tracking Session Activity with DBMS_APPLICATION_INFO

Use `DBMS_APPLICATION_INFO` to label what a session is doing. Helps with performance analysis using ASH reports.

## What It Does

- Sets `MODULE` and `ACTION` in `V$SESSION`
- Useful for tracking session activity
- Helps identify bottlenecks or long-running steps

### Definitions
- **MODULE**: High-level program name or component (e.g. 'OrderProcessing')
- **ACTION**: Specific task or step within the module (e.g. 'InsertOrder')
- **CLIENT_INFO**:Identify the user, department, or application component
- **ASH Report**: Shows session activity sampled over time. Helps identify performance issues by analyzing what sessions were doing at different moments. stands for *A*ctive *S*ession *H*istory report.

## Example

```plsql
BEGIN
  DBMS_APPLICATION_INFO.SET_MODULE('OrderProcessing', 'InsertOrder');
  
  DBMS_APPLICATION_INFO.SET_CLIENT_INFO('SagivClnt');

INSERT INTO orders (order_id, customer_id, order_date)
  VALUES (1001, 200, SYSDATE);

  DBMS_APPLICATION_INFO.SET_ACTION('OrderInserted');

  UPDATE inventory   SET quantity = quantity - 1  WHERE product_id = 300;

  DBMS_APPLICATION_INFO.SET_ACTION('InventoryUpdated');

  COMMIT;

  DBMS_APPLICATION_INFO.CLEAR_MODULE;
EXCEPTION
  WHEN OTHERS THEN
    DBMS_APPLICATION_INFO.SET_ACTION('Failed');
    ROLLBACK;
    RAISE;
END;
```

## How to View It
1. Use ASH reports to see session history (`$ORACLE_HOME/rdbms/admin/ashrpt.sql`)
2. Query `V$SESSION`: `SELECT module, action FROM v$session WHERE sid = SYS_CONTEXT('USERENV','SID');`

## More

- [Oracle Docs – DBMS_APPLICATION_INFO](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_APPLICATION_INFO.html)
- Use ASH reports to see session history (`$ORACLE_HOME/rdbms/admin/ashrpt.sql`)
- [Gather info for  performance tuning](/oracle/2020/02/17/Oracle-DB-Gather-info-for-performance-tuning.html)
