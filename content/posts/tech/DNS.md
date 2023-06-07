---
title: "域名解析"
date: 2023-06-06T13:53:38+09:00
draft: false
categories: [tech]
tags: ["vercel","deploy","domains","aliyun"]
summary: 如何利用aliyun快速配置你的域名解析
author: "MGMCN"
showToc: false
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
> 因为你的域名并没有在中国备案，所以在中国是无法直接访问到你在Netlify和Vercel上部署的博客等服务的。

Notice : 你部署的服务应当符合中国社会主义核心价值观。

## 配置你的云解析DNS
此处默认你已经在阿里云购买了你自己的域名。(其他云服务提供商也可以)你需要配置以下信息。
```
主机记录 blog.mgmcn.net
A记录 76.223.126.88 # 中转，主要方便大陆地区的访问，你也可以配置CNAME
```
需要注意的是A记录必须根据你部署服务的提供商提供的ip修改。例如我这里使用的是Vercel部署我的blog。如果你也是部署在Vercel上的，那么你可以直接copy我的A记录。
## 在你的Vercel APP Dashboard添加Domains
![image](/img/vercel-domains.jpg)  

这样你就不需要备案也能让你国内小伙伴访问到你在Netlify或者Vercel上部署的服务啦！🤪

