---
title: "C# 语句关键字（三）：goto, yield"
date: 2021-03-28T16:22:51+08:00
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
本期介绍 **goto, yield** 关键字

|类别|C#关键字|
|:---|:---|
|选择语句	|if、else、switch、case|
|迭代语句	|do、for、foreach、in、while|
|跳转语句	|break、continue、default、**goto**、return、**yield**|
|异常处理语句	|throw、try-catch、try-finally、try-catch-finally|
|Checked 和 unchecked	|checked、unchecked|
|fixed 语句	|fixed|
|lock 语句	|lock|



## goto
### 直接跳转到标记语句
```C#
int val = 10;
if(val is int) 
    goto FoundInt;
else
    goto Finish;
FoundInt:
    Console.WriteLine("Found Integer");
Finish:
    Console.WriteLine("Finish");

```
### 跳转到特定的 case 标签或 default 标签。
```c#
switch (n)
{
    case 1:
        break;
    case 2:
        goto case 1;
    case 3:
        goto default;
    default:
        break;
}
```
### 跳出多层嵌套循环。
```c#
for (int i = 0; i < x; i++)
{
    for (int j = 0; j < y; j++)
    {
        if (array[i, j].Equals(myNumber))
        {
            goto FoundInt;
        }
    }
}
goto Finish;
FoundInt:
    Console.WriteLine("Found Integer");
Finish:
    Console.WriteLine("Finish");
```


## yield
`yield` 同时也属于上下文关键字。  
`yield` 出现在迭代器内部。

### yield return
 `yield return` 表示下一次迭代要返回的数据
```C#
foreach (int i in test())
{
    Console.Write("{0} ", i);
}
public static IEnumerable<int> test()
{
    int result = 0;
    for (int i = 0; i < 10; i++)
    {
        result = result + i;
        yield return result;
    }
}
```
另外一种应用,可轻松实现可枚举的对象 Cities,Users
```C#
public static IEnumerable<string> Cities()
{   
    yield return "GUANGDONG";
    yield return "BEIJING";
    yield return "SHANGHAI";
    yield return "SHENZHEN";
}
public static IEnumerable<User> Users()
{
    yield return new User { UserId = "1", UserName = "GUANGDONG" };
    yield return new User { UserId = "2", UserName = "BEIJING" };
    yield return new User { UserId = "3", UserName = "SHANGHAI" }; 
    yield return new User { UserId = "4", UserName = "SHENZHEN" }; 
}
public class User
{
    public string UserName {get;set;}
    public string UserId { get; set; }
}
```

### yield break
在遇到 `yield break;` 便会终止迭代。
```C#
public static IEnumerable<string> Cities()
{   
    yield return "GUANGDONG";
    yield return "BEIJING";
    yield return "SHANGHAI";
    yield break;
    yield return "SHENZHEN";
}
```