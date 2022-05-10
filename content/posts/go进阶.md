---
title: "Go进阶"
date: 2022-03-30T00:00:00+08:00
tags: ["go", "微服务"]
draft: false
---


<!--more-->

## 微服务（微服务概览与治理）

### 微服务概览

#### 单体架构

- 化繁为简，分而治之

#### 微服务起源

- SOA 面向服务的架构模式
- 微服务想成是 SOA 的一种实践。

#### 微服务定义

- 围绕业务功能构建的，服务关注单一业务，服务间 采用轻量级的通信机制，可以全自动独立部署，可 以使用不同的编程语言和数据存储技术。
- 原子服务、独立进程、隔离部署、去中心化服务治理
- 基础设施的建设、负责度高

#### 微服务不足

- 分布式系统会带来固有的复杂性
- 分布式事务
- 微服务测试复杂
- 服务模块间依赖，应用升级会设计多个模块
- 运维、基础设施挑战大

#### 组件服务化

- kit: 微服务基础库
- service: 业务 + kit + 依赖
- rpc + mq: 轻量级通讯
- 本质上等同于，多个微服务组合(compose)完成了一个完整 的用户场景(usecase)。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324103830.png" width="500"/>

#### 按业务组织划分

- 开发团队对软件在生产环境的运行负全部责任！

#### 去中心化

- 数据去中心化
- 治理去中心化
- 技术去中心化
- 每个服务独享自身的数据存储设施(缓存，数据库等)，不像传统应用共享一个缓存和数据 库，这样有利于服务的独立性，隔离相关干扰 。

#### 基础设施自动化

- CICD: Gitlab + Gitlab Hooks + k8s
- Testing: 测试环境、单元测试、API自动化测试
- 服务监控: k8s，以及一系列 Prometheus、ELK、Conrtol Panle

#### 可用性 & 兼容性设计

- Design For Failure
- 隔离、超时控制、负载保护、限流、降级、重试、负载均衡
- 发送时要保守，接收时要开放
  - 发送的数据要更保守， 意味着最小化的传送必要的信息，接收时更开放意味着要最大限度的容忍冗余数据，保证兼容性

### 微服务设计

#### API Geteway

- 困难
  - 客户端到微服务直接通信，强耦合。
  - 需要多次请求，客户端聚合数据，工作量巨 大，延迟高。
  - 协议不利于统一，各个部门间有差异，需要端来兼容。
  - 面向“端”的API适配，耦合到了内部服务。
  - 多终端兼容逻辑复杂，每个服务都需要处理。
  - 统一逻辑无法收敛，比如安全认证、限流。
- 内聚模式
- 新增app-interface用于统一协议出口 （BFF）
  - 轻量交互：协议精简、聚合。
  - 差异服务：数据裁剪以及聚合、针对终端定制化API。
  - 动态升级：原有系统兼容升级，更新服务而非协议。
  - 沟通效率提升，协作模式演进为移动业务+网关小组。
- BFF 可以认为是一种适配服务，将后端的微服务进行适配(主要包括聚合裁剪和格式适配等逻辑)，向无线端 设备暴露友好和统一的 API，方便无线设备接入访问后端服务。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324105238.png" width="400" />
- app-interface属于single point of failure
  - 单个模块也会导致后续业务集成复杂度高
  - 跨横切面逻辑，比如安全认证，日志监控，限流熔断等。代码逐渐复杂
- 引入 API Gateway
  - 把业务集成度高的 BFF 层和通用功能服务层 API Gateway 进行了分层处理。
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324105348.png" width="400" />
  - 单块 BFF 实现了解耦拆分
  - BFF 可以是 nodejs 来做服务端渲染 (SSR，Server-Side Rendering)

#### Microservice划分

- 如何划分服务的边界
  - 业务职能(Business Capability): 由公司内部不同部门提供的职能。例如客户服务部 门提供客户服务的职能，财务部门提供财务相关的职能。
  - 限界上下文(Bounded Context): 限界上下文是 DDD 中用来划分不同业务边界的元素，这里业务边界的含义是“解决不同业务问题”的问题域和对应的解决方案域，为了解决某种类型的业务问题， 贴近领域知识，也就是业务。
