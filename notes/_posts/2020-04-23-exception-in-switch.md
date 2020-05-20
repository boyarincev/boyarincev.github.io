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

Есть несколько вариантов типа выбрасываемого исключения, в этой заметке я хочу рассмотреть какое исключение, в какой ситуации, будет правильно использовать.

## А нужно ли вообще бросать исключение

Во-первых, я должен заметить, что могут быть ситуации, когда отсутствие подходящего case - это вообще не ошибка, так как ваш switch может изначально предусматривать только обработку ограниченного скопа значений.

Во-вторых, если то что подходящего case не нашлось - это предусмотренная вами ошибка, часть потока выполнения программы, часть её алгоритма работы, то best practice - это не использовать исключения для управления потоком выполнения программы (читайте [Are exceptions as control flow considered a serious antipattern? If so, Why?](https://softwareengineering.stackexchange.com/q/189222/236856)), перепишите метод так, чтобы возвращать ошибку как результат выполнения метода, например.

Здесь я рассматриваю вопрос, какое исключение нужно бросать, только в контексте ситуации, когда switch должен содержать обработку всего возможного диапазона значений, но по какой-то причине не содержит (добавили новое значение enum'а или добавили новый тип и т.п., а switch не изменили) - в большинстве случаев это будет исключительной ситуацией, в которой лучше всего как можно скорее упасть (почему лучше как можно скорее упасть читайте в статье [Fail Fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)) и сообщить пользователю или вызывающему модулю о возникновении непредвиденной ошибки выбросив исключение.

## Имеет ли значение какое исключение бросать

В C# не принято бросать исключение базового типа Exception, Microsoft изначально создало большое количество типов исключений на все возможные случаи жизни и общепринятая практика выбирать исключение подходящее под ситуацию для которой оно было создано, но само по себе бросание исключения абстрактного "правильного типа" несёт в себе только семантическую чистоту и согласованность кода, когда программные конструкции используются по назначению, для которого были созданы.

С практической же точки зрения использование конкретного типа исключения может обоснываться двумя причинами:

1. В программе существуют специальные обработчики для этого типа исключения
2. Упрощение процесса поиска места возникновения и причины исключения, а также передача дополнительной информации о контексте возникновения исключения

Здесь я буду рассматривать вопрос "правильности" используемого типа, только с точки зррения семантической релевантности использования конкретного типа исключения, в конкретной ситуации.

## Какие исключения подходят для выбрасывания из switch

### ArgumentException

> Исключение, которое выдается, если один из передаваемых методу аргументов является недопустимым.

[ArgumentException](https://docs.microsoft.com/ru-ru/dotnet/api/system.argumentexception?view=netcore-3.1) отличный вариант, но нужно понимать, что семантика его использования предполагает то, что значение enum'а пришло как параметр метода. 

Поэтому пример кода в заголовке статьи не подойдёт для этой ситуации. С другой стороны, обработка значения enum всегда может быть вынесена в отдельный метод, так что значение будет являться параметром этого метода:

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

[InvalidEnumArgumentException](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.invalidenumargumentexception?view=netcore-3.1) - малоизвестный тип исключения, а всё из-за пространства имён: `System.ComponentModel` и сборки в которую он помещён: `System.ComponentModel.Primitives.dll`

### InvalidOperationException


