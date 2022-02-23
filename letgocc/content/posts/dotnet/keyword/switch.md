---
title: "C# 语句关键字（一）：switch"
date: 2021-03-28T16:13:47+08:00
draft: false
toc: true
images:
tags: 
  - dotnet
  - keyword
categories: 
- dotnet
---


## 介绍
语句就是我们所说的程序指令。一般语句都是按照顺序执行的，除非遇到下表中介绍的关键字。
## 语句类别
|类别|C#关键字|
|:---|:---|
|选择语句	|if、else、**switch、case**|
|迭代语句	|do、for、foreach、in、while|
|跳转语句	|break、continue、default、goto、return、yield|
|异常处理语句	|throw、try-catch、try-finally、try-catch-finally|
|Checked 和 unchecked	|checked、unchecked|
|fixed 语句	|fixed|
|lock 语句	|lock|


## switch
>switch 是一个选择语句，它根据与匹配表达式匹配的模式，从候选列表中选择单个开关部分进行执行。
### Case 标签
`switch` 语句包含至少一个开关，每个开关包含至少一个 `case 标签`。

每个 `case 标签`指定一个模式与匹配表达式进行比较。 如果它们匹配，则执行**首次匹配** `case 标签`的开关部分

因为 C# 6 仅支持常量模式且禁止重复常量值，所以 case 标签定义了互斥值，而且只能有一个模式与匹配表达式匹配。 因此，case 语句显示的顺序并不重要。 

然而，在 C# 7.0 中，因为支持其他模式，所以 case 标签不需要定义互斥值，并且多个模式可以与匹配表达式相匹配。 因为仅执行包含匹配模式的首次开关部分中的语句，所以 case 语句显示的顺序很重要。

以下示例说明了使用各种非互斥模式的 switch 语句。 如果你移动 case 0: switch 部分，使之不再是 switch 语句中的第一部分，C# 会生成编译器错误，因为值为零的整数是所有整数的子集（由 case int val 语句定义的模式）。

``` C#
// Calculate the sum of n dice rolls.
public static int DiceSum(IEnumerable<object> values)
{
	ar sum = 0;
	foreach (var item in values)
	{
		  switch (item)
      {
        // A single zero value.
        case 0:
          break;
        // A single value.
         case int val:
            sum += val;
				    break;
        // A non-empty collection.
        case IEnumerable<object> subList when subList.Any():
            sum += DiceSum(subList);
            break;
        // An empty collection.
        case IEnumerable<object> subList:
            break;
        //  A null reference.
        case null:
            break;
        // A value that is neither an integer nor a collection.
        default:
             throw new InvalidOperationException("unknown item type");
		}
	}
	return sum;
}
```
你可以通过以下两种方法之一更正此问题并消除编译器警告：
 - 更改开关部分的顺序。
 - 在 case 标签中使用 `when 子句`。
 
 ### 模式匹配：类型模式
 ``` C#
 case type varname
 ```
其中 type 是 expr 结果要转换到的类型的名称，varname 是 expr 结果要转换到的对象（如果匹配成功）。

如果以下任一条件成立，则 case 表达式为 true：
- expr 是与 type 具有相同类型的一个实例。
- expr 是派生自 type 的类型的一个实例。 换言之，expr 结果可以向上转换为 type 的一个实例。
- expr 具有属于 type 的一个基类的编译时类型，expr 还具有属于 type 或派生自 type 的运行时类型。 变量的编译时类型是其类型声明中定义的变量类型。 变量的运行时类型是分配给该变量的实例类型。
- expr 是实现 type 接口的类型的一个实例。

**请注意，null 与任何类型都不匹配**。 若要匹配 null，请使用以下 case 标签：
 ``` C#
 case null:
 ```
 
示例：
```C#
switch (coll)
{
  case Array arr:
      Console.WriteLine($"An array with {arr.Length} elements.");
      break;
  case IEnumerable<int> ieInt:
      Console.WriteLine($"Average: {ieInt.Average(s => s)}");
      break;
  case IList list:
      Console.WriteLine($"{list.Count} items");
      break;
  case IEnumerable ie:
      string result = "";
      foreach (var e in ie)
          result += $"{e} ";
      Console.WriteLine(result);
      break;
  case null:
      // Do nothing for a null.
      break;
  default:
      Console.WriteLine($"A instance of type {coll.GetType().Name}");
      break;
}
```
**如果 coll 是泛型，将不能使用 case null 。因为编译器只能将 T 转换为 object 。**
 
 ### when 子句
 从 C# 7.0 开始，因为 case 语句不需要互相排斥，因此可以添加 when 子句来指定必须满足的附加条件使 case 语句计算为 true。 when 子句可以是返回布尔值的任何表达式。
 ```C#
 private static void ShowShapeInfo(Shape sh)
{
  switch (sh)
  {
	// Note that this code never evaluates to true.
	case Shape shape when shape == null:
        Console.WriteLine($"An uninitialized shape (shape == null)");
        break;
	case null:
        Console.WriteLine($"An uninitialized shape");
        break;
	case Shape shape when sh.Area == 0:
        Console.WriteLine($"The shape: {sh.GetType().Name} with no dimensions");
        break;
	case Square sq when sh.Area > 0:
        Console.WriteLine("Information about square:");
        Console.WriteLine($"   Length of a side: {sq.Side}");
        Console.WriteLine($"   Area: {sq.Area}");
        break;
	case Rectangle r when r.Length == r.Width && r.Area > 0:
        Console.WriteLine("Information about square rectangle:");
        Console.WriteLine($"   Length of a side: {r.Length}");
        Console.WriteLine($"   Area: {r.Area}");
        break;
	case Rectangle r when sh.Area > 0:
        Console.WriteLine("Information about rectangle:");
        Console.WriteLine($"   Dimensions: {r.Length} x {r.Width}");
        Console.WriteLine($"   Area: {r.Area}");
        break;
	case Shape shape when sh != null:
        Console.WriteLine($"A {sh.GetType().Name} shape");
        break;
	default:
        Console.WriteLine($"The {nameof(sh)} variable does not represent a Shape.");
        break;
  }
}
   ```
   请注意，不会执行尝试测试 `Shape` 对象是否为 `null` 的示例中的 `when` 子句。 测试是否为 `null` 的正确类型模式是 `case null:`。
   
## switch 表达式 (C# 8.0 )
    
**常量模式**
   
```C#
var direction = Directions.Right;
Console.WriteLine($"Map view direction is {direction}");

var orientation = direction switch
{
  Directions.Up    => Orientation.North,
  Directions.Right => Orientation.East,
  Directions.Down  => Orientation.South,
  Directions.Left  => Orientation.West,
};
 Console.WriteLine($"Cardinal orientation is {orientation}");
```
    
**类型模式**
   
```C#
public static T ExhaustiveExample<T>(IEnumerable<T> sequence) =>
sequence switch
{
  System.Array { Length : 0}       => default(T),
  System.Array { Length : 1} array => (T)array.GetValue(0),
  System.Array { Length : 2} array => (T)array.GetValue(1),
  System.Array array               => (T)array.GetValue(2),
  IEnumerable<T> list
      when !list.Any()             => default(T),
  IEnumerable<T> list
      when list.Count() < 3        => list.Last(),
  IList<T> list                    => list[2],
  null                             => throw new ArgumentNullException(nameof(sequence)),
  _                                => sequence.Skip(2).First(),
};
```
`_ `模式与未匹配的所有输入相匹配。 它必须在 `null` 检查之后执行，否则将与 `null` 输入匹配。

