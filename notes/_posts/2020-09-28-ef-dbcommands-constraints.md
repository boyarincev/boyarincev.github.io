---
title: Entity Framework и ограничение параметров в команде
tags: entityframework
published: false
---

На работе у нас есть некое подобие самописной ORM, работающей напрямую с ADO.NET и одна из проблем с которой мы сталкивались при её разработе - это то что у количества параметров используемых в DbCommand[^1] есть ограничение - это ограничение накладывается базой данных и у разных баз данных оно разное[^2].

Например, у PostgreSQL - в каждом SQL statement (под SQL statement имеется в виду то, что обычно называют sql-запросом) может использоваться не больше 65535 параметров (в 1 dbCommand можно отправить множество sql-запросов и таким образом в общем DbCommand может содержать больше 65535 параметров).

В большинстве запросов довольно сложно преодолеть разрешённую планку, мы столкнулись с этим ограничением в двух запросах - в `Insert`, когда вставляется множество строк:

```sql
insert into table_name ([column_nam, ... ]) values (value1, ...)
```

И в запросе с использованием оператора `IN`:

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...);
```

## Ссылки

[^1]: [ADO.NET Commands and Parameters](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/commands-and-parameters)
[^2]: [What are the max number of allowable parameters per database provider type?](https://stackoverflow.com/q/6581573/5402731)
