---	
title: Инкапсуляция - что ты такое?
tags: oop misconceptions
published: true
---	

Вроде живёшь себе спокойно, думаешь, что понимаешь кое-что в программировании, но вдруг во время беседы в зуме с друзьями программистами говоришь:

>Инкапсуляция? - Ну это сокрытие деталей реализации.

А тебе в ответ:

>Вообще-то инкапсуляция - это объединение данных и методов оперирующих этими данными, а сокрытие тут совсем не обязательно.

Начинаем разбираться, идём в [Википедию](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)), а она пишет:

>Под инкапсуляцией понимают либо объединение вместе данных и методов, либо скрытие внутренних данных и состояния объекта и ограничение прямого доступа к ним, либо комбинацию этих определений. В общем разные авторы по разному её понимают.

*Это не дословная цитата*

И в качестве примера рассматривающего инкапсуляции исключительно как механизма для объединения данных и методов вместе даётся ссылка на статью ["Encapsulation is not information hiding"](https://www.javaworld.com/article/2075271/encapsulation-is-not-information-hiding.html) - а [Information Hiding](https://en.wikipedia.org/wiki/Information_hiding) - это по сути и есть сокрытие деталей реализации, так что явный аргумент пока не в мою пользу.

Но я то инкапсуляцию без сокрытия деталей реализации никогда не представлял, да, мы объединяем вместе данные и методы, но не это главное, а у статьи, к тому же, никаких рефференсов на материалы, которые она использовала нет, так что предлагаю посмотреть ещё источники.

<!--excerpt-->

Берём книгу "Чистая Архитектура" Роберта Мартина, смотрим что там:

>Это возможность очертить круг связанных данных и функций. За пределами круга эти данные невидимы и доступны только некоторые функции.

*Глава 5 Объектно-ориентированное программирование, 55 стр.*

Берём книгу "Объектно ориентированный анализ и проектирование с примерами приложений Третье издание" Гради Буч:

>Абстракция и инкапсуляция дополняют друг друга. В центре внимания абстракции находится наблюдаемое поведение объекта, а инкапсуляция сосредоточена на реализации, обеспечивающей заданное поведение. Как правило, инкапсуляция осуществляется с помощью сокрытия информации (а не просто сокрытия данных), т.е. утаивания всех несущественных деталей объекта.

*Глава 2 Объектная модель, 82 стр.*

Открываем "Совершенный код Second edition" Стива Макконелла:

>Абстракция говорит: "Вы можете рассмотреть объект с общей точки зрения". Инкапсуляция добавляет: "Более того, вы не можете рассмотреть объект с иной точки зрения".
>
>...
>
>Инкапсуляция позволяет вам смотреть на дом, но не даёт подойти достаточно близко, чтобы узнать, из чего сделана дверь. Инкапсуляция позволяет вам знать о существовании двери, о том открыта она или заперта, но при этом вы не можете узнать, из чего она сделана.

*Глава 5 Проектирование при конструировании, 87 стр.*

>Абстракция помогает управлять сложностью, предоставляя модели, позволяющие игнорировать детали реализации. Инкапсуляция не позволяет узнать детали реализации, даже если вы этого захотите.

*Глава 6 Классы, 135 стр.*

Ну и наконец берём "Object oriented software construction Second edition" Бертрана Мейера:

>One of the reasons for the misunderstanding mentioned above is very term "information hiding", which tends to suggest physical protection. "Encapsulation", sometimes used as a synonim for information hiding, is probably preferable in this respect.

*3.2 Five Rules, 53 стр.*

Это он про то что не нужно термин "Information Hiding" понимать буквально как ограничение физического доступа к исходному коду, а только как языковые правила, помогающие ограничивать доступ одних модулей к свойствам других модулей.

В общем все лидеры мнений согласны с моим определением инкапсуляции. 

И тут мой оппонент находит документ ["Abstraction, Encapsulation, and Information Hiding"](http://www.tonymarston.co.uk/php-mysql/abstraction.txt) за авторством Edward V. Berard:

>I recently read a magazine article that said, "Encapsulation is just a fancy name for information hiding." Since the writer was non-technical, I just assumed that he was attempting to show that he really did not understand technical matters.
>
>...
>
>As a process, encapsulation means the act of enclosing one or more items within a (physical or logical) container. Encapsulation, as an entity, refers to a package or an enclosure that holds (contains, encloses) one or more items. It is extremely important to note that nothing is said about "the walls of the enclosure." Specifically, they may be "transparent," "translucent," or even "opaque."

[Edward V. Berard](https://en.wikiquote.org/wiki/Edward_V._Berard) похоже более менее известная фигура, а статья судя по [scholar.google.com](https://scholar.google.com/scholar?hl=ru&as_sdt=0%2C5&as_ylo=1971&as_yhi=2000&q=Edward+V.+Berard+Abstraction%2C+encapsulation%2C+and+information+hiding&btnG=) была напечатана в книге ["Essays on Object-Oriented Software Engineering"](https://www.amazon.com/Essays-Object-Oriented-Software-Engineering-Edward/dp/0132888955) и датируется 1993 годом (но пдф книги, чтобы лично в этом убедиться я не нашёл).

Хороший аргумент в пользу точки зрения о том, что инкапсуляция - это только про объединение вместе данных и методов.

Наконец решаем посмотреть, что обо всё этом думает stackoverflow. Сначала ничего интересного не находим, большинство определений так или иначе крутятся вокруг моей интерпретации:

[Difference between abstraction and encapsulation?](https://stackoverflow.com/q/742341/5402731)

[Encapsulation vs Abstraction?](https://stackoverflow.com/q/8960918/5402731)

[encapsulation vs abstraction real world example](https://stackoverflow.com/q/12072980/5402731)

Но встречается и [другой взгляд на инкапсуляцию](https://stackoverflow.com/a/28614407/5402731):

>Encapsulation is probably the most misunderstood concept of OOP.
>
>...
>
>Encapsulation is NOT data hiding!
>
>...
>
>This doesn't require data hiding at all. The advantage is that the code is more manageable, because you can clearly see all the related "things" (that is, data and functions) grouped together, so at a glance you know what you have (data) and what you can do with it (methods).

А в качестве рефференсов к ответу уже известные нам статьи ["Encapsulation is not information hiding"](http://www.javaworld.com/article/2075271/core-java/encapsulation-is-not-information-hiding.html) и ["Abstraction, Encapsulation, and Information Hiding"](http://www.tonymarston.co.uk/php-mysql/abstraction.txt) 

На этом уже собирались рассходиться, так и не найдя правды - истоки определения слова Инкапсуляция лежат где-то в глубине времён, никто не упоминает кто этот термин ввёл, чтобы прочитать в источнике, что автор вложил в него, а разные точки зрения на него существуют как минимум 30 лет. Но тут каким-то чудом я натыкаюсь на [этот ответ](https://stackoverflow.com/a/39499367/5402731) и я думаю, что именно он должен быть в википедии. Приведу самую интересную его часть - это историю использования термина инкапсуляция:

>Encapsulation and information hiding are very closely linked concepts, though their precise definitions vary depending on who you talk to.
>
>The concept of "information hiding" was first described by [Parnas (1971)](http://repository.cmu.edu/compsci/1829/) who suggested that access to information should be restricted to reduce the interconnectedness of a system. He proposed that this would facilitate splitting of a system into modules while maintaining a user-friendly external interface and allowing implementation details to be changed without affecting clients.
>
>The term "encapsulation" was coined by [Zilles (1973)](http://dl.acm.org/citation.cfm?doid=800021.808305) to describe the use of procedures to control access to underlying data in order to reduce system complexity and protect data from dangerous modification.
>
>Subsequently, [Parnas (1978)](http://dl.acm.org/citation.cfm?id=800099.803218) described information hiding and encapsulation (and abstraction) as synonymous terms, which describe the hiding of details of a system that are likely to change. However, distinctions have been drawn between information hiding and encapsulation, such as by [Micallef (1987)](http://dl.acm.org/citation.cfm?id=36160.36167), who described encapsulation as "the strict enforcement of information hiding". Some authors, such as [Cohen (1984)](http://doi.acm.org/10.1145/948415.948418) and [Abreu and Melo (1996)](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.49.3479) describe "encapsulation mechanisms", especially in object-oriented programming languages, as allowing information hiding.
>
>[Meyers (2000)](http://www.drdobbs.com/cpp/how-non-member-functions-improve-encapsu/184401197) suggests that the degree to which a piece of code is encapsulated depends on the amount of code which would be broken if it changed. In this sense, private data and methods are more encapsulated the fewer methods by which they can be accessed. In contrast, public data and methods are completely unencapsulated, as the amount of code by which they can be accessed is unknown.
>
>Conversely, [Rogers (2001)](http://www.javaworld.com/article/2075271/core-java/encapsulation-is-not-information-hiding.html) suggests that encapsulation is simply the language mechanism that allows data to be bundled with methods that operate on that data. He claims that encapsulation fundamentally has nothing to do with information hiding. However, this definition is counter to almost all usage of the term in the academic literature in the 28 years prior to the publication of his article. There are a few other examples of this usage, for example [Archer and Stinson (1995)](http://repository.cmu.edu/sei/214/), but they are few and far between and not particularly notable.
>
>In conclusion, information hiding is the idea that information should be hidden so that a design can be changed without affecting clients. This allows for increased flexibility and robustness. Encapsulation may be considered to be the same as information hiding, but the term is often used to describe the practical implementation of information hiding, especially in object-oriented programming.

[Rogers 2001](http://www.javaworld.com/article/2075271/core-java/encapsulation-is-not-information-hiding.html) - это уже известная нам статья на которую ссылается википедия и по мению автора ответа: "Взгляд на то что инкапсуляция сводится к объединению данных и методов вместе противоречит почти всем примерам использования этого термина в академической литературе за почти 28 лет предшествовавших этой статье".

Но самое интересное - это конечно то, что в ответе упоминается происхождение термина инкапсуляция - его ввёл Stephen N. Zitles в статье ["Procedural Encapsulation: A Linguistic Protection Technique"](https://dl.acm.org/doi/pdf/10.1145/800021.808305), я попробовал найти более ранние упоминания инкапсуляции и не нашёл, так что вероятно так и есть. В этой статье автор берёт для примера данные потока операционной системы и рассматривает техники, которые бы помогли представить их, только в качестве операций (процедур), которые над потоком могут производиться и скрыть от клиента сами данные, над которыми эти операции прозводятся.

И вот то что он понимал под инкапсуляцией:

>We will use the term procedural encapsulation to refer to the technique of representing system components in terms of one or more procedures such that interactions among compoments are limited to procedure calls
>When only the defined operations are used, a program is independent of the particular representation
chosen for a stream. **The term "procedural encapsulation" is derived from the fact that the procedures encapsulate or isolate the representational aspects of the data.**
>
>...
>
>**Using the technique of encapsulation, a data type is completely characterized by the set of operations defined on the data of that type and the observable relationships between those operations.**
>
>...
>
>Procedural encapsulation also supports the "minimal assumptions" property. When each module is a set of related procedures, the assumptions about one module which can be exploited by another module are limited by the ways in which each procedure can access other procedures.
>
>...
>
>**Procedural encapsulation** is a useful technique for building modular, reliable operating systems. With appropriate support from the operating system, it **can provide protection for data objects. Both accidental destruction or
modification, and intentional subversion are prevented.** This form of protection is easy to use correctly since it is integrated with a natural building block for system components: procedures. The set of assumptions that can be made
about a module is minimized. Because **representational details are encapsulated within procedures, the representation cannot be manipulated nor its form discovered. Therefore, the representation can be modified to fix bugs or to enhance performance without introducing problems caused by invalidating assumptions made in other modules.**

То есть с самого своего рождения в программировании термин инкапсуляция подразумевал сокрытие информации.

А вывод такой: всегда ищите первоисточник, а не читайте чью-то интерпретацию, иначе можно потерять изначально вложенный смысл.
