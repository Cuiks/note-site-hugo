---
title: "计算机网络"
date: 2021-05-27T15:25:29+08:00
tags: ["计算机网络"]
draft: false
---

- 计算机网络之概述
- 计算机网络之网络层
- 计算机网络之传输层
- 计算机网络之应用层
- 计算机网络实践

<!--more-->

## 计算机网络之概述篇

### 计算机网络的发展简史

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517202959.png)

- 第一阶段：单个网络ARPANET
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517203241.png)
- 第二阶段：三层结构互联网
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517203314.png)
- 第三阶段：多层次ISP互联网
  - ISP(Internet Service Provider)：网络服务提供商
  - 中国电信、中国移动、中国联通等
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517203540.png)

#### 中国互联网发展简史

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517204041.png)

### 层次结构设计的基本原则

#### 层次结构设计的基本原则

- 需要考虑的问题（采用分层设计的原因）
  - 保证数据通路顺畅
  - 目的计算机状态
  - 识别目的计算机
  - 数据是否错误

- 设计原则
  - 各层之间相互独立
  - 每一层要有足够的灵活性
  - 各层之间完全解耦


#### OSI七层模型

七层模型：

- 应用层
  - 为计算机用户提供接口和服务
- 表示层
  - 数据处理（编码解码、加密解密等）
- 会话层
  - 管理（建立、维护、重连）通信会话
- 传输层
  - 管理端到端的通信连接
- 网络层
  - 数据路由（决定数据在网络中的路径）
- 数据链路层
  - 管理相邻节点之间的数据通信
- 物理层
  - 数据通信的光点物理特性

other：

- OSI欲成为全球计算机都遵循的标准
- OSI在市场化的过程中困难重重，TCP/IP在全球范围成功运行
- OSI最终并没有成为广为使用的标准模型

OSI没有广泛推广的原因：

- OSI的专家缺乏实际经验
- OSI标准制定周期过长，按OSI标准生产的设备无法及时进入市场
- OSI模型设计的并不合理，一些功能在多层中重复出现

#### TCP/IP四层模型

四层模型：

- 应用层
  - 应用层、表示层、会话层
  - HTTP、FTP。。。
- 传输层
  - 传输层
  - TCP、UDP
- 网络层
  - 网络层
  - IP、ICMP
- 网络接口层
  - 数据链路层、物理层
  - Ethernet、ARP、RARP

数据传输：
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517210618.png)
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210517210821.png)

### 现代互联网的网络拓扑

边缘部分

核心部分


### 计算机网络的性能指标

速率： bps=bit/s

时延：

- 发送时延
  - =数据长度(bit)/发送速率(bit/s)
- 排队时延
  - =传播路径距离/传播速度(bit/s)
- 传播时延
  - 数据包在网络设备中等待被处理的时间
- 处理时延
  - 数据包到达设备或者目的机器被处理所需要的时间

总时延 = 发送时延 + 排队时延 + 传播时延 + 处理时延

RTT(Route-Trip Time)

- 表示的是数据报文在端到端通信的来回一次的时间
- 通常使用ping命令查看RTT


### 数据链路层概述

- 封装成帧
  - “帧”是数据链路层数据的基本单位
  - 发送端在网络层的一段数据前后添加特定标记形成“帧”
  - 接收端根据前后特定标记识别出“帧”
  - 物理层不管是不是“帧”
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518112503.png)
  - 帧首部和尾部是特定的控制字符（特定比特流）
    - 首部(SOH)：00000001
    - 尾部(EOT): 00000100
- 透明传输
  - "透明"在计算机领域是一个非常重要的术语
  - 即是控制字符在帧数据中，但要当做不存在的去处理
  - ESC  转义字符
- 差错监测
  - 物理层只管传输比特流，无法控制是否出错
  - 数据链路层负责起“差错检测”的工作


### 数据链路层的差错检测

- 奇偶校验码
  - 数据流求和。偶数就在末尾添加0，否则在末尾添加1
  - 出错位数为偶数时，校验不出
