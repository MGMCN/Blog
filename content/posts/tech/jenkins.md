---
title: "部署Jenkins定时任务"
date: 2023-06-14T10:25:42+09:00
draft: false
categories: [tech]
tags: ["jenkins","ci/cd","deploy","auto pull request"]
summary: 如何部署你自己的CI/CD服务并按时自动执行爬虫代码。
author: "MGMCN"
showToc: True
TocOpen: false
hidemeta: false
comments: true
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
---

> 有很多方法可以帮助你部署一个按时执行的爬虫代码，这里给出了如何使用Jenkins部署，Jenkins会在我们规定的时间拉取我们存放在Github仓库的代码并编译运行，然后将爬取的结果以pull request的方式更新到我们的Github仓库去。

## 安装Jenkins
请参照[官方教程](https://www.jenkins.io/doc/book/installing/)安装Jenkins。
## 安装插件
因为我的爬虫代码是Go语言编写的，所以我需要安装Go的相关[插件](https://plugins.jenkins.io/golang/)。你应当根据你自己的需求安装对应的插件。这里我默认Jenkins已经安装了Git相关插件。
## Jenkins配置
以下主要给出一些较重要的配置。你可以根据你自己的需求修改你自己的配置。
### 全局工具配置
下载你的工程依赖版本的Go语言。
![image](/img/jenkins-0.png)
点击保存。
### 系统配置
在系统配置里面添加系统管理员邮箱地址。
![image](/img/jenkins-1.jpg)
配置你的SMTP服务器地址和你的邮箱地址。邮箱地址应当与系统管理员邮箱地址相同。
![image](/img/jenkins-2.jpg)
配置好后你可以测试下邮件发送功能，然后点击保存。
### 工程配置
配置你的Github仓库地址，我这里使用的是access_token这种访问方式，你也可以使用其他Github提供的访问方式。
![image](/img/jenkins-3.jpeg)
在构建触发器这里我们选择定时构建。这样我们的爬虫代码会定时在每个月的1号执行。
![image](/img/jenkins-4.png)
勾选下图中的配置，你也可以按照自己需求勾选。
![image](/img/jenkins-5.png)
这里我们选择通过执行shell脚本来自动化我们的爬虫代码构建和运行，同时添加构建失败后通知的邮件地址。
![image](/img/jenkins-6.png)
点击保存。
## 结语
现在你就全都配置好啦！只要你的Jenkins服务不宕掉，你的代码会在每个月1号执行，并且自动将爬取的结果PR到你的仓库去。请参照[hugoThemesRanking](https://github.com/MGMCN/hugoThemesRanking)。

