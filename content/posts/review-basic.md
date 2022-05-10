---
title: "基础复习"
date: 2021-10-12T13:47:43+08:00
tags: ["basic", "review"]
draft: false
---

- 网络协议
- 操作系统
- 锁、同步、通信

<!--more-->

### 1、网络协议之HTTP、HTTPS、DNS

#### OSI七层模型

为什么需要分层？

- 不同层实现不同功能

七层模型

- 应用层：为计算机用户提供接口和服务
- 表示层：数据处理（编码解码、加密解密等）
- 会话层：管理（建立、维护、重连）通信会话
- 传输层：管理端到端的通信连接
- 网络层：数据路由（决定数据在网络的路径）
- 数据链路层：管理相邻节点之间的数据通信
- 物理层：数据通信的光电物理特性

四层模型（映射）

- 应用层：（应用层、表示层、会话层）
  - HTTP/FTP/SMTP/DNS...
  - 定义了运行在不同端系统上的应用程序进程如何相互传递报文
  - 应用层定义了进程交换的报文类型、报文的语法、字段的含义、进程如何发生数据、怎样发送数据等
- 传输层：（传输层）
  - TCP/UDP
  - 传输层属于主机间不同进程的通信，传输层向上面的应用层提供通信服务，并屏蔽了下面的核心网络细节，使得面向传输层编程就像是两个主机进程之间有一条端到端的逻辑通信信道一样；当传输层采用TCP协议时，这条逻辑通信信道就是一条可靠的通信信道，尽管下面的网络是不可靠的
- 网络层：（网络层）
  - IP/ICMP
  - 网络层属于主机之间的通信，它的目的是向上提供简单灵活的、无连接的、尽最大努力交付的**数据报服务**，网络层不提供服务质量的承诺。
  - 不需要建立连接、每个数据报单独路由、每个数据报有完整的目标地址（IP）、不提供可靠的连接、到达终点可能无序、由终点进行差错检测
- 网络接口层：（数据链路层、物理层）
  - Ethernet/ARP/RARP

报文结构

![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211007211719.png)



#### HTTP版本

HTTP/0.9

- HTTP协议原型、设计缺陷、只支持GET方法、不支持多媒体内容、只有HTML对象

HTTP/1.0

- 广泛使用、增加多种方法、支持多媒体对象、无连接无状态、...

HTTP/1.1

- 长连接、管道化、缓存处理、断点传输、...

HTTP/2.0

- 性能进一步提升、二进制分帧、多路复用、首部压缩、服务端推送、...
- 多路复用：多路复用通常表示在一个信道上传输多路信号或数据流的过程和技术。通过使用多路复用，通信运营商可以避免维护多条线路，从而有效地节约运营成本
  - 二进制分帧是基础，通信单位为帧。
  - 多请求并行不依赖多TCP连接。
  - 并行在一个TCP连接交互多种类型信息。
- 头部压缩：1、客户端和服务端维护一个同样的头部静态字典，传输通过key进行交互。2、通过`Huffman`编码进行字符压缩。
- 服务端推送：服务端主动推送资源。

keep-alive长连接

- TCP建立连接需要成本



#### HTTP状态码及含义

HTTP报文结构

- 请求报文
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211008184820.png)
- 应答报文
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211008184837.png)



HTTP请求方法

- GET：最常用的方法，常用于请求服务器发送某个资源
- HEAD：和GET类似，但服务器在响应中只返回首部
- POST：向服务器写入数据
- TRACE：观察请求报文到达服务器的最终样子
- PUT：和GET相反，向服务器写入资源
- DELETE：请求服务器删除请求URL所指定的资源
- OPTIONS：一般用于中间服务，用于向服务器请求它所支持的请求方法

HTTP请求

- 幂等操作：任意多次执行所产生的影响均与第一次执行的影响相同
- 幂等函数：指可以使用相同参数重复执行，并能获取相同结果的函数

HTTP状态码