- 循环冗余校验码CRC
  - 一种根据串或保存的数据而产生固定位数校验码的方法
  - 检测数据传输或者保存后可能出现的错误，生成的数字计算出来并且附加导数据后面
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518134703.png)
  - 计算步骤
    - 选定一个用于校验的多项式G(x)，并在数据尾部添加r个0
    - 将添加r个0后的数据，使用模"2"除法除以多项式的位串
    - 得到的余数填充在原数据r个0的位置得到可校验的位串
  - CRC的错误检测能力与位串的阶数r有关
  - 数据链路层只进行数据的检测，不进行纠正（对于错误的数据，直接丢弃）
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518135325.png)


### 最大传输单元MTU

- MTU(Maximum Transmission Unit)
- 数据链路层的数据帧也不是无限大的
- 数据帧长度受MTU限制
- 数据帧过大或者过小都会影响传输的效率(以太网一般为1500字节)
- 路径MTU由链路中MTU最小值决定


### 以太网协议详解

#### MAC地址

- 物理地址、硬件地址
- 每个设备都拥有唯一的MAC地址
- MAC地址共48位，使用十六进制表示

#### 以太网协议

- 以太网是一种使用广泛的局域网技术
- 以太网是一种应用于数据链路层的协议
- 使用以太网可以完成相邻设备的数据帧传输
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518172545.png)
- MAC地址表
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518172640.png)
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518172808.png)
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210518172900.png)


## 计算机网络之网络层篇

### IP协议详解

#### 虚拟互联网络

- IP协议是把复杂的实际网络变成一个虚拟互联的网络
- IP协议是的网络层可以屏蔽底层细节而专注网络层的数据转发
- IP协议解决了在虚拟网络中数据报传输路径的问题 

#### IP协议

- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519151741.png)
- 长度32位，常分为4个8位
- IP地址常使用点分十进制来表示
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519152028.png) 
- ip首部
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519152105.png)
  - 版本：占4位，指的是IP协议的版本。即IPV4或IPV6
  - 首部位长度：占4位，表示IP首部长度，即是IP首部最大长度为60字节
  - 总长度，占16位，最大数值25535，表示IP数据报总长度(IP首部+IP数据)
  - 标识
  - 标志
  - 片偏移
  - TTL：占8位，表明IP数据报文在网络中的寿命，没经过一个设备，TTL减1，当TTL=0时，网络设备必须丢弃该报文
  - 协议：占8位，表明IP数据所携带的具体数据是什么协议的(如：TCP、UDP等)
  - 首部校验和：占16位，校验IP首部是否出错

### IP协议的转发流程

#### 路由表的简介

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519165603.png)

#### IP协议的转发流程

- 数据帧每一跳的MAC地址都在变化
- IP数据报每一跳的IP地址始终不变


### ARP协议与RARP协议

ARP(Address Resoution Protocol)地址解析协议

- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519170705.png)
- ARP缓存表
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519170739.png)
  - ARP缓存表是ARP协议和RARP协议运行的关键
  - ARP缓存表缓存了IP地址到硬件地址之间的映射关系
  - ARP缓存表中的记录并不是永久有效的，有一定的期限
    ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519171328.png)

RARP(Reverse Address Resolution Protocol)逆地址解析协议
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519171438.png)
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519171716.png)

- (R)ARP协议是TCP/IP协议栈里面基础的协议
- ARP和RARP的操作对程序员是透明的
- 理解(R)ARP协议有助于理解网络分层的细节


### IP地址的子网划分

#### 分类的IP地址

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519172536.png)
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519173605.png)

- 特殊主机号
  - 主机号全0表示当前网络段，不可分配为特定主机
  - 主机号全1表示广播地址，向当前网络段所有主机发消息
- 特殊网络号
  - A类地址网络段全0(00000000)表示特殊网络
  - A类地址网络段后7位全1(01111111：127)表示回环地址
  - B类地址网络段(10000000.00000000：128.0)是不可使用的
  - C类地址网络段(192.0.0)是不可使用的
- 127.0.0.1
  - 通常被称为本地回环地址(Loopback Address)，不属于任何一个有类别的地址类。

