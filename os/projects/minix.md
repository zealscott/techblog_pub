<p align="center">
  <b>这学期的操作系统课我们使用Minix3.3进行教学。折腾了一上午，在此记录下自己安装和配置Minix的过程。</b>
</p>



# 系统介绍
Minix是一种基于微内核架构的类UNIX系统，与最受欢迎的Linux系统的最大区别就是：Linux是巨内核，Minix是微内核。Minix是由Andrew S. Tanenbaum大神发明的，其最初设想就是为了教学。由于Minix内核代码只有几千行，因此我们这学期也用它来学习操作系统。

Minix3.3已经增加了图形界面，但由于有不少bug，且从学习的角度，本文依然采用命令行的方式进行安装和使用。

# 环境配置
本文为Windows下的环境配置，Mac OS下的配置类似。



1. 需要先从[Minix官网][1]上下载iso镜像文件。
2. 下载[VMware][2]等其他虚拟机软件。
3. 下载[MobaXterm][3]等其他远程终端软件，便于从物理机上开多个窗口访问虚拟机。

# 安装Minix
1. 打开VMware，选择`Create a New Virtual Machine`，点击`Typical`,选择`I will install the Operating System later`，再1.  操作系统类型和版本都设置为 `other/unknown`，将将虚拟光驱路径设置为Minix镜像文件。设置内存大于512MB，硬盘大于4GB（确保足够资源），确保网络模式为NAT/网络地址转换(便于访问外网)。
2. 启动虚拟机，按照提示一路回车（使用默认配置）。
3. 输入shutdown -h now，关闭虚拟机。
4. 将虚拟机移除Minix镜像文件（否则每次启动都需要重新配置），重新启动。
5. 通过输入一些基本命令如ls，ps指令测试是否安装正确。

# 安装开发环境
1.  在线更新软件仓库元数据，输入 `pkgin update`  
2.  在线安装git版本控制器,输入  `pkgin install git-base`
3.  在线安装SSH,输入  `pkgin install openssh`
4.  在线安装VIM,输入  `pkgin install vim`
5.  在线安装clang编译器， 输入 `pkgin install clang`
6.  在线安装运行链接库，输入 `pkgin install binutils` 

# 通过SSH设置远程控制
直接在虚拟机中使用Minix中不太方便，一方面不能很好的与物理机进行交互，另外一方面是不能打开多个命令行窗口。因此，本文使用SSH进行连接和文件交换。
1. 将VMware中Minix的虚拟机的网络连接改为桥接（bridge）模式，这一步骤是为了让虚拟机拥有自己的IP地址。
2. 打开Minix，输入`ifconfig`，查看本机的IP地址。
3. 在命令行中输入`passwd root`，设置账号为`root`的密码（很奇怪为什么Minix没有初始密码）。注意，只有设置了密码后才能使用SSH。
3. 打开MobaXterm，选择使用SSH连接，输入账号和密码即可进入。
4. 可以使用MobaXterm方便的进行文件传输和远程控制。


# 使用FTP配置文件共享
虽然说使用了MobaXterm就没有必要再专门配置文件共享了，但我们最开始是使用FTP进行文件传输，既然掉进了这个坑，就还是记录下心得吧。
1. 下载`fiezilla`等ftp服务器。
2. 在fiezilla中，选择`Edit - Users`，添加账号，选择一个文件夹传输文件，配置好后点击ok完成。
3. 在Windows下打开命令行使用命令`ipconfig`查看当前电脑的IP地址，在`Windows文件框`中输入`ftp://user：password@IP地址`，若没有设置密码可以不填写。看是否能访问设置文件传输的文件夹。
4. 在Minix虚拟机中，登陆ftp客户端，输入`ftp 物理机ip`。
5. 输入`lcd 下载路径`选择文件传输后的下载路径。
6. 使用`ls`，`cd`等命令移动到物理机中所需的文件目录。
7. 输入`get + 文件名`，下载当前文件。
8. 使用`exit`退出ftp客户端，查看文件是否下载。


# 测试
到此时，Minix的开发环境就已经设置好了。你可以新建一个hello.c文件，放到Minix中进行测试。注意，Minix3已经不支持GCC，因此我们必须用Clang进行编译，使用`clang hello.c –o hello`看是否能成功编译。再输入`./hello`进行执行，查看结果。

[1]: http://www.minix3.org/
[2]: https://www.vmware.com/cn.html
[3]: https://mobaxterm.mobatek.net/
