
# 文件系统的基本概念

## 文件系统的基本要求

- 必须能够存储大量的信息。
- 在使用信息的进程终止时，信息必须保存下来。
- 多个进程可以并发地存取信息。

## 文件

### 文件命名

文件是一个抽象机制，它提供了一种把信息保存在磁盘上而且便于以后读取的方法。它必须这样来实现，使用户不必了解信息存储的方法、位置以及磁盘实际运作方式等细节。

### 文件结构

- 文件是**一个无结构字节序列**，Unix采用这种方式
- 文件是**一个固定长度记录的序列**，每条记录都有内部结构
- 文件由**一棵记录树构成**，每条记录长度不等。在记录的固定位置包含一个关键字域。记录树按关键字域进行排序，这便于对特定关键字进行快速查找。

### 文件类型

- **常规文件(regular file)**中包含有用户信息。
  - 是**ASCII文件或者二进制文件**。
  - ASCII文件由多行文本组成。
    - ASCII文件的最大优点是可以**原样地显示和打印，也可以用通常的文本编辑器进行编辑。**
  - 二进制文件，**无法直接查看**，但对于使用它的程序而言，其具有一定的内部结构。
- **目录(directory)**是管理文件系统结构的系统文件。
- **字符设备文件(character special file)**和输入/输出有关，用于处理串行I/O设备。
- **块设备文件(block special file)**则用于处理磁盘。

### 文件访问

- **顺序访问(sequential access)**
  - 进程可以从文件开始处顺序读取文件中所有字节或者记录，但不能够略过某些内容，也不能够非顺序读取。
  - 顺序存取文件可以重绕，只要需要，可以多次读取该文件。
  - 主要用于磁带
- **随机访问(random access)**
  - 可以非顺序地读取文件中的字节或记录，或者根据关键字而不是位置来存取记录
  - 主要用于磁盘

### 文件属性

每个文件除了文件名和数据本身之外，操作系统还给文件赋以其他信息，比如，文件创建日期、文件长度等等。

## 目录

为了记录文件，文件系统通常需要用到目录，在许多系统中，**目录本身也是文件。**

-  一个目录通常包含多个目录项，每个目录项代表一个文件
-  目录项包含了文件名字、属性和文件数据存储地址等信息
-  目录项仅包含文件名字、以及一个指向另一个数据结构的指针

### 目录系统

1. 每个系统的目录数目各不相同，最简单的设计方案是**维护一个单独的目录，其中包含所有用户的全部文件**
2. 对于整个系统中使用单独一个目录管理所有文件的想法的改进是，**每个用户拥有一个目录**
   - 这种设计消除了不同用户之间的文件名冲突，但仍然难以使那些有许多文件的用户感到满意（需要子目录）。
3. 更进一步，需要的是一般的层次结构(即目录树)。
   - 使用层次结构，每个用户可以拥有所需的多个目录，以便自然地组织他们的文件。

### 路径名

- 使用目录树来组织文件系统时，需要相应的方法指定文件名
- 每个文件都赋予一个**绝对路径名(absolute path name)**，它由从根目录到文件的路径组成。
- 另一种文件名是**相对路径名(relative path name)。**它常和工作目录(也称作当前目录)的概念一起使用。用户可以指定一个目录作为当前的工作目录。这时，所有的路径名，如果不是从根目录开始，都是相对于工作目录的。

# 文件系统实现

## 文件系统布局

- **主引导记录(MBR)**：磁盘的扇区0，用于启动计算机。
  - 在MBR未尾有一个**分区表**，里面记录了每个分区的**起始地址和结束地址**，其中有一个活动分区。
- 机器启动后，BIOS读入并执行MBR中的代码。
- MBR程序确定活动分区，并读入它的第一个磁盘块，即引导块，然后执行之。
- 引导块把保存在该分区中的操作系统装入内存并运行。
- 在类UNIX中，文件系统由**超级块**管理，它包含了关于文件系统的所有关键参数，当文件系统第一次被加载时，超级块的内容被装入内存。其后，是**空闲空间管理**，记录文件系统中的空闲物理块信息。之后是**索引节点**，每个索引节点对应于一个文件，记录了文件的属性信息及在磁盘上的存储地址。

## 文件实现

### 连续分配法

- 把每个文件作为连续数据块存储在磁盘上。例如，在具有1K大小块的磁盘上，50K的文件要分配50个连续的块。

**优点**