#### 划分子网

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519173904.png)

- 子网掩码
  - 子网掩码和IP地址一样，都是32位
  - 子网掩码由连续的1和连续的0组成
  - 某一个子网的子网掩码具备网络号位数个连续的1
  - 例子
    - A类：255.0.0.0
    - B类：255.255.0.0
    - C类：255.255.255.0
  - 用于快速判断IP属于哪一个子网号

#### 无分类编址CIDR

- CIDR中没有A、B、C类网络号和子网划分的概念
- CIDR把网络前缀相同的IP地址成为一个"CIDR地址块"
- 斜线记法
  - 193.10.10.129/25
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210519174547.png)


### 网络地址转换NAT技术

背景：

- IPV4只有40+亿个IP地址
- 早期IP地址的不合理规划导致IP号浪费

内网地址

- 内部机构使用
- 避免与外网地址重复
- 三类内网地址
  - 10.0.0.0 ~ 10.255.255.255（支持千万数量及设备）
  - 172.16.0.0 ~ 172.31.255.255（支持百万数量级设备）
  - 192.168.0.0 ~ 192.168.255.255（支持万数量级设备）

外网地址

- 全球范围使用
- 全球公网唯一


- 网络地址转换NAT（Network Adress Translation）
- NAT技术用于多个主机通过一个公有IP访问互联网的
- NAT减缓了IP地址的消耗，但是增加了网络通信的复杂度
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520103506.png)


### ICMP协议详解

- 网际控制报文协议(Internet Control Message Protocol)
- ICMP协议可以报告错误信息或者异常情况
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520104518.png)
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520111555.png)

报文种类

- 差错报告报文
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520111733.png)
  
  
- 询问报文
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520112422.png)

### ICMP协议的应用

#### Ping应用

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210520112616.png)

- Ping回环地址127.0.0.1
- Ping网关地址
- Ping远端地址

#### Traceroute应用

- Traceroute可以探测IP数据在网络中走过的路径
- Traceroute依次发出TTL加一的报文，获取操作不可达的返回


### 网络层的路由概述

#### 背景：

- 路由表怎么维护的
- 下一跳怎么来的
- 下一跳地址唯一吗
- 下一跳地址是最佳的吗
- 路由器这么多，他们怎么协同工作

#### 要求：

- 算法是正确的、完整的
- 算法在计算上应该尽可能的简单
- 算法可以适应网络中的变化
- 算法是稳定的和公平的

#### 路由算法本质是图的算法

#### 对互联网进行划分

- 互联网的规模是非常大的
- 互联网的环境是非常复杂的

#### 自制系统（Autonomous System）

- 一个自治系统(AS)是处于一个管理机构下的网络设备群
- AS内部网络自行管理，AS对外提供一个或者多个出（入）口

- 自治系统内部路由的协议称为：内部网关协议(RIP、OSPF)
- 自治系统外部路由的协议称为：外部网关协议（BGP）


### 内部网络路由协议RIP协议

#### 距离矢量(DV)算法

- 每一个节点使用两个向量Di和Si
- Di描述的是当前节点到别的节点的距离
- Si描述的是当前节点到别的节点的下一节点
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521145503.png)

- 每一个节点与相邻的节点交换向量Di和Si的信息
- 每一个节点根据交换的信息更新自己的节点信息

- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521153327.png)


#### RIP协议的过程

- RIP(Routing Information Protocol)协议
- RIP协议是使用DV算法的一种路由协议
- RIP协议把网络的跳数(hop)作为DV算法的距离
- RIP协议每隔30秒交换一次路由信息
- RIP协议认为跳数>15的路由则为不可达路由

RIP协议过程

1. 路由器初始化路由信息(两个向量Di和Si)
2. 对相邻路由器X发过来的信息，对信息的内容进行修改（下一跳地址设置为X，所有距离加1）
   1.  检索本地路由，将信息中新的路由插入到路由表里面
       - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521154430.png) 
   2.  检索本地路由，对于下一跳为X的，更新为修改后的信息
       - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521154555.png) 
   3.  检索本地路由，对比相同目的地距离，如果新信息的距离更小，则更新本地路由表
       - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521154853.png) 
