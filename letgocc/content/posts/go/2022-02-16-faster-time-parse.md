---
title: "Faster Time Parse"
date: 2022-02-18T10:32:43+08:00
author:
description:
draft: false
type: posts
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocPosition: inner
tocLevels: ["h2", "h3", "h4"]
tags:
- golang
series: 
-
categories:
- golang
image:
---
**本文翻译自原文：** [Faster time parsing](https://syslog.ravelin.com/faster-time-parsing-4afd48abade3 "Faster time parsing")  
**作者：** Phil Pearl

在[Ravelin](https://www.ravelin.com/careers "Ravelin")，我们有大量的数据，有大量的时间戳。大多数时间戳在 BigQuery 中被存储为字符串，而我们的大多数 Go 结构用 Go time.Time 类型表示时间。

我很遗憾地说出上述这些事实。我们确实有大量的数据。而且我们确实有很多时间戳。一段时间以来，我一直在围着一个结论打转，随着时间的推移，我确信我将会朝着这个结论落下。

**朋友不会让朋友在数据库中用字符串表示时间。**

无论如何，决定已经做出，我们被它们所困。但被决定所困并不意味着我们被所有不幸的后果所困。我们可以把事情做到最好。对我来说，现在把事情做到最好不可避免地包括找到一种比使用[time.Parse](https://pkg.go.dev/time#Parse "time.Parse")更快的方法来解析 RFC3339 时间戳。

事实证明这很容易。`time.Parse`有两个参数：一个是描述要解析的数据的格式，另一个是需要解析的数据字符串。格式参数并不只是选择适合该格式的专用解析程序。格式参数描述了应该如何解析数据。`time.Parse`不仅要解析时间，而且要解析、理解和实现对如何解析时间的描述。如果我们写一个专门的解析例程，只解析 RFC3339，应该比这更快。

但在我们开始之前，让我们写一个快速的基准测试来看看`time.Parse`有多快。

```
func BenchmarkParseRFC3339(b *testing.B) {
	now := time.Now().UTC().Format(time.RFC3339Nano)
	for i := 0; i < b.N; i++ {
		if _, err := time.Parse(time.RFC3339, now); err != nil {
			b.Fatal(err)
		}
	}
}
```

以下是结果

```
name             time/op
ParseRFC3339-16  150ns ± 1%
```

现在我们可以编写我们专用的 RFC3339 解析函数了。这很无聊。它不漂亮。但是（据我所知！）它有效。

(它确实很长，也不漂亮，所以与其把它放在这篇文章里，让你们都滑过去，不如在这里提供一个链接，让你们看到应用了下面讨论的所有优化的[最终版本](https://github.com/philpearl/avro/blob/master/time/parse.go "parse.go")。如果你想象一下一个很长的函数，其中有很多对`strconv.Atoi`的调用，你就会明白的)

如果我们调整我们的基准以使用我们新的解析函数，我们会得到以下结果。

```
name             old time/op  new time/op  delta
ParseRFC3339-16   150ns ± 1%    45ns ± 4%  -70.15%  (p=0.000 n=7+8)
```

它确实比`time.Time`快了不少。很好。我们完成了。

**当然，我们还没有完成。**

如果我们得到一个 CPU 配置文件，我们就会发现很多时间是在调用`strconv.Atoi`时消耗的。

```
> go test -run ^$ -bench BenchmarkParseRFC3339 -cpuprofile cpu.prof
> go tool pprof cpu.prof
Type: cpu
Time: Oct 1, 2021 at 7:19pm (BST)
Duration: 1.22s, Total samples = 960ms (78.50%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 950ms, 98.96% of 960ms total
Showing top 10 nodes out of 24
      flat  flat%   sum%        cum   cum%
     380ms 39.58% 39.58%      380ms 39.58%  strconv.Atoi
     370ms 38.54% 78.12%      920ms 95.83%  github.com/philpearl/blog/content/post.parseTime
      60ms  6.25% 84.38%      170ms 17.71%  time.Date
```

`strconv.Atoi`将 ASCII 字符串中的数字转换为整数。这是 Go 标准库的一个基本部分，所以它的编码肯定非常好，而且已经被优化了。当然，我们不能在此基础上进行改进？
好吧，我们的大多数数字都正好是 2 字节长或 4 字节长。我们可以利用这些事实来编写数字解析函数，而不需要那些讨厌的慢速 for 循环。

```
func atoi2(in string) (int, error) {
	a, b := int(in[0]-'0'), int(in[1]-'0')
	if a < 0 || a > 9 || b < 0 || b > 9 {
		return 0, fmt.Errorf("can't parse number %q", in)
	}
	return a*10 + b, nil
}
func atoi4(in string) (int, error) {
	a, b, c, d := int(in[0]-'0'), int(in[1]-'0'), int(in[2]-'0'), int(in[3]-'0')
	if a < 0 || a > 9 || b < 0 || b > 9 || c < 0 || c > 9 || d < 0 || d > 9 {
		return 0, fmt.Errorf("can't parse number %q", in)
	}
	return a*1000 + b*100 + c*10 + d, nil
}
```

如果我们再次运行我们的基准测试，我们可以看到我们已经取得了很好的进一步改进。

```
name             old time/op  new time/op  delta
ParseRFC3339-16  44.9ns ± 4%  39.7ns ± 3%  -11.51%  (p=0.000 n=8+8)
```

好了，我们现在不仅写了一个自定义的时间分析器，而且还写了自定义的数字分析器。这肯定够了。现在肯定是完成了。

**当然，我们还没有完成。**

啊，但是让我们再看一下 CPU 配置文件。再让我们看看反汇编。在`atoi2`中有两个切片长度检查（它们是对 panicIndex 的调用，在下面的绿色反汇编中看到）。这不是有一个技巧吗？

![20220216-timeparse](https://img.letgo.cc/blog/202202181114572.png)

下面是用这个技巧更新的代码。\_ = in[1]在函数的开头给了编译器足够的提示，它不会在我们以后每次引用它时都检查字符串是否足够长。

```
func atoi2(in string) (int, error) {
	_ = in[1] // This helps the compiler reduce the number of times it checks `in` is long enough
	a, b := int(in[0]-'0'), int(in[1]-'0')
	if a < 0 || a > 9 || b < 0 || b > 9 {
		return 0, fmt.Errorf("can't parse number %q", in)
	}
	return a*10 + b, nil
}
```

一个小小的改变，但足以让我们有一个明确的改善

```
name             old time/op  new time/op  delta
ParseRFC3339-16  39.7ns ± 3%  38.4ns ± 2%  -3.26%  (p=0.001 n=8+7)
```

`atoi2`非常短。为什么它不被内联？也许如果我们简化错误，它就会被简化？如果我们去掉对`fmt.Errorf`的调用，用一个简单的错误来代替它，这就降低了我们 atoi 函数的复杂性。这可能足以让 Go 编译器决定不把这些函数作为单独的代码块来实现，而是直接在调用函数中实现。

```
var errNotNumber = errors.New("not a valid number")
func atoi2(in string) (int, error) {
	_ = in[1]
	a, b := int(in[0]-'0'), int(in[1]-'0')
	if a < 0 || a > 9 || b < 0 || b > 9 {
		return 0, errNotNumber
	}
	return a*10 + b, nil
}
```

事实的确如此，并产生了明显的改进。

```
name             old time/op  new time/op  delta
ParseRFC3339-16  38.4ns ± 2%  32.9ns ± 5%  -14.39%  (p=0.000 n=7+8)
```

我们的故事就到此为止了。在大约 120 纳秒的时间里做了很多工作。但是，纳秒加起来，这些改进使 Ravelin 的一些机器学习特征提取管道的运行时间减少了一个小时或更多。正如我所说，我们确实有*大量*的数据和*大量*的时间戳!

