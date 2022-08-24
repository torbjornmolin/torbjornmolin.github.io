+++
title = "Diving into .NET source"
description = "A quick look at some .NET source code"
date = 2022-08-23T08:13:50Z
author = "Torbjörn Molin"
+++

## Once upon a time...

Microsoft was, as many of us remember, not always a proponent of open source and free software. In the 1990s and 2000s the company did what it could to stop open source from gaining momentum and internal memos talked about embrace-extend-extinguish as a strategy that could be adopted.

The Microsoft of today is very different and some development tools that used to be closed source and very expensive are now free in one or two senses of the word. One product that is free in both senses of the word is .NET (core and 5 and 6, not framework) through the .NET Foundation.

A recurring theme for me as a developer is that I enjoy those moments when I get see how things that initially seem like magic actually work. One such thing is standard libraries, I imagine them to be written by ancient wizards who probably don't even have to type to get code to appear in their editor. Thanks to the .NET foundation we can have a look at the code written by these mysterious people.

## Looking at the source

Microsoft has [a page](https://dotnet.microsoft.com/en-us/platform/open-source) introducing the .NET open source project, with a link to [GitHub](https://github.com/dotnet?WT.mc_id=dotnet-35129-website) where it is hosted. Looking in GitHub we see that the project has 218 repositories, but if we're interested in the standard library, the runtime project is where we need to look.

![dotnet GitGub](/dotnetplatformgithub.jpg)

Let's have a look at something that we hopefully will understand, LINQs Where-method. If someone asked me to write it, I would probably come up with something like this:

```csharp {linenos=inline}
public static IEnumerable<T> NaiveWhere<T>(
    this IEnumerable<T> source,
    Func<T, bool> predicate
)
{
    if (source is null)
        throw new ArgumentException(nameof(source));
    if (predicate is null)
        throw new ArgumentException(nameof(predicate));

    foreach (var o in source)
    {
        if (predicate(o))
        {
            yield return o;
        }
    }
}
```

This works and returns the expected result for a simple case using a couple of small tests:

```csharp {linenos=inline}

var colors = new List<string>() { "red", "blue", "green" };
var red = colors.NaiveWhere(c => c == "red");

System.Console.WriteLine($"Number of elements is 1: {red.Count() == 1}");
System.Console.WriteLine($"Color is red: {red.FirstOrDefault() == "red"}");
```

Outputs:

```
Number of elements is 1: True
Color is red: True
```

How does this compare to what is in the actual .NET runtime codebase, then? Well, to begin with, there is `Where.SpeedOpt.cs` and `Where.cs`, but let's look at [the latter](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Linq/src/System/Linq/Where.cs).

This file is about 400 lines long and has two public methods:

```csharp
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
```

and

```csharp
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, int, bool> predicate)
```

The second one is a version of the first one that allows us to use the index of the item in the predicate function. We can abuse it a bit and do:

```csharp
new[] { "red", "blue", "green" }
.Where((data, index) =>
{
    System.Console.WriteLine($"Color {index}: {data}");
    return true;
}).ToList();
```

which will print

```
Color 0: red
Color 1: blue
Color 2: green
```

But lets focus on the first method, the one with the predicate that only takes the object as input.

```csharp {linenos=inline}
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
    if (source == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.source);
    }

    if (predicate == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.predicate);
    }

    if (source is Iterator<TSource> iterator)
    {
        return iterator.Where(predicate);
    }

    if (source is TSource[] array)
    {
        return array.Length == 0 ?
            Empty<TSource>() :
            new WhereArrayIterator<TSource>(array, predicate);
    }

    if (source is List<TSource> list)
    {
        return new WhereListIterator<TSource>(list, predicate);
    }

    return new WhereEnumerableIterator<TSource>(source, predicate);
}
```

This method has four cases, the one that would be taken for our test case with a `List<string>` is the third one that returns a `WhereListIterator` that is defined in the same file.

```csharp
/// <summary>
/// An iterator that filters each item of a <see cref="List{TSource}"/>.
/// </summary>
/// <typeparam name="TSource">The type of the source list.</typeparam>
private sealed partial class WhereListIterator<TSource> : Iterator<TSource>
{
    private readonly List<TSource> _source;
    private readonly Func<TSource, bool> _predicate;
    private List<TSource>.Enumerator _enumerator;

    public WhereListIterator(List<TSource> source, Func<TSource, bool> predicate)
    {
        Debug.Assert(source != null);
        Debug.Assert(predicate != null);
        _source = source;
        _predicate = predicate;
    }

    public override Iterator<TSource> Clone() =>
        new WhereListIterator<TSource>(_source, _predicate);

    public override bool MoveNext()
    {
        switch (_state)
        {
            case 1:
                _enumerator = _source.GetEnumerator();
                _state = 2;
                goto case 2;
            case 2:
                while (_enumerator.MoveNext())
                {
                    TSource item = _enumerator.Current;
                    if (_predicate(item))
                    {
                        _current = item;
                        return true;
                    }
                }

                Dispose();
                break;
        }

        return false;
    }

    public override IEnumerable<TResult> Select<TResult>(Func<TSource, TResult> selector) =>
        new WhereSelectListIterator<TSource, TResult>(_source, _predicate, selector);

    public override IEnumerable<TSource> Where(Func<TSource, bool> predicate) =>
        new WhereListIterator<TSource>(_source, CombinePredicates(_predicate, predicate));
}
```

This class is an explicit implementation of an enumerator. It's not using the syntax sugar of `yield return` and `yield break` that our naive implementation did (read more [here](https://devblogs.microsoft.com/oldnewthing/20080812-00/?p=21273)). Otherwise it is mostly the same. Every time `MoveNext` is called it advances until it finds an element that matches the predicate and sets `_current` to the found item.
When we reach the end of the enumerator we `Disose()`, `break` and `return false`.

One interesting feature is that the iterator returned overrides the Where-method so that it returns a new WhereListIterator with a new predicate that is obtained using `CombinePredicates`.
This is a simple little helper defined in [Utilities.cs](https://github.com/dotnet/runtime/blob/57bfe474518ab5b7cfe6bf7424a79ce3af9d6657/src/libraries/System.Linq/src/System/Linq/Utilities.cs) that does exactly what it says on the box:

```csharp
/// A new predicate that will evaluate to <c>true</c> only if both the first and
/// second predicates return true. If the first predicate returns <c>false</c>,
/// the second predicate will not be run.
/// </returns>
public static Func<TSource, bool> CombinePredicates<TSource>(Func<TSource, bool> predicate1, Func<TSource, bool> predicate2) =>
    x => predicate1(x) && predicate2(x);
```

## Summing things up

Even if this was perhaps a trivial example, it's nice to be able to dive into some of the code that you'd use every day as a .NET developer and understand how it's implemented. Even if we didn't go through all of the code paths or try to deep dive into why the various cases are needed, we got to see a bit of how the standard library code is structured.