3. 如果3分钟没有收到相邻的路由信息，则把相邻路由设置为不可达(16跳) 

RIP协议优点：

- 实现简单，开销很小

RIP协议缺点：

- 限制了网络的规模
- 故障消息传递慢，更新收敛时间过长


### Dijkstra(迪杰斯特拉)算法

- 著名的图算法
- 解决有权图从一个节点到其他节点的最短路径问题
- 广度优先

最短路径问题
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521160357.png)


### 内部网关路由协议之OSPF协议

#### 链路状态(LS)协议

- 向所有路由器发送信息
- 消息描述该路由器与相邻路由器的链路状态
- 只有链路状态发生变化时， 才发送更新信息

#### OSPF协议的过程

- OSPF(Open Shortest Path First: 开放最短路径优先)
- OSPF协议的核心是Dijkstra算法


- 向所有的路由器发送消息
  - 获得网络中的所有信息 -> 网络的完整拓扑
  - 也称为“链路状态数据库”
  - “链路状态数据库”是全网一致的
  - Dijkstra算法
- 消息描述该路由器与相邻路由器的链路状态
  - OSPF协议更加客观、更加先进
- 只有链路状态发生变化时， 才发送更新信息
  - 减少了数据的交换，更快收敛


- 五种消息类型
  - 问候(Hello)
  - 链路状态数据库描述信息
  - 链路状态请求信息
  - 链路状态更新信息
  - 链路状态确认信息

过程：
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521163210.png)

对比：
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521163242.png)

### 外部网关路由协议之BGP协议

- BGP(Border Gateway Protocol: 边际网关协议)
- BGP协议是运行在AS之间的一种协议
- BGP协议可以找到一条到达目的比较好的路由

背景：

- 互联网规模很大
- AS内部使用不同的路由协议
- AS之间需要考虑网络特性意外的一些因素（政治、安全...）

BGP发言人(speaker)

- BGP并不关心内部网络拓扑
- AS之间通过BGP发言人交流信息
- BGP Speaker可以人为配置策略

## 计算机网络之传输层

- 使用端口来标记不同的网络进程
- 端口使用16比特位表示（0~65535）

### UDP协议详解

- UDP(User Datagram Protocol:用户数据报协议)
- UDP是一个非常简单的协议

数据报(Datagram)：

- 不合并 不拆分

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521222510.png) 
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521223434.png)

特点：

- UDP是无连接协议
- UDP不能保证可靠的交付数据
- UDP是面向报文传输的
- UDP没有拥塞控制
- UDP的首部开销很小


### TCP协议详解

- TCP(Transmission Control Protocol:传输控制协议)
- TCP协议是计算机网络中非常复杂的一个协议

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521223941.png)

特点：

- TCP是面向连接的协议
- TCP的一个连接有两端（点对点通信）
- TCP提供可靠的传输服务
- TCP协议提供全双工的通信
- TCP是面向字节流的协议

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521224357.png)

- 序号
  - 0 ~ 2^32-1
  - 一个字节一个序号
  - 数据首字节序号
- 确认号
  - 0 ~ 2^32-1
  - 一个字节一个序号
  - 期望收到的首字节序号
- 数据偏移
  - 占4位：0 ~ 15，单位为：32位字
  - 数据偏移首部的距离
- TCP标记
  - 占6位，每位各有不同意义
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521224908.png)
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210521224931.png)
- 窗口
  - 占16位，0 ~ 2^16-1
  - 窗口指明允许对方发送的数据量
- 校验和
- 紧急指针
  - 紧急数据（URG=1）
  - 置顶紧急数据在报文的位置
- TCP选项
  - 最多40字节
  - 支持未来的拓展

### 可靠传输的基本原理

#### 停止等待协议

- 发送的消息在路上丢失了
- 确认的消息在路上丢失了
- 确认的消息很久才到

超时定时器

- 每发送一个消息后都需要设置一个定时器


- 停止等待协议是最简单的可靠传输协议
- 停止等待协议对信道的利用效率不高

