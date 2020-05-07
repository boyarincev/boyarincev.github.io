---	
title: Инкапсуляция - это не про сокрытие деталей реализации
tags: oop
published: false	
---	

Вроде живёшь себе спокойно, думаешь, что понимаешь кое-что в программировании, но вдруг во время дружеской беседы в зуме с друзьями программистами говоришь:

> Инкапсуляция? - Ну это сокрытие деталей реализации.

А тебе в ответ:

> Вообще-то инкапсуляция - это объединение данных и методов оперирующих этими данными вместе, а сокрытие тут совсем не обязательно.

Начинаем разбираться, идём в [Википедию](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)), а она говорит:

> Под инкапсуляцией понимают либо объединение вместе данных и методов, либо скрытие внутренних данных и состояния объекта и ограничение прямого доступа к ним, либо комбинацию этих определений. В общем разные авторы по разному её понимают.

И в качестве примера рассматривающего инкапсуляции исключительно как механизма для объединения данных и методов вместе даётся ссылка на статью [Encapsulation is not information hiding](https://www.javaworld.com/article/2075271/encapsulation-is-not-information-hiding.html) - а [Information Hiding](https://en.wikipedia.org/wiki/Information_hiding) - это по сути и есть сокрытие деталей реализации, так что явный аргумент пока не в мою пользу.

Но я то всегда воспринимал инкапсуляцию исключительно в контексте сокрытия деталей реализации, а у статьи никаких рефференсов на материалы, которые она использовала нет, так что предлагаю посмотреть ещё источники.

Берём книгу "Чистая Архитектура" Роберта Мартина, смотрим что там:

> Это возможность очертить круг связанных данных и функций. За пределами круга эти данные невидимы и доступны только некоторые функции.
>
>(Глава 5 Объектно-ориентированное программирвоание, 55 стр.)

Берём книгу "Объектно ориентированный анализ и проектирование с примерами приложений Третье издание" Гради Буч:

> Абстракция и инкапсуляция дополняют друг друга. В центре внимания абстракции находится наблюдаемое поведение объекта, а инкапсуляция сосредоточена на реализации, обеспечивающей заданное поведение. Как правило, инкапсуляция осуществляется с помощью сокрытия информации (а не просто сокрытия данных), т.е. утаивания всех несущественных деталей объекта.
>
>(Глава 2 Объектная модель, 82 стр.)

Открываем "Совершенный код Second edition" Стива Макконелла:

> Абстракция говорит: "Вы можете рассмотреть объект с общей точки зрения". Инкапсуляция добавляет: "Более того, вы не можете рассмотреть объект с иной точки зрения".
>
> Инкапсуляция позволяет вам смотреть на дом, но не даёт подойти достаточно близко, чтобы узнать, из чего сделана дверь. Инкапсуляция позволяет вам знать о существовании двери, о том открыта она или заперта, но при этом вы не можете узнать, из чего она сделана.
>
>(Глава 5 Проектирование при конструировании, 87 стр.)

> Абстракция помогает управлять сложностью, предоставляя модели, позволяющие игнорировать детали реализации. Инкапсуляция не позволяет узнать детали реализации, даже если вы этого захотите.
>
>(Глава 6 Классы, 135 стр.)

Ну и наконец берём "Object oriented software construction Second edition" Бертрана Мейера:

> One of the reasons for the misunderstanding mentioned above is very term "information hiding", which tends to suggest physical protection. "Encapsulation", sometimes used as a synonim for information hiding, is probably preferable in this respect.
>
>(3.2 Five Rules, 53 стр.)

Это он про то что не нужно термин "Information Hiding" понимать буквально как ограничение физического доступа к исходному коду, а только как языковые правила, помогающие ограничивать доступ одних модулей к свойствам других модулей.

В общем все лидеры мнений согласны с моим определением инкапсуляции. 

И тут мой оппонент находит этот документ: [Abstraction, Encapsulation, and Information Hiding](http://www.tonymarston.co.uk/php-mysql/abstraction.txt) By Edward V. Berard The Object Agency:

> I recently read a magazine article that said, "Encapsulation is just a fancy name for information hiding." Since the writer was non-technical, I just assumed that he was attempting to show that he really did not understand technical matters.
>
>As a process, encapsulation means the act of enclosing one or more items within a (physical or logical) container. Encapsulation, as an entity, refers to a package or an enclosure that holds (contains, encloses) one or more items. It is extremely important to note that nothing is said about "the walls of the enclosure." Specifically, they may be "transparent," "translucent," or even "opaque."

[Edward V. Berard](https://en.wikiquote.org/wiki/Edward_V._Berard) похоже более менее известная фигура, а статья выше судя по [scholar.google.com](https://scholar.google.com/scholar?hl=ru&as_sdt=0%2C5&as_ylo=1971&as_yhi=2000&q=Edward+V.+Berard+Abstraction%2C+encapsulation%2C+and+information+hiding&btnG=) была напечатана в книге [Essays on Object-Oriented Software Engineering](https://www.amazon.com/Essays-Object-Oriented-Software-Engineering-Edward/dp/0132888955) и датируется 1993 годом (пдф книги, правда, я найти не смог, чтобы точно в этом убедиться).

Хороший аргумент в пользу точки зрения о том, что инкапсуляция - это только про объединение вместе данных и методов.

Наконец решаем посмотреть, что обо всё этом думает stackoverflow. Сначала ничего интересного не находим, большинство определений так или иначе крутятся вокруг моей интерпретации:

[Difference between abstraction and encapsulation?](https://stackoverflow.com/q/742341/5402731)

[Encapsulation vs Abstraction?](https://stackoverflow.com/q/8960918/5402731)

[encapsulation vs abstraction real world example](https://stackoverflow.com/q/12072980/5402731)

Но встречается и [другой взгляд на инкапсуляцию](https://stackoverflow.com/a/28614407/5402731):

>Encapsulation is probably the most misunderstood concept of OOP.
>
>Encapsulation is NOT data hiding!
>
>This doesn't require data hiding at all. The advantage is that the code is more manageable, because you can clearly see all the related "things" (that is, data and functions) grouped together, so at a glance you know what you have (data) and what you can do with it (methods).

А в качестве рефференсов к ответу уже известные нам статьи [Encapsulation is not information hiding](http://www.javaworld.com/article/2075271/core-java/encapsulation-is-not-information-hiding.html) и [Abstraction, Encapsulation, and Information Hiding](http://www.tonymarston.co.uk/php-mysql/abstraction.txt) 

На этом уже собирались рассходиться, так и не найдя правды - истоки определения слова Инкапсуляция лежат где-то в глубине времён, никто не упоминает кто этот термин ввёл, чтобы прочитать в источнике, что автор вложил в него, а разные точки зрения на него существуют как минимум 30 лет. Но тут каким-то чудом я натыкаюсь вот на [этот ответ](https://stackoverflow.com/a/39499367/5402731)
