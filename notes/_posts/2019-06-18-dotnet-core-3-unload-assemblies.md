---
title: Выгрузка сборок в dotnet core 3
tags: dotnetcore3
---

Благая весть идёт: предстоящий .NET Core 3 принесёт нам возможность выгружать сборки из памяти. Сделать это можно будет с помощью класса [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext). 
[А вот тут инструкция по использованию](https://docs.microsoft.com/en-us/dotnet/standard/assembly/unloadability-howto).

Так как .NET Core [забрал у нас Домены Приложений](https://stackoverflow.com/questions/27266907/no-appdomains-in-net-core-why), а это была единственная возможность выгрузить сборку в управляемом коде, то новость просто отличная.
