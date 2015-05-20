---
layout: post
title: Comparing Tables in Postgres
comments: True
---

How to tell if two tables are exactly the same?

-----

### Method 1: Use SQL EXCEPT

If you google around, you will find 
[this post]
(http://dba.stackexchange.com/questions/72641/checking-whether-two-tables-have-identical-content-in-postgresql),
where the use of `EXCEPT` is suggested.  To obtain rows in t1 that are
missing in t2:

```sql
select * from (table t1 except table t2) foo;
```

To obtain all rows that are different between t1 and t2:

```sql
-- note: do not drop the parens
select * from ((table t1 except table t2) union all (table t2 except table t1)) foo;
```

Note that an empty result set from the above query does not indicate
that t1 and t2 are EQUAL. For that, we would also need to assert that
t1 and t2 has the same cardinality, i.e., same number of rows.

### Method 2: Hash it ourselves

```sql

with A as (
    select hashtext(textin(record_out(t1))) as h, count(*) as c
      from t1
      group by 1
),
 B as (
    select hashtext(textin(record_out(t2))) as h, count(*) as c
    from t2
    group by 1
)
select *
  from A full outer join B on (A.h + A.c = B.h + B.c)
 where A.h is null or B.h is null
 limit 5;

```

The SQL above gives you up to 5 differences between t1 and
t2, and if there are 0 differences, then t1 and t2 are indeed EQUAL.

To see exactly which row is different, take the h value
(e.g. `12345`) of the result and match it:

```sql
select *
from t1     -- or t2
where hashtext(textin(record_out(t1))) = 12345;

```

### Which method is better?

In general, the built-in EXCEPT clause works better. 