- 本质上也促进了组织结构的演进：Service per team
- CQRS
  - 将应用程序分为两部分：命令端和查询端。命令端处理程序创建，更新和删除请求，并在数据更改时发出事件。查询端通过针对一个或多个物化视图执行查询来处理查询，这些物化视图通过订阅数据更改时发出的事件流而保持最新。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324110939.png" width="400"/>
  - 架构也从 Polling publisher -> Transaction log tailing进行了演进(Pull vs Push)。

#### Microservice安全

- 外网的请求来说，通常在 API Gateway 进行统一的认证拦截，认证成功，使用 JWT 方式通过 RPC 元数据传递的方式带到 BFF 层，BFF 校验 Token 完整性后把身份信息注入到应用的 Context 中
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324111310.png" width="400"/>
  - API Gateway -> BFF -> Service
  - Biz Auth -> JWT -> Request Args
- 服务内部，一般要区分身份认证和授权
  - Full Trust: 验证服务身份且加密传输
  - Half Trust: 只验证服务身份
  - Zero Trust: 调用无需任何验证

### gRPC & 服务发现

#### gRPC

- 高性能开源统一RPC框架
- 支持多种语言
- 高性能，序列化支持Protocol Buffer
  - PB代码即文档
- 可插拔，支持插件
- IDL：文件定义服务，通过proto3工具生成指定语言代码
  - 通过PB生成HTTP和GRPC代码
- 基于标准的HTTP2设计，支持双向流、消息头压缩，单TCP的多路复用、服务端推送等
  - gRPC上公网
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220324111517.png" width="400" />
- 服务而非对象、消息而非引用。粗粒度消息
- 负载无关。可以使用不同的消息类型，pb、JSON、XML、Thrift
- 流式API
- 元数据交换
- 标准化的状态码

不要过早关注性能问题，先标准化

#### gRPC - HealthCheck

- 主动健康检查
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220323230511.png" width="400" />  
- 平滑发布
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220323230637.png" width="400" />  
- 平滑退出
  1. 首先服务收到kill信号。
  2. 服务向注册中心发送注销请求，同时标记自己为unhealth。
  3. 使用gRPC或者HTTP的shutDown接口，入参支持Context设置时间为2个心跳周期。
  4. 服务退出超时，K8S强制kill -9。

#### 服务发现 - 客户端发现

- 服务实例被启动时，它的网络地址会被写到注册表上；当服务实例终止时，再从注册表中删除
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325153746.png" width="400"/>

#### 服务发现 - 服务端发现

- 客户端通过负载均衡器向一个服务发送请求，这个负载均衡器会查询服务注册表，并将请求路 由到可用的服务实例上。
- 服务实例在服务注册表上被注册和注销
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325154712.png" width="400"/>

#### 服务发现

- 客户端发现
  - 直连，比服务端服务发现少一次网络跳转
  - Consumer 需要内置特定的服务发现客户端和发现逻辑。
- 服务端发现
  - Consumer 无需关注服务发现具体细节，只需知道服务的 DNS 域名即可，支持异构语言开发
  - 需要基础设施支撑
  - 多了一次网络跳转，可能有性能损失

- 微服务的核心是去中心化，推荐使用客户端发现模式。
- 推荐Nacos

- Zookpeeper。AP系统
  - 分布式协调服务(要求任何时刻对 ZooKeeper的访问请求能得到一致的数据，从而牺牲可用性)。
  - 网络抖动或网络分区会导致的 master 节点因为其他节点失去联系而重新选举或超过半数不可用导致服务注册发现瘫痪。
  - 大量服务长连接导致性能瓶颈

- euerka实现原理
  - AP 发现服务
  - 牺牲一致性，最终一致性
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325155154.png" width="400"/>


### 多集群 & 多租户

#### 多集群

- 多集群必要性
  - 从单一集群考虑，多个节点保证可用性，通常使用 N+2 的方式来冗余节点
  - 从单一集群故障带来的影响面角度考虑冗余多套集群。
  - 单个机房内的机房故障导致的问题

- 集群建立
  - 利用 paas 平台，给某个 appid 服务建立多套集群
  - 对于不同集群服务启动后，从环境变量里可以获取当下服务的 cluster，在服务发现注册的时候，带入 这些元信息
  - 不同集群可以隔离使用不同的缓存资源等。多套冗余的集群对应多套独占的缓存，带来更好的性能和冗余能力
  - 尽量避免业务隔离使用或者 sharding 带来的 cache hit 影响（按照业务划分集群资源）