- 简单、容易实现，记录每个文件用到的磁盘块仅需记住一个数字即可，也就是第一块的磁盘地址。


- **性能较好，在一次操作中，就可以从磁盘上读出整个文件。**

**缺点**

- 首先，除非在文件创建时就知道了文件的最大长度，否则这一方案是行不通的。
- 该分配方案会造成**磁盘碎片**

### 链表分配法

- 为每个文件构造磁盘块的链接表，每个块的第一个字用于指向下一块的指针，块的其他部分存放数据。

**优点**

- 每个磁盘块都可以被利用，不会因为磁盘碎片而浪费存储空间；**目录项中只需记录一个整数(起始块号)**

**缺点**

- 尽管顺序读取文件非常方便，但是**随机存取却相当缓慢**。
- 此外，因为指针占去了一些字节，每个磁盘块存储数据的字节数不再是2的幂。可能造成性能损失

### 文件分配表法（FAT）

- 如果取出每个磁盘块的指针字，把它放在内存的表或索引中，就可以消除链接表法的两个不足，即**文件分配表法（FAT）**

**优点**

- 整个磁盘块都可以存放数据；随机存取容易；目录项中只需记录一个整数(起始块号) 。MS-DOS就使用这种方法进行磁盘分配。

**缺点**

- 主要缺陷是必须把整个链表都存放在内存中，**占用较多的存储空间。**

### 索引节点法

给每个文件赋予一张称为**i-node(索引节点)的数据结构**，其中列出了**文件属性和各块在磁盘上**的地址。

![file2](/images/file2.png)

## 目录实现

- 在读文件前，必须先打开文件。打开文件时，操作系统利用用户给出的路径名找到相应目录项，目录项中提供了查找文件磁盘块所需的信息


- 由于系统不同，这些信息可能是**整个文件的磁盘地址(连续分配方案)、第一个块的块号(对于两种链接表分配方案)或者是i-节点号。**

### 共享文件

**提供文件共享的方法：**利用多个目录中的不同文件名来描述同一共享文件（即文件别名，该方法的访问速度快，但会影响文件系统的树状结构，适用于经常访问的文件共享，同时存在一定的限制）。

#### 硬链接

基于改进的多级目录结构，将目录内容分为两部分：**文件名和索引结点**。

前者包括文件名和索引结点编号，后者包括文件的其他内容（包括属主和访问权限）。

例如：

```shell
ln source target ; rm source
```

通过多个文件名链接(link)到同一个索引结点，可建立同一个文件的多个彼此平等的别名。**别名的数目记录在索引结点的链接计数中，若其减至0，则文件被删除。**

限制：不能指向另一个文件系统中的i-node。

#### 软链接

一种特殊类型的文件，其内容是到另一个目录或文件路径名。建立符号链接文件(软链接)，**并不影响原文件，实际上它们各是一个文件。**可以建立任意的别名关系，甚至原文件是在其他计算机上。

```shell
ln -s /user/a /tmp/b
```

"cd /tmp/b ; cd .."是进入目录"/user"而不是"/tmp"；

当一个文件被删除时，相应的所有链接都无效。

## 磁盘空间管理

- 存储n个字节的文件可以有两种策略：分配n个字节的连续磁盘空间，或者把文件分成许多个(并不一定要)连续的块。


- 几乎所有的文件系统都把文件分割成固定大小的块来存储，各块不必相邻。

### 块大小

书中P352给出了块大小与访问速率、磁盘空间利用率的关系，可以发现，性能与空间利用率是相互冲突的。

![file3](/images/file3.png)

### 空闲块管理

1. 使用磁盘块的链接表
   - 磁盘块中包含尽可能多的空闲磁盘块号
2. 使用位图
   - n个块的磁盘需要n位位图。
   - 在位图中，空闲块用1表示，分配块用0表示(或者反之)。

# 文件系统的可靠性

## 备份

- 备份整个文件系统、还是部分备份
- 增量备份、常规备份
- 是否采取数据压缩
- 快照技术
- 存储介质的物理安全


## 备份策略

1. 物理转储

   当将磁盘的内容备份至磁带上时，从磁盘的第0个块开始，按照顺序把所有的磁盘块依次写到输出磁带中。

   - 优点：简单快捷
   - 缺点：备份了一些无用的信息(空闲块、坏块)，不支持增量备份、不能根据需要恢复特定的数据

