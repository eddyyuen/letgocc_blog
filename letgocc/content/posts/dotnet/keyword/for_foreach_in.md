---
title: "C# 语句关键字（二）：for, foreach in"
date: 2021-03-28T16:21:19+08:00
draft: false
toc: true
images:
tags: 
  - dotnet
  - keyword
categories: 
- dotnet
---

## 语句类别

本期介绍 **for、foreach、in** 关键字

|类别|C#关键字|
|:---|:---|
|选择语句	|if、else、switch、case|
|迭代语句	|do、**for、foreach、in**、while|
|跳转语句	|break、continue、default、goto、return、yield|
|异常处理语句	|throw、try-catch、try-finally、try-catch-finally|
|Checked 和 unchecked	|checked、unchecked|
|fixed 语句	|fixed|
|lock 语句	|lock|



## for
一般我们使用 `for(int i=0;i<100;i++)` 这样的语句。但是实际上。在**初始化、迭代器**两个部分都可以调用其他方法。这是因为，在**初始化、迭代器**可以包括由逗号分割的零个或多个语句
```c#
int i;
int j = 10;
for (i = 0, Console.WriteLine($"Start: i={i}, j={j}"); i < j; 
i++, j--, Console.WriteLine($"Step: i={i}, j={j}"))
{
    // Body of the loop.
}
```

## foreach , in
`foreach` 语句为类型实例中实现 [System.Collections.IEnumerable](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.ienumerable "System.Collections.IEnumerable") 或 [System.Collections.Generic.IEnumerable< T>](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.generic.ienumerable-1 "System.Collections.Generic.IEnumerable<T>") 接口的每个元素执行语句或语句块
```C#
var fibNumbers = new List<int> { 0, 1, 1, 2, 3, 5, 8, 13 };
int count = 0;
foreach (int element in fibNumbers)
{
    Console.WriteLine($"Element #{count}: {element}");
    count++;
}
Console.WriteLine($"Number of elements: {count}");
```

 `foreach` 语句还可以与满足以下条件的任何类型的实例一起使用：
- 类型具有公共无参数 GetEnumerator 方法，其返回类型为类、结构或接口类型。 从 C# 9.0 开始，GetEnumerator 方法可以是类型的扩展方法。  
- GetEnumerator 方法的返回类型具有公共 Current 属性和公共无参数 MoveNext 方法（其返回类型为 Boolean）。

**示例**
```C#
class Program
{
    static void Main(string[] args)
    {
        foreach (int number in 5)
        {
            Console.Write($"{number} ");
        }
        Console.ReadLine();
    }
}
```
输出结果` 4 3 2 1 0`。  

之所以能这样做是因为我们基于 C# 9.0 实现了 `int` 的扩展方法 `GetEnumerator`:
```C#
public static Enumerator GetEnumerator(this int val)
{
    var enumerator = new Enumerator();
    enumerator.Current = val;
    return enumerator;
}
public struct Enumerator
{
    public int Current { get; set; }
    public bool MoveNext()
    {
        if (Current <= 0) return false;
        Current -= 1;
        return true;
    }
}
```
### ref readonly
从 C# 7.3 开始，如果枚举器的 Current 属性返回引用返回值（ref T，其中 T 为集合元素类型），就可以使用 ref 或 ref readonly 修饰符来声明迭代变量
```C#
public class ForeachRefExample
{
    public static void Main()
    {
        Span<int> storage = stackalloc int[10];
        int num = 0;
        foreach (ref int item in storage)
        {
            item = num++;
        }

        foreach (ref readonly var item in storage)
        {
            Console.Write($"{item} ");
        }
        // Output:
        // 0 1 2 3 4 5 6 7 8 9
    }
}
```
### await foreach
从 C# 8.0 开始，可以使用 `await foreach` 语句来使用异步数据流，即实现 IAsyncEnumerable< T > 接口的集合类型。 异步检索下一个元素时，可能会挂起循环的每次迭代。
C# 8.0 的  await foreach 后续有需要的话单独再介绍。
```C#
await foreach (var item in GenerateSequenceAsync())
{
    Console.WriteLine(item);
}
```
---
