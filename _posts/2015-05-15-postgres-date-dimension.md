---
layout: post
title: Date Dimension Table in Postgres
---

One of the first tables required for any Data Warehouse projects is
the Date Dimension table. 

-----

```sql
drop table x;
create table x as
with
DR1 as (
    select n,
       '2000-01-01'::date as first_date,
       '2000-01-01'::date + n as date
       from generate_series(0, 366*10) n
),
DR2 as (
    select *,
        extract(dow from first_date) as day_of_first_date,
        (date - first_date) + 1 as doe,
        extract(year from date)::int - 2000 + 1 as yoe,
        extract(week from date)::int as woy,
        extract(day from date)::int as dom,
        extract(dow from date)::int as dow,
        extract(doy from date)::int as doy, 
        extract(month from date)::int as m,
        extract(quarter from date)::int as q, 
        extract(year from date)::int as y
    from DR1
),
DR3 as (
    select *,
         (case when m = 1 and woy > 50 then y - 1 else y end) as yow,
         (8 - (case when day_of_first_date = 0 then 7 else day_of_first_date end))::int % 7 as dayoffset

    from DR2
)
select
    n+1 as k,
    date,
    y,                          -- year
    m,                          -- month
    dow as d,                   -- day of week
    q,                          -- quarter
    ((m - 1) / 6) + 1 as h,     -- half of year
    dom,                        -- day of month
    doy,                        -- day of year
    yow,                        -- year of week
    woy,                        -- week of year
    doe,
    (doe - 1 - dayoffset + 7) / 7 + 1 as woe,
    (yoe - 1) * 12 + m as moe,
    (yoe - 1) * 4 + ((m - 1) / 3) + 1 as qoe,
    yoe,
    (1 <= dow and dow <= 5) as is_weekday,
    not (1 <= dow and dow <= 5) as is_weekend,
    yow||'-W'||woy as yw,
    to_char(date, 'yyyy-mm') as ym
from DR3
;

```



For how to use this, refer here...

* [Ref1](http://link1.example.com)
* [Ref2](http://lilnk2.example.com)

### The SQL

Step by step:

* step 1
* step 2
* step 3

### Additional Notes