#### 连续ARQ协议

ARQ(Automatic Repeat reQuest: 自动重传请求)
    - 滑动窗口
        - 累计确认

### TCP协议的可靠传输

#### TCP的可靠传说基于连续ARQ协议

- 滑动窗口
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210524172511.png)
- 选择重传
  - 选择重传需要指定需要重传的字节
  - 每一个字节都有唯一的32位序号
  - 需要重传的边界序号存放在TCP首部的“TCP选项”中
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210524173740.png)

#### TCP的滑动窗口以字节为单位


### TCP协议的流量控制

- 流量控制指让发送方发送速率不要太快
- 流量控制是使用滑动窗口来实现的
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525113649.png)

坚持定时器

- 当接收到窗口为0的消息，则启动坚持定时器
- 坚持定时器每隔一段时间发送一个窗口探测报文


### TCP协议的拥塞控制

拥塞原因：

- 一条数据链路经过非常多的设备
- 数据链路中各个部分都有可能成为网络传输的瓶颈

和流量控制的区别：

- 流量控制考虑点对点的通信量的控制
- 拥塞控制考虑整个网络，是全局性的考虑

报文超时则认为是拥塞

#### 慢启动算法

- 由小到大逐渐增加发送数据量
- 每收到一个报文确认，就加一
  - 指数增长
- 慢启动阈值(ssthresh)
  - 增长到慢启动阈值不再增长

#### 拥塞避免算法

- 维护一个拥塞窗口的变量
- 只要网络不拥塞，就试探着拥塞窗口调大

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525115024.png)


### TCP连接的建立  三次握手

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525133143.png)

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525140052.png)

为什么发送方要发出第三个确认报文呢？

- 已经失效的连接请求报文传送到对方，引起错误
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525140546.png)


### TCP的释放  四次挥手

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525142503.png)

#### 等待计时器

- 等待2MSL时间
- 为什么等待2MSL
  - 确保发送方的ACK可以到达接收方
    - 2MSL时间内没有收到，则接收方会重发。保证接收方收到了最后一次挥手
  - 确保当前连接的所有报文都已经过期


MSL:

- MSL(Max Segment Lifetime)：最长报文段寿命
- MSL建议设置2分钟


### 套接字与套接字编程

- 使用端口(Port)来标记不同的网络进程
- 端口(Port)使用16比特位表示(0~65535)


- 套接字(Socket)是抽象概念，表示TCP连接的一端
- 通过套接字可以进行数据发送或接收
- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525143952.png)

套接字编程流程：

- 服务端
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525144058.png)
- 客户端
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210525144117.png)

域套接字和网络套接字

- Unix域套接字只能用于在同一个计算机的进程间进行通信。但是使用Unix域套接字效率会更高，因为Unix域套接字仅仅进行数据复制，不会执行在网络协议栈中需要处理的添加、删除报文头、计算校验和、计算报文顺序等复杂操作


## 计算机网络之应用层篇

- 传输层及以下提供完整的通信服务
- 应用层是面向用户的一层


- UDP
  - 多媒体信息分发
    - 视频
    - 语音
    - 实时信息
- TCP
  - 可靠信息传输
    - 金融交易
    - 可靠通讯
    - MQ

应用层概述：（定义应用间通信的规则）

- 应用进程的报文类型（请求报文、应答报文）
- 报文的语法、格式
- 应用进程发送数据的时机、规则


### DNS详解

DNS(Domain Name System:域名系统)

- 把域名转换为IP
- UDP协议

背景：

- IP不方便记忆，使用域名帮助记忆

域名：

- 域名由点、字母和数字组成
- 点分割不同的域
- 域名可以分为顶级域、二级域、三级域

域名分级：

- ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526135858.png)
- 顶级域：
  - 国家
    - cn 中国
    - uk 英国
    - us 美国
    - ca 加拿大
  - 通用
    - com 公司
    - net 网络服务机构
    - gov 政府机构
    - org 组织
- 二级域

域名服务器

- 根域名服务器
- 顶级域名服务器
- 域名服务器

