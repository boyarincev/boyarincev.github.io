---	
title: Производительность ToList() vs ToArray()
tags: perfomance
published: false	
---	

Два самых популярных варианта материализовать linq-запрос (в этой статье под linq-запросом я подразумеваю только [LINQ to Objects](https://docs.microsoft.com/ru-ru/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects)) - это вызвать `ToList()` или `ToArray()` на нём. Давайте разберёмся есть ли какие-то отличия в их производительности.

## Какая концептуальная разница что использовать

Cначала нужно остановится на том, есть ли какие-то другие причины, кроме производительности, выбрать тот или другой метод. При использовании `ToList()` результатом операции будет [List](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.generic.list-1?view=netcore-3.1), а при использовании `ToArray()` - [Array](https://docs.microsoft.com/ru-ru/dotnet/api/system.array?view=netcore-3.1#methods). Соответственно выбор метода сводится к вопросу, что для вас предпочтительнее на выходе.

Лично для себя я не вижу каких-то преимуществ массива перед списком, а вот минусы у него есть.

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

Приведя массив к интерфейсу [IList](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.ilist?view=netcore-3.1), вы получите runtime-ошибку при [вызове Add на нём](https://docs.microsoft.com/ru-ru/dotnet/api/system.array?view=netcore-3.1#explicit-interface-implementations). Да, IList имеет свойство [IsFixedSize](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.ilist.isfixedsize?view=netcore-3.1#System_Collections_IList_IsFixedSize) для того, чтобы узнать изменяемая ли коллекция находится под капотом, но по мне так это просто ещё один пример [протекающей абстракции](https://en.wikipedia.org/wiki/Leaky_abstraction) - думаю, что свойство было добавлено только потому что массив реализует интерфейс IList.

Есть ещё вещи, которые нельзя отнести к минусам, скорее просто к особенностям.

### Массивы разрешают изменения в массиве при foreach, а List - нет

[Enumerator используемый List](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.enumerator?view=netcore-3.1) не разрешает изменения коллекции во время итерации, а [Enumerator используемый Array](https://github.com/dotnet/runtime/blob/v5.0.0-preview.4.20251.6/src/libraries/System.Private.CoreLib/src/System/Array.Enumerators.cs) разрешает. И такая разница в поведении вообще довольна опасна, так как вы можете использовать массив, всё под тем же многострадальным интерфейсом IList и при этом изменение массива при итерации у вас будет работать, а потом кто-то изменит нижележащий тип с массива на лист и вы получите runtime-ошибку.

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

## Какая теоретическая разница в производительности материализации списка и массива из linq-запроса

Список реализован внутри с использованием массива. Изначально это массив небольшого размера (на данный момент [4 элемента](https://github.com/dotnet/corefx/blob/v3.1.0/src/Common/src/CoreLib/System/Collections/Generic/List.cs#L25), если мы явно не задаём Capacity при создании List), по мере добавления элементов в список, когда место в текущем массиве подходит к концу, происходит создание нового массива в два раза больше предыдущего и копирование элементов из предыдущего. Если мы при создании списка используем [конструктор принимающий Capacity](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.generic.list-1.-ctor?view=netcore-3.1#System_Collections_Generic_List_1__ctor_System_Int32_) - то экземпляр списка сразу создаётся с массивом указанного размера.

Не трудно догадаться, что множественное пересоздание массива и копирование в него данных из старого - операция более затратная, чем сразу создать массив нужного размера и заполнить его единожды, но количество элементов в материализуемом linq-запросе нам неизвестно, поэтому, вероятно, при материализации создание пойдёт именно путём пересоздания нижележащего массива и копированием данных из старого.

Массивы, в свою очередь, всегда создаются фиксированного размера и в случае, когда размер итогового массива неизвестен (опять же - это наш случай), нужно придумать механизм динамического создания массива - итоговая производительность материализации массива будет зависеть от способа реализации этого механизма, если он будет похож на тот что использует список - с постепенным увеличением размера массива и копированием данных из старого, то разницы в производительности вообще может не оказаться.

## Как реализован ToList

[Реализация](https://referencesource.microsoft.com/#System.Core/System/Linq/Enumerable.cs,947) `ToList()` - это просто вызов конструктора `List<T>`

```csharp
public static List<TSource> ToList<TSource>(this IEnumerable<TSource> source) {
    if (source == null) throw Error.ArgumentNull("source");
    return new List<TSource>(source);
}
```

[Конструктор](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs,74) `List<T>` имеет "хак" на случай, если коллекция на которой он вызывается по-настоящему реализует интерсейс ICollection, для того, чтобы сразу создать массив нужного размера, иначе будет в цикле добавлять элементы по одному.

```csharp
public List(IEnumerable<T> collection) {
    if (collection==null)
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.collection);
    Contract.EndContractBlock();

    ICollection<T> c = collection as ICollection<T>;
    if( c != null) {
        int count = c.Count;
        if (count == 0)
        {
            _items = _emptyArray;
        }
        else {
            _items = new T[count];
            c.CopyTo(_items, 0);
            _size = count;
        }
    }    
    else {                
        _size = 0;
        _items = _emptyArray;
        // This enumerable could be empty.  Let Add allocate a new array, if needed.
        // Note it will also go to _defaultCapacity first, not 1, then 2, etc.

        using(IEnumerator<T> en = collection.GetEnumerator()) {
            while(en.MoveNext()) {
                Add(en.Current);                                    
            }
        }
    }
}
```

## Как реализован ToArray

Реализация `ToArray()` - это просто вызов конструктора Buffer и последующий вызов `ToArray()` на созданном экземпляре Buffer.

```csharp
public static TSource[] ToArray<TSource>(this IEnumerable<TSource> source) {
    if (source == null) throw Error.ArgumentNull("source");
    return new Buffer<TSource>(source).ToArray();
}
```
