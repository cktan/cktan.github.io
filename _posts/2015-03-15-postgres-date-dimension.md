---
layout: post
title: Date Dimension Table in Postgres
comments: True
---

One of the first dimension tables required for any Data Warehouse projects is
the Date Dimension table. See [Calendar Date Dimensions @kimballgroup] (http://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/calendar-date-dimension/). 

-----

Here is the SQL statement that generates the Date Dimension table in Postgres. The table will look like this:

| Date Dimension 
| -------------- 
| Column         | Type  | Description 
| -------------- | ----- | ----------- 
| date_key       | int   | A number corresponding to the date. e.g. 20150315. 
| date           | date  | SQL date. 
| y              | int   | Year in number, e.g. 2015. 
| m              | int   | Month in number (1..12). 
| d              | int   | Day of month (1..31). 
| q              | int   | Quarter (1..4). 
| h              | int   | First / second half of year (1 or 2). 
| dow            | int   | Day of week (1..7). 
| doy            | int   | Day of year (1..366). 
| yow            | int   | Year of week. Can be different from y only during first week of the year. 
| woy            | int   | Week of year (1..52). Note: first few days of the year may be week 52 of last year. 
| doe            | int   | Day of epoch, a serial number. 
| woe            | int   | Week of epoch, a serial number. 
| moe            | int   | Month of epoch, a serial number. 
| yoe            | int   | Year of epoch, a serial number. 
| is_weekday     | bool  | True if Mon..Fri. 
| is_weekend     | bool  | True if Sat..Sun. 
| first_date_of_week | date | Date of Monday of this week. 
| last_date_of_week | date | Date of Sunday of this week. 
| yw                | text | e.g. 2015-W5 for week 5 of 2015. 
| ym                | text | e.g. 2015-12 for December 2015. 
| yq                | text | e.g. 2015-Q4 for Q4 of 2015. 



```sql

create table datedim as
with
DR1 as (
    -- You can change the range here!
    -- epoch starting 2000-01-01, for 10 years
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
         (8 - (case when day_of_first_date = 0
                    then 7 else day_of_first_date end))::int % 7 as dayoffset
    from DR2
),
DR4 as (
    select *,
        ((m - 1) / 6) + 1 as h,     -- half of year
        (doe - 1 - dayoffset + 7) / 7 + 1 as woe,
        (yoe - 1) * 12 + m as moe,
        (yoe - 1) * 4 + ((m - 1) / 3) + 1 as qoe,
        (1 <= dow and dow <= 5) as is_weekday,
        not (1 <= dow and dow <= 5) as is_weekend

    from DR3
),
MonthGroup as (
    select moe, max(date) as last_date_of_month from DR4 group by 1
),
WeekGroup as (
    select woe, min(date) as first_date_of_week,
           max(date) as last_date_of_week from DR4 group by 1
)
select
    to_char(date, 'YYYYMMDD')::int as date_key,
    date,
    y,                          -- year (YYYY)
    m,                          -- month (1..12)
    dom as d,                   -- day of month (1..31)
    q,                          -- quarter (1..4)
    h,				-- half (1..2)
    dow,                        -- day of week (1..7)
    doy,                        -- day of year (1..366)
    yow,                        -- year of week
    woy,                        -- week of year
    doe,                        -- day of epoch 
    DR4.woe,                    -- week of epoch
    DR4.moe,                    -- month of epoch
    yoe,                        -- year of epoch
    is_weekday,                 -- true if Mon .. Fri
    is_weekend,                 -- true if Sat or Sun 
    first_date_of_week,         -- date of Mon in week
    last_date_of_week,          -- date of Sun in week
    last_date_of_month,         -- date of last day in month
    yow||'-W'||woy as yw,       -- e.g.2015-W5
    to_char(date, 'yyyy-mm') as ym, -- e.g. 2015-07
    y||'-Q'||q as yq                -- e.g. 2015-Q3
from DR4
join MonthGroup using (moe)
join WeekGroup using (woe)
;

```


{% comment %}
For how to use this, refer here...

* [Ref1](http://link1.example.com)
* [Ref2](http://lilnk2.example.com)

### The SQL

Step by step:

* step 1
* step 2
* step 3

### Additional Notes
{% endcomment %}


