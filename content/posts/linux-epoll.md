---
title: "linux的epoll机制"
date: 2021-07-20T08:58:27+08:00
tags: ["linux", "epoll"]
draft: false
---

- IO多路复用
- epoll池原理

<!--more-->

### IO多路复用

- 实现形式：后台程序只需要1个就可以负责管理多个`fd`句柄，负责应对所有业务放的IO请求。这种一对多的IO模式叫做`IO多路复用`
  - 多路是指：多个业务方（句柄）并发下来的IO
  - 复用是指：复用这个一个后台处理程序

#### 1、最朴实的实现方式 - `for`循环

- 使用`for`循环，每次都尝试IO一下，读/写到了就处理，否则`sleep`

  - ```c
    while True:
        for each 句柄数组 {
            read/write(fd, /* 参数 */)
        }
        sleep(1s)
    ```

  - 存在问题：

    - 默认情况下，`create`出的句柄是阻塞类型的。读写数据的时候，如果数据没有准备好，会需要等待。所以上面代码第三行可能直接被卡死，导致整个线程都得不到运行。

  - 解决方案

    - 只需要把`fd`都设置成非阻塞的模式。这样`read/write`的时候，如果数据没有准备好，返回`EAGIN`的错误即可，不会卡住线程，从而整个系统就运转起来了。

  - 缺点：

    - `for`循环每次都要定期`sleep`，会导致吞吐能力极差。
    - 不加`sleep`会导致`CPU`一直告诉运转，浪费资源
    
  - 解决思路：
  
    - 求助内核提供机制协助。因为内核才能及时的管理这些事件的通知和调度。
    - 要把所有的时间都用在处理句柄的IO上，不能有任何空转、`sleep`的时间浪费。

#### 2、`Linux`内核机制

- 内核提供了3中工具`select`、`poll`、`epoll`
- 这三种都方式都能够管理`fd`的可读可写事件，在所有`fd`不可读不可写的时候，可以阻塞线程，切走`cpu`。`fd`有情况的时候，线程能够被唤醒。
- 这三种方式`epoll`池效率最高。`select`和`poll`都需要遍历知道`fd`，所以效率低。



### epoll 池原理

#### 1、epoll 涉及的系统调用

`epoll`涉及下面三个系统调用：

- ```shell
  epoll_create
  epoll_ctl
  epoll_wait
  ```

- `epoll_create`：负责创建一个池子，一个监控和管理句柄`fd`的池子

- `epoll_ctl`：负责管理这个池子里`fd`的增、删、改

- `epoll_wait`：负责等待，让出`CPU`调度，但是只有有事，立马会从这里唤醒

#### 2、epoll 高效的原理

- `epoll`的实现几乎没有做任何无效功

实现步骤：

1. `epoll`创建一个池子。使用`epoll_create`

   ```c
   // 原型
   int epoll_create(int size)
   // 示例
   epollfd = epoll_create(1024);
   if (epollfd == -1) {
       perror("epoll_create");
       exit(EXIT_FAILURE);
   }
   ```

   - 代码解释
     - `epollfd`：唯一代表这个`epoll`池
     - 用户可以创建多个`epoll`池

