---
title: "使用 Hugo 建立博客"
date: 2020-07-30T15:59:34+08:00
draft: false
toc: true
images:
tags: 
  - blog
  - hugo
---





想要备份一些技术文章和资料，很多博客都是php或要求数据库，找了一圈发现 Hugo 不错，单一文件，还是 golang 写的。

具体怎么使用搜索一下一大堆，这里写一下别的。

### 修改 Themes 后怎么编译

修改了别人的 scss 文件的话需要下载 extended 版本的hugo才能正常编译。只是修改HTML的话就不用。

### 显示行号、代码高亮

`lineos  ` 代表显示行号。

` hl_line` 代表代码里需要高亮的行，`[2,3,5-7]`表示第2、3、5至7 行都高亮。

`linenostart` 表示行号从199开始，这里的行号跟高亮的行无关。

``` bash
​``` csharp  {linenos=inline,hl_lines=[2,3,5-7],linenostart=199}
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
​```
```



### 支持高亮的类型

- **actionscript、actionscript3** actionscript, actionscript3, as, as3
- **Angular2**  ng2
- **ApacheConf** aconf, apache, apacheconf, conf, htaccess
- **Bash** bash, bash_*, bashrc, ebuild, eclass, exheres-0, exlib, ksh, sh, shell, zsh, zshrc
- **Batchfile** bat, batch, cmd, dosbatch, winbatch
- **C** c, h, idc
- **C#** c#, cs, csharp
- **C++**  C, CPP, H, c++, cc, cp, cpp, cxx, h++, hh, hpp, hxx
- **CSS** css
- **Docker**  docker, dockerfile
- **Go** go, golang
- **HTML** htm, html, xhtml, xslt
- **PHP** inc, php, php3, php4, php5, php[345]
- **PowerShell**  posh, powershell, ps1, psm1
- **Python** py, python, pyw, sage, sc, tac, py3, python3
- **其他** json, java, javascript, js, kotlin, lua, mysql, sql, objective-c, vue, vuewjs ,txt



### Github Webhook

想要偷懒，在GitHub Push/Comment 后自动生成 Blog，以下是Golang的代码

- **main.go**

  监听3000端口，提供一个接口 /githubhook 。在GitHub新建一个 Webhooks 指向此处即可。

``` go
package main

import (
	"bufio"
	"fmt"
	"gopkg.in/go-playground/webhooks.v5/github"
	"io"
	"io/ioutil"
	"net/http"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
)

const (
	path = "/githubhook"
	conf = "app.conf"
)

func main() {
	hook, _ := github.New(github.Options.Secret("eddyyuenblog"))

	http.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
		payload, err := hook.Parse(r, github.IssueCommentEvent, github.PushEvent)
		if err != nil {
			if err == github.ErrEventNotFound {
				// ok event wasn;t one of the ones asked to be parsed
				_, _ = w.Write([]byte("EventNotFound!"))

			} else {
				_, _ = w.Write([]byte("Error!"))
			}
			return
		}
		switch payload.(type) {
		case github.IssueCommentPayload:
			//IssueComment := payload.(github.IssueCommentPayload)
			// todo 发送回复的通知
			//fmt.Printf("%+v", IssueComment)
			fmt.Printf("github.IssueComment")
			_, _ = w.Write([]byte("ok!"))

		case github.PushPayload:
			//pushRequest := payload.(github.PushPayload)
			// todo pull github & run hugo.exe
			runCommand()
			fmt.Printf("github.PushPayload")
			_, _ = w.Write([]byte("ok!"))
		default:
			_, _ = w.Write([]byte("NOT HANDLER!"))
		}

	})
	_ = http.ListenAndServe(":3000", nil)
}

func runCommand() {
	dir, _ := os.Executable()
	exPath := filepath.Dir(dir)
	//读取文件的信息
	bytes, err := ioutil.ReadFile(exPath+"\\"+conf)
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	//按照换行符分割
	text := string(bytes)
	cmdarr := strings.Split(text, "\r\n")

	//是否新的开始
	isBegin := 1
	for _, val := range cmdarr {
		tmpval := strings.TrimSpace(val)

		//如果是新命令开始，那么是切换目录操作
		if tmpval != "" && isBegin == 1 {
			os.Chdir(tmpval)
		} else if tmpval != "" {
			//分割命令
			cmdarr := strings.Split(tmpval, " ")
			//命令名称
			command := cmdarr[0]
			//命令参数
			params := cmdarr[1:]
			//执行cmd命令
			execCommand(command, params)
		}
		//如果是空行，说明新的命令开始
		if tmpval == "" {
			isBegin = 1
			continue
		} else {
			isBegin = 0
		}
	}
}


//执行命令函数
//commandName 命名名称，如cat，ls，git等
//params 命令参数，如ls -l的-l，git log 的log等
//注：本函数转载#http://studygolang.com/articles/4004
func execCommand(commandName string, params []string) bool {
	cmd := exec.Command(commandName, params...)

	//显示运行的命令
	fmt.Println(cmd.Args)

	stdout, err := cmd.StdoutPipe()

	if err != nil {
		fmt.Println(err)
		return false
	}

	cmd.Start()

	reader := bufio.NewReader(stdout)

	//实时循环读取输出流中的一行内容
	for {
		line, err2 := reader.ReadString('\n')
		if err2 != nil || io.EOF == err2 {
			break
		}
		fmt.Println(line)
	}

	cmd.Wait()
	return true
}


```

- **app.conf**
	获取最新的代码，然后调用 Hugo 生成站点，最后复制到网站的目录。

  ``` bash {linenos=inline,hl_lines=[1,5,8],linenostart=2}
  D:\Webapplication\Hugo\sites\blog
  git fetch
  git merge origin
  
  D:\Webapplication\Hugo\sites\blog
  D:\Webapplication\Hugo\sites\hugo.exe
  
  D:\Webapplication\Hugo\sites\blog
  xcopy D:\Webapplication\Hugo\sites\blog\public D:\Webapplication\Hugo\sites\public /e /k /y
  ```

  