- 集群问题
  - 业务隔离集群带来的问题是 cache hit ratio 下降，不同业务形态数据正交，我们退而求其次整个集群全部连接。（所有的consumer连接所有的集群）
  - 全部节点，全部连接。导致内存和 CPU 开销很大。短连接极大的资源成本和延迟。
  - 合适的子集大小和选择算法
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325160050.png" width="400"/>

#### 多租户

- 在一个微服务架构中允许多系统共存是利用微服务稳定性以及模块化最有效的方式之一，这种方式一般被称为多租户(multi-tenancy)
- 租户可以是测试，金丝雀发布，影子系统(shadow systems)，甚至服务层或者产品线，使用租户能够保证代码的隔离性并且能够基于流量租户做路由决策。

- 通常来说，微服务架构有两种基本的 集成测试方式：并行测试和生产环境测试。
- 挑战
  - 混用环境导致的不可靠测试。
  - 多套环境带来的硬件成本。
  - 难以做负载测试，仿真线上真实流量情况。

- 染色发布
  1. 把待测试的服务 B 在一个隔离的沙盒环境中启动，并且在沙盒环境下可以访问集成环境(UAT) C 和 D
  2. 把测试流量路由到服务 B，同时保持生产流量正常流入到集成服务。服务 B 仅仅处理测试流量而不处理生产流量。另外要确保集成流量不要被测试流量影响。
  3. <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325160557.png" width="400"/>
- 生产中的测试提出了两个基本要求，它们也构成了多租户体系结构的基础：
  - 流量路由：能够基于流入栈中的流量类型做路由。
  - 隔离性：能够可靠的隔离测试和生产中的资源，这样可以保证对于关键业务微服务没有副作用。
- 灰度测试成本代价很大，影响 1/N 的用户。其中 N 为 节点数量。
- 实现：
  - 给入站请求绑定上下文(如: http header)， inprocess 使用 context 传递，跨服务使用 metadata 传递
  - 在这个架构中每一个基础组件都能够理解租户信息，并且能够基于租户路由隔离流量，同时平台中允许对运行不同的微服务有更多的控制，比如指标和日志
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220325160817.png" width="400"/>
- 多租户架构本质上描述为：
  - 跨服务传递请求携带上下文(context)，数据隔离的流量路由方案。
- 利用服务发现注册租户信息，注册成特定的租户。





## Go 语言实践 - error

### Error Vs Exception

#### Eoor

- Go error就是普通的一个接口
  - skd/go/src/errors
- `errors.New()`来返回一个`error`对象
- 基础库中大量自定义的`error`
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220328160449.png" width="400"/>
- `errors.New()`返回的是内部`errorString`对象的指针
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220328160523.png" width="400"/>
  - 返回指针原因: 避免比较的时候比较字符串会相等。直接返回对象，比较的时候实际是挨个比较两个struct中的元素既string
  - 

#### Error vs Exception

- 各个语言
- Go 的处理异常逻辑是不引入exception，支持多参数返回，所以你很容易的在函数签名中带上实现了 error interface 的对象，交由调用者来判定。
- go的panic != exception。Go panic 意味着 fatal error(就是挂了)
- 使用多个返回值和一个简单的约定，Go 解决了让程序员知道什么时候出了问题，并为真正的异常情况保留了 panic。
- 对于真正意外的情况，那些表示不可恢复的程序错误，才使用 panic。对于其他的错误情况，我们应该是期望使用 error 来进行判定。
- Error模型的优势
  - 简单
  - 考虑失败，而不是成功
  - 没有隐藏的控制流
  - 完全交给你来控制error
  - Error are values

### Error Type

#### Sentinel Error

- 预定义的特定错误称为 Sentinel Error
- 使用一个特定值来表示不可能进行进一步处理的做法。对于 Go，我们使用特定的值来表示错误。
- 使用 sentinel 值是最不灵活的错误处理策略，因为调用方必须使用 == 将结果与预先声明的值进行比较。无法携带更多的上下文，携带上下文会导致破坏调用方 == 的相等性检查
- Sentinel errors 成为你 API 公共部分。
- Sentinel errors 在两个包之间创建了依赖。
- 结论: 尽可能避免 sentinel errors。

