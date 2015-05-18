---
layout: post
title: Comparing Tables in Postgres
---

How to tell if two tables are exactly the same?

-----

```sql

with A as (
    select hashtext(textin(record_out(t1))) as h, count(*) as c
      from table1 t1
      group by 1
),
 B as (
    select hashtext(textin(record_out(t2))) as h, count(*) as c
    from table2 t2
    group by 1
)
select *
  from A full outer join B on (A.h + A.c = B.h + B.c)
 where A.h is null or B.h is null
 limit 5;

```

The SQL above gives you up to 5 differences between table1 and
table2. To see exactly which row is different, take the h value
(e.g. 12345) of the result and match it:

```sql
select *
from table1 t1     -- or t2
where hashtext(textin(record_out(t1))) = 12345;

```