- 200~299：成功状态码
  - 200：OK  请求没问题，实体的主题部分包含了请求的资源
  - 204：No Content  响应报文中包含若干首部和一个状态行，但没有实体的主体部分
- 300~399：重定向状态码
  - 304：Not Modified  所请求的资源未修改，服务器返回此状态码时，不会返回任何资源
- 400~499：客户端错误状态码
  - 400：Bad Request  客户端请求的语法错误，服务器无法理解
  - 401：Unauthorized  请求客户端在获取对资源的访问权之前，对自己进行认证
  - 403：Forbidden  请求被服务器拒绝了
  - 404：Not Found  说明服务器无法找到所请求的URL
- 500~599：服务端错误状态码
  - 500：Internal Server Error  服务器内部错误，无法完成请求
  - 502：Bad Gateway  作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应
  - 503：Service Unavailable  说明服务器现在无法为该请求提供服务
  - 504：Gateway Timeout  网关或代理服务器，未及时从远端服务器获取请求

#### 安全传输

对称加密

- 通过同一个密钥既可以加密信息也可以解密信息
- DES、3DES、AES

非对称加密

- RSA、ECC、DH

散列算法

- 一种从任何一种数据中创建小的数字“指纹”的方法。散列函数吧消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来
- MD5
- 加盐处理

#### HTTPS加密认证 -- TLS技术

HTTP vs HTTPS

- HTTPS(Secure)是安全的HTTP协议
- 安全、复杂度高、效率低、端口443

TLS

- TLS：传输层安全性协议（Transport Layer Security）
- 数据安全和数据完整。对传输层数据进行加密后传输。
- 综合了对称加密、非对称加密技术设计的安全协议

数字证书

- 是可信任组织颁发给特定对象的认证

SSL安全参数握手

- 非对称密钥、对称密钥
- 双方分别生成密钥，没有经过传输

#### 域名系统DNS服务

DNS

- Domain Name System：域名系统

DNS工作原理

- 迭代查询、递归查询

#### DNS安全 -- DNS攻击

DNS劫持

- 黑客攻击本地DNS服务 运营商进行劫持

DNS欺骗

DDoS攻击

- denial-of-service attack  拒绝服务攻击

防范手段

- DNS服务商角度
- 个人用户角度



### 2、网络协议之IP、TCP、UDP

#### TCP和UDP的区别

端口

- 套接字（Socket）：IP地址+端口号

UDP协议

- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211008213653.png)

TCP协议

- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211008213815.png)
- 序号
  - 4个字节[0, 4294967295]
  - TCP数据是字节流 -- 每个字节都有唯一的序号
  - 起始序号在建立TCP连接的时候设置
  - 序号表示本报文段数据的第一个字节的序号
- 确认号
  - 4个字节
  - 期待收到对方下一个报文的第一个数据字节序号
- 控制位
  - 6个比特位
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211008214206.png)
- 窗口
  - 2字节
  - 窗口指明允许对方发送的数据量

UDP vs TCP

- TCP提供的是可靠的有连接服务。UDP提供的是不可靠的无连接服务
- TCP要求系统资源较多，UDP较少
- TCP是字节流，UDP是数据报
- TCP保证数据的正确性，UDP可能丢包
- TCP保证数据顺序，UDP不保证



#### TCP连接的建立过程

TCP协议为什么需要三次握手

- TCP协议提供的是可靠的有连接的服务
- 避免两次握手导致的建立多个连接

#### TCP连接的释放过程

TCP连接四次挥手

TIME-WAIT状态

- 第四次挥手后，主动中断连接方所处的状态，这个状态下，主动方尚未完全关闭TCP连接，端口不可复用
- 2MSL（2*2min）

TIME-WAIT为什么2MSL

- 2MSL时间内，如果第四次确认报文没有到达对方，那么对方会重新进行第三次挥手，确保连接正常释放
- 确保当前连接的所有报文都已过期

#### TCP的可靠传输

停止-等待协议

