+++ 
draft = true
date = 2024-09-19T11:04:17-04:00
title = "Recognizing paired records in CDC"
description = "Patterns in paired records & What They Mean in SQL Server Change Data Capture"
slug = "cdc-paired-records-patterns"
authors = ["Peter Vandivier"]
tags = ["sql-server","change-data-capture"]
categories = []
externalLink = ""
series = []
+++

# Getting more information out of paired rows in CDC base tables 

When using [SQL Server Change Data Capture](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server), the most visible artifacts are the `*_CT` suffixed tables nested under the "System Tables" in SSMS Object explorer. These tables are a similar in structure to a [Temporal Table](https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables) but have ✨special extra bits✨.

You can use the ✨special extra bits✨ to dig deeper into the rows and row pairs than might be obvious at first glance.

## First some commentary on the docs

> [cdc.<capture_instance>_CT (Transact-SQL) - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/system-tables/cdc-capture-instance-ct-transact-sql)

For the purposes of the demos that follow:
* `__$start_lsn`, `__$end_lsn`, & `__$seqval` are `binary(10)` and not easy to read in full. I'll shorten them in example data to make them easier to read. They reflect the same relative sort order as values you would see if you run the demos yourself and the shortened values are there for you to infer this ordering.
* date/time values may be truncated for readability in sample data, but as above: relative ordering is preserved.

### `__$operation`

I'll copy the `__$operation` enum values here because I can never remember them and always end up looking them up anyhow 😅

| id | description | explanation | update mask |
| --- | --- | --- | -- | 
| 1 | delete | Row values at the time the row was deleted. | Always full row |
| 2 | insert | Initial row values. | Always full row |
| 3 | update-old | Row values _before_ executing the update statement. | |
| 4 | update-new | Row values _after_ executing the update statement. | |

### `__$update_mask`

`__$update_mask` is one of my favorite columns. In chunks of 8 it maps which columns in the base table changed for a given `_CT` row. 

| update mask | binary | description |
| --- | --- | --- |
| 0x00 | 00000000 | impossible to find in CT - means no rows were updated |
| 0x02 | 00000010 | the 2nd from the left column was updated |
| 0x0002 | 00000000 00000010 | the 2nd from the left column was updated _and_ the table has between 9 and 16 columns |

It's rare see an 0x01 (or 0x0001 etc.) mask because the leftmost column is _usually_ the clustered index column. Updating a primary key value is done under the hood by deleting and re-inserting the row. Therefore you'll only see this bitmask if the leftmost column is updated and is **not** the clustered index key.

Tables with 8 / 16 / etc. columns are fun because you can get bitmasks like 0xFF, 0xFFFF bitmasks on insertion & deletion. You can get the full-row bitmask for your table without needing to convert to/from hex in your head by visually scanning for a row with a 1 or 2 `__$operation` when you query the `_CT` table.

### `__$update_mask`


## For example...

One distinction between temporal tables & CDC is that each temporal row is self-contained, while CDC rows may "double-up" and convey information in a pair of rows not otherwise interpretable in a single row. The most obvious of these in the base update. 

Given a one-row table of the form...

```sql 
create table dbo.foo (
    id int not null primary key,
    a char(1)
);

insert dbo.foo 
values (1,'a');
```

...what can we infer when querying `cdc.dbo_foo_CT`.

### Simple update

```sql
update foo 
set a = 'b'
where id = 1
```

* same `__$start_lsn`
* same `__$seqval`
* same `__$command_id`
* operations 3 & 4

| __$start_lsn | __$end_lsn | __$seqval | __$operation | __$update_mask | id | a | __$command_id |
|---|---|---|---|---|---|---|---|
| 0x01 | _NULL_ | 0x02 | 3 | 0x02 | 1 | a | 1 | 
| 0x01 | _NULL_ | 0x02 | 4 | 0x02 | 1 | b | 1 | 

This is also the pattern you will see for merge-matched-updates _when the PK is not specified in the `update`_.

```sql
merge foo as t
using (values (1,'b')) as s (id,a)
    on s.id = t.id
when matched
    then update 
        a  = s.a
```

There is a different pattern for [Merge-matched command WITH PK](#merge-matched-command-with-pk).

### Update Primary Key

```sql
update foo 
set id = 2
where id = 1
```

* same `__$start_lsn`
* same `__$seqval`
* incremented `__$command_id`
* operations 1 & 2
* full bitmask
* PK scalar value update

| __$start_lsn | __$end_lsn | __$seqval | __$operation | __$update_mask | id | a | __$command_id |
|---|---|---|---|---|---|---|---|
| 0x01 | _NULL_ | 0x02 | 1 | 0x03 | 1 | a | 1 | 
| 0x01 | _NULL_ | 0x02 | 2 | 0x03 | 2 | a | 2 | 

### Merge matched command WITH PK

If the PK column is specified in the merge command, a full-row deletion and insertion will occur _even_ if no data change occurs.

```sql
merge foo as t
using (values (1,'b')) as s (id,a)
    on s.id = t.id
when matched
    then update 
        id = s.id,
        a  = s.a
```

* same `__$start_lsn`
* incremented `__$seqval`
* incremented `__$command_id`
* operations 1 & 2
* full bitmask

| __$start_lsn | __$end_lsn | __$seqval | __$operation | __$update_mask | id | a | __$command_id |
|---|---|---|---|---|---|---|---|
| 0x01 | _NULL_ | 0x02 | 1 | 0x03 | 1 | a | 1 | 
| 0x01 | _NULL_ | 0x02 | 2 | 0x03 | 1 | a | 2 | 

### No-entry commands

The following commands produce _no entry_ in CDC even if they have a non-zero `@@rowcount`.

#### Same-value update

All the below queries produce a positive `@@rowcount`, but do not insert a CDC `_CT` record.

```sql
update bar set a = 'a' where a = 'a';

update bar set id = 1 where id = 1;

merge foo as t
using (values ('a')) 
    as s (a)
    on s.a = t.a
when matched
    then update 
        a = s.a
```