#### Error types

- Error type 是实现了 error 接口的自定义类型。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329134320.png" width="400"/>
- 调用者可以使用断言转换成这个类型，来获取更多的上下文信息。 
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329134644.png" width="400"/>
- 错误类型的一大改进是它们能够包装底层错误以提供更多上下文。
- 调用者要使用类型断言和类型 switch，就要让自定义的 error 变为 public。这种模型会导致和调用者产生强耦合，从而导致 API 变得脆弱。
- 尽量避免使用 error types，虽然错误类型比 sentinel errors 更好，因为它们可以捕获关于出错的更多上下文，但是 error types 共享 error values 许多相同的问题。

#### Opaque errors

- 不透明错误处理
- 要求代码和调用者之间的耦合最少，虽然知道有错误但是没有能力看到错误的内部。关于操作的结果，知道他起作用了或者没起作用即可。
- 只需返回错误而不假设其内容
- Assert errors for behaviour, not type
  - 可以断言错误实现了特定的行为，而不是断言错误是特定的类型或值
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329142111.png" width="400"/>


### Handing Error

#### 缩进流用于处理错误

- 无错误的正常流程代码，将成为一条直线，而不是缩进的代码。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329150347.png" width="400"/>

#### 通过消除错误来消除错误处理

- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329150834.png" width="300"/> --> <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220329150900.png" width="300"/>

#### 封装错误

- 日志记录与错误无关且对调试没有帮助的信息应被视为噪音，应予以质疑。记录的原因是因为某些东西失败了，而日志包含了答案。
  - 错误要被日志记录。
  - 应用程序处理错误，保证100%完整性。
  - 之后不再报告当前错误。
  - github.com/pkg/errors
- 通过使用 pkg/errors 包，您可以向错误值添加上下文，这种方式既可以由人也可以由机器检查。
  - `errors.Wrap`包装错误，携带堆栈信息  `errors.WithMessage`携带更多错误信息，不添加堆栈信息。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330105226.png" width="400"/>
  - `errors.Cause`获取原始错误信息
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330105347.png" width="400"/>
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330105414.png" width="400"/>
- 总结
  - 在许多项目中重复使用的软件包仅返回根错误值。
    - 选择 wrap error 是只有 applications 可以选择应用的策略。具有高可重用性的包只能返回根错误值。
  - 如果错误不会被处理，包装并返回堆栈信息。
    - 如果函数/方法不打算处理错误，那么用足够的上下文 wrap errors 并将其返回到调用堆栈中。
    - 例如，额外的上下文可以是使用的输入参数或失败的查询语句。确定您记录的上下文是足够多还是太多的一个好方法是检查日志并验证它们在开发期间是否为您工作。
  - 一旦错误被处理，它就不再被允许向上传递到调用堆栈。
    -  一旦确定函数/方法将处理错误，错误就不再是错误。
    -  如果函数/方法仍然需要发出返回，则它不能返回错误值。它应该只返回零(比如降级处理中，你返回了降级数据，然后需要 return nil)。

### Go 1.13 errors

#### Unwrap

- go1.13为 errors 和 fmt 标准库包引入了新特性，以简化处理包含其他错误的错误。
- 包含另一个错误的 error 可以实现返回底层错误的 Unwrap 方法。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330141114.png" width="500"/>
- go1.13 errors 包包含两个用于检查错误的新函数：Is 和 As。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330141724.png" width="500"/>
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330141742.png" width="500"/>

#### Wrapping errors with %w

-  Go 1.13中 fmt.Errorf 支持新的 %w 谓词。
   - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330141841.png" width="500"/>
-  用 %w 包装错误可用于 `errors.Is` 以及 `errors.As`:
   - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330142312.png" width="500"/>
-  %w类似pkg/errors
   - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330143323.png" width="500"/>

#### 使用 Is and As methods 自定义错误类型判断

- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220330143445.png" width="500"/>



## Go 语言实践 - concurrency

### Goroutine

#### Processes and Threads

- 操作系统会为应用程序创建一个进程。作为一个应用程序，它像一个为所有资源而运行的容器。这些资源包括内存地址空间、文件句柄、设备和线程。
- 线程是操作系统调度的一种执行路径，用于在处理器执行我们在函数中编写的代码。