- 每发送完一个分组就停止发送，等待对方的确认。在收到确认后再发送下一个分组。
- 出差错情况：超时重传
- 停止-等待协议是最简单的可靠传输协议。停止-等待协议对信道的利用效率不高

连续ARQ协议

- 累计确认：只确认最后收到的字节 
- 滑动窗口
  - 窗口指明允许对方发送的数据量
  - TCP协议是传输数据流的协议，通过TCP协议头部序列号、确认好以及窗口等字段的控制，可以在有限的资源下，接受几乎无限的数据

#### 拥塞避免

网络拥塞

- 某段时间内，对网络中的某一资源的需求超过了该资源所能提供的可用部分，网络性能就会变坏，这种情况成为网络拥塞。

慢开始与拥塞避免

- 拥塞窗口：TCP协议基于窗口的拥塞控制需要的一个变量配置。发送方在发送数据时会维持一个拥塞窗口cwnd的状态变量，并且可以动态变化，在TCP报文头部，发送方让自己的发送窗口等于拥塞窗口。
- 门限值：拥塞避免算法的启动阈值，cwnd超过门限值ssthresh时，启动慢启动算法
- 传输轮次：一次报文发送和确认的时间成为一次传播轮次，RTT定义的是一次传播轮次的往返时间。
- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211009174444.png)

快重传和快恢复

- 快重传：让发送方尽快知道个别报文段的丢失，并立即重传，以避免发送方认为网络发生了拥塞，从而因为拥塞避免算法降低发送数据。
- 快恢复：出现快重传后 门限值=拥塞窗口/2，拥塞窗口=门限值。没有慢开始过程，所以称为快恢复。

#### TCP粘包

 应用层的数据拆分

- TCP协议是面向字节流的协议，不存在消息、数据包的概念。它可能会组合或者拆分应用层协议的数据。
- 粘包并不是TCP协议造成的，而是应用层协议涉及缺陷导致的问题。应用层协议需要自行设计消息边界，以正确分离消息，避免消息粘连。
- HTTP
  - `Content-Length`：是一个实体消息的首部，用来指明发送给接收方的消息主体的大小，即用十进制数字表示的数据字节的大小。 
- 数据拆分方法
  - 基于长度的标识。例如HTTP
  - 基于特殊分割符。
- Nagle算法：一种通过减少数据包的方式提高TCP传输性能的算法。
  - 因为网络带宽有限，它不会将小的数据块直接发送到目的主机，而是会在本地缓冲区中等待更多待发送的数据，这种批量发送数据的策略虽然会影响实时性和网络延迟，但是能够减低网络拥堵的可能性并减少额外开销。
  - 需要额外处理粘包的问题

#### TCP协议安全性

SYN flood攻击

- 利用三次握手的过程漏洞。大量发送第一次握手(假IP)的报文；攻击方忽略第二次握手的报文；被攻击方多个TCP连接处于（SYNC-RCVD）阶段，耗费大量资源；最终因为资源耗尽，拒绝服务（Dos）。
- **todo** 怎么避免？

资源耗尽类攻击

- 攻击方通过和被攻击目标完成三次握手后保持连接但不做任何事情，消耗TCP连接资源。或者马上断开连接有重新发起新连接。最终服务器不堪重负，影响正常服务或者直接宕机。

协议特性漏洞攻击

- 攻击方与攻击目标简历正常连接；依据TCP流量控制的特性，马上把TCP窗口设置为0,；攻击方断开连接，被攻击目标还在等待窗口重新打开；最终服务器不堪重负，影响正常服务或者直接宕机。

防范手段

- 识别异常流量
- 分流异常流量
- 高可用部署
- 合理设计参数
- 实时监控告警
- ...

#### 虚拟专用网技术VPN

VPN

- 虚拟隧道、专用IP地址
- 网络层
- NAT
- IPSec、PPTP、L2TP

专用IP地址

- 内部通信的IP地址
- 10.0.0.0 ~ 10.255.255.255、172.16.0.0 ~ 172.31.255.255、192.168.0.0 ~ 192.168.255.255



