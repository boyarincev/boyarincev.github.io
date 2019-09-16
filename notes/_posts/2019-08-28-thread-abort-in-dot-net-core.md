---
title: Прерывание выполнения потока в dot.net core
tags: dotnetcore
published: true
---

В dot.net core выпилили метод `Thread.Abort()` и это значит, что теперь невозможно насильственное прерывание выполнения потока из вне этого потока средствами управляемого кода.

Так что все эти знания о нюансах использования `Thread.Abort()` и чудесных способностях выбираться из try/catch блоков исключения `ThreadAbortException`, которые мы заучивали в Рихтере, теперь бесполезны - сегодня выкидывание ThreadAbortException невозможно.

[Цитата из документации](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadabortexception?view=netcore-3.0#remarks):
> Even though this type exists in .NET Core, since Abort is not supported, the common language runtime won't ever throw ThreadAbortException.

В этом [Issue на GitHub](https://github.com/dotnet/coreclr/issues/20705) подробное обсуждение плюсов и минусов такого решения.

Если кратко о причинах, то это было сделано, потому что очень дорого писать библиотеки, которые гарантировали бы защиту от corrupted state всего процесса, то есть гарантировали бы безопасность дальнейшей работы процесса, в условиях, когда в любой момент выполнение потока может быть прервано - а именно такие библиотеки нужно было написать разработчика .NET Core. Поэтому если вам нужно прерывать выполнение потока, то используйте CancellationToken, если это ваш код или код чужих разработчиков поддерживающий его, или запускайте код, который потребует прерывание выполнения потока в отдельном процессе и прекращайте его целиком.

Вот некоторые цитаты из этого обсуждения:

> The key problem with thread abort is that it affects reliability of the whole stack. If you are using thread abort, all code (ie all libraries) running in the process have to be robust against being killed by thread abort at any point. It is extremely expensive to audit and write libraries with this constrain.

> Try to review any code that is doing a more complex managed/unmanaged interop (e.g. sockets in CoreFX) and try to find places where inserting a thread abort exception would cause bad things to happen. I am sure that you are going to find many places where inserting thread abort would lead to hangs, crashes or data corruptions. And there will be a lot more that the tests would discover. People just do not naturally write code that is able to recover from being aborted at any point.

> It is hard to tell where it is "safe" to insert the thread aborts. Libraries that need to be robust in presence of Thread.Abort need to be annotated for it, coded in a special way and stress tested.

> We have tried to do this in .NET Framework: It was a full time job for several people to run a stress harness that inserted thread about at random points in .NET Framework, and file and fix bugs on the crashes, hangs and data corruptions that it hit. This was done only for a subset of .NET Framework that was usable in SQLCLR, and still it was never ending stream of issues.

> Even with this effort, we often got a support escalation (from paid support) where people hit problems with Thread.Abort in production. Some of these issues require a very ugly hacks to workaround because of there was just no right fix to them. We had to resort to hacks like decoding assembly instructions and suppress or adjust thread abort behavior for particular instruction pattern that was known to hit the problem.

> I am not even talking about larger .NET ecosystem - if you take a random NuGet package from nuget.org, it is almost guaranteed that it has reliability bugs in the presence of thread abort.

Да и Джон Скит, Эрик Липперт и Джо Даффи, задолго до dot.net core тоже [не рекомендовали использовать](https://stackoverflow.com/a/1856312/5402731) `Thread.Abort()`
