---
layout: post
title: Представление чисел с плавающей точкой в памяти в дотнете
tags: floating-point-numbers
published: false
---

Это не полноценная статья, я не буду подробно рассказывать о устройстве чисел с плавающей точкой, для этого я порекомендую ссылки на другие статьи рассказывающие об этом:

У Джона Скита есть две коротенькие статьи на эту тему, первая [Binary floating point and .NET](https://csharpindepth.com/Articles/FloatingPoint) про double и float, вторая [Decimal floating point in .NET](https://csharpindepth.com/Articles/Decimal) про decimal, можно сначала прочитать их.

Сам Скит рекомендует к прочтению эту статью: [Floating Point in .NET part 1: Concepts and Formats](https://www.extremeoptimization.com/resources/Articles/FPDotNetConceptsAndFormats.aspx) - она  более подробная и позволит уже более основательно разобраться в вопросе.

В этой статье я только разберу, конкретные байтовые представления чисел в памяти, подробно остановившись на том, как они получаются. Мне кажется, таких конкретных примеров не хватает в статьях выше для лучшего понимания.