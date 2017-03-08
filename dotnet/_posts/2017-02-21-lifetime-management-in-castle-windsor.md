---
layout: post
title: Управление жизненным циклом объектов в Castle Windsor
tags: castle-windsor
published: false
---

Возможно, наиболее непонимаемая вещь в Castle Windsor - это его система управления жизненным циклом объектов. Надеюсь эта запись поможет прояснить некоторые сложные моменты.

# Castle Windsor хранит ссылки на создаваемые им объекты

Одна из ключевых обязанностей IoC-контейнера (далее просто контейнер) - это управление жизненным циклом, полученных из него объектов. Контейнер создает объект, 
настраивает его и отдаем вам, после чего вы можете им пользоваться, а когда объект становится не нужен - уничтожает его.
Для того, чтобы знать какие объекты он должен уничтожить, когда придет их время и получить к ним доступ и 
хранятся ссылки на создаваемые объекты.

# Что подразумевается под "Уничтожением объектов" ?

Давайте рассмотрим пример:

```
public class UnitOfWork
{
   public void Init()
   {
      // some initialization logic...
   }
   public void Commit()
   {
      // commit or rollback the UoW
   }
}
```

# Должен ли я вызывать Dispose метод на объектах полученных из Castle Windsor?

В .NET фреймворке есть довольно простое правило:

> Диспосить то что вы создали, когда вы закончили использовать его (Dispose what you’ve created, when you’re done using it).

Так как контейнер создал объект, то это его обязанность диспосить этот объект.

Если вы хотите больше узнать о диспосинге объектов, то вот еще несколько ссылок:

1. [Dispose паттерн в .NET](https://msdn.microsoft.com/en-us/library/b1yfkh5e(v=vs.110).aspx)

2. [Забавный ответ Эрика Липперта о диспосинге объектов](http://stackoverflow.com/a/35567469/5402731)

3. [Первая часть статьи Эрика Липперта о финалайзерах](https://ericlippert.com/2015/05/18/when-everything-you-know-is-wrong-part-one/)

4. [Вторая часть статьи Эрика Липперта о финалайзерах](https://ericlippert.com/2015/05/21/when-everything-you-know-is-wrong-part-two/)

# Castle Windsor хранит ссылки на объекты которые не требуется диспосить (не реализуют интерфейс IDisposable)?

По дефолту - да. Castle Windsor будет хранить ссылки на любые объекты.

Ссылки:

1. [Типы жизненных стилей объектов в Castle Windsor](https://github.com/castleproject/Windsor/blob/master/docs/lifestyles.md)

2. [Жизненный цикл объектов в Castle Windsor](https://github.com/castleproject/Windsor/blob/master/docs/lifecycle.md)

3. [Release Policy в Castle Windsor](https://github.com/castleproject/Windsor/blob/master/docs/release-policy.md)
