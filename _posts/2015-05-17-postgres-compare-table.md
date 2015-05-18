---
layout: post
title: Comparing Tables in Postgres
---

How to tell if two tables are exactly the same?

-----

```sql

with A as (
    select hashtext(textin(record_out(a))) as h, count(*) as c
      from table1 a
      group by 1
),
 B as (
    select hashtext(textin(record_out(b))) as h, count(*) as c
    from table2 b
    group by 1
)
select *
  from A full outer join B on (A.h + A.c = B.h + B.c)
 where A.h is null or B.h is null
 limit 5;

```
