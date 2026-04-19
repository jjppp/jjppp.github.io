---
title: Network02 Application
date: 2022-09-12
permalink: /posts/2022/09/Network02-Application/
tags:
  - Network
---

# Intro

应用一般跑在端系统上，此时网络对应用而言是透明的，所有的网络 API 起到进程间通信的作用。

# Application architecture

## C/S

服务器长时间开机提供服务，客户端与服务器建立连接来交换数据。服务器通常有固定 IP。当然服务器可以有很多（集群）

## P2P

用户之间直接通信（当然也可能要预先借助服务器建立连接）。比如大学生都爱玩的 Bittorrent。

# Process Communication

网络本质上提供的是进程间通信的服务，只不过这里的进程可以跑在不同的主机上。基于字符串的网络通信可以屏蔽很多底层细节。

在一对通信中，通常把提出建立连接的一方称为客户端，另一端称为服务端。

## socket

socket 是操作系统提供的通信 API，一个 socket 指示了一对连接的一端。unix socket 可以看成特殊的文件，向其中写入或读取就能完成消息的发送和接收。

当然，连接是建立在进程-进程之间的，因此 socket 在建立连接时需要获得

1. 主机 IP
2. 端口号

可以认为端口号上守候的进程即为目标通信进程。

## Transport layer services

**传输层**提供的服务有几个可以度量的方面

* Reliable data transfer，即是否向应用层提供可靠的传输服务
* Throughput，某些传输层协议可以提供带宽保障
* Timing，某些协议提供延时保障
* Security

### TCP

* 面向连接，在应用层穿消息之前，传输层会首先进行若干次通信以建立连接、设定状态（称为握手）。在结束通信前需要解除连接。
* 可靠数据传输，保证顺序、正确、完整。
* 拥塞控制，降低流量强度
* **没有延时和带宽的保证**

### UDP

* 无连接，直接发
* 不可靠
* **没有延时和带宽的保证**

TCP UDP 都是不保证安全性的，新的 TLS(Transport Layer Security) 解决了这个问题。TLS 实质上是在应用层实现的，即可以看成是 TCP 的应用层代理，所有原本途径 TCP 的数据现在绕道经过 TLS 加密。

而延时和带宽则是无法加抽象层解决的问题，本质是因为下层提供的服务太弱了（或没有利用好下层的强服务）。

至于为什么只做增量式的修修补补，我觉得这就是维护屎山艺术了。

## Application layer protocol

本质上就是 API，和 PA 里面写过的 `frame_buffer` 作为显存的格式规定是一模一样的。协议可以是公开的（例如 HTTP）也可以是私有的（例如你自己写的即时聊天软件）

# Web

web 就是很经典的 C/S 架构，用户只会在需要的时候向服务器发出请求。

文件（包括页面）被称为对象，页面中可以出现对不同对象的引用（通过 URL）。URL 则由 主机名 + 路径 构成，指向了一个对象。

## HTTP/1.0

在建立 TCP 连接后，客户端将发起请求（提供 URL 和若干指令），等待服务器返回请求的对象。

* 无状态，即通信双方无需维护历史信息（client-server 都是纯函数）
* HTTP/1.1 额外支持了 持久化/非持久化 连接，即可以控制本次连接是否是一次性的（通信后即销毁）。持久化的通信可以减少 TCP 连接建立的次数。

### cookie

本质上是对 HTTP 协议无状态的一个补丁，其中

* cookie 随着 HTTP 消息的传输一起传输（实质上是 cookie 的标识符/编号）
* cookie 在客户端和服务器各存一份

书上的原话是 cookie creates a user session layer on top of HTTP，事实如此

## HTTP/2

TCP 的特性之一是保证所有连接共享带宽，这样的特性使得浏览器倾向于打开更多的 TCP 连接来抢占更多的带宽。而 HTTP/2 则是为了缓解这样的情况，尽可能重用 TCP 连接，降低服务器的负载。

另一个问题是关于等待的。仅凭 URL 很难得知对象的大小，而大对象的传输又会推迟小对象的传输，这就使得浏览器倾向于打开更多 TCP 连接来并行地传输对象。

### frames

简单的说就是在应用层做了传输层的活，请求和传输将会首先被切割成块，然后以块为单位做真正的请求和传输。这样就实现了带宽在文件之间的共享（也就是细化了并发的粒度）。没啥好说的。

### priority

请求块可以被赋予优先级。是不是很像实时操作系统的套路？

### push

服务端可以预先推送未经请求的对象给客户端，也就是协议级别的 prefetch

## Web cache/Proxy server

本质上是一台联网的计算机。用户的请求会被先给到 proxy server，然后由 proxy server 请求真正的页面、缓存、返回，很好理解。

之所以这么做是因为网络带宽由瓶颈链路决定，而代理服务器通常到客户端有较高的带宽，向外的请求则可以合并成一个降低流量强度。这么做要求请求的命中率要比较好。

当然 cache 就带来了缓存一致性的问题，解决的办法就是加一个 conditional get 请求，用来获得某个对象的修改时间。如果发现修改时间变动了说明缓存过期，那么重新 get 一次或者踢出缓存就好了。顺便一提，这也是社团仙贝 304 Not Modified 这个昵称的由来（

# Mail

主要是几个比较有历史厚重感的协议

## 发

发送端通过 SMTP 协议发给服务器，发送服务器通过 SMTP 协议发给接收服务器。