#### Goroutines and Parallelism

- Concurrency is not Parallelism.  并发不是并行

#### Keep yourself busy or do the work yourself

- 使用空的 select 语句将永远阻塞。
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412162733.png" width="500"/>
- 如果你的 goroutine 在从另一个 goroutine 获得结果之前无法取得进展，那么通常情况下，你自己去做这项工作比委托它( go func() )更简单。

#### Leave concurrency to the caller （把并发交给调用者）

- 通常，将异步执行函数的决定权交给该函数的调用方通常更容易。

#### Never start a goroutine without knowning when it will stop （永远不要启动一个不知道什么时候结束的goroutine）

- 任何时候启动一个goroutine都要确认：
  - 它什么时候会停止
  - 怎样防止它终止
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412164911.png" width="500"/>。
  - 程序在两个不同的端口上提供 http 流量，端口8080用于应用程序流量，端口8001用于访问 /debug/pprof 端点。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412165044.png" width="500"/>。
  - 函数解耦，并确保 serveApp 和 serveDebug 将它们的并发性留给调用者。
  - 如果 serveApp 返回，则 main.main 将返回导致程序关闭
  - 但是 serveDebug 是在一个单独的 goroutine 中运行的，如果它返回，那么所在的 goroutine 将退出，而程序的其余部分继续运行。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412165343.png" width="500"/>。
  - log.Fatal 调用了 os.Exit，会无条件终止程序；defer 不会被调用到。
- 通过chan运行
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412172200.png" width="400"/>  
  - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220412172229.png" width="400"/>

#### Incomplete Work （不完整的工作）
- 无法保证创建的 goroutine 生命周期管理，会导致的问题，就是在服务关闭时候，有一些事件丢失。
- 使用 sync.WaitGroup 来追踪每一个创建的 goroutine。
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220418164455.png" width="600"/>
- 将 wg.Wait() 操作托管到其他 goroutine，owner goroutine 使用 context 处理超时。
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220418164601.png" width="600"/>
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220418164616.png" width="600"/>

### Memory model
#### Memory Reordering (内存重排)
- 为了提高读写内存的效率，会对读写指令进行重新排列，这就是所谓的 内存重排
- 还有编译器重排
- 在多核心场景下,没有办法轻易地判断两段程序是“等价”的。
- 三级缓存会导致数据写入 store buffer 而不写入 Memory 。
- 对于多线程的程序，所有的 CPU 都会提供“锁”支持，称之为 barrier，或者 fence。barrier 指令要求所有对内存的操作都必须要“扩散”到 memory 之后才能继续执行其他对 memory 的操作。

#### Memory model
- 在单一的独立的 goroutine 中先行发生的顺序即是程序中表达的顺序。
- 当多个 goroutine 访问共享变量 v 时，它们必须使用同步事件来建立先行发生这一条件来保证读操作能看到需要的写操作。 
- 注
    - 对变量 v 的零值初始化在内存模型中表现的与写操作相同。
    - 对大于 single machine word 的变量的读写操作表现的像以不确定顺序对多个 single machine word的变量的操作。

### Package sync
#### Share Memory By Communicating  (通过通信共享内存)
- 通常，共享数据结构由锁保护，线程将争用这些锁来访问数据。
- Go 没有显式地使用锁来协调对共享数据的访问，而是鼓励使用 chan 在 goroutine 之间传递对数据的引用。确保在给定的时间只有一个goroutine 可以访问数据。
- Do not communicate by sharing memory; instead, share memory by communicating.
    - 不要通过共享内存进行通信;相反，通过通信来共享内存。

#### Detecting Race Conditions With Go  (用Go检测竞态条件)
- `go build -race xxx.go`
- `go test -race xxx.go`  更推荐
- 解决原子赋值的问题，使用 Go 同步语义: Mutex
- struct 由 Type 和 Data 两部分组成，Type 指向实现了接口的 struct，Data 指向了实际的值。所以结构体的更新也不是原子的。
- 没有安全的 data race(safe data race)。您的程序要么没有 data race，要么其操作未定义。
    - 原子性
    - 可见行

#### sync.atomic
- Go 同步语义
    - Mutex
    - RWMutex
    - Atomic
