---
title: "部署Blog教程"
date: 2023-05-25T12:04:04+09:00
draft: false
categories: ["tech"]
tags: ["netlify","deploy","free","hugo"]
summary: 如何利用netlify部署你的blog
author: "MGMCN"
showToc: false
TocOpen: false
hidemeta: false
comments: false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
---
> 本文将介绍如何使用netlify部署你的blog。
## 注册netlify账号
首先你需要注册一个netlify账号，注册地址为[netlify-signup](https://app.netlify.com/signup)。
## 部署流程🔥
### step 1 首先在你的仓库根目录下添加netlify.toml文件，内容如下：
```toml
[build]
  command = "hugo"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.111.0"
```
我这里使用的是hugo，你也可以使用其他的静态网站生成器。博客的目录结构请参考右上角Wiki。
### step 2 点击Import from Git
![image](/img/1import-from-git.png)
### step 3 这里我选择的是Github，你也可以选择其他的平台。
![image](/img/2choose-github.png)
### step 4 选择你的Blog对应的Github仓库，点击install。
![image](/img/3install.png)
### step 5 选择你刚刚添加的Github仓库。
![image](/img/4import-rep.png)
### step 6 Netlify会自动读取netlify.toml的配置，此处直接点击deploy即可。
![image](/img/5config.png)
### step 7 你可以在Netlify的dashboard查看你的blog部署状态。
![image](/img/6check-deploy.png)
### step 8 你可以通过Permalink获取你的Blog地址。
![image](/img/7get-link.png)
### step 9 你可以定制你的site name。
![image](/img/8customize-link.png)
## 结语
Netlify还有很多功能强大的插件等待你自己去探索。

