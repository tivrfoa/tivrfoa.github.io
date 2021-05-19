---
layout: post
title:  "MySQL Anti Join"
date:   2021-05-19 15:30:00 -0300
categories: mysql join antijoin
---

### MySQL Antijoin

Today I saw a Twitter about [JOOQ](https://github.com/jOOQ/jOOQ) where it mentioned
`leftAntiJoin`:

[![JOOQ leftAntiJoin](/assets/images/jooq-leftAntiJoin-Screenshot-2021-05-19-15-46-43.png)](https://twitter.com/anghelleonard/status/1394600475090169857/photo/2)

I had never seen this before. I knew about `outer join`, but I didn't
know about `antijoin`.

I searched and found interesting info about this functionality in MySQL:

>In MySQL 8.0.17, we made an observation in the well-known TPC-H benchmark for one particular query. The query was executing 20% faster than in MySQL 8.0.16. This improvement is because of the “antijoin” optimization which I implemented. Here is its short mention in the release notes:
>
>“The optimizer now transforms a WHERE condition having NOT IN (subquery), NOT EXISTS (subquery), IN (subquery) IS NOT TRUE, or EXISTS (subquery) IS NOT TRUE internally into an antijoin, thus removing the subquery.”

[Antijoin in MySQL 8](https://mysqlserverteam.com/antijoin-in-mysql-8/)


But the `anti join` syntax is not available in MySQL. It's just an *internal* implementation ...

[8.2.2.1 Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations](https://dev.mysql.com/doc/refman/8.0/en/semijoins.html)


### Testing using MySQL and Docker

Starting MySQL server:
```sh
sudo docker run --name mysql-8-0-25 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123 mysql:8.0.25
```

On another terminal:
```sh
sudo docker run -it --network host mysql:8.0.25 mysql -h0.0.0.0 -uroot -p
```

Another option is to start a shell and then log in to mysql:
```sh
sudo docker exec -it mysql-8-0-25 sh

mysql -p
```

```sql
create database antijoin;

use antijoin;

create table A (
	id int primary key auto_increment,
	name varchar(50)
);

create table B (
	id int primary key auto_increment,
	a_id int,
	xyz varchar(30),
	foreign key(a_id) references A(id)
);

insert into A (name)
values
('hello'), ('world'), ('goodbye');

insert into B (a_id, xyz)
values
(1, 'ola'), (2, 'mundo');

select *
from A
where not exists (
  select 1
  from B
  where A.id = B.a_id);

+----+---------+
| id | name    |
+----+---------+
|  3 | goodbye |
+----+---------+
1 row in set (0.00 sec)

-- These don't work :/

select A.*
from A antijoin B on
  A.id = B.a_id;

select A.*
from A anti join B on
  A.id = B.a_id;

select A.*
from A anti join (B) on
  ((A.id = B.a_id));

```

Using `explain` and `show warnings` to understand the query:
```sql
mysql> explain select *
    -> from A
    -> where not exists (
    ->   select 1
    ->   from B
    ->   where A.id = B.a_id);
+----+-------------+-------+------------+------+---------------+------+---------+---------------+------+----------+--------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref           | rows | filtered | Extra                                |
+----+-------------+-------+------------+------+---------------+------+---------+---------------+------+----------+--------------------------------------+
|  1 | SIMPLE      | A     | NULL       | ALL  | NULL          | NULL | NULL    | NULL          |    3 |   100.00 | NULL                                 |
|  1 | SIMPLE      | B     | NULL       | ref  | a_id          | a_id | 5       | antijoin.A.id |    1 |   100.00 | Using where; Not exists; Using index |
+----+-------------+-------+------------+------+---------------+------+---------+---------------+------+----------+--------------------------------------+
2 rows in set, 2 warnings (0.00 sec)

mysql> show warnings\g;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                                                       |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1276 | Field or reference 'antijoin.A.id' of SELECT #2 was resolved in SELECT #1                                                                                                                     |
| Note  | 1003 | /* select#1 */ select `antijoin`.`A`.`id` AS `id`,`antijoin`.`A`.`name` AS `name` from `antijoin`.`A` anti join (`antijoin`.`B`) on((`antijoin`.`B`.`a_id` = `antijoin`.`A`.`id`)) where true |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

ERROR: 
No query specified
```

`explain format=tree`
```sql
explain format=tree select *
from A
where not exists (
  select 1
  from B
  where A.id = B.a_id);
```

```
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                                     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
| -> Nested loop antijoin  (cost=1.60 rows=3)
    -> Table scan on A  (cost=0.55 rows=3)
    -> Index lookup on B using a_id (a_id=A.id)  (cost=0.28 rows=1)
 |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set, 1 warning (0.00 sec)
```


### References

[https://mysqlserverteam.com/antijoin-in-mysql-8/](https://mysqlserverteam.com/antijoin-in-mysql-8/)

[https://dev.mysql.com/doc/refman/8.0/en/semijoins.html](https://dev.mysql.com/doc/refman/8.0/en/semijoins.html)

[https://twitter.com/anghelleonard/status/1394600475090169857](https://twitter.com/anghelleonard/status/1394600475090169857)

[https://mysqlserverteam.com/a-must-know-about-not-in-in-sql-more-antijoin-optimization/](https://mysqlserverteam.com/a-must-know-about-not-in-in-sql-more-antijoin-optimization/)
