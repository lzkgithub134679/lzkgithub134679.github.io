---
title: IP协议
date: 2020-01-14 09:02:40
tags:
- Show All
- 网络
header-img: "Demo.png"
---

### 问题

阿里云的对象存储(OSS)为什么Region（地域）不同，访问速度会受影响？

### TCP/IP协议体系结构

通常我们各个电脑或服务器之间传输数据是使用的TCP/IP协议。
应用层： 应用进程间通信和交互的规则, 如域名DNS系统协议、以及HTTP协议以及邮件协议SMTP等
传输层： 传输层主要使用两种传输协议：传输控制协议（TCP）与 用户数据报协议（UDP）, 其中TCP是面向连接的，可靠的数据传输服务，数据不会出现丢失的情况。UDP是面向无连接，但是数据传输可能会有丢包的情况出现。
网络层：
IP协议, ARP协议, 路由协议, 网络层的主要工作是定义网络地址、区分网段、子网内MAC寻址、对于不同子网的数据包进行路由
网络接口层:分为两层,分为数据链路层:MAC地址所在层，两个相邻的节点通过节点的MAC地址进行帧的传送。
物理层： 线路，光纤口等。
![img](http://192.168.12.24/Public/Uploads/2020-01-08/5e15734e0cd1f.png)

### IP协议概念

Internet Protocol简称IP，互联网协议，也可以交又网际协议，IP协议是网络层最重要的协议，也是TCP/IP体系中最重要的协议。.

### 网络层

网络层也可以叫IP层，提供互联网主机之间无连接的通信服务,就相当于人身体的心脏,主要功能是各器官之间的通讯。利用数据链路层提供的服务，向传输层提供主机间的分组传递服务。那就产生了三个问题：网络层编址，路由选择，和拥塞控制。IP层主要是解决网络层编址，路由选择，为了提升效率，将拥塞控制主要留给了高层解决。

具体就是几个问题:
1.IP编址和地址的分配
2.IP包的转发
3.路由表的产生和动态刷新
4.差错处理
5.IP组播

### IP编址和地址的分配

TCP/IP的设计者使用一种类似于物理网络的编址方案，给因特网上每个主机分配一个32比特的唯一地址，用于与该主机的所有通信。
早时期，IP编址方法将IP地址分为两个部分：前缀和后缀。前缀便是所属的物理网络，称为网络号，后缀用于区分物理网络内的主机，称为主机号。显然前缀包含的比特越长，后缀就越少，，这种网络数多，支持的主机数少。反之前缀短后缀就长，主机数多，仅能支持较少的网络。这种称为分类编址方案。
最初的分类编址方案包含5中形式的IP地址，A,B,C是三种主类别.
分类IP地址是自标识的，从地址本身就能确定前缀和后缀之间的边界，这使路由器更快的使用地址的网络号就行路由选择。
![img](http://192.168.12.24/Public/Uploads/2020-01-09/5e16c5420b9b6.png)

例：
某主机32比特的IP地址:
10000001 00000001 01000110 00001111
最高两位是10，属于B类
转成十进制可写成129.1.70.15 网络号为129.1 主机号70.15
![img](http://192.168.12.24/Public/Uploads/2020-01-09/5e16d9ff11ccd.png)

随着网络时代的快速发展，到20世纪80年代，分类IP网络地址不够用了，并且已分配的地址也没有得到充分利用。随后设计人员在分类编址的基础上，提出了子网编址、代理ARP、无编号的点对点链路。90年代，又创造了无分类编址方法，进一步提高32比特地址空间的利用率以及路由效率。

##### 子网编址：

子网编址即给每个地址编上一个序号。
现在一个 IP 地址的网络标识和主机标识已不再受限于该地址的类别，而是由一个叫做“子网掩码”的识别码通过子网网络地址细分出比 A 类、B 类、C 类更小粒度的网络。这种方式实际上就是将原来 A 类、B 类、C 类等分类中的主机地址部分用作子网地址，可以将原网络分为多个物理网络的一种机制。
子网掩码用二进制方式表示的话，也是一个 32 位的数字。它对应 IP 地址网络标识部分的位全部为 “1”，对应 IP 地址主机标识的部分则全部为 “0”。由此，一个 IP 地址可以不再受限于自己的类别，而是可以用这样的子网掩码自由地定位自己的网络标识长度。当然，子网掩码必须是 IP 地址的首位开始连续的 “1”。
IP的子网编址解决了IP编址的诸多问题例如：
(1).路由表急剧膨胀；
(2).地址管理的开销大；
(3).用网桥连接不方便网络的管理；
(4).地址数目过大造成浪费，而地址空间日渐紧缩；

##### 代理ARP

内网的机器想要知道外网的IP主机的MAC地址，但是我们知道MAC是工作在数据链路层，MAC是不可能跨路由的，所以，可以通过网关的MAC绑定源主机的IP来实现通信！这就是代理ARP。

##### 无编号的点对点链路

因为IP把点对点连接看成是一个网络，在使用IP编制方案时，就需要给这样的网络分配一个唯一的前缀。一般都使用C类地址。
为了避免给每条点对点连接都分配前缀，就发明了：匿名联网。这种技术通常用在通过租用数字线路连接一对路由器时，线路两端的路由器接口都不需要分配IP地址。

### IP包的转发

互联网通讯的双方，一般都是在不同的局域网内，要使IP从原始地到目的地，就得使用路由器。
路由器中有路由表，保存各个物理网络的路由信息，路由器通过路由表为经过的每个IP包选择一条路由，再进行逐跳转发，使IP包不断接近目的站。

### 路由表的产生和动态刷新

路由表存储有关怎么样到达目的网络的信息？
主机和路由器都有路由表。路由表不存储主机地址信息，这就可以减少路由表的大小，因为网络数据远小于主机数量，还有利于提高路由表查询效率以及降低路由表维护开销。

一张路由表的建立和刷新有两种不同的方式：静态路由和动态路由。
• 静态路由：由网络管理员设置并随时更新 网络管理员的工作负担重，容易出错，无法根据网络状态，进行调 整，适应性差； 简单、开销小，只适用于小型网络。
• 动态路由：路由器运行过程中根据网络情况动态地维护 减轻了网络管理员的工作负担重实时性好，适应性好； 能够满足大型网络的需要； 因要搜集网络运行状态，网络开销有所增加，实现也比较复杂

路由器自动获取路径信息的有两种方法：向量-距离算法，链路-状态算法。
向量-距离算法(vector-distance,简称VD)，它的基本思想是：路由器周期性的向与他相邻的路由器广播刷新报文,个路由器收到报文后，按照最短路径优先原则对各自的路由表进行刷新。
该算法虽然简单，易于实现，但是它的信息交换量大(当交换路由信息的时候，几乎传输整个路由表)，该算法不适合那些网络结构频繁变化的或者大型的网络结构。
链路-状态算法(link-status,简称L-S)，也叫最短路径优先(shortest path first SPF)算法，它的主要做法如下：
1).首先由路由器向相邻路由器发送查询报文，测试是否可以正常通信
2).在收到链路状态后，向系统中所有参加最短路径优先算法的路由器发送链路状态报文；
3).各路由器收到其他路由器发来的链路状态报文后，根据报文中的数据刷新本路由器所保存的网络拓扑结构图。如果链路发生变化，路由器将启用Dijkstra算法生成新的最短路径优先数，并刷新本地路由表；

