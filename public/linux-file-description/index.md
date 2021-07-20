# 文件描述符 fd


- `fd`是什么
- Linux内核层面解释

<!--more-->

### fd 是什么

- `fd`是`File description`的缩写，中文名叫做：文件描述符。文件描述符是一个非负整数，**本质上是一个索引值**。

##### 什么时候获取`fd`？

- 当打开一个文件时，内核向进程返回一个文件描述符（`open`系统调用得到），后续`read`、`write`这个文件时，则只需要用这个文件描述符来标识该文件，将其作为参数传入`read`、`write`

##### `fd`的范围是什么？

- 在`POSIX`语义中，0、1、2这三个`fd`值已经被赋予特殊含义，分别是标准输入（`STDIN_FILENO`），标准输出（`STDOUT_FILENO`）、标准错误（`STDERR_FILENO`）
- 文件描述符是有一个范围的：`0 ~ OPEN_MAX-1`，现在的主流系统但就这个值来说，变化范围几乎不受限制，只受到系统硬件配置和系统管理员配置的约束
- 使用`ulimit`查看当前系统的配置

### Linux内核层面解释

- 用户使用系统调用`open`或者`create`来打开或者创建一个文件，用户态得到的结果值就是`fd`，后续的IO操作全都是用`fd`来标识这个文件。

#### `task_struct`

- 进程的抽象是基于`struct task_struct`结构体

  ```c
  struct task_struct {
      // ...
      /* Open file information: */
      struct files_struct     *files;
      // ...
  }
  ```

- `files`是一个指针，指向一个为`struct files_struct`的结构体。这个结构体用于管理该进程打来的所有文件的管理结构。

- `struct task_struct`是进程的抽象封装，标识一个进程，在Linux里面的进程各种抽象视角，都是这个结构体给到的。当创建一个进程，其实也就是`new`一个`struct task_struct`出来。

#### `files_struct`

- 这个结构体管理某些进程打开的所有文件的管理结构。

  ```c
  /*
   * Open file table structure
   */
  struct files_struct {
      // 读相关字段
      atomic_t count;
      bool resize_in_progress;
      wait_queue_head_t resize_wait;
  
      // 打开的文件管理结构
      struct fdtable __rcu *fdt;
      struct fdtable fdtab;
  
      // 写相关字段
      unsigned int next_fd;
      unsigned long close_on_exec_init[1];
      unsigned long open_fds_init[1];
      unsigned long full_fds_bits_init[1];
      struct file * fd_array[NR_OPEN_DEFAULT];
  };
  ```

- `files_struct`这个结构体是用来管理所有打开的文件的。本质上就是数组管理的方式，所有打开的文件结构都在一个数组里。数组在那里？有两个地方：

  1. `struct file * fd_array[NR_OPEN_DEFAULT]`是一个静态数组，随着`files_struct`结构体分配出来的，在64位系统上，静态数组大小为64；
  2. `struct fdtable fdtab;`也是数组管理结构，只不过是一个动态数组，数组边界是用字段描述的；

- 为什么会有静态数组 + 动态数组存储的方式？

  - 性能和资源的权衡。
  - 大部分进程机会打开少量的文件，所以静态数组就够了，这样不用另外分配内存。
  - 如果超过了静态数组的阈值，那么就动态扩展。

##### `fdtable`

- `fdtable`结构体就是封装用来管理`fd`的结构体：

  ```c
  struct fdtable {
      unsigned int max_fds;
      struct file __rcu **fd;      /* current fd array */
  };
  ```

- `fdtable.fd`字段是一个耳机指针，就是指向`fdtable.fd`的一个指针字段，指向的内存地址还是存储指针的。即，`fdtable.fd`指向一个数组，数组元素为指针（指针类型为`struct file *`）

- `max_fds`指明数组边界

##### `files_struct`小结

- `file_struct` 本质上是用来管理所有打开的文件的，内部的核心是由一个**静态数组**和**动态数组**管理结构实现。
- **`fd` 本质上就是索引**：**`fd` 就是这个数组的索引，也就是数组的槽位编号而已。**
- `fd` 真的就是 `files` 这个字段指向的指针数组的索引而已（仅此而已）。通过 `fd` 能够找到对应文件的 `struct file` 结构体；

##### `file`

