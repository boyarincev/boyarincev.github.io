---	
title: Суть принципа Open-Closed	
tags: SOLID	
published: false	
---	

Вообще есть такое чувство, что на данный момент SOLID-принципы уже как-то совсем устарели, никто вокруг про них не говорит и не пишет, но если вдруг вспоминать о них, то Open-Closed в них самый неоднозначный.	

Если посмотреть на формальное определение: 	
> Модули должны быть открыты для расширения, но закрыты для модификации 	
То с первой частью всё более менее понятно: продумываем дизайн модуля так, чтобы его поведение было расширяемо - объясняя этот принцип, все в общем-то и сосредотачиваются на этой первой части, показывая разные подходы к расширяемости, полезность расширяемости интуитивно тоже сразу понятна, но что значит вторая часть?	

"Модули должны быть закрыты для модификации" - Чего мы достигаем закрывая модули от модификации, какую цель преследует этот запрет? Мы говорим про запрет вносить изменения прямо в модули? Или что такое значит закрыть?	

Для объяснений [слово предоставляется Роберту Мартину](https://blog.cleancoder.com/uncle-bob/2013/03/08/AnOpenAndClosedCase.html), который и популяризировал этот принцип, а через 17 лет после этого решил подробнее объяснить, что значит вторая часть принципа:	

> It should be easy to change the behavior of a module without changing the source code of that module. This doesn’t mean you will never change the source code, this doesn’t mean you can stop using your version control systems (sheesh!). What it means is that you should strive to get your code into a position such that, when behavior changes in expected ways, you don’t have to make sweeping changes to all the modules of the system. Ideally, you will be able to add the new behavior by adding new code, and changing little or no old code.	
Разберём каждое предложение отдельно:	

> It should be easy to change the behavior of a module without changing the source code of that module. 	
То есть, при следовании этому принципу точки расширения должны позволять нам произвести расширения модуля, не трогая его исходный код - это может быть полезно, например, когда у нас нет доступа к этому исходному коду. Но из этого не следует, что модуль должен быть закрыт для модификации - это только дополнительные требования к тому, как нужно продумать дизайн расширяемости модуля.	

> This doesn’t mean you will never change the source code, this doesn’t mean you can stop using your version control systems (sheesh!). 	
Ну вот, похоже, что Роберт Мартин согласен со мной.	

> What it means is that you should strive to get your code into a position such that, when behavior changes in expected ways, you don’t have to make sweeping changes to all the modules of the system. 	
А вот это что-то новенькое - изменения поведения модуля не должны вызывать каскад изменений в системе - но это тоже больше про дизайн расширяемости, про сокрытие информации и инкапсуляцию, но ничего про отказ от модификации модуля тут нет.	

> Ideally, you will be able to add the new behavior by adding new code, and changing little or no old code.	
А тут взгляд под другим углом - дизайн расширяемости должен позволять добавление нового поведения без изменения старого кода - могу предположить, что это может быть полезно, если у нас он не покрыт юнит-тестами и мы не хотим проводить тестирование старого функционала, после внесения изменений в старый код. Но тут тоже ничего нет про то, что мы должны отказаться от модификации этого старого кода, когда юнит-тесты есть или мы не против провести тестирование.	

Но вообще в итоге намного понятнее не стало и не мне одному надо сказать.	

Вот, например, статья Сергея Теплякова [Open/Closed Principle](http://sergeyteplyakov.blogspot.com/2014/08/open-closed-principle.html), в которой он пытаясь понять, что имелось в виду в этом принципе, обращается к настоящему отцу принципа Бертрану Мейеру и приходит к выводу, что в этом принципе:	

> Мейер говорит о том, что интерфейс модуля должен быть закрытым, а реализация и точное поведение может варьироваться и оставаться открытым к изменениям.	
И это определение довольно близко по смыслу к словам Роберта Мартина о том, что изменения в модуле не должны вызывать каскад изменений в других модулях системы. Потому, что если наш интерфейс не меняется, то никто из пользователей модуля не будет побеспокоен изменениями вносимыми в модуль.	

Другим человеком, который пытался разобраться с этим принципом был Джон Скит. В своей статье [THE OPEN-CLOSED PRINCIPLE, IN REVIEW](https://codeblog.jonskeet.uk/2013/03/15/the-open-closed-principle-in-review/) он тоже сначала уходит к Бертрану Мейеру, правда по его словам, Бертран Мейер имел в виду, что:	

> The idea was that once completed, the implementation of a class could only be modified to correct errors; new or changed features would require that a different class be created. That class could reuse coding from the original class through inheritance. The derived subclass might or might not have the same interface as the original class.	
То есть закрытие модификации - это именно закрытие исходного кода модуля от внесения в него дальнейших изменений.	

Как видите Сергей Тепляков и Джон Скит нашли довольно разные по смыслу объяснения принципа Бертраном Мейером. Джон Скит сказал, что взял эту цитату из Википедии, Сергей Тепляков вообще не сказал откуда взял свою версию.	

Как бы то ни было с такой точкой зрения Бертрана Мейера Джон Скит не соглашается и продолжает размышлять над принципом дальше и приходит к выводу, что "закрыт" наиболее близко по смыслу к идее под названием Protected Variation, которая звучит как:	

> Identify points of predicted variation and create a stable interface around them.	
В общем приходит к тому же выводу, что и Сергей Тепляков, что суть принципа в том, что изменения должны быть за закрытым/стабильным интерфейсом, который не будет меняться и соответственно изменения поведения модуля не будут заметны внешним пользователям модуля.	

Что в итоге:	

- Скорее всего закрыт в принципе значит: Дизайн модуля продуман таким образом, что дальнейшие расширения поведения (открыт для расширения) не вызывают необходимость изменять публичный интерфейс модуля и таким образом интерфейс модуля можно назвать закрытым от дальнейших изменений.	

- Open-Closed - мутный никому не ясный до конца принцип, не тратьте своё время на него, прекращайте сразу же разговор о нём, когда его кто-то начинает, ну или спросите собеседника, что в этом принципе имеется в виду под закрыт и чем это закрытие полезно.

## В чём проблема принципа	

Принцип противоречиво и неясно описан, приходится прилагать очень много ментальных усилий, чтобы понять настоящий смысл этого принципа.	

## Определение принципа от Боба Мартина	

Краткий экскурс - Мейер, Статья Боба Мартина, почему я начинаю описание с книги, а не со статьи. 	

Начнём с определения принципа данного в книге "Принципы, паттерны и методики гибкой разработки на языке C#"	

> Программные сущности (класы, модули, функции и т.п.) должны быть открыты для расширения, но закрыты для модификации	
И дальнейшего краткого объяснения принципа	

> Если единственное изменение в каком-то месте программы приводит к каскаду изменений в зависимых модулях, то дизайн попахивает жесткостью. Принцип OCP рекомендует переработать систему так, чтобы в будущем аналогичные изменения можно было реализовать путем добавления нового кода, а не изменения уже работающего. На первый взгляд это кажется недостижимым идеалом, но существуют относительно простые и эффективные способы приблизиться к нему.	
Что нам теперь известно о принципе:	
1. Если внесение одного какого-либо изменения влечёт каскад изменений в зависимых модулях, то дизайн жёсткий	
2. Следование принципу OCP означает, то что внести изменение из первого пункта в программу можно добавлением нового кода, а не изменением старого	

Хотя это явно и не проговаривается, но то как построены предложения позволяет предположить, что если мы внесём изменения в программу с помощью добавления нового кода, а не изменения старого, то следствием этого будет, то что внесение такого изменения не повлечёт каскад изменений в зависимых модулях. С другой стороны - это не значит, что внести изменения без каскада изменений можно только следуя принципу OCP.	

Теперь давайте поговорим про ценность каждого из этих пунктов. Ценность первого пункта (возможность внести изменения в модуль без каскада изменений в зависимых модулях) очевидна и понятна, ценность второго пункта, а именно: возможность внести изменение добавлением нового кода, а не правки старого - сама по себе не очевидна, да это может быть полезно в каких-то ситуациях, но мы говорим про принцип проектирования, которому должно соответствовать максимальное количество программных сущностей, в определении Боба Мартина нет никаких уточнений или исключений - все программные сущности должны соответствовать принципу OCP. Лично я не готов при проектировании всегда следовать принципу только ради того, чтобы получить возможность внести изменение, добавив новый код, с другой стороны, вероятно, я практически всегда готов был бы следовать тому принципу, который поможет избежать каскада изменений в зависимых модулях. Но как я уже написал выше, на данный момент не говорится, что достижение этой цели возможно только через использование OCP.