- Mutex vs Atomic ，Mutex 相对更重。因为涉及到更多的 goroutine 之间的上下文切换 pack blocking goroutine，以及唤醒 goroutine。
- `go test -bench=.`
- Copy-On-Write(写时复制),指的是，写操作时候复制全量老数据到一个新的对象中，携带上本次新写的数据，之后利用原子替换(atomic.Value)，更新调用者的变量。来完成无锁访问共享数据。
    - 在微服务降级或者 local cache 场景中经常使用

#### Mutex
- Barging. 这种模式是为了提高吞吐量，当锁被释放时，它会唤醒第一个等待者，然后把锁给第一个等待者或者给第一个请求锁的人。
- Handsoff. 当锁释放时候，锁会一直持有直到第一个等待者准备好获取锁。它降低了吞吐量，因为锁被持有，即使另一个 goroutine 准备获取它。
    - 一个互斥锁的 handsoff 会完美地平衡两个goroutine 之间的锁分配，但是会降低性能，因为它会迫使第一个 goroutine 等待锁。
- Spinning. 自旋在等待队列为空或者应用程序重度使用锁时效果不错。Parking 和 Unparking goroutines 有不低的性能成本开销，相比自旋来说要慢得多。
- Go 1.8 使用了 Barging 和 Spining 的结合实现。当试图获取已经被持有的锁时，如果本地队列为空并且 P 的数量大于1，goroutine 将自旋几次(用一个 P 旋转会阻塞程序)。自旋后，goroutine park。在程序高频使用锁的情况下，它充当了一个快速路径。
- Go 1.9 通过添加一个新的**饥饿模式**来解决先前解释的问题，该模式将会在释放时候触发 handsoff。所有等待锁超过一毫秒的 goroutine(也称为有界等待)将被诊断为饥饿。当被标记为饥饿状态时，unlock 方法会 handsoff 把锁直接扔给第一个等待者。在饥饿模式下，自旋也被停用，因为传入的goroutines 将没有机会获取为下一个等待者保留的锁。
- 最晚加锁，最早释放
- 锁里面的代码越简单越好

#### errgroup
- https://pkg.go.dev/golang.org/x/sync/errgroup
- 核心原理: 利用 sync.Waitgroup 管理并行执行的 goroutine。
    - 并行工作流
    - 错误处理 或者 优雅降级
    - context 传播和取消
    - 利用局部变量+闭包
    - https://github.com/go-kratos/kratos/tree/master/pkg/sync/errgroup

#### sync.Pool
- sync.Pool 的场景是用来保存和复用临时对象，以减少内存分配，降低 GC 压力(Request-Driven 特别合适)。


### chan
#### Channels
- channels 是一种类型安全的消息队列，充当两个 goroutine 之间的管道
- 当创建的 chan 没有容量时，称为无缓冲通道。反过来，使用容量创建的 chan 称为缓冲通道。

#### Unbuffered Channels
- `ch := make(chan struct{})`
- 无缓冲 chan 没有容量，因此进行任何交换前需要两个 goroutine 同时准备好。
- 当 goroutine 试图将一个资源发送到一个无缓冲的通道并且没有goroutine 等待接收该资源时，。当 goroutine 尝试从无缓冲通道接收，并且没有 goroutine 等待发送资源时，该通道将锁住接收 goroutine 并使其等待。
- 无缓冲信道的本质是保证同步。
- 总结
    - Receive 先于 Send 发生。
    - 好处: 100% 保证能收到。
    - 代价: 延迟时间未知。

#### Buffered Channels
- 在 chan 创建过程中定义的缓冲区大小可能会极大地影响性能。
- 总结
    - Send 先于 Receive 发生。
    - 好处: 延迟更小。
    - 代价: 不保证数据到达，越大的 buffer，越小的保障到达。buffer = 1 时，给你延迟一个消息的保障。

#### Design Philosophy （设计原则）
- If any given Send on a channel CAN cause the sending goroutine to block
- If any given Send on a channel WON’T cause the sending goroutine to block
- Less is more with buffers.


### Package context
#### Request-scoped context (请求范围上下文)
- 在 Go 服务中，每个传入的请求都在其自己的goroutine 中处理。
- 如何将 context 集成到 API 中：
    - The first parameter of a function call （首参数传递）
        - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419170231.png" width="500"/>
    - Optional config on a request structure （请求结构上的可选配置）
        - 通过携带给定的 context 对象，返回一个新的 Request 对象。
        - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419170314.png" width="500"/>

