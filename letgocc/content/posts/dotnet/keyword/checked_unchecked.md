---
title: "C# 语句关键字（四）：checked, unchecked"
date: 2021-03-28T16:23:43+08:00
draft: false
images:
tags: 
  - dotnet
  - keyword
categories: 
- dotnet
series:
- CSharp 关键字
enableToc: true
enableTocContent: false
tocFolding: false
tocPosition: inner
--- 

## 语句类别

本期介绍 **checked, unchecked, fixed, lock** 关键字

|类别|C#关键字|
|:---|:---|
|选择语句	|if、else、switch、case|
|迭代语句	|do、for、foreach、in、while|
|跳转语句	|break、continue、default、goto、return、yield|
|异常处理语句	|throw、try-catch、try-finally、try-catch-finally|
|Checked 和 unchecked	|**checked、unchecked**|
|fixed 语句	|**fixed**|
|lock 语句	|**lock**|



## checked
>checked 关键字用于对整型类型算术运算和转换显式启用溢出检查。

一般我们在运行时，进行整型的运算时，是不会进行溢出检查的。例如
```C#
int a = 27;
//int b = 2147483648; 超出目标范围外，编译异常
int b = int.MaxValue; //2147483647
int c = a + b;
Console.Write($"{c} ");
//output -2147483622
```
相加的数值因为超出 `2147483647`，以负数 `-2147483622` 展示。很明显这不是我们想要的结果。那么我们加上`checked`关键字进行溢出检查
```c#
checked{
    int c = a + b;
    Console.Write($"{c}");
}
或者
int c = checked(a + b);
```
运行时会抛出异常 `System.OverflowException:“Arithmetic operation resulted in an overflow.”`

### Visual Studio 启用溢出检查
1. 打开项目`属性`页
1. 打开`生成`属性页
1. 点击 `高级` 按钮
1. 勾选`检查算术溢出`属性



## unchecked
跟`checked`关键字相反， `unchecked`关键字取消运行时的溢出检查。原因在于检查溢出需要时间，在没有溢出风险时取消检查可以提高程序的性能。

## fixed
在了解fixed之前要先了解 CLR。

CLR管理着一块连续的地址空间。当没有足够的地址空间来分配新建的对象时，就会触发GC。GC 时，会删除垃圾对象，并把非垃圾对象移动到大的连续的内存块里面以压缩这个地址空间。这时，指向这些对象的指针就会失效。GC会自行修改对象移动后的指针地址。

但是！但是！这个是在托管环境下的代码，如果是 `unsafe` 代码下的指针，GC 是不会理会的！这样GC后就会造成 `unsafe` 代码失效。

所以，我们需要用到 `fixed` 关键字，**不允许**垃圾回收器移动我们的对象。


```C#
class Point
{
    public int x;
    public int y;
}

unsafe private static void ModifyFixedStorage()
{
    // Variable pt is a managed variable, subject to garbage collection.
    Point pt = new Point();
    // Using fixed allows the address of pt members to be taken,
    // and "pins" pt so that it is not relocated.
    fixed (int* p = &pt.x)
    {
        *p = 1;
    }
}
```

`System.Span<T>` 和 `System.ReadOnlySpan<T>` 也可以被固定。
  
```C#
Span<int> RowFive = new Span<int>(PascalsTriangle, 10, 5);
fixed (int* ptrToRow = RowFive)
{
    // Sum the numbers 1,4,6,4,1
    var sum = 0;
    for (int i = 0;i < RowFive.Length;i++)
    {
        sum += *(ptrToRow + i);
    }
    Console.WriteLine(sum);
    }
```
更多使用方法请查看[fixed 关键字参考](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/fixed-statement "fixed 关键字")。
                                          
## lock
lock 获取指定对象的互斥锁，执行语句块，然后释放锁。
### 锁定对象
避免使用以下对象
- lock(this)  
`this` 可能是两个不同的实例
- lock(MyType)  
可以用 `lock(typeof(MyType))`。但是也不建议使用，因为`typeof(MyType)`范围有可能很大
- lock(string)  
`string`实例有可能只是暂时存在的

建议使用非`public`的专用于锁定的对象实例。
```C#
private readonly object myLock = new object();
```


