+++ 
draft = false
date = 2021-10-05T09:11:07-04:00
title = "Shredding JSON to Key-Value Pairs in SQL Server"
description = "Shredding JSON to Key-Value Pairs in SQL Server"
slug = "json-shred-sql-server"
authors = ["Peter Vandivier"]
tags = ["sql-server","json"]
categories = []
externalLink = ""
series = []
+++

<!--
- https://dbfiddle.uk/?rdbms=sqlserver_2017&fiddle=c797e4fb3575db31e75eb60b56a3f999&hide=2
TODO: 
- add "getParent" path 
- add "just the key" attribute
- rename "key" to "fullPath"
-->

# Shredding JSON in SQL Server

For a while now ([1](https://chat.stackexchange.com/transcript/179?m=55146340#55146340),[2](https://dba.stackexchange.com/questions/239180/find-ancestry-from-json#comment471849_239206)), I've been a fan of shredding arbitrary JSON to key-value mapping. I've had some success doing this in PowerShell ([3](https://topanswers.xyz/powershell?q=930)) and using jq ([4](https://topanswers.xyz/nix?q=915)). These exercises were helpful in identifying aggregate changes to ElasticSearch logs and MongoDB documents on a few occasions.

Sadly, I've never been able to grok the syntax in T-SQL. I blame this on [SQL Server's very limited support for JSON functions](https://docs.microsoft.com/en-us/sql/t-sql/functions/json-functions-transact-sql) - as opposed to [JSON support in PostgreSQL](https://www.postgresql.org/docs/current/functions-json.html) for example. Recently though I had a bit of a breakthrough. Something clicked into place and I'd like to document below the successful approach with some notes on what it does and why. To start with the best bit, the code:

## The code

The table-valued function below turns JSON input into a KVP table where each key is the fully qualified jPath to the element within the top-level JSON document.

{{< detail-tag "dbo.json_shred()" >}}

```sql
create or alter function dbo.json_shred (
    @json nvarchar(max)
)
returns table 
as
return (
/*
Author:      Peter Vandivier
Date:        2021-10-04
Description: Takes an arbitrary JSON document and returns all valid
             json_paths within the document as well as the values
             present at said path and the depth within the document
             to which the given path with probe
Example:
    -- Filtering to `[Type] not in (4,5)` returns all leaf-node
    -- paths and values for the given document
    --
    declare @json nvarchar(max) = N'{"a":"foo","b":[1,2,{"d":"bar","e":[0]}],"f":{"g":"baz"},"h":null,"i":"zap"}';
    --
    select * 
    from dbo.json_shred(@json)
    where [Type] not in (4,5);
Example_Output:
    +-------+----------+-------------+-------+------+
    | Level | Parent   | Key         | Value | Type |
    +-------+----------+-------------+-------+------+
    | 1     | $.       | $.a         | foo   | 1    |
    | 1     | $.       | $.h         | NULL  | 0    |
    | 1     | $.       | $.i         | zap   | 1    |
    | 2     | $.f      | $.f.g       | baz   | 1    |
    | 2     | $.b      | $.b[0]      | 1     | 2    |
    | 2     | $.b      | $.b[1]      | 2     | 2    |
    | 3     | $.b[2]   | $.b[2].d    | bar   | 1    |
    | 4     | $.b[2].e | $.b[2].e[0] | 0     | 2    |
    +-------+----------+-------------+-------+------+
*/
    with level_0 as (
        select
            convert(int,0) as [Level],
            convert(nvarchar(4000),N'$') as [Key],
            @json as [Value],
            convert(
                int,
                case left(@json,1)
                    when N'[' then 4
                    when N'{' then 5
                    else 0
                end
            ) as [Type]
    )
    , key_value_unwrap as(
        select 
            l0.[Level] + 1 as [Level],
            convert(nvarchar(max),null) as Parent,
            l0.[Key] + iif(l0.[Type] = 5, '.' + oj.[Key], quotename(-1 + row_number() over (order by (select null)))) collate database_default as [Key],
            oj.[Value],
            oj.[Type]
        from level_0 l0
        outer apply openjson(l0.[Value]) as oj
        where l0.[Value] is not null 
        union all
        select 
            kvu.[Level] + 1 as [Level],
            convert(nvarchar(max),kvu.[Key]) as Parent,
            kvu.[Key] + iif(kvu.[Type] = 5, '.' + oj.[Key], quotename(-1 + row_number() over (order by (select null)))) as [Key],
            oj.[Value],
            oj.[Type]
        from key_value_unwrap as kvu
        outer apply openjson(kvu.[Value], 'lax $') as oj
        where kvu.[Type] in (4,5)
    ), _union as (
        select 
            l0.[Level],
            convert(nvarchar(max),null) as Parent,
            l0.[Key] + N'.' as [Key],
            l0.[Value],
            l0.[Type]
        from level_0 as l0
        union all
        select 
            kvu.[Level],
            kvu.Parent,
            kvu.[Key],
            kvu.[Value],
            kvu.[Type]
        from key_value_unwrap as kvu
    ) 
    select 
        u.[Level],
        iif(u.[Level]=1,N'$.',u.Parent) as Parent,
        u.[Key],
        u.[Value],
        u.[Type]
    from _union as u
);
```

{{< /detail-tag >}}

## The Explanation

An invocation of the single-argument-no-schema overload of [`OPENJSON()`](https://docs.microsoft.com/en-us/sql/t-sql/functions/openjson-transact-sql) will unpack all elements at the first level. If you then recurse your level-1 resultset into another application of single-argument `OPENJSON()`, you'll get all elements from the 2nd level; and so on. 

### level_0

Why the clunky `level_0` CTE? Well, dear reader, because `APPLY` is still a `JOIN`. The first iteration of `OPENJSON` over your document has a left-to-right relationship to the document; **NOT** an up-to-down relationship. Consider the following:

```sql
select *
from (values 
    (1, '{"a":1,"b":{"c":2}}'),
    (2, '{"d":3}')
) as v (id,doc)
cross apply openjson(v.doc) as js;
```

The resultset might be visualized like...

| id | doc | key | value | type |
| --- | --- | --- | --- | --- |
| 1  | {"a":1,"b":{"c":2}} | {{< rawhtml >}}a<br>b{{< /rawhtml >}} | {{< rawhtml >}}1<br>{"c":2}{{< /rawhtml >}} | {{< rawhtml >}}2<br>5{{< /rawhtml >}} |
| 2 | {"d":3} | d | 3 | 2 |

<!-- 
+----+---------------------+-----+---------+------+
| id | doc                 | key | value   | type |
+----+---------------------+-----+---------+------+
| 1  | {"a":1,"b":{"c":2}} | a   | 1       | 2    |
|    |                     | b   | {"c":2} | 5    |
+----+---------------------+-----+---------+------+
| 2  | {"d":3}             | d   | 3       | 2    |
+----+---------------------+-----+---------+------+
-->

Note that key-value pairs associate directly to the containing document. A JSON document can itself contain arbitrary JSON documents as constituent data (consider the data in `[Key]` "b" for document id 1). Therefore for a JSON shredder, I consider the document _itself_ as a data value. If this were [`jq`](https://stedolan.github.io/jq/), the query string I would execute to get the _whole document_ would be "`.`" (single dot) - meaning, simply: "retrieve everything". 

What I _want_ is all level-1 key-value rows _appended_ to the the level-0 document row - at a deeper `[Level]`. Something like this:

| id | key | value | Level |
| --- | --- | --- | --- |
| 1  | {{< rawhtml >}}.<br>.a<br>.b{{< /rawhtml >}} | {{< rawhtml >}}{"a":1,"b":{"c":2}}<br>1<br>{"c":2}{{< /rawhtml >}} | {{< rawhtml >}} 0<br>1<br>1{{< /rawhtml >}}  |
| 2  | {{< rawhtml >}}.<br>.d{{< /rawhtml >}}   | {{< rawhtml >}}{"d":3}<br>3 {{< /rawhtml >}} | {{< rawhtml >}}0<br>1 {{< /rawhtml >}} |

<!--
+----+-----+---------------------+-------+
| id | key | value               | Level |
+----+-----+---------------------+-------+
| 1  | .   | {"a":1,"b":{"c":2}} | 0     |
|    | .a  | 1                   | 1     |
|    | .b  | {"c":2}             | 1     |
+----+-----+---------------------+-------+
| 2  | .   | {"d":3}             | 0     |
|    | .d  | 3                   | 1     |
+----+-----+---------------------+-------+
-->

In order to achieve this, I've separately defined the level-0 tuple and I prepend it to the final resultset.

### key_value_unwrap

This is the "[meat & potatoes](https://www.merriam-webster.com/dictionary/meat-and-potatoes)" of the shredder. We iterate over the keys of the document and recurse to a higher level for any key whose corresponding value is an array or object type ([OPENJSON() return types](https://learn.microsoft.com/en-us/sql/t-sql/functions/openjson-transact-sql#return-value)).

We decrement by one the output of `row_number()` where used to accomodate 0-based indexing. We suffix the local `[Key]` to the parent value to build the fully qualified path as we spelunk "deeper" into the object.

### _union

The `_union` CTE is _partially_ redundant but serves to simplify some expression redundancy that would otherwise need to appear in the final `select`. 

## The homage

One of my favorite StackOverflow answers ever is [this T-SQL XML shredder](https://stackoverflow.com/a/10885014/4709762). You can see it in action against a complex document [on db<>fiddle](https://dbfiddle.uk/u7otQIYv?hide=2). It stuck in my brain from the first time I used it and obviously influenced the design of this JSON shredder.

These 2 functions share more than just output style though. They both serve to do on SQL Server something you probably ought to be doing elsewhere in the stack - but sometimes you need to JFDI in a pinch. Just don't look too close at the execution plan for either 😅