### 分组交换（packet switching，包交换）

分组交换也称为包交换，它将用户通信的数据划分成多个更小的等长数据段，在每个数据段的前面加上必要的控制信息作为数据段的首部，每个带有首部的数据段就构成了一个分组

首部指明了该分组发送的地址，当交换机收到分组之后，将根据首部中的地址信息将分组转发到目的地，这个过程就是分组交换

### 差错处理

没有哪个采用分组交换技术的网络在任何时候都能运转正常的。正因会出现各类的故障，所以互联网需要差错检查与纠正机制。如果网路出现差错，IP协议仅通过IP首部校验和提供一种传输差错检测手段，将错误信息高速给某个应用程序，或采取其他措施来纠错。而网络层的补充协议ICMP，就是提供一种差错报告机制的。
一、目的不可达报文（类型3）
0：网络不可达； 1：主机不可达；
2：协议不可达； 3：端口不可达；
4。需要分片单DF被置位； 5：源路由失败；
0、1、4、5一般由路由器发出。2、3一般由主机发出。
二、超时报文（类型11）
0：运输过程中TTL超时；
1：分片重装超时；
TTL:生存时间，指明数据报在互联网系统中允许保留的时间。
0的是路由器发出，1一般来自主机。
三、源抑制报文（类型4）
只有一个代码0
路由器或主机都可以发出。

### IP组播

数据报的多点交付，即IP组播。一对多的通信，用IP组播实现可以大大节约网络资源。
![img](http://192.168.12.24/Public/Uploads/2020-01-10/5e17d9cb59957.png)