- `fd`索引指向的元素为`struct file`结构体：

  ```c
  struct file {
      // ...
      struct path                     f_path;
      struct inode                    *f_inode;
      const struct file_operations    *f_op;
  
      atomic_long_t                    f_count;
      unsigned int                     f_flags;
      fmode_t                          f_mode;
      struct mutex                     f_pos_lock;
      loff_t                           f_pos;
      struct fown_struct               f_owner;
      // ...
  }
  ```

- 这个结构体非常重要，它标示一个进程打开的文件，IO相关的几个重要字段：

  - `f_path`：标识文件名；
  - `f_inode`：`inode`这个是 vfs 的`inode`类型，是基于具体文件系统之上的抽象封装；
  - `f_pos`：当前文件的偏移。`open`时设置成默认值，`seek`时可以更改，从而影响`read/write`的位置；

- 问题1：`files_struct`结构体只属于一个进程，那么`struct file`这个结构体呢？

  - `struct file`是属于系统级别的结构，可以共享于多个不用的进程。

- 问题2：什么时候出现多个进程的`fd`指向同一个`file`结构体？

  - `fork`的时候，父进程打开了文件，`fork`出一个子进程共享`file`的场景

- 问题3：同一个进程中，多个`fd`可以指向同一个`file`结构吗？

  - 可以。`dup`函数就是做这个的。

    ```c
    #include <unistd.h>
    int dup(int oldfd);
    int dup2(int oldfd, int newfd);
    ```

##### `inode`

- `struct file`结构体里面有一个`inode`指针。这个指向的`inode`并没有直接指出具体文件系统的`inode`，而是操作系统抽象出来的一层虚拟文件系统，叫做`VFS(Virtual File System)`，然后在 VFS 之下才是真正的文件系统，比如`ext4`等。

- 完整架构图

  ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210720152350.png)

- 为什么会有这一层封装呢？

  - 解耦
  - 如果让`struct file`直接和`struct exts_inode`这样的文件系统对接，会导致`struct file`的处理逻辑非常复杂，因为每对接一个具体的文件系统，就要考虑一种实现。
  - 所以，操作系统必须把底层文件系统屏蔽掉，对外提供统一的`inode`概念，对下定义好接口进行回调注册。这样让`inode`的概念得以统一，Unix一切皆文件的基础就来源于此。
  
- VFS 的`inode`结构

  ```c
  struct inode {
      // 文件相关的基本信息（权限，模式，uid，gid等）
      umode_t             i_mode;
      unsigned short      i_opflags;
      kuid_t              i_uid;
      kgid_t              i_gid;
      unsigned int        i_flags;
      // 回调函数
      const struct inode_operations   *i_op;
      struct super_block              *i_sb;
      struct address_space            *i_mapping;
      // 文件大小，atime，ctime，mtime等
      loff_t              i_size;
      struct timespec64   i_atime;
      struct timespec64   i_mtime;
      struct timespec64   i_ctime;
      // 回调函数
      const struct file_operations    *i_fop;
      struct address_space            i_data;
      // 指向后端具体文件系统的特殊数据
      void    *i_private;     /* fs or device private pointer */
  };
  ```

  - 包括了一些基本的文件信息，`uid`、`gid`、大小、模式、类型、时间等

  - 一个 vfs 和后端具体文件系统的纽带：`i_private`字段。用来传递一些集具体文件系统使用的数据结构。

  - `i_op`回调函数是在构造`inode`的时候，就注册成了后端的文件系统函数，比如`ext4`等

  - 问题：怎么通过 `vfs inode`得到具体文件系统的`inode`呢？ext4 的 inode 类型是 `struct ext4_inode_info` 。

    - 通过c语言常见的编程手法：强转类型。

    - 分配`inode`的时候，其实分配的是`ext4_inode_info`结构体，包含了`vfs inode`，然后对外给出去`vfs_inode`字段的地址即可。VFS 层拿`inode`的地址使用，底下文件系统强转类型后，取外层的`inode`地址使用。

      ```c
      static struct inode *ext4_alloc_inode(struct super_block *sb)
      {
          struct ext4_inode_info *ei;
      
          // 内存分配，分配 ext4_inode_info 的地址
          ei = kmem_cache_alloc(ext4_inode_cachep, GFP_NOFS);
      
          // ext4_inode_info 结构体初始化
      
          // 返回 vfs_inode 字段的地址
          return &ei->vfs_inode;
      }
      ```

    - `inode`的内存由后端文件系统分配，`vfs inode`结构体嵌在不同的文件系统的`inode`中。

    - 不同层次用不同的地址，`ext4`文件系统用`ext4_inode_info`的结构体的地址，vfs 层用`ext4_inode_info.vfs_inode`的地址

  - 问题：怎么理解 vfs `inode` 和 `ext2_inode_info`，`ext4_inode_info` 等结构体的区别？

    - 所有文件系统共性的东西抽象到 vfs `inode` ，不同文件系统差异的东西放在各自的 `inode` 结构体中。



