---	
title: Какое исключение бросать в swith, если подходящего case не нашлось?
tags: cleancode
published: false	
---	

## В чём проблема

```charp
enum SomeEnum
{
  One,
  Two,
  Three
}

void someFunc()
{
  SomeEnum value = someOtherFunc();
  switch(value)
  {
     case One:
       //Обрабатываем
       ...
       break;
     case Two:
      //Обрабатываем
       ...
       break;
     default:
       throw new Exception();   
       //Какое исключение правильно бросать?
  }
}
```

Есть несколько вариантов типа выбрасываемого исключения, в этой заметке я хочу поразмышлять - какое исключение, в какой ситуации будет правильнее использовать.

## А нужно ли вообще бросать исключение

Во-первых, я должен заметить, что могут быть ситуации, когда отсутствие подходящего case - это вообще не ошибка, так как ваш switch может изначально предусматривать только обработку ограниченного скопа значений.

Во-вторых, если то что подходящего case не нашлось - это предусмотренная вами ошибка, часть потока выполнения программы, часть её алгоритма работы, то best practice - это не использовать исключения для управления потоком выполнения программы (читайте [Are exceptions as control flow considered a serious antipattern? If so, Why?](https://softwareengineering.stackexchange.com/q/189222/236856)), перепишите метод так, чтобы возвращать ошибку как результат выполнения метода, например.

Здесь я рассматриваю вопрос, какое исключение нужно бросать, только в контексте ситуации, когда switch должен содержать обработку всего возможного диапазона значений, но по какой-то причине не содержит (добавили новое значение enum'а или добавили новый тип и т.п., а switch не изменили) - в большинстве случаев это будет исключительной ситуацией, в которой лучше всего как можно скорее упасть (почему лучше как можно скорее упасть читайте в статье [Fail Fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)) и сообщить пользователю или вызывающему модулю о возникновении непредвиденной ошибки выбросив исключение.

## Имеет ли значение какое исключение бросать

В C# не принято бросать исключение базового типа Exception, Microsoft изначально создало большое количество типов исключений на все возможные случаи жизни и общепринятая практика выбирать исключение подходящее под ситуацию для которой оно было создано, но само по себе бросание исключения абстрактного "правильного типа" несёт в себе только семантическую чистоту и согласованность кода, когда программные конструкции используются по назначению, для которого были созданы.

С практической же точки зрения использование конкретного типа исключения может обоснываться двумя причинами:

1. В программе существуют специальные обработчики для этого типа исключения
2. Упрощение процесса поиска места возникновения и причины исключения, а также передача дополнительной информации о контексте возникновения исключения

Здесь я буду в большей мере рассматривать вопрос "правильности" используемого типа, с точки зррения семантической релевантности использования конкретного типа исключения, в конкретной ситуации.

## Какие исключения подходят для выбрасывания из switch

### ArgumentException

> The exception that is thrown when one of the arguments provided to a method is not valid.

[ArgumentException](https://docs.microsoft.com/ru-ru/dotnet/api/system.argumentexception?view=netcore-3.1) отличный вариант, но нужно понимать, что семантика его использования предполагает то, что значение enum'а пришло как параметр метода. 

Поэтому пример кода в заголовке статьи не подойдёт для этой ситуации. С другой стороны, обработка значения enum всегда может быть вынесена в отдельный метод, так что значение будет являться параметром этого метода.

```charp
enum SomeEnum
{
  One,
  Two,
  Three
}

void someFunc()
{
  SomeEnum value = someOtherFunc();
  HandleSomeEnum(value);
}

void HandleSomeEnum(SomeEnum value)
{
  switch(value)
  {
     case One:
       //Обрабатываем
       ...
       break;
     case Two:
      //Обрабатываем
       ...
       break;
     default:
       throw new ArgumentException(message: $"Unexpected enum value: {value}", paramName: nameof(value));
  }
}
```

### ArgumentOutOfRangeException

> The exception that is thrown when the value of an argument is outside the allowable range of values as defined by the invoked method.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception?view=netcore-3.1) - наследуется от ArgumentException и тоже подразумевает, что обрабатываемое значение является параметром метода. Отличием от ArgumentException является то, что конструктор принимает текущее значение параметра отдельным параметром (не нужно передавать его как часть сообщения об ошибке).

```charp
enum SomeEnum
{
  One,
  Two,
  Three
}

void someFunc()
{
  SomeEnum value = someOtherFunc();
  HandleSomeEnum(value);
}

void HandleSomeEnum(SomeEnum value)
{
  switch(value)
  {
     case One:
       //Обрабатываем
       ...
       break;
     case Two:
      //Обрабатываем
       ...
       break;
     default:
       throw new ArgumentOurOfRangeException(paramName: nameof(value), actualValue: value, message: "Unexpected enum value");
  }
}
```

Решарпер, например, раньше использовал именно этот тип исключения при автогенерации switch/case блока (возможно на данный момент это уже не так).

### InvalidEnumArgumentException

>The exception thrown when using invalid arguments that are enumerators.
>
>This exception is thrown if you pass an invalid enumeration value to a method or when setting a property.

[InvalidEnumArgumentException](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.invalidenumargumentexception?view=netcore-3.1) - малоизвестный тип исключения, а всё из-за пространства имён: `System.ComponentModel` и сборки в которую он помещён: `System.ComponentModel.Primitives.dll` - вряд ли будет хорошей идеей использовать ссылки на них бездумно по всему вашему приложению.

> The [System.ComponentModel](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel?view=netcore-3.1) namespace provides classes that are used to implement the run-time and design-time behavior of components and controls. This namespace includes the base classes and interfaces for implementing attributes and type converters, binding to data sources, and licensing components.

В остальном это мог бы быть идеальный тип исключения, который позволяет передать и необработанное исключение и тип Enum (за исключением случаев использования swith для типов: [C# Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching#using-pattern-matching-switch-statements) - а не для Enum).

```charp
enum SomeEnum
{
  One,
  Two,
  Three
}

void someFunc()
{
  SomeEnum value = someOtherFunc();
  HandleSomeEnum(value);
}

void HandleSomeEnum(SomeEnum value)
{
  switch(value)
  {
     case One:
       //Обрабатываем
       ...
       break;
     case Two:
      //Обрабатываем
       ...
       break;
     default:
       throw new InvalidEnumArgumentException(paramName: nameof(value), invalidValue: value, enumClass: typeof(SomeEnum));
  }
}
```

### InvalidOperationException

> The exception that is thrown when a method call is invalid for the object's current state.
>
> InvalidOperationException is used in cases when the failure to invoke a method is caused by reasons other than invalid arguments. Typically, it is thrown when the state of an object cannot support the method call.

[InvalidOperationException](https://docs.microsoft.com/ru-ru/dotnet/api/system.invalidoperationexception?view=netcore-3.1) очень популярный тип исключения для данной ситуации, но всё же семантика использования этого исключения, предполагает наличие некоторого внутреннего состояния объекта, для которого данная операция некорректна и само по себе отсутствие обработчика в switch, по моему мнению, не подходит под это требование. С другой стороны, если обрабатываемое значение - это часть внутреннего состояния, тогда использование InvalidOperationException оправданно.

```charp
enum SomeEnum
{
  One,
  Two,
  Three
}

class SomeEnumClass
{
  private SomeEnum someEnumField;

  void someFunc()
  { 
    switch(value)
    {
       case One:
         //Обрабатываем
         ...
         break;
       case Two:
        //Обрабатываем
         ...
         break;
       default:
         throw new InvalidOperationException(message: $"Unexpected enum value: {someEnumField}");
    }
  }
}
```