#### Do not store Contexts inside a struct type (不要在结构类型内存储上下文)

#### context.WithValue
- context.WithValue 内部基于 valueCtx
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419170455.png" width="600"/>
- 为了实现不断的 WithValue，构建新的 context，内部在查找 key 时候，使用递归方式不断从当前，从父节点寻找匹配的 key，直到 root context(Backgrond 和 TODO Value 函数会返回 nil)。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419170538.png" width="600"/>

#### Debugging or tracing data is safe to pass in a Context (在上下文中传递调试或跟踪数据是安全的)
- context.WithValue 方法允许上下文携带请求范围的数据。这些数据必须是安全的，以便多个 goroutine 同时使用。
- 同一个 context 对象可以传递给在不同 goroutine 中运行的函数；上下文对于多个 goroutine 同时使用是安全的。对于值类型最容易犯错的地方，在于 context value 应该是 immutable 的，每次重新赋值应该是新的 context，即: context.WithValue(ctx, oldvalue)
- Context.Value should inform, not control
- 仅对运输进程和API的请求范围数据使用上下文值，而不是将可选参数传递给函数。
    - 比如 染色，API 重要性，Trace

#### context.WithValue
- 使用 copy-on-write 的思路，解决跨多个 goroutine 对一个 context 使用数据、修改数据的场景
- 使用WithCancel、WithDeadline、WithTimeout或WithValue替换上下文。

#### When a Context is canceled, all Contexts derived from it are also canceled （取消上下文时，派生的所有上下文也都已取消。级联取消）
- WithCancel(ctx) 参数 ctx 认为是 parent ctx，在内部会进行一个传播关系链的关联。
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419171255.png" width="500"/>
- Done() 返回 一个 chan，当我们取消某个parent context, 实际上上会递归层层 cancel 掉自己的 child context 的 done chan 从而让整个调用链中所有监听 cancel 的 goroutine退出。
    - <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419171348.png" width="500"/>
    - 每一层都要监控`done`， 做相应的退出操作

#### All blocking/long operations should be cancelable （所有阻塞操作都应该是可以被取消的）
- 如果要实现一个超时控制，通过上面的context 的parent/child 机制，其实我们只需要启动一个定时器，然后在超时的时候，直接将当前的 context 给 cancel 掉，就可以实现监听在当前和下层的额context.Done() 的 goroutine 的退出。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419171609.png" width="500"/>
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220419171655.png" width="500"/>

#### 总结
- 对服务器的传入请求应创建一个上下文。
- 对服务器的传出调用应该接受一个上下文。
- 不要将上下文存储在结构类型中； 相反，将 Context 显式传递给需要它的每个函数。
- 它们之间的函数调用链必须传播上下文。
- 使用 WithCancel、WithDeadline、WithTimeout 或 WithValue 替换上下文。
- 当一个上下文被取消时，所有从它派生的上下文也被取消。
- 相同的 Context 可以传递给在不同的 goroutine 中运行的函数； 上下文对于多个 goroutine 同时使用是安全的。
- 即使函数允许，也不要传递 nil 上下文。 如果您不确定要使用哪个上下文，请传递 TODO 上下文。
- 仅将上下文值用于传输流程和 API 的请求范围数据，而不用于将可选参数传递给函数。
- 所有阻塞/长操作都应该是可取消的。
- Context.Value 使程序的流程模糊。
- Context.Value 应该通知，而不是控制。
- 尽量不要使用 context.Value。

## 工程化实践

### 工程项目结构

#### Standard Go Projet Layout (标准Go项目布局)
- 参考“https://github.com/golang-standards/project-layout/blob/master/README_zh-CN.md”
- /internal 目录
    - 私有应用程序和库代码
    - internal包可以添加额外结构，以分隔共享和非共享的内部代码(/internal/app/myapp)
    - https://golang.org/doc/go1.4#internalpackages
- /pkg 目录
    - 外部应用程序可以使用的库代码
    - https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/