### 小结梳理

当用户打开一个文件，用户只得到了一个 `fd` 句柄，但内核做了很多事情，梳理下来，我们得到几个关键的数据结构，这几个数据结构是有层次递进关系的，我们简单梳理下：

1. 进程结构 `task_struct` ：表征进程实体，每一个进程都和一个 `task_struct` 结构体对应，其中 `task_struct.files` 指向一个管理打开文件的结构体 `fiels_struct` ；

2. 文件表项管理结构 `files_struct` ：用于管理进程打开的 open 文件列表，内部以数组的方式实现（静态数组和动态数组结合）。返回给用户的 `fd` 就是这个数组的**编号索引**而已，索引元素为 `file` 结构；

3. - `files_struct` 只从属于某进程；

4. 文件 `file` 结构：表征一个打开的文件，内部包含关键的字段有：**当前文件偏移，inode 结构地址**；

5. - 该结构虽然由进程触发创建，但是 `file` 结构可以在进程间共享；

6. vfs `inode` 结构体：文件 `file` 结构指向 的是 vfs 的 `inode` ，这个是操作系统抽象出来的一层，用于屏蔽后端各种各样的文件系统的 `inode` 差异；

7. - inode 这个具体进程无关，是文件系统级别的资源；

8. ext4 `inode` 结构体（指代具体文件系统 inode ）：后端文件系统的 `inode` 结构，不同文件系统自定义的结构体，ext2 有 `ext2_inode_info`，ext4 有`ext4_inode_info`，minix 有 `minix_inode_info`，这些结构里都是内嵌了一个 vfs `inode` 结构体，原理相同；

   ![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210720162030.png)

##### 文件读写（IO）是时候发生了什么？

- 在完成 write 操作后，在文件 `file` 中的当前文件偏移量会增加所写入的字节数，如果这导致当前文件偏移量超处了当前文件长度，则会把 inode 的当前长度设置为当前文件偏移量（也就是文件变长）
- `O_APPEND` 标志打开一个文件，则相应的标识会被设置到文件 `file` 状态的标识中，每次对这种具有追加写标识的文件执行 `write` 操作的时候，`file` 的当前文件偏移量首先会被设置成 `inode` 结构体中的文件长度，这就使得每次写入的数据都追加到文件的当前尾端处（该操作对用户态提供原子语义）；
- 若一个文件 `seek` 定位到文件当前的尾端，则 `file` 中的当前文件偏移量设置成 `inode` 的当前文件长度；
- `seek` 函数值修改 `file` 中的当前文件偏移量，不进行任何 `I/O` 操作；
- 每个进程对有它自己的 `file`，其中包含了当前文件偏移，当多个进程写同一个文件的时候，由于一个文件 IO 最终只会是落到全局的一个 `inode` 上，这种并发场景则可能产生用户不可预期的结果；

### 总结

一切IO的行为到系统层面都是以`fd`的形式进行的。

- 用户`open`文件得到一个非负数句柄`fd`，之后对该文件的IO操作都是基于这个`fd`
- 文件描述符`fd`本质上来讲就是数组索引，`fd`等于5，那对应数组的第5个元素，高数组是进程打开的所有文件的数组，数据元素类型为`struct file`
- 结构体`task_struct`对应一个抽象的进程，`files_struct`是这个进程管理该进程打开的文件数组管理器。`fd`则对应了这个数组的编号，每一个打开的文件用`file`结构体表示，内含当前偏移等信息。
- `file`结构体可以为进程间共享，属于系统级资源，同一个文件可能对应多个`file`结构体，`file`内部有个`inode`指针，指向文件系统的`inode`
- `inode`是文件系统级别的概念，只由文件系统管理维护，不因进程改变（`file`是进程出发创建的，进程`open`同一个文件会导致多个`file`，指向同一个`inode`）










