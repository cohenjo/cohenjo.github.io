---
layout: post
section-type: post
title: 'Table Pruning for Unions '
date: '2015-12-23 00:32'
category: PostgreSQL
tags:
  - Partitions
  - PostgreSQL
---

### Table Pruning

One of the most useful usages of partitions in terms of query performance is the ability to prune partitions which do not contain relevant data.
In PostgreSQL this is done using `CHECK` validations on the partition (inherited table).
It turns out PostgreSQL can use the same optimization to prune tables which are not required in queries using `UNION ALL`.

Consider the following schema:

<pre><code data-trim class="sql">
Table "public.t<n>"
Column |  Type   | Modifiers
--------+---------+-----------
id     | integer |
c      | text    |
v      | integer |
Check constraints:
"t<n>_v_check" CHECK (v = <n>)
</code></pre>

for `n = 1:3`.

The following query:
`SELECT count(*) from (select * from t1 union all select * from t2 union all select * from t3) foo where foo.v=1; `

will be executed only on the first partition (i.e. `t1`)

<pre><code data-trim class="sql">
postgres=# explain select count(*) from (select * from t1 union all select * from t2 union all select * from t3) foo where foo.v=1;
                               QUERY PLAN                               
------------------------------------------------------------------------
 Aggregate  (cost=24346.00..24346.01 rows=1 width=0)
   ->  Append  (cost=0.00..21846.00 rows=1000000 width=0)
         ->  Seq Scan on t1  (cost=0.00..21846.00 rows=1000000 width=0)
               Filter: (v = 1)
(4 rows)
</code></pre>

Compare this with the same query only this time without the `CHECK` constrains:

<pre><code data-trim class="sql">
postgres=# explain select count(*) from (select * from t1 union all select * from t2 union all select * from t3) foo where foo.v=1;
                               QUERY PLAN                               
------------------------------------------------------------------------
 Aggregate  (cost=68038.01..68038.01 rows=1 width=0)
   ->  Append  (cost=0.00..65538.00 rows=1000002 width=0)
         ->  Seq Scan on t1  (cost=0.00..21846.00 rows=1000000 width=0)
               Filter: (v = 1)
         ->  Seq Scan on t2  (cost=0.00..21846.00 rows=1 width=0)
               Filter: (v = 1)
         ->  Seq Scan on t3  (cost=0.00..21846.00 rows=1 width=0)
               Filter: (v = 1)
(8 rows)
</code></pre>

At this point you're probably wondering _why not simply use partitions?_
well, one reason is to avoid the locks that are required for partition maintenance.
Dropping a partition takes a lock on the parent table which is usually very busy and under constant load, while dropping an unrelated table takes no locks on the accessed tables.

### Usage
Let's design a solution to hold time-series data for a year.
we'd like to partition the data over months for the usual reasons.

At this point we should probably remind the reader that partitions in PostgreSQL are created manually and it is left up to the development to maintain the inherited tables.

So, how can we avoid the horrors of locking our extremely busy table?
create a table for each month, drop tables which are no longer useful.
upon query simply `UNION ALL` all the relevant tables and let PG prune for you :)

For example:

<pre><code data-trim class="sql">
explain select count(*)
from
(select * from t1
union all
select * from t2
union all
select * from t3
union all
select * from t4
union all
select * from t5
union all
select * from t6
union all
select * from t7
union all
select * from t8
union all
select * from t9
union all
select * from t10
union all
select * from t11
union all
select * from t12
) foo
where foo.v in (12, 11, 10);
</code></pre>

And PG runs:

<pre><code data-trim class="sql">
QUERY PLAN                                
-------------------------------------------------------------------------
Aggregate  (cost=76788.00..76788.01 rows=1 width=0)
->  Append  (cost=0.00..69288.00 rows=3000000 width=0)
->  Seq Scan on t10  (cost=0.00..23096.00 rows=1000000 width=0)
Filter: (v = ANY ('{12,11,10}'::integer[]))
->  Seq Scan on t11  (cost=0.00..23096.00 rows=1000000 width=0)
Filter: (v = ANY ('{12,11,10}'::integer[]))
->  Seq Scan on t12  (cost=0.00..23096.00 rows=1000000 width=0)
Filter: (v = ANY ('{12,11,10}'::integer[]))
</code></pre>

It even works for conditions such as `where foo.v > 10;`

<pre><code data-trim class="sql">
QUERY PLAN                                
-------------------------------------------------------------------------
Aggregate  (cost=48692.00..48692.01 rows=1 width=0)
->  Append  (cost=0.00..43692.00 rows=2000000 width=0)
->  Seq Scan on t11  (cost=0.00..21846.00 rows=1000000 width=0)
Filter: (v > 10)
->  Seq Scan on t12  (cost=0.00..21846.00 rows=1000000 width=0)
Filter: (v > 10)
</code></pre>

### Conclusion
Partition pruning is a powerful optimization tool but it is not limited to partitions.
As the underlying mechanism is shared with `UNION ALL` we can use it for cases were we must avoid locks on the parent table and pay the price of longer query syntax.
