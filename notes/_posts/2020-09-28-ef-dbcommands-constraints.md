---
title: Entity Framework и лимит параметров в команде
tags: entityframework npgsql
published: true
---

На работе у нас есть некое подобие самописной ORM, работающей напрямую с ADO.NET и одна из проблем с которой мы сталкивались при её разработке - это то, что у количества параметров используемых в [DbCommand](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/commands-and-parameters) есть лимит - этот лимит накладывается базой данных и у разных баз данных он разный.

Например, у PostgreSQL в каждом SQL statement (под SQL statement имеется в виду то, что в разговорной речи называют SQL запросом) может использоваться [не больше 65535 параметров](https://stackoverflow.com/q/6581573/5402731) (в одну DbCommand можно отправить множество SQL statement и таким образом в общем DbCommand может содержать больше 65535 параметров).

В большинстве запросов довольно сложно преодолеть разрешённую планку, мы столкнулись с этим ограничением в двух случаях - в `Insert`, когда за один запрос вставляется множество строк:

```sql
INSERT INTO table_name (column_namе(s)) values (value1, ...), (valueN, ...), ...
```

Если таблица содержит 10 столбцов, то с проблемой мы столкнёмся уже при попытке вставить за один раз 656 строк.

И в запросах с использованием оператора `IN`:

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...);
```

Здесь список, в котором ищутся значения, должен превышать 65535 элементов.

Первым делом я конечно же полез смотреть как эту проблему решает и решает ли вообще Entity Framework.

_Очень многое в поведении может определить конкретный дата-провайдер, я исследовал только связку Entity Framework + Npgsql, поэтому что-то из текста может быть неактуально для других дата-провайдеров_

## Зачем вообще использовать параметры

Основная причина в использовании параметров - это [защита от SQL инъекций](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/configuring-parameters-and-parameter-data-types):

> Command objects use parameters to pass values to SQL statements or stored procedures, providing type checking and validation. Unlike command text, parameter input is treated as a literal value, not as executable code. This helps guard against "SQL injection" attacks, in which an attacker inserts a command that compromises security on the server into an SQL statement.

OWASP, например, [рекомендует](https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html#data-access) всегда использовать параметризированные запросы:

> - Use Parameterized SQL commands for all data access, without exception.
> - Do not use SqlCommand with a string parameter made up of a concatenated SQL String.

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

_Кстати, по поводу массового INSERT в дотнете и PostgreSQL недавно был доклад у DotNetRu: [Евгений Фирстов - PostgreSQL: Under Pressure](https://youtu.be/ZH7VtsyYSGk)_

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

Тут проблемы в Entity Framework тоже нет, просто потому что он вообще не использует параметры при формировании такого запроса, беря на себя риски с возможными SQL инъекциями.

_Кстати, если интересно можете изучить в коде, как Entity Framework генерирует запросы (ссылки в конце)_

## Как мы решили проблему в своей ORM

Мы считаем параметры и при достижении лимита начинаем новый SQL statement.

## Другие ограничения на которые тоже стоит обратить внимание

Как мы выяснили, проблема с лимитом параметров не актуальна для Entity Framework, но есть и другие ограничения, с которыми вероятность столкнуться больше.

### Таймаут времени выполнения команды

У команды есть лимит времени, отведённый на её выполнение, если команда выполняется дольше, то выполнение прерывается выбрасыванием исключения. Выше мы видели, что при вставке множества новых строк, Entity Framework объединят INSERT'ы в одну команду, таким образом потенциально общее время выполнения такой команды может быть большим. Точно также он поступает и в случае операций обновления и удаления и вообще он склонен все возможные операции проводимые на базе данных в один момент времени упаковывать в одну команду (при вызове `SaveChanges`, например).

В логах Entity Framework выше, можно увидеть что CommandTimeout для запросов он устанавливает в 30 секунд:

```
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (7ms) [Parameters=[@p0='?', @p1='?'], CommandType='Text', CommandTimeout='30']
```

Выбирает такое значение он, вероятно, потому что это [значение по умолчанию для Npgsql](https://www.npgsql.org/doc/connection-string-parameters.html).

Изменить его можно либо задав прямо в connection string подключения к базе данных, либо в [коде](https://stackoverflow.com/a/6234593/5402731)

### Ограничение длины запроса

## Дополнительные ссылки

1. Генерация запросов чтения данных: [Entity Framework QuerySqlGenerator](https://github.com/dotnet/efcore/blob/v3.1.8/src/EFCore.Relational/Query/QuerySqlGenerator.cs#L570), [Npgsql QuerySqlGenerator](https://github.com/npgsql/efcore.pg/blob/v3.1.4/src/EFCore.PG/Query/Internal/NpgsqlQuerySqlGenerator.cs)
2. Генерация запросов изменения данных [Entity Framework UpdateSqlGenerator](https://github.com/dotnet/efcore/blob/v3.1.8/src/EFCore.Relational/Update/UpdateSqlGenerator.cs), [Npgsql UpdateSqlGenerator](https://github.com/npgsql/efcore.pg/blob/v3.1.4/src/EFCore.PG/Update/Internal/NpgsqlUpdateSqlGenerator.cs)
