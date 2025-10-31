---
title: "如何活用V2ray抵御合法的中间人攻击🤪"
date: 2025-10-31T10:25:48+09:00
draft: false
categories: []
tags: []
summary: 本文教你怎么轻松绕过公司网络安全策略中的中间人攻击
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

# 背景
公司为了防止机密泄漏，一般会使用证书替换也就是我们常说的中间人攻击对我们向外发出的请求进行审查。公司会预先在我们的电脑上安装企业根证书（公钥部分），我们发出的向外的请求被公司劫持后公司会给我们的请求下发一个签名证书，我们使用本地受信任的公司预装的根CA公钥去验证，发现是合法的且是被信任的CA签发的，同时证书并没有过期，域名也匹配。然后我们和公司的服务器之间就能通过后续约定的临时密钥进行加密解密通信。公司能够拿到我们对外请求的所有明文信息，然后针对里面的诸如域名等等信息作限制，然后选择性的放行请求，实现细粒度的控制。常见的这类中间人攻击工具有Netskope
# 方法原理
首先我们必须得有一个被公司信任的人存在，例如cloudflare这类CDN公司，我们部署V2ray Server在我们自己服务器上，使用cloudflare tunnel技术定向到与这个Server通信。（针对没有公共ip却有自己domain也有自己家用服务器-树莓派等的情况）然后我们在公司电脑上安装V2ray Client，定义好我们想要绕过公司中间人攻击安全策略的routing规则。然后在浏览器或者系统网络配置里配置好相应的V2ray Client Proxy，配置好后，所有的请求将通过这个Client发送。所以现在的请求就变成了，用户请求 -> V2ray Client Proxy -> VLESS 包装后请求 -> 公司拦截解密 -> body内容无法解析已经被加密 -> 放行 -> cloudflare tunnel -> home proxy 解密 -> 代替我们请求然后返回给用户 
# Server配置
生成UUID
```bash
$ sudo apt install -y uuid-runtime
$ UUID=$(uuidgen)
$ echo $UUID
# 记录这个 UUID，后面 server/client 都要用同一个
```
在树莓派上安装v2ray
```bash
$ bash <(curl -sL https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
$ sudo systemctl enable v2ray
$ sudo systemctl start v2ray
$ sudo systemctl status v2ray
$ vim /usr/local/etc/v2ray/config.json # 内容见下，注意替换为你自己的配置
$ sudo systemctl restart v2ray
```
如何在cloudflared上建立隧道请参照[官方文档](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/get-started/)。
你需要配置abc.your_domain.xyz -> http://127.0.0.1:10086 然后在你的server(树莓派)上运行cloudflare来维持你的tunnel运行。
```json
{
  "log": {
    "loglevel": "info"
  },
  "inbounds": [
    {
      "port": 10086,
      "listen": "127.0.0.1",
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "<UUID>",
            "level": 0
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/v2ray"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
你也可以选择使用vmess，但注意server和client协议要匹配。这里使用vless是因为比较快。
# Client配置
```bash
# 以下指令在Mac上验证过，你需要根据你的操作系统调整
$ brew install v2ray
$ vim ~/abc/xyz/config.json # 内容见下，注意替换为你自己的配置
$ v2ray run -c  ~/abc/xyz/config.json
```
```json
{
  "log": {
    "loglevel": "info"
  },
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": "127.0.0.1"
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],
  "outbounds": [
    {
      "tag": "home-proxy",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "abc.your_domain.xyz",
            "port": 443,
            "users": [
              {
                "id": "<UUID>",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/v2ray"
        },
        "tlsSettings": {
          "serverName": "abc.your_domain.xyz",
          "allowInsecure": false
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "domain": [
          "geosite:openai",
          "domain:openai.com",
          "domain:chatgpt.com",
          "domain:oaistatic.com",
          "domain:oaiusercontent.com",
          "domain:auth0.openai.com",
          "domain:challenges.cloudflare.com",
	      "domain:mail.google.com",
	      "domain:linkedin.com"
        ],
        "outboundTag": "home-proxy"
      },
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "port": "0-65535",
        "outboundTag": "direct"
      }
    ]
  },
  "dns": {
    "servers": [
      "https+local://1.1.1.1/dns-query",
      "8.8.8.8"
    ]
  }
}

```
然后你需要在你的浏览器(例如FoxyProxy之类的插件)或者网络配置里添加这个client的proxy配置，转发请求到127.0.0.1:1080，Type选SOCKS5就行了，然后你再打开浏览器访问被公司block的网页，你应该能成功访问了！

# Warning
请不要参照本文档做任何违法违规的事，如因搭建V2ray导致任何违法违规影响，一切后果自负！


