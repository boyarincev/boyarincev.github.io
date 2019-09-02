---
title: Несколько интересных статей про Entity Framework Core
tags: entityframework
---

Не помню уже, как велики были возможности логирования в классическом Entity Framework, но у Entity Framework Core с этим точно всё в порядке, статья на эту тему: [Настройка логирования в Entity Frmework Core](https://msdn.microsoft.com/magazine/mt830355)

Оказывается Entity Framework Core [может использовать INotifyPropertyChanged интерфейс](https://blog.oneunicorn.com/2016/11/16/notification-entities-in-ef-core-1-1/) для того, чтобы напрямую узнавать о изменениях в моделях и не использовать снапшот данных для их поиска. Думаю, что это можно попробовать использовать в высокопроизводительных сценариях.

[Статья](https://benjii.me/2018/01/expression-projection-magic-entity-framework-core/) с разбором продвинутых техник использования проекций в Entity Framework Core.
1. Как хранить проекции для их переиспользования.
2. Как хранить и использовать вложенные проекции для проекций.
3. И наконец самое интересное - способ создания вложенных проекций не для коллекций, а для одиночных сущностей (для вложенной проекции одиночной сущности нельзя вызвать Select, а значит и нельзя сделать проекцию.

Ну и наконец две статьи в которых рассказывается, как можно изменить генерацию SQL в Entity Framework Core.

1. [Extending SQL Generation in Entity Framework Core](https://www.chasingdevops.com/sql-generation-ef-core/)
2. [Реализуем свой оператор в Entity Framework Core](https://habr.com/ru/post/351556/)
