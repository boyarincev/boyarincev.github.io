---	
title: Производительность ToList() vs ToArray()
tags: perfomance
published: false	
---	

Два самых популярных варианта материализовать linq-запрос - это вызвать `ToList()` или `ToArray()` на нём. Давайте посмотрим есть ли какие-то отличия в их производительности.

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

Думаю, все знают, что [List](https://docs.microsoft.com/ru-ru/dotnet/api/system.collections.generic.list-1?view=netcore-3.1) реализован внутри с помощью массива, но не многие