2. 逻辑转储

   从一个或多个指定目录开始，对该目录下的每个文件和目录，进行备份，或以某个时间基点备份其文件和目录的修改数据

   - 优点：支持增量备份、避免备份无效的数据
   - 不足：相对物理转储，需要一定的技巧，以避免引入错误

## 文件系统的一致性

分为数据块和文件的一致性。

### 数据块

![file4](/images/file4.png)

对于数据块，可能出现快丢失（不严重），重复块（严重）。

### 文件

检验程序从根目录开始，沿着目录树递归下降，检查文件系统中的每个目录。对每个目录中的文件，其i节点对应的计数器加1。

当全部检查完成后，得到一张表，对应于每个i节点号，表中给出了指向这个i节点的目录数目。

#### i节点的链接数大于指向i节点的目录项个数

- 这一错误并不严重，可是却浪费了磁盘空间。即使所有的文件都被删除，文件链接数仍然为非0值，该i节点不会被删除。
- 措施：把i节点中的文件链接数设置成正确的值。

#### i节点的链接数小于指向i节点的目录项个数

- 该错误具有严重的潜在问题。如果两个目录项都链接到同一个文件，但其i节点的文件链接数只为1，如果删除任何一个目录项，i节点链接数变为0。文件系统将该i节点标志为“未使用”，并释放该文件的所有磁盘块。这将导致另一个文件数据丢失
- 措施：把i节点中的链接数设置为目录项的实际数目

# 文件系统性能

## 高速缓存

- 减少磁盘访问次数最常用技术是块高速缓存，其中，高速缓存是一些块，它们逻辑上属于磁盘，但基于性能的考虑而保存在内存中。
- 一般来说，使用一个哈希表查找需要的数据块，而使用LRU算法将所有块用双向链表连接起来。
- 高速缓存向磁盘的写入
  - 直写，即当数据块被修改，写高速缓存时亦立即写入磁盘中
  - 写回，当数据块被修改，只将其写到高速缓存中，之后再根据需要(需要被换出、关系一致性问题而特意保存、定时强制被保存)再写回到磁盘中

## 块预读

- 在数据块被访问之前，预先把它们读入高速缓存，从而提高高速缓存的命中率
-  适用于顺序访问文件，不适用于随机访问

## 减少磁头臂移动技术

- 将可能顺序访问的块存放在一起，最好在同一个柱面上，从而减少磁头臂的移动次数
  - 对于位图法管理空闲块的情况，在分配新的数据块时，尽量找与顺序访问时前一个块(已分配了)最邻近的空闲块
  - 对于链表法管理空闲块的情况，在分配新的块时，一次分配一组连续的块，在顺序访问亦可减少寻道时间
- 控制信息与数据信息的存放位置
  - 在使用i节点或者与i节点等价结构的系统中，另一个性能瓶颈在于，即使读取一个很短的文件也要访问两次磁盘：一次是读取i节点，另一次是读取文件块
  - 改进方法一：把i节点放在磁盘中部，在i节点和第一块之间的平均寻道时间减为原来的一半
  - 改进方法二：把磁盘分成多个柱面组，每个柱面组有自己的i节点、数据块和空闲表，在创建文件时，可以选取任一个i节点，然而分配块时，在该i节点所在的柱面组上进行查找

# 文件系统的安全性

## 安全环境

- 数据的机密性
  - 确保敏感数据处于机密状态下，保证未被授权的用户无法看到这些数据
- 数据的完整性
  - 未被授权的用户不能去修改任何数据，防止数据被篡改
  - 系统可用性
  - 确保系统的正常运行，防止拒绝服务攻击

## 恶意程序

- 病毒
  - 拒绝服务（大量消耗计算机资源）
- 蠕虫
  - 独立的程序，通过网络来传播
- 特洛伊木马
- 逻辑炸弹

# 保护机制

- 采用机制与策略分离
- 使用**参照监视器(reference monitor)**
- ![file5](/images/file5.png)
- **保护域** 
- **保护矩阵**
- **访问控制列表**（按列存储保护矩阵）
  - 主体：一个主动的实体，通常是用户和进程
  - 客体(对象)：一个被动的实体，通常是文件和资源
- **权能表(capability list)**或称C表（按行表存储保护矩阵）
- 秘密通道
  - 调节CPU使用强度
  - 调节页面率、文件锁
  - 申请和释放专用资源

# 参考资料

1. Operating System:Design and Implementation,Third Edition 