2. 往这个`epoll`池放`fd`。使用`epoll_ctl`

   ```c
   // 原型
   int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
   // 示例
   if (epoll_ctl(epollfd, EPOLL_CTL_ADD, 11, &ev) == -1) {
       perror("epoll_ctl: listen_sock");
       exit(EXIT_FAILURE);
   }
   ```

   - 代码解释

     - 上述操作把句柄`11`放入池子
     - `op(EPOLL_CTL_ADD)`表明操作时增加、修改、删除
     - `event`结构体可以指定监听事件类型，可读、可写

   - 高效问题1：添加`fd`进了池子，如果是修改、删除，怎样做到高效？

     - 红黑树。时间复杂度`O(log n)`

   - 高效问题2：怎样保证数据准备好后，立马感知？

     - 通过设置回调
     - `epoll_ctl`的内部实现中，除了把句柄用红黑树管理，还会设置`poll`回调

   - `poll`回调是什么？

     - `file_operations->poll`：定制监听事件的机制实现。通过`poll`机制让上层能告诉底层，我这个`fd`一旦读写就绪了，请底层硬件（比如网卡）回调的时候自动把这个`fd`相关的结构体放到指定队列中，并且唤醒操作系统。
     - 这个`poll`事件回调机制是`epoll`池高效最核心原理
     - `epoll`池管理的句柄只能是支持了`file_operations->poll`的文件`fd`。如果一个“文件”所在的文件系统没有实现`poll`接口，那么就用不了`epoll`机制

   - `poll`怎么设置？

     - 在`epoll_ctl`的实现中，有一步是调用`vfs_poll`，这个里面会有个判断，如果`fd`所在的文件系统的`file_operations`实现了`poll`，那么就会直接调用，否则报告相应的错误码

       ```c
       static inline __poll_t vfs_poll(struct file *file, struct poll_table_struct *pt)
       {
           if (unlikely(!file->f_op->poll))
               return DEFAULT_POLLMASK;
           return file->f_op->poll(file, pt);
       }
       ```

   - `poll`调用里面实现了什么？

     - 挂了个钩子，设置了唤醒的回调路径。`epoll`跟底层对接的回调函数是`ep_poll_callback`，这个函数做两件事：
       1. 把事件就绪的`fd`对应的结构体放到一个特定的队列（就绪队列，`ready list`）
       2. 唤醒`epoll`
     - 当`fd`满足可读可写的时候，就会经过层层回调，最终调用到这个回调函数，把`对应fd的结构体`放入就绪队列中，从而把`epoll`从`epoll_wait`唤醒
       - 结构体叫做`epitem`，每个注册到`epoll`池的`fd`都会对应一个
       - 就绪队列，因为没有查找需求，所以就绪队列使用最简单的双指针链表

   - 小结。`epoll`之所以做到了高效，最关键的两点：

     - 内部管理`fd`使用了高效的红黑树
     - `epoll`池添加`fd`的时候，调用`file_operations->poll`，把这个`fd`就绪之后的回调路径安排好。通过事件通知的形式，做到最高效的运行
     - `epoll`池核心的两个数据结构：红黑树和就绪列表。

     ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210720101222.png)

#### 3、哪些`fd`可以用`epoll`来管理

- 由于并不是所有的`fd`对应的文件系统都实现了`poll`接口，所以自然并不是所有的`fd`都可以放进`epoll`池，那么有哪些文件系统的`file_operations`实现了`poll`接口？

- 类似`ext2`、`ext4`、`xfs`这种常规的文件系统是没有实现的。最常见的、真正是文件的文件系统反倒是用不了`epoll`机制的

- 哪些支持

  - 最常见的就是网络套接字`socket`。网络也是`epoll`池最常见的应用地点。Linux下一些皆文件，`socket`实现了一套`socket_file_operations`的逻辑（`net/socket.c`）:

    ```c
    static const struct file_operations socket_file_ops = {
        .read_iter =    sock_read_iter,
        .write_iter =   sock_write_iter,
        .poll =     sock_poll,
        // ...
    };
    ```

    - `socket`实现了`poll`调用，所以`socket fd`是天然可以放到`epoll`池管理的

  - `eventfd`：`eventfd`实现非常简单，故名思义就是专门用来做事件通知用的。使用系统调用 `eventfd` 创建，这种文件`fd`无法传输数据，只用来传输事件，常常用于生产消费者模式的事件实现；

  - `timerfd`：这是一种定时器`fd`，使用 `timerfd_create` 创建，到时间点触发可读事件；

- 小结

  - ext2，ext4，xfs 等这种真正的文件系统的 fd ，无法使用 epoll 管理；
  - socket fd，eventfd，timerfd 这些实现了 poll 调用的可以放到 epoll 池进行管理；



### 总结

- IO 多路复用的原始实现很简单，就是一个 1 对多的服务模式，一个 loop 对应处理多个 fd 
- IO 多路复用想要做到真正的高效，**必须要内核机制提供**。因为 IO 的处理和完成是在内核，如果内核不帮忙，用户态的程序根本无法精确的抓到处理时机；
- **fd 记得要设置成非阻塞的哦**，切记；
- epoll 池通过高效的内部管理结构，并且结合操作系统提供的 **poll 事件注册机制**，实现了高效的 fd 事件管理，为高并发的 IO 处理提供了前提条件；
- epoll 全名 eventpoll，在 Linux 内核下以一个文件系统模块的形式实现，所以有人常说 **epoll 其实本身就是文件系统**也是对的；
- socketfd，eventfd，timerfd 这三种“文件”fd 实现了 poll 接口，所以网络 fd，事件fd，定时器fd 都可以使用 epoll_ctl 注册到池子里。我们最常见的就是网络fd的多路复用；
- **ext2，ext4，xfs 这种真正意义的文件系统反倒没有提供 poll 接口实现，所以不能用 epoll 池来管理其句柄。**那文件就无法使用 epoll 机制了吗？不是的，有一个库叫做 libaio ，通过这个库我们可以间接的让文件使用 epoll 通知事件





