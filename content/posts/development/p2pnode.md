---
title: "P2P-File-Sharing Tutorial"
date: 2023-06-29T10:14:09+09:00
draft: false
categories: [development]
tags: ["p2p","file sharing","golang","mdns"]
summary: A tutorial on how to use the P2P-File-Sharing application we developed
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
File sharing in P2P manner through LAN
## Usage
Build p2pnode from [source code](https://github.com/MGMCN/P2P-File-Sharing).
```bash
$ go get -t github.com/libp2p/go-libp2p@v0.28.0
$ go get -t github.com/chzyer/readline
$ go mod tidy
$ go build -o p2pnode ./main
```
Enter ```-help``` option to view usage.
```bash
$ ./p2pnode -help
Peer-to-peer file sharing over LAN.
  -help
    	Display Help
  -host string
    	The bootstrap node host listen address
    	 (default "0.0.0.0")
  -port int
    	node listen port
    	 (default 6666)
  -rendezvous string
    	Unique string to identify group of nodes. Share this with your friends to let them connect with you
    	 (default "default")
  -src string
    	Path to shared directory
    	 (default "./")
```
## Example
Create a directory for peer1 and store a test file. Please note that all files under this directory will be discovered by other nodes within the LAN.
```bash
$ ls
.
├── p2pnode
└── peer1.txt
$ cat peer1.txt
hello from peer1
```
Create a directory for peer2 and store a test file.
```bash
$ ls
.
├── p2pnode
└── peer2.txt
$ cat peer2.txt
hello from peer2
```
Execute the following command in each node's directory to start the two nodes respectively, it should be noted that the ports cannot conflict.
```bash
$ ./p2pnode -port 6667 # peer1:6667 peer2:6668
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
```
The ```peer echo``` command sends a hello message to all nodes within this LAN and also receives their replies.
```bash
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
peer echo
2023/06/25 11:15:23 Hello from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
```
The ```peer search``` command will search for resources of all online nodes in the current LAN.
```bash
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
peer echo
2023/06/25 11:15:23 Hello from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
peer search
2023/06/25 11:15:29 UpdateOthersSharedResources from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
```
The ```cache list``` command allows you to view the details of the shared resources in the current LAN.
```bash
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
peer echo
2023/06/25 11:15:23 Hello from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
peer search
2023/06/25 11:15:29 UpdateOthersSharedResources from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
cache list
2023/06/25 11:15:32 We share the following resources: | p2pnode ( 30002322 bytes ) | peer1.txt ( 17 bytes )
2023/06/25 11:15:32 The resources shared by other nodes are listed in the table below
2023/06/25 11:15:32 Resource             | Size           | Peers
2023/06/25 11:15:32 p2pnode              | 30002322 bytes | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
2023/06/25 11:15:32 peer2.txt            | 17 bytes       | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
```
The ```peer download resourceName``` command can download the shared resources within the current LAN, and if there are multiple nodes with the same resource, the resources will be downloaded from multiple nodes in parallel.
```bash
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
peer echo
2023/06/25 11:15:23 Hello from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
peer search
2023/06/25 11:15:29 UpdateOthersSharedResources from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
cache list
2023/06/25 11:15:32 We share the following resources: | p2pnode ( 30002322 bytes ) | peer1.txt ( 17 bytes )
2023/06/25 11:15:32 The resources shared by other nodes are listed in the table below
2023/06/25 11:15:32 Resource             | Size           | Peers
2023/06/25 11:15:32 p2pnode              | 30002322 bytes | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
2023/06/25 11:15:32 peer2.txt            | 17 bytes       | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
peer download peer2.txt
2023/06/25 11:15:45 Caculated fileChunkSize:17 bytes
2023/06/25 11:15:45 Received file chunk from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
2023/06/25 11:15:45 Merge chunk of peer2.txt successfully
2023/06/25 11:15:45 Total download time 0.01s
```
The ```peer leave``` command causes the current node to gracefully exit the LAN.
```bash
2023/06/25 11:15:12 Peer listening on: 0.0.0.0 with port: 6667 hostID: QmWjGSAJxhKzDqD9QTmHt9XBe5coDJdgZUWJv4ecDQPrRW
peer echo
2023/06/25 11:15:23 Hello from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
peer search
2023/06/25 11:15:29 UpdateOthersSharedResources from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
cache list
2023/06/25 11:15:32 We share the following resources: | p2pnode ( 30002322 bytes ) | peer1.txt ( 17 bytes )
2023/06/25 11:15:32 The resources shared by other nodes are listed in the table below
2023/06/25 11:15:32 Resource             | Size           | Peers
2023/06/25 11:15:32 p2pnode              | 30002322 bytes | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
2023/06/25 11:15:32 peer2.txt            | 17 bytes       | [QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH]
peer download peer2.txt
2023/06/25 11:15:45 Caculated fileChunkSize:17 bytes
2023/06/25 11:15:45 Received file chunk from QmdixMouioQwsBxWo1Bj3US3Q5YujB1KLsY2jSa1XCbptH
2023/06/25 11:15:45 Merge chunk of peer2.txt successfully
2023/06/25 11:15:45 Total download time 0.01s
peer leave
2023/06/25 11:15:53 Node leave gracefully
```
Now we go back to the peer1 directory. You can see that peer2.txt has been downloaded successfully.
```bash
$ ls
.
├── p2pnode
├── peer1.txt
└── peer2.txt
$ cat peer2.txt
hello from peer2
```