### 3、操作系统之进程、线程与协程

#### 进程

多道程序设计

- 使得批处理系统可以一次处理多个任务
- 操作系统实现了对计算机硬件资源的管理和抽象

进程存在的意义

- 进程是系统进行资源分配和调度的基本单位
- 进程作为程序独立运行的载体保障程序正常运行
- 进程的存在使得操作系统资源的利用率大幅提升

#### 进程状态模型。同步、异步

五状态模型

- 创建
- 就绪：其他资源都准备好、只差CPU资源的状态
- 终止
- 阻塞：进程因某种原因，放弃CPU的状态
- 执行：进程获得CPU，其程序正在执行

阻塞、非阻塞、异步、同步

- 同步和异步强调的是消息通信机制
- 阻塞和非阻塞强调的是程序在等待调用结果时的状态

#### 线程

线程

- 操作系统进行运行调度的最小单位
- 包含在进程之中，是进程实际运行工作的单位
- 一个进程可以并发多个线程，每个线程执行不同的任务 

进程与线程的关系

- 进程的线程共享进程资源

#### 操作系统用户态/内核态

内核态

- 内核空间：存放的是内核代码和数据
- 进程执行操作系统内核的代码
- CPU可以访问内存所有的数据，包括外围设备

用户态

- 用户空间：存放用户程序的代码和数据
- 进程在执行用户自己的代码（非系统调用之类的函数）
- CPU只可以访问有限的内存，不允许访问外设

内核态/用户态切换

- 系统调用、异常中断、外围设备中断

#### 程序运行类型

计算密集型

- CPU密集型
- 完成一项任务的时间取决于CPU的速度
- CPU利用率高、其他事情处理慢

IO密集型

- 频繁读写网络、磁盘等任务
- 完成一项任务的时间取决于IO设备的舒服
- CPU利用率低，大部分时间在等待设备完成

#### 协程

上下文切换

- 寄存器级上下文、用户级上下文、系统级上下文

协程Coroutine

- 微线程、纤程、协作式线程
- 比线程更小的力度；运行效率更高；可以支持更高的并发
- 本质是用户级的线程

协程优缺点

- 调度、切换、管理更加轻量
- 内核无法感知协程的存在
- 可以减少上下文切换的成本
- 无法发挥CPU的多核优势
- 协程主要运用在多IO的场景

### 4、操作系统之存储系统

#### 缓存

存储器的层次结构

- 缓存、主存、辅存
- 局部性原理：CPU访问存储器时，无论是存取指令还是存取数据，所访问的存储单元都趋于聚集在一个较小的连续的区域中。
- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010203135.png)
- 缓存-主存层次
  - 原理：局部性原理
  - 实现：在CPU与主存之间增加一层速度快（容量小）的Cache
  - 目的：解决主存速度不足的问题
- 主存-辅存层次
  - 原理：局部性原理
  - 实现：主存之外增加辅助存储器（磁盘、SD卡、U盘等）
  - 目的：解决主存容量不足的问题

缓存的设计

- 原理：局部性原理
- 分离冷热数据，降低热点数据服务的负载
- 提升吞吐量、并发量，提升服务质量

#### 虚拟内存

逻辑地址空间

- 指进程可以使用的内存空间
- 逻辑地址空间的大小仅受CPU地址长度限制。32位地址最大逻辑空间为4G
- 逻辑地址空间是一个进程运行时程序指令与程序数据可以用的相对地址空间

物理地址空间

- 物理地址指向物理内存的存储空间
- 物理地址空间是指程序运行过程在物理内存分配和使用的地址空间

虚拟内存概述

- 程序运行时，无需全部装入内存，装载部分即可
- 如果访问页不在内存，则发出缺页中断，发起页面置换
- 从用户层面看，程序拥有很大的空间，即虚拟内存
- 虚拟内存实际是对物理内存的补充，速度接近于内存，成本接近于辅存

#### 缺页中断

