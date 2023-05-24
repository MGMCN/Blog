---
title: "tc指令学习记录"
date: 2023-05-24T21:32:33+09:00
draft: false
categories: ["tech"]
tags: ["network", "tc", "linux"]
summary: 使用tc命令模拟网络延迟
author: "Me"
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
> 当我们在做一些与网络相关实验的时候，可能会遇到需要模拟真实网络延迟的场景。这里记录下如何使用tc这个linux自带的高级流控指令模拟网络延迟。
## 实验环境
```
node1 Ubuntu 20.04.5 LTS 163.221.29.105
node2 Ubuntu 20.04.5 LTS 163.221.29.106
node3 Ubuntu 20.04.5 LTS 163.221.29.208
```
## 实验目的
我们希望在node1和node2之间制造50ms延迟，在node2和node3之间制造100ms延迟，在node1和node3之间制造150ms延迟。
## 实验手段
利用tc指令建立流队列，接管各个节点网卡的流，利用filter规则过滤流，最终制造延迟。  
在node1上执行以下指令(注意:请替换ens160为你虚拟机的网卡)
```bash
$ sudo tc qdisc add dev ens160 root handle 1: prio
$ sudo tc filter add dev ens160 parent 1: protocol ip prio 1 u32 match ip dst 163.221.29.106 flowid 1:1
$ sudo tc qdisc add dev ens160 parent 1:1 handle 10: netem delay 50ms
```
在node2上执行以下指令
```bash
$ sudo tc qdisc add dev ens160 root handle 1: prio
$ sudo tc filter add dev ens160 parent 1: protocol ip prio 1 u32 match ip dst 163.221.29.208 flowid 1:1
$ sudo tc qdisc add dev ens160 parent 1:1 handle 10: netem delay 100ms
```
在node3上执行以下指令
```bash
$ sudo tc qdisc add dev ens160 root handle 1: prio
$ sudo tc filter add dev ens160 parent 1: protocol ip prio 1 u32 match ip dst 163.221.29.105 flowid 1:1
$ sudo tc qdisc add dev ens160 parent 1:1 handle 10: netem delay 150ms
```
现在你就全都配置好啦
## 实验结果
在node1上ping node2和node3
```bash
root@node1:~# ping 163.221.29.106 -c 4
PING 163.221.29.106 (163.221.29.106) 56(84) bytes of data.
64 bytes from 163.221.29.106: icmp_seq=1 ttl=64 time=50.3 ms
64 bytes from 163.221.29.106: icmp_seq=2 ttl=64 time=50.4 ms
64 bytes from 163.221.29.106: icmp_seq=3 ttl=64 time=50.3 ms
64 bytes from 163.221.29.106: icmp_seq=4 ttl=64 time=50.5 ms

--- 163.221.29.106 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 50.302/50.357/50.456/0.060 ms
root@node1:~# ping 163.221.29.208 -c 4
PING 163.221.29.208 (163.221.29.208) 56(84) bytes of data.
64 bytes from 163.221.29.208: icmp_seq=1 ttl=64 time=151 ms
64 bytes from 163.221.29.208: icmp_seq=2 ttl=64 time=151 ms
64 bytes from 163.221.29.208: icmp_seq=3 ttl=64 time=151 ms
64 bytes from 163.221.29.208: icmp_seq=4 ttl=64 time=151 ms

--- 163.221.29.208 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 150.548/150.604/150.681/0.053 ms
```
在node2上ping node1和node3
```bash
root@node2:~# ping 163.221.29.105 -c 4
PING 163.221.29.105 (163.221.29.105) 56(84) bytes of data.
64 bytes from 163.221.29.105: icmp_seq=1 ttl=64 time=50.5 ms
64 bytes from 163.221.29.105: icmp_seq=2 ttl=64 time=50.3 ms
64 bytes from 163.221.29.105: icmp_seq=3 ttl=64 time=50.5 ms
64 bytes from 163.221.29.105: icmp_seq=4 ttl=64 time=50.4 ms

--- 163.221.29.105 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 50.348/50.436/50.508/0.057 ms
root@node2:~# ping 163.221.29.208 -c 4
PING 163.221.29.208 (163.221.29.208) 56(84) bytes of data.
64 bytes from 163.221.29.208: icmp_seq=1 ttl=64 time=101 ms
64 bytes from 163.221.29.208: icmp_seq=2 ttl=64 time=101 ms
64 bytes from 163.221.29.208: icmp_seq=3 ttl=64 time=101 ms
64 bytes from 163.221.29.208: icmp_seq=4 ttl=64 time=100 ms

--- 163.221.29.208 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 100.394/100.506/100.573/0.068 ms
```
在node3上ping node1和node2
```bash
root@node3:~# ping 163.221.29.105 -c 4
PING 163.221.29.105 (163.221.29.105) 56(84) bytes of data.
64 bytes from 163.221.29.105: icmp_seq=1 ttl=64 time=150 ms
64 bytes from 163.221.29.105: icmp_seq=2 ttl=64 time=150 ms
64 bytes from 163.221.29.105: icmp_seq=3 ttl=64 time=150 ms
64 bytes from 163.221.29.105: icmp_seq=4 ttl=64 time=150 ms

--- 163.221.29.105 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 150.371/150.427/150.499/0.046 ms
root@node3:~# ping 163.221.29.106 -c 4
PING 163.221.29.106 (163.221.29.106) 56(84) bytes of data.
64 bytes from 163.221.29.106: icmp_seq=1 ttl=64 time=100 ms
64 bytes from 163.221.29.106: icmp_seq=2 ttl=64 time=101 ms
64 bytes from 163.221.29.106: icmp_seq=3 ttl=64 time=100 ms
64 bytes from 163.221.29.106: icmp_seq=4 ttl=64 time=101 ms

--- 163.221.29.106 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 100.443/100.563/100.661/0.097 ms
```
## 讨论
tc是一个神奇的linux指令，还有很多高级功能有待挖掘，对于正在研究SDN或者涉及网络通信的应用之类课题的人帮助极大。关于tc指令的更多细节请参考[link](https://man7.org/linux/man-pages/man8/tc.8.html)。(ps:其实这也是我为了测试blog post水的first post😂)