SMTP 协议要求报文是ASCII可显示的，这座屎山使得中文和附件这些东西得通过特殊的编码来发送。

注意SMTP的报文本身有一层格式要求（表明身份、发出指令等等），同时邮件的内容也有格式的要求（主题、内容等等）

## 收

接收端则主动向接收服务器查询收件箱（HTTP/POP3/IMAP）

这么做的一个好处是客户端可以不需要随时在线，同时一个服务器可以服务很多客户端。

POP3收取后会删除服务器上的数据。

实际上收发邮件就是在做文本传输的事情，因此用 HTTP 协议也是可以的。

# DNS

互联网上的主机需要区分彼此，因此都有一个（可以视为唯一的）标识符 IP 地址。但是为了方便称呼，还会有一个字符串组成的主机名。

类比文件系统，DNS 就是主机名解析。例如浏览器里输入的网址最后会被解析成 IP 地址（文件系统中打开的文件最后会被解析成inode）。DNS 是跑在 UDP 上的。

当然 DNS 有两层含义

* DNS 系统，是一个世界范围的分布式解析系统，提供解析服务
* DNS 协议，通过协议来进行报文传递

除了域名解析，DNS 还能做

* Host aliasing. 即可以给一个主机赋予不同的名字，其中一个叫 canonical name，其余的都是 alias。
* Load distribution. 可以把一个主机名解析到不同的 IP 上，实现负载均衡。

## DNS Server Hierarchy

* Root server，实际上有很多个
* TLD server，TLD 对应于域名中最后一段，例如 .cn .jp .fr .org。每个 TLD 都会有自己的 DNS 服务器。
* Authoritative  server。每个 hostname 都将最终被一台 DNS server 给解析，这样的 server 叫做这个 hostname 的 authoritative server。

本质上是把域名按照 "." 划分成若干段，每段只由一类服务器负责。

网上则还有两层，每次解析一般会按照顺序从上到下查询 DNS server，最终定位到一个 authoritative server，然后从中获得 IP 地址。

当然 ISP 还可以提供 local DNS server 这样的东西，本质上是一个 DNS proxy。

* recursive query，说的是客户端的解析请求由另一个主机代理完成查询，仅和代理主机通信。
* iterative query，说的是客户端的解析请求全部由自己完成，其与其它主机直接通信。

DNS server 内部还可以做 caching，没啥好说的。

## Resource Records

格式形如 (Name, Value, Type, TTL)，其中 (Name, Value) 是 Key-Value pair

* Type A: (hostname, IP)
* Type NS: (hostname, dns server hostname)，同时还会有 (dns server hostname, IP, A, TTL)
* Type CNAME: (hostname, canonical name)
* Type MX: (hostname, canonical name)

# P2P File Distribution

大学生都爱玩的 PT

## Tracker

指一类特殊的 host。所有 BT 客户端上线后都将向 Tracker 发出请求，随后 Tracker 将返回一份客户主机的名单，这份名单上的主机称为这个新上线客户端的邻居节点。

## Download

1. 客户端会向邻居发出请求，得到若干份邻居所持有资源的表
2. 找到最稀有的一份资源，发出下载请求

## Upload

1. 客户端会给所有邻居打分，上传时优先满足分数最高的邻居。分数与之前的上传情况相关
2. 隔一段时间会随机挑选邻居重新评估分数

有点 MFLQ 的味道在里面。

关于 P2P 有一个定量的分析，比较简单就不说了。

# Video Streaming

杀手级应用

## DASH (Dynamic Adapting Streaming over HTTP)

说的是在向服务器请求文件的时候，服务器会返回同一个视频在不同码率下的 URL，客户端就可以动态作出播放何种码率的视频的决策...

同时 HTTP GET 请求中可以附带块(chunks)座标，这样就可以按需加载以节省带宽。在我小学的时候老师很喜欢打开一个视频页面，等整个视频加载完再看，现在大概是做不到了。

## CDN (Content Distribution Network)

CDN 说的是把提供内容的服务器架设在各个地方，通过某种方式从中央服务器分发内容到各个 CDN 服务器上，用户对于中央服务器中内容的请求最终会被重定向到 CDN 服务器。

可以发现网络相关的技术都是这么玩的——怎么在客户端无法感知的前提下提供更优质的 underlying 服务

CDN 后请求的大概流程会变成这样：

1. 用户通过官方 URL 请求一个文件，这个 URL 中的域名部分会被 local DNS 服务器解析
2. local DNS 的 DNS request 经过层层查询来到了权威 DNS 服务器
3. 权威 DNS 服务器返回 CDN 权威服务器的域名
4. CDN 权威服务器**挑出**一个 CDN 服务器，返回这个 CDN 服务器的 IP 地址
5. 客户端拿到了 CDN 服务器的 IP 地址，就能请求到真正的视频文件了。

可以发现这个“重定向”的操作是建立在 DNS 之上的。即官方的服务器并没有直接提供数据，而是把请求重定向到了一个 CDN 的服务器。因此也可以说 CDN 本质上只是加了一层域名解析。

### Content Update Strategy

大概讲了两类

* Pull caching. 说的是用户最终的请求定向到了一个 CDN 服务器，但是这个 CDN 服务器上没有这个资源，那么 CDN 服务器就会向中央服务器请求这个资源、缓存、回传给用户
* Push. 说的是中央服务器会主动把内容分发到各个 CDN 服务器上
* 当然它们可以混合

### CDN Selection Strategy

这就五花八门了，但总归还是评分排序