内存管理

- 页式存储管理
  - 将进程逻辑空间等分成若干大小的页面
  - 相应的把物理内存空间分成页面大小的物理块
  - 以页面为单位把进程空间装进物理内存中分散的物理块
  - 页表：页表记录进程逻辑空间与物理空间得映射
- 段式存储管理
  - 将进程逻辑空间划分成若干段（非等分）
  - 段的长度由连续逻辑的长度决定
  - 段式存储管理相比页式存储管理更加灵活
- 段页式存储管理
  - 分页管理有效提高内存利用率；分段管理更加灵活。两者结合，想成段页式存储管理
  - 先把逻辑空间按段式管理分成若干段
  - 再把段内空间按页式管理等分成若干页

缺页中断

- 当所要访问的页面不在内存时，会产生一次缺页中断。此时操作系统会根据页表中的外存地址在外存中找到所缺的一页，调入内存
- 磁盘属于外设，读写存盘需要系统调用
- 在指令执行期间产生和处理中断信号
- 一条指令执行期间可能产生多次缺页中断

#### 页面置换算法

页面置换的时机

- 高速缓存替换
- 主存页面的替换

缓存置换算法

- 先进先出算法 FIFO
- 最近最少使用算法 LRU
- 最不经常使用算法 LFU

#### Linux文件系统 -- 软链接与硬链接

常见文件系统

- FAT、NTF、ext2/3/4
- 操作系统中负责管理和存储文件信息的软件机构成为文件管理系统，简称文件系统

Ext文件系统

- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010211740.png)
- Inode Table
  - 存放文件Inode的地方
  - 每一个文件（目录）都有一个Inode
  - 每一个文件（目录）的索引节点
- Inode
  - 文件的相关元信息
  - 文件名不是存放在Inode节点上，而是存放在目录的Inode节点。所以列出目录文件的时候无需加载文件的Inode
- Inode botmap
  - Inode的位示图
  - 记录已分配的Inode和未分配的Inode
- Data block
  - 存放文件内容的地方
  - 每个block都有唯一的编号
  - 文件的block记录在文件的Inode上
- Block bitmap
  - 记录Data block的使用情况
- Superblock
  - 记录整个文件系统相关信息
  - Block和Inode的使用情况
  - 时间信息、控制信息等

软连接-硬链接

- 硬链接直接指向源文件的Inode
- 软链接有自己的Inode和block。里面存放的是源文件的路径，指向源文件
- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010212317.png)
- 具有相同Inode节点号的文件护卫硬链接文件
- 删除硬链接文件或者删除源文件任意之一，文件实体并未删除
- 删除源文件，软连接依然存在，但无法访问源文件内容

#### 磁盘冗余阵列 RAID

RAID

- Redundant Array of Independent Disks。磁盘冗余阵列
- 利用虚拟化存储技术把多个硬盘组合起来，成为一个或多个硬盘阵列组，目的提升性能或减少冗余。

RAID分级方案

- RAID 0
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010212906.png)
  - 性能：单块磁盘的N倍
  - 不提供数据校验和数据冗余
  - 某块磁盘损坏，数据直接丢失且无法恢复
- RAID 1
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010213109.png)
  - 数据无差别的双写工作磁盘和镜像磁盘
  - 性能：单块磁盘的N/2倍
  - 数据可靠性强，只要不是同事损坏，都可以恢复
- RAID 5
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010213349.png)
  - 数据中心最常见的RAID等级
  - 提供纠错海明码实现数据冗余校验
  - 分散校验盘，提高写性能，降低检验盘出错概率
- RAID10
  - ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010213942.png)
- ![img](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20211010214019.png)

### 5、锁、同步、通信

#### 死锁

死锁的产生

- 竞争资源
- 进程调度顺序不当

死锁的四个必要条件

- 互斥条件
- 请求保持条件
- 不可剥夺条件
- 环路等待条件

预防死锁的方法

- 破坏上述死锁必要条件

银行家算法