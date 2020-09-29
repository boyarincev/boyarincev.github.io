---
title: Entity Framework и ограничение параметров в команде
tags: entityframework
published: true
---

На работе у нас есть некое подобие самописной ORM, работающей напрямую с ADO.NET и одна из проблем с которой мы сталкивались при её разработке - это то, что у количества параметров используемых в DbCommand[^1] есть лимит - этот лимит накладывается базой данных и у разных баз данных он разный[^2].

Например, у PostgreSQL - в каждом sql-statement (под sql-statement имеется в виду то, что обычно называют sql-запросом) может использоваться не больше 65535 параметров (в 1 DbCommand можно отправить множество sql-statement и таким образом в общем DbCommand может содержать больше 65535 параметров).

В большинстве запросов довольно сложно преодолеть разрешённую планку, мы столкнулись с этим ограничением в двух случаях - в `Insert`, когда вставляется множество строк:

```sql
insert into table_name ([column_nam, ... ]) values (value1, ...), (valueN, ...)
```

И в запросах с использованием оператора `IN`:

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...);
```

Первым делом я конечно же полез смотреть как эту проблему решает Entity Framework.

## Insert множества значений в Entity Framework

В Entity Framework эта проблема не встречается (во всяком случае при работе с PostgreSQL), потому что для каждой вставляемой строки генерируется свой `INSERT` запрос, а для каждого запроса мы можем использовать 65535 параметров, соответственно, чтобы превысить разрешённый лимит нужна таблица с 65536 столбцами:

```csharp
dbContext.Countries.Add(new Models.Country { Name = "Албания"});
dbContext.Countries.Add(new Models.Country { Name = "Словения"});
dbContext.SaveChanges();
```

```
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (7ms) [Parameters=[@p0='?', @p1='?'], CommandType='Text', CommandTimeout='30']
      INSERT INTO "Countries" ("Name")
      VALUES (@p0)
      RETURNING "Id";
      INSERT INTO "Countries" ("Name")
      VALUES (@p1)
      RETURNING "Id";
```

В общем-то в том числе и поэтому для массовой вставки Entity Framework не очень пригоден.

> Кстати, по поводу массового INSERT в дотнете и PostgreSQL недавно был доклад у DotNetRu[^3]

## IN оператор в Entity Framework

`IN` оператор генерируется, например, при вызове LINQ метода `Contains()`:

```csharp
var countryNames = Enumerable.Range(0, 100).Select(item => item.ToString()).ToList();
dbContext.Countries.Where(c => countryNames.Contains(c.Name)).Select(c => c.Name).ToArray();
```

```
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT c."Name"
      FROM "Countries" AS c
      WHERE c."Name" IN ('0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39', '40', '41', '42', '43', '44', '45', '46', '47', '48', '49', '50', '51', '52', '53', '54', '55', '56', '57', '58', '59', '60', '61', '62', '63', '64', '65', '66', '67', '68', '69', '70', '71', '72', '73', '74', '75', '76', '77', '78', '79', '80', '81', '82', '83', '84', '85', '86', '87', '88', '89', '90', '91', '92', '93', '94', '95', '96', '97', '98', '99')
```

Тут проблемы в Entity Framework тоже нет, просто потому что он вообще не использует параметры при формировании такого запроса, беря на себя риски с возможными sql-инъекциями.

Кстати, если интересно можете изучить в коде, как Entity Framework генерирует запросы[^4], а вот здесь[^5] SqlGenerator используемый в Npgsql

## Ссылки

1. [^1]: [ADO.NET Commands and Parameters](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/commands-and-parameters)
2. [^2]: [What are the max number of allowable parameters per database provider type?](https://stackoverflow.com/q/6581573/5402731)
3. [^3]: [Евгений Фирстов - PostgreSQL: Under Pressure](https://youtu.be/ZH7VtsyYSGk)
3. [^4]: [Entity Framework QuerySqlGenerator](https://github.com/dotnet/efcore/blob/v3.1.8/src/EFCore.Relational/Query/QuerySqlGenerator.cs#L570)
4. [^5]: [Npgsql QuerySqlGenerator](https://github.com/npgsql/efcore.pg/blob/v3.1.4/src/EFCore.PG/Query/Internal/NpgsqlQuerySqlGenerator.cs)

