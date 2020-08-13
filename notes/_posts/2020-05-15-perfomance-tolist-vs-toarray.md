---	
title: Производительность ToList() vs ToArray()
tags: perfomance
published: false	
---	

Два самых популярных варианта материализовать linq-запрос (в этой статье под linq-запросом я подразумеваю только [LINQ to Objects](https://docs.microsoft.com/ru-ru/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects)) - это вызвать `ToList()` или `ToArray()` на нём. Давайте разберёмся есть ли какие-то отличия в их производительности.

## Какая концептуальная разница что использовать

Cначала нужно остановится на том, есть ли какие-то другие причины, кроме производительности, выбрать тот или иной метод. При использовании `ToList()` результатом операции будет [List](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.generic.list-1?view=netcore-3.1), а при использовании `ToArray()` - [Array](https://docs.microsoft.com/ru-ru/dotnet/api/system.array?view=netcore-3.1#methods). Соответственно выбор метода сводится к вопросу, что для вас предпочтительнее на выходе.

Я не вижу каких-то преимуществ массива перед списком, а вот минусы у него есть.

### Массивы не типобезопасны при ковариации

```csharp
object[] array = new String[10];  
// The following statement produces a run-time exception.  
// array[0] = 10;  
```

Массив элементов дочернего типа может быть приведён к массиву элементов родительского типа (это может произойти неявно, например, если метод в котором вы работаете с массивом возвращает массив элементов базового типа) и дальше вы при попытке вставить в такой массив элемент базового типа вы получите runtime-ошибку.

### Массивы изменяемая коллекция с неизменяемым размером

Это довольно редкий кейс, когда вам нужна коллекция, в которой вам нужно заменять элементы, но при этом не нужно добавлять или удалять что-то из неё. В общем случае, если вам нужна неизменяемая коллекция, то ни лист ни массив вам одинаково не подходят, а если вам нужна изменяемая коллекция, то вам не подходит массив.

### Массивы реализуют интерфейс IList, но не поддерживают операцию добавления элемента

Приведя массив к интерфейсу [IList](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.ilist?view=netcore-3.1), вы получите runtime-ошибку при [вызове Add на нём](https://docs.microsoft.com/ru-ru/dotnet/api/system.array?view=netcore-3.1#explicit-interface-implementations). Да, `IList` имеет свойство [IsFixedSize](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.ilist.isfixedsize?view=netcore-3.1#System_Collections_IList_IsFixedSize) для того, чтобы узнать изменяемая ли коллекция находится под капотом, но по мне так это просто ещё один пример [протекающей абстракции](https://en.wikipedia.org/wiki/Leaky_abstraction) - не удивлюсь, если свойство было добавлено в интерфейс только потому что массив реализует интерфейс `IList`.

Есть ещё вещи, которые нельзя отнести к минусам, скорее просто к особенностям.

### Массивы разрешают изменения в массиве при foreach, а List - нет

[Enumerator используемый List](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.enumerator?view=netcore-3.1) не разрешает изменения коллекции во время итерации, а [Enumerator используемый Array](https://github.com/dotnet/runtime/blob/v5.0.0-preview.4.20251.6/src/libraries/System.Private.CoreLib/src/System/Array.Enumerators.cs) разрешает. И такая разница в поведении вообще довольна опасна, так как вы можете использовать массив, всё под тем же многострадальным интерфейсом `IList` и при этом изменение массива при итерации у вас будет работать, а потом кто-то изменит нижележащий тип с массива на лист и вы получите runtime-ошибку.

```csharp
IList<int> source = Enumerable.Range(1, 10).ToArray();

foreach (var x in source)
{
  if (x == 5)
    source[8] *= 100;
  Console.WriteLine(x);
}

//Вывод:
//1
//2
//3
//4
//5
//6
//7
//8
//900
//10

IList<int> source = Enumerable.Range(1, 10).ToList();

foreach (var x in source)
{
  if (x == 5)
    source[8] *= 100;
  Console.WriteLine(x);
}

//1
//2
//3
//4
//5
//System.InvalidOperationException
```

Пример взят [отсюда](https://stackoverflow.com/a/41246500/5402731).

Ну и можете ещё прочитать статью Эрика Липперта [Arrays considered somewhat harmful](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/arrays-considered-somewhat-harmful), чтобы узнать его мнение по поводу использования массивов.

## Как реализован ToList()

[Реализация](https://github.com/dotnet/corefx/blob/v3.1.0/src/System.Linq/src/System/Linq/ToCollection.cs#L23) `ToList()` - это либо вызов `ToList()` на интерфейсе [IIListProvider](https://github.com/dotnet/corefx/blob/v3.1.0/src/System.Linq/src/System/Linq/IIListProvider.cs), либо просто вызов конструктора списка с передачей в него последовательности.

```csharp
return source is IIListProvider<TSource> listProvider ? listProvider.ToList() : new List<TSource>(source);
```

На `IIListProvider` остановимся позже, пока разберём конструктор списка принимающий последовательность - по этому пути выполнение пойдёт, если мы `ToList()` вызовем на любой обычной коллекции, массиве или при использовании итератора (потому что они не реализуют интерфейс `IIListProvider`).

[Конструктор](https://github.com/dotnet/corefx/blob/v3.1.0/src/Common/src/CoreLib/System/Collections/Generic/List.cs#L61) имеет "хак" на случай, если последовательность на которой он вызывается по-настоящему реализует интерсейс `ICollection`, для того, чтобы сразу создать массив нужного размера, иначе будет в цикле добавлять элементы по одному.

```csharp
if (collection is ICollection<T> c)
{
    int count = c.Count;
    if (count == 0)
    {
        _items = s_emptyArray;
    }
    else
    {
        _items = new T[count];
        c.CopyTo(_items, 0);
        _size = count;
    }
}
else
{
    _size = 0;
    _items = s_emptyArray;
    using (IEnumerator<T> en = collection!.GetEnumerator())
    {
        while (en.MoveNext())
        {
            Add(en.Current);
        }
    }
}
```

Вообще список внутри реализован с использованием массива. Изначально это пустой массив (если мы явно не задаём Capacity при создании `List` - а это другой конструктор `List`), после добавления в список первого элемента создаётся новый массив небольшого размера (на данный момент [4 элемента](https://github.com/dotnet/corefx/blob/v3.1.0/src/Common/src/CoreLib/System/Collections/Generic/List.cs#L25)), по мере добавления элементов в список, когда место в текущем массиве подходит к концу, происходит создание нового массива в два раза больше предыдущего и копирование элементов из предыдущего. Если бы мы при создании списка использовали [конструктор принимающий Capacity](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.generic.list-1.-ctor?view=netcore-3.1#System_Collections_Generic_List_1__ctor_System_Int32_) - то экземпляр списка сразу бы создался с массивом нужного размера.

Не трудно догадаться, что множественное пересоздание массива и копирование в него данных из старого - операция более затратная, чем сразу создать массив нужного размера и заполнить его единожды, поэтому в случае вызова `ToList()` на коллекции реализующей `ICollection` материализация будет проходить гораздо быстрее и поэтому тестировать производительность `ToList()` просто создав список и приведя его к интерфейсу `IEnumerable` - смысле немного.

## Как реализован ToArray()

[Реализация](https://github.com/dotnet/corefx/blob/v3.1.0/src/System.Linq/src/System/Linq/ToCollection.cs#L11) `ToArray()` - это либо вызов `ToArray()` на `IIListProvider`, либо вызов `EnumerableHelpers.ToArray(source)`.

```csharp
return source is IIListProvider<TSource> arrayProvider
    ? arrayProvider.ToArray()
    : EnumerableHelpers.ToArray(source);
```

Выполнение пойдёт по пути вызова `EnumerableHelpers.ToArray(source)` при вызове `ToArray()` на любой обычной коллекции, массиве или последовательности сгенерированной с помощью итератора (потому что они не реализуют интерфейс `IIListProvider`).

[Реализация](https://github.com/dotnet/corefx/blob/v3.1.0/src/Common/src/System/Collections/Generic/EnumerableHelpers.Linq.cs#L93) `ToArray()` в `EnumerableHelper`.

```csharp
if (source is ICollection<T> collection)
{
    int count = collection.Count;
    if (count == 0)
    {
        return Array.Empty<T>();
    }

    var result = new T[count];
    collection.CopyTo(result, arrayIndex: 0);
    return result;
}

var builder = new LargeArrayBuilder<T>(initialize: true);
builder.AddRange(source);
return builder.ToArray();
```

Опять же проверка на `ICollection` и сразу создание массива нужного размера или вызов конструктора `LargeArrayBuilder<T>`.

Так как массивы всегда создаются фиксированного размера, то в случае, когда размер итогового массива неизвестен, нужно придумать механизм динамического создания массива, таким механизмом и является `LargeArrayBuilder<T>`, а итоговая производительность материализации массива будет зависеть от способа реализации этого механизма, если он будет похож на тот что использует список - с постепенным увеличением размера массива и копированием данных из старого, то производительности материализации массива будет сравнима с материализацией списка.

## Вывод 1 про разницу производительности

Если метод `ToList()` или `ToArray()` вызывается на коллекции реализующей интерфейс `ICollection` (массив тоже её реализует), то в обоих случаях код сведётся к:

```charp
_items = new T[count];
c.CopyTo(_items, 0);
```
И поэтому разница в производительности будет минимальной.

## Что за интерфейс IIListProvider



## Ссылки

- [Arrays considered somewhat harmful by Erick Lippert](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/arrays-considered-somewhat-harmful)
- [Is it better to call ToList() or ToArray() in LINQ queries? -Stack Overflow](https://stackoverflow.com/q/1105990/5402731)