访问方式：

- 先访问本地DNS服务器，没有的话在向上请求

DNS查询方式：

- 递归查询
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526141611.png)
- 迭代查询
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526141636.png)
- 一般来说，域名服务器之间的查询使用迭代查询方式，以免根域名服务器的压力过大。


### DHCP协议详解

- DHCP(Dynamic Host Configuration Protocol:动态主机设置协议)
- DHCP是一个局域网协议
- DHCP是应用UDP协议的应用层协议


- 临时IP
- 租期

工作流程：

- DHCP服务器监听默认端口：67
- 主机使用UDP协议广播DHCP发现报文
- DHCP服务器发出DHCP提供报文
- 主机向DHCP服务器发出DHCP请求报文
- DHCP服务器回应并提供IP地址


### HTTP协议详解

- HTTP(HyperText Transfer Protocol:超文本传输协议)
  - 超文本：带超链接的文本
- http(s)://<主机>:<端口>/<路径>
- HTTP协议是可靠的数据传输协议
- 应用TCP协议

c/s架构：

- Web服务器
  - 硬件部分
  - 软件部分
  - 请求流程
    1. 接受客户端连接
    2. 接收请求报文
    3. 处理请求
    4. 访问Web资源
    5. 构造应答
    6. 发送应答
  - 请求方法
    - GET
      - 获取指定的服务端资源
    - POST
      - 提交数据到服务端
    - DELETE
      - 删除指定的服务器资源
    - UPDATE
      - 更新指定的服务端资源
    - PUT
    - OPTIONS
    - PATCH
    - HEAD
    - TRACE
  - 指定资源
    - 在地址中指定
    - 在请求数据中指定
  - 请求报文
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526145234.png)
  - 应答报文
    - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526145246.png) 
    - 状态码
      - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526145437.png)

### HTTP工作的结构

- Web缓存
- Web代理
  - 正向代理
  - 反向代理
- CDN
  - Content Delivery Network: 内容分发网络
- 爬虫


### HTTPS协议详解

- HTTP是明文传输的
- HTTPS(Secure)是安全的HTTP协议
- http(s)://<主机>:<端口443>/<路径>

#### 加密模型

- 对称加密
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526150151.png)
- 非对称加密
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526150234.png)
  - A和B是拥有一定数学关系的一组密钥
    - 私钥：私钥自己使用，不对外公开
    - 公钥：公钥给大家使用，对外公开

#### 数字证书

- 数字证书是可信任组织颁发给特定对象的认证
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526150558.png)

#### SSL

- SSL(Secure Socket Layer:安全套接层)
  - 数据安全和数据完整
  - 对传输层数据进行加密后传输

#### HTTPS工作流程

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526150837.png)

- SSL安全参数握手过程
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526151041.png)
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526151319.png)
  - 综合使用非对称加密
    - 随机数3的传递使用非对称加密
    - 秘钥根据随机数1、2、3生成，进行对称加密传输


## 计算机网络实践

1. 搭建服务基本框架
2. python操作字节序列
3. 实现IP报文解析器
4. 实现UDP报文解析器
5. 实现TCP报文解析器


### 搭建服务基本框架

网卡工作模式：

- 混杂模式
  - 接受所有经过网卡设备的数据
- 非混杂模式
  - 只接受目的地址指向自己的数据


### Python操作字节序列

- 字节序
  - 255 = 00000000,11111111
  - 大端字节序
    - 高位字节放在前面 低位字节放在后面
      - 255 = 00000000,11111111
    - 主要用于网络
  - 小端字节序
    - 位高字节放在后面，低位字节放在前面
      - 255 = 11111111,00000000
    - 主要用于计算机
  - 原因
    - 计算机电路先处理低位字节效率比较高
    - 人类习惯读写大端字节序


- 格式字符
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526154338.png)
- Python操作字节序列
  - ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526154907.png) 


### 实现IP报文解析器

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526155904.png)

protocol：
![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210526174642.png)


### 实现UDP报文解析器

### 实现TCP报文解析器



代码：https://github.com/Cuiks/review