#### Kit Project Layout
- 为不同的微服务建立一个统一的kit工具包项目
- https://www.ardanlabs.com/blog/2017/02/package-oriented-design.html
- kit项目必须具备特点
    - 统一
    - 标准库方式布局
    - 高度抽象
    - 支持插件

#### Service Application Project Layout (服务端程序项目布局)
- /api
    - API协议定义目录，API协议放在同一个目录方便查找。xxx.proto protobuf文件，已经生成的go文件
- /configs
    - 配置文件模板或默认配置
- /test
    - 额外的测试脚本。测试数据集
    - Go会忽略以“.”或“_”开头的目录或文件
- 不应该包含: /src
- 组织结构
    - 一个gitlab的project里放置多个微服务的app
    - 按照gitlab的group建立多个project，每个project对应一个app
- app的服务类型
    - interface。即BFF，接受来自用户的请求
    - service。对内的微服务，仅接受来自内部其他服务或者网关的请求
    - admin。区别于service，面向运营侧，数据权限更高
    - job。流式任务处理的服务
    - task。定时任务，部署到task托管平台
- cmd应用目录负责程序的：启动、关闭、配置初始化等
- v1
    - model -> dao -> service -> api。model struct串联各个层，直到api需要做DTO对象转换
    - model目录。存储表一一映射
    - dao目录。数据读取层，数据库和缓存在这层统一处理。包括cache miss处理
    - service目录。数据组合，业务逻辑。
    - server目录。可以通过kit库提供快捷启动，干掉server层。
    - api目录。定义api
- DTO(Data Transfer Object)：数据传输对象。泛指用于展示层/API层与服务层(业务逻辑层)之间的数据传输对象。
- DO(Domain Object): 领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。
- V2
    - internal目录。为了避免同业务下有人跨目录引用内部struct
        - biz。业务逻辑的组装层，类似DDD的domain层，data类似DDD的repo，repo接口在这里定义，采用依赖倒置的原则。
        - data。业务数据访问。包含cache、db等封装没实现了biz的repo接口。data偏重业务的含义，他所做的是将领域对象重新拿出来，去掉了DDD的infra层。
        - service。实现了api定义的服务层，类似于DDD的application层，处理DTO到biz领域实体的转换（DTO->DO），协同biz交互，但是不应该处理复杂逻辑
    - PO(Persistent Object)。持久化对象
        - https://github.com/facebook/ent

#### Lifecycle (生命周期)
- 考虑服务应用对象初始化及生命周期的管理，所有HTTP/gRPC依赖的前置资源初始化。
- <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220509212310.png"/>
- 依赖注入核心：
    - 方便测试
    - 单次初始化和复用
- https://github.com/go-kratos/kratos/blob/main/app.go
- uber-go/fx

#### wire
- https://go.dev/blog/wire
- 手动控制资源的初始化和关闭繁琐易出错
- - https://github.com/google/wire <- 管理所有资源的依赖注入。

#### 工程化qs
- https://golang.org/doc/go1.4#internalpackages
- https://www.ardanlabs.com/blog/2017/02/package-oriented-design.html
- https://go.dev/blog/wire
- https://github.com/google/wire
- gogs
- 依赖倒置
- IOC控制反转 依赖注入
- DTO数据传输对象
- DO 领域对象  DTO -> DO
- 失血模型、贫血模型、充血模型
- https://github.com/go-kratos/kratos-layout
- java注解
- https://github.com/go-kratos/kratos/blob/main/transport/http/router.go


#### 扩展视野
- service mesh
- go内存模型"https://golang.org/ref/mem"
- 内存屏障实现
- https://cch123.github.io/ooo/ (memory reording)



#### 待看源码
- error group 源码
- kratos errorgroup
- consul源码
- euerka/discovery
- go context
- sync.pool
- nginx

#### 项目
- bilibili  overlord

#### 电子书
- unix环境高级编程
- Google SRE
- **effective go**
- 极客大学 丁琦 mysql



问题:
- grpc上公网ppt
- spring AOP
- mysql  double write buffer
- redis cow bgsave
- for循环空转优化  pause指令。  intel pause。  nginx for循环指令优化
- https://talks.golang.org/2014/gotham-context.slide#1



回答问题：
- 单体架构迁移微服务
  - 通过nginx rewrite已完成的微服务模块


技巧 :
- 查看底层汇编  `go tool compile -S xxx.go`