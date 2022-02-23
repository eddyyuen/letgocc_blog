---
title: "用 ONLYOFFICE 在线查看Word Excel PPT"
date: 2020-08-11T11:56:38+08:00
draft: false
toc: true
images:
tags: 
  - onlyoffice
  - office
  - docker
---

项目需要在线浏览 Office 文件，以前用过 SharePoint ，现在有 Office Online Server 可以处理。但是OOS安装配置比较复杂且需要 Windows 的服务器，不合适。一番查找，最后发现 ONLYOFFICE 的功能就是我需要的，马上部署！

 为了方便，就采用Docker镜像部署的方式 

### 创建容器

首先我们必须安装有Docker（假设Docker宿主机IP：192.168.1.2），然后进入网页 [Docker安装帮助文档](https://helpcenter.onlyoffice.com/server/docker/document/docker-installation.aspx)，创建容器：

``` bash
sudo docker run -i -t -d -p 9080:80 --restart=always onlyoffice/documentserver
```

在生产环境，数据不要存储在容器内，就自行创建保存目录，并使用以下命令：

``` bash
sudo docker run -i -t -d -p 9080:80 --restart=always \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice  \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data  \
    -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
    -v /app/onlyoffice/DocumentServer/db:/var/lib/postgresql  onlyoffice/documentserver
```

- `/var/log/onlyoffice` for **Document Server** logs
- `/var/www/onlyoffice/Data` for certificates
- `/var/lib/onlyoffice` for file cache
- `/var/lib/postgresql` for database



### 查看是否成功

浏览器访问宿主IP+端口（http://192.168.1.2:9080），如果出现以下字样代表容器运行正常

![](http://img.bugs.red/blog/20200811150218.png?imageView2/1/w/400/h/100)

### 建立文档编辑HTML页面

创建一个html文件

- 嵌入编辑器 

  ```html
  <div id="placeholder"></div>
  ```

- 引用 js 

  ```html
  <script type="text/javascript" src="http://192.168.1.2:9080/web-apps/apps/api/documents/api.js"></script>
  ```

- 设置编辑器的参数，具体参数含义[查看这里](https://api.onlyoffice.com/editors/advanced)

``` html
config = {
       "document": {
          "fileType": "XLSX",
		   "info": {
            "folder": "Example Files",
            "owner": "John Smith",
            "sharingSettings": [
                {
                    "permissions": "Full Access",
                    "user": "John Smith"
                },
                {
                    "permissions": "Read Only",
                    "user": "Kate Cage"
                },
            ],
            "uploaded": "2010-07-07 3:46 PM"
        },
          "key": "12NaaaaAFE2", //每个文档唯一的KEY
          "title": "test6.docx",
          "url": "https://bugs.red/201901.XLSX", //文档的地址
		  "permissions": { //权限
            "comment": false,
            "download": false,
            "edit": true,
            "fillForms": false,
            "modifyContentControl": false,
            "modifyFilter": false,
            "print": false,
            "review": true
			},
      },
      "documentType": "spreadsheet", // WORD:text，PPT:presentation
      //"width": "1600px", 
     "height": "900px",
      "editorConfig": {
           // "actionLink": ACTION_DATA,
        "callbackUrl": "",
        "createUrl": "",
        "customization": {
            "autosave": false,
            "chat": false,
            "commentAuthorOnly": false,
            "comments": false,
            "compactHeader": false,
            "compactToolbar": false,
            "customer": {
                "address": "My City, 123a-45",
                "info": "Some additional information",
                "logo": "https://example.com/logo-big.png",
                "mail": "john@example.com",
                "name": "John Smith and Co.",
                "www": "example.com"
            },
            "feedback": {
                "url": "https://example.com",
                "visible": false
            },
            "forcesave": false,
            "goback": {
                "blank": false,
                "requestClose": true,
                "text": "Open file location",
                "url": "https://bugs.red/"
            },
            "help": false,
            "hideRightMenu": true,
            "logo": {
                "image": "https://bugs.red/favicon-32x32.png",
                "imageEmbedded": "https://bugs.red/favicon-32x32.png",
                "url": "https://bugs.red/"
            },
            "mentionShare": false,
            "reviewDisplay": "original",
            "showReviewChanges": false,
            "spellcheck": false,
            "toolbarNoTabs": true,
            "unit": "cm",
            "zoom": 100
        },
		"embedded": {
            "embedUrl": "https://example.com/embedded?doc=exampledocument1.docx",
            "fullscreenUrl": "https://example.com/embedded?doc=exampledocument1.docx#fullscreen",
            "saveUrl": "https://example.com/download?doc=exampledocument1.docx",
            "shareUrl": "https://example.com/view?doc=exampledocument1.docx",
            "toolbarDocked": "top"
        },
        "lang": "zh-cn",
        "mode": "edit",
        }	,
		"type": "desktop",
	 "width": "100%"

};
```

- 实例化编辑器

``` javascript
var docEditor = new DocsAPI.DocEditor("placeholder", config);
```

### 最终效果

![image-20200812173704709](http://img.bugs.red/blog/20200812173704.png)

### 相关链接

[ONLYOFFICE 社区版下载地址](https://www.onlyoffice.com/zh/download.aspx)

[ONLYOFFICE API 基本设置](https://api.onlyoffice.com/editors/basic)