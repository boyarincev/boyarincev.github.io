---
layout: post
title: Представление чисел с плавающей точкой в памяти в дотнете
tags: floating-point-numbers
published: false
---

Это не полноценная статья, я не буду подробно рассказывать о устройстве чисел с плавающей точкой, а порекомендую ссылки на другие статьи рассказывающие об этом:

У Джона Скита есть две коротенькие статьи на эту тему, первая [Binary floating point and .NET](https://csharpindepth.com/Articles/FloatingPoint) про double и float, вторая [Decimal floating point in .NET](https://csharpindepth.com/Articles/Decimal) про decimal, можно сначала прочитать их.

Сам Скит рекомендует к прочтению эту статью: [Floating Point in .NET part 1: Concepts and Formats](https://www.extremeoptimization.com/resources/Articles/FPDotNetConceptsAndFormats.aspx) - она  более подробная и позволит уже более основательно разобраться в вопросе.

В этой статье я только разберу, конкретные байтовые представления чисел в памяти, подробно остановившись на том, как они получаются. Мне кажется, таких конкретных примеров не хватает в статьях выше для лучшего понимания.

## Кратко о том как Double хранится в памяти 


## Как можно получить байтовое представление Double
[Класс BitConverter](https://docs.microsoft.com/ru-ru/dotnet/api/system.bitconverter?view=netframework-4.8) позволяет получить байтовое представление базовых типов или наоборот преобразовать байтовое представление в базовый тип.

```csharp
var bytesArray = BitConverter.GetBytes(1.0d); //Получить представление 1 типа double в виде массива байт

// 1.0d представленное как массив байт: 0; 0; 0; 0; 0; 0; 240; 63
Console.WriteLine($"1.0d представленное как массив байт: {string.Join("; ", bytesArray)}"); 

//1.0d представленное как массив байт в виде 16-ричных чисел: 00-00-00-00-00-00-F0-3F
Console.WriteLine($"1.0d представленное как массив байт в виде 16-ричных чисел: {BitConverter.ToString(bytesArray)}"); 

```

Double занимает 64 бита в памяти, поэтому метод `GetBytes` возвращает нам массив из 8 байт (каждый байт занимает 8 бит).

Но для разбора удобнее всего будет числа представлять в двоичном виде. Можно для этого каждый элемент массива представить в двоичном виде:

```csharp
//1.0d представленное в двоичном виде: 00000000000000000000000000000000000000000000000011110000111111
Console.WriteLine($"1.0d представленное в двоичном виде: {string.Join("", bytesArray.Select(ba => ba == 0 ? "00000000" : Convert.ToString(ba, 2)))}"); 
```

А можно воспользоваться методом `DoubleToInt64Bits` - он возвращает 64 битное целое число, которое в двоичном виде соответствует байтовому представлению числа типа double:

```csharp
var oneinlong = BitConverter.DoubleToInt64Bits(1.0d);
var onvebinary = Convert.ToString(oneint64bits, 2);
```

Можно использовать статический метод `GetBits` типа Double

```csharp

```
