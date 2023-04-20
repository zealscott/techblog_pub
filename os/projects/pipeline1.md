<p align="center">
  <b>操作系统的第一个大作业是做一个简单的Shell，实现重定向、管道等功能。奋战了好几天终于基本搞定了= =</b>
</p>


# 基本要求
**Shell能够解析的命令行法如下：**


- 带参数的程序运行功能。

> program arg1 arg2 … argN 


- 重定向功能，将文件作为程序的输入/输出。
- “>”表示覆盖写

>program arg1 arg2 … argN  > output-file

- “>>”表示追加写

> program arg1 arg2 … argN  >> output-file

 - “<”表示文件输入

> program arg1 arg2 … argN  < input-file


- 管道符号“|”，在程序间传递数据(最后也可用重定向符号)

> programA arg1 … argN | programB arg1 … argN | programC …


- 后台符号& ,表示此命令将以后台运行的方式执行

> program arg1 arg2 … argN  &


- 工作路径移动命令cd


- Shell退出命令exit


- history显示开始任务后执行的命令；history n显示最近执行的n条指令 


# 基本思路
很明显本次实验主要是以考察Shell基本功能以及管道的实现为主。之前已经详细讲解了管道的实现，可以参考[这篇文章][1]。
## 熟悉命令
首先我们先在UNIX自带的Shell下实现重定向和管道功能，示例命令可以参考如下：



> \# ps &
>
> \# cat numbers.txt | sort > temp.txt
>
> \# sort < numbers.txt | grep 1 > a.txt
>
> \# ps -ef | grep \-sh 
>
> \# cd ..

我们不难发现：



1. “>”，“>>”重定向命令只能在命令中**出现一次**，一旦出现后，之后还有什么命令也是无效的。

2. “<”命令也只能出现一次，但是后面可以接管道命令。

3. “|”管道命令可以出现多次，且管道之后还可以使用重定向符号。

4. 实际上所有命令进入程序之后都是一串字符串，因此对字符串的解析是最重要的。

5. 对于**ps**，**ls**，**cd**等命令，可以使用**exceve**命令进行操作，并不需要我们自己实现。

6. 如果注意，可以发现系统Shell在实现后台进程时，可能会出现如下情况：
  - 我们让**ls**的结果在后台运行，但为什么会在结果前多一个“#”呢？
  - 原因是因为后台运行的子进程和前台运行的父进程同时进行，谁先谁后不能确定，图中就是父进程先运行，打印了“#”，子进程再打印**ls**的结果，因此出现了这种情况。


## 难点
1. 管道的实现以及**fork()**的使用。
2. 子父进程进行信号交互，以及回收僵尸进程。
3. 多文件的协调和编译。

# 大体框架
## 主函数入口
由于我们在Windows下写这个Shell无法编译，每次必须在UNIX下编译，因此必须在写之前就想好模块布局，不然很难debug和进行单元测试。
一个Shell其实就是一个**while(1)**的死循环，每次输出提示符到屏幕，然后执行输入的字符串命令。因此不难写出大体框架：
```C
    int main() {
    /*Command line*/
    while (1) {

        printf("cmd >");
        /*set buf to empty*/
        memset(buf, 0, sizeof(buf));
        /*Read from keyboard*/
        fgets(buf, MAXLINE, stdin);

        /*The function feof() tests the end-of-file indicator
        for the stream pointed to by stream,
        returning non-zero if it is set. */
        if (feof(stdin))
            exit(0);
        /*update the command history*/
        UpdateHistory();
        /*command exceve*/
        command();
    }
    return 0;
}
```

主程序的确很简单，就是每次用**buf**读取输入的字符串，然后更新输入列表（为了 **history**功能的实现），然后再解析命令（**command**）即可。

## 字符串命令存储方式
Shell主要就是对得到的命令进行操作，因此命令如何存储是至关重要的。最简单的想法就是用一个**char\*[]**字符串数组存储，但是我们后面对命令解析需要 **命令的下标**等其他信息，因此这里选择用**struct**进行存储更为方便。
定义结构体如下：
```C
struct CommandInfomation {
    char* argv[512]; /*store the command after Parsing*/
    int argc;        /*the number of argv,split with space*/
    int index;       /*store the index of special character*/
    int background;  /*whether it is a background command*/
    enum specify type[50];
    int override; /* in case after < command has muti pipes */
    char* file;
};
```

初始化函数为：
```C
void initStruct(struct CommandInfomation* a) {
    a->argc = 0;
    a->index = 0;
    a->background = 0;
    a->override = 0;
    a->file = NULL;
    memset(a->type, 0, sizeof(a->type));
}
```

## 特殊字符命令
对于重定向">"，管道"|"等特殊命令，我们需要使用额外的标识来注明，方便后面的操作。这里使用**eunm**实现。
```C
/*the enum stand for different command*/
enum specify {NORMAL, OUT_REDIRECT, IN_REDIRECT, OUT_ADD, PIPE};
```

## 主要函数详解
### pipe(fd[2])
此函数用于实现无名管道，将fd[2]数组中的两个文件描述符分别标记为管道读（fd[0]）和管道写（fd\[1\]）。
### dup(fd)
为复制文件操作符的系统函数，可以定向目前未被使用的最小文件操作符到fd所指的文件。相类似的函数还有**dup2[fd1,fd2]**,意思是 **未被使用**的文件描述符**fd2**
作为**fd1**的副本，进过此函数后，**fd1**和**fd2**都可访问同一个文件。
### execlp(const char \*file, const char \*arg, ...)
属于exec()函数族，会从PATH环境变量所指的目录中查找符合参数file的文件名，找到后便执行该文件，然后将第二个以后的参数当做该文件的argv[0]、argv[1]……，最后一个参数必须用空指针(NULL)作结束。
命令中的**ls**，**ps**等内置系统命令都可以由此函数进行解析。要注意，此函数一经调用就不会再返回。

### chdir(const char * path)
改变当前的工作路径以参数path所指的目录，使用比较简单，支持常用的改变路径的方式，例如退回上一级：**cd .. **，也支持绝对路径。

# 执行命令
由主函数可知，我们得到了命令需要进行解析，由于我们知道**exceve**函数一旦调用就不会返回，因此要使用**fork()**函数对其子进程进行处理。
这里需要注意的是，由于子进程一定要比父进程先结束，因此我们需要将执行的命令放到**子进程中**，父进程进行等待或者执行后面的命令，否则会出现父进程结束子进程还在运行的错误。
## 父子进程进行通讯
需要注意的是，Shell支持**后台程序运行**，因此，父进程不一定要等待子进程运行结束才做后面的事情，但这就涉及到子进程结束后，父进程需要回收僵尸进程。那么，如何做到这一点呢？
### Linux上进行信号屏蔽
在Linux系统上，我们可以使用**signal(int signum, sighandler_t handler)**函数来设置某一类的信号处理或者屏蔽。我们知道，子进程要**exit()**之前，会发送**SIGCHLD**信号给父进程，提醒父进程来回收子进程的退出状态和其他信息。
在这里，我们可以使用一个特殊的技巧：
> signal(SIGCHLD, SIG_IGN) 

这里是让父进程屏蔽子进程的信号，为什么这样就可以做到回收僵尸进程的作用呢？原来是因为在Linux中，**当我们忽略SIGCHLD信号时，内核将把僵尸进程交由init进程去处理，能够省去大量僵尸进程占用系统资源**。因此，屏蔽了子信号后，子程序在要结束时发送信号没人应答，内核就会认为这是一个孤儿进程，因此被init进程去回收，可以很好的解决我们面临的问题。 
### BSD系统上的信号处理
而笔者使用的是Minix3.3的系统，经过实测，内核并不会在父进程屏蔽信号后主动回收孤儿进程，因此不能使用这种方法。
那怎么办呢？因此只能自己写一个handler，规定父进程在收到子进程结束的信号后再wait，这样也可以实现此功能。但缺点就是**wait**函数需要阻塞父进程直到子进程结束为止，对于并发要求较高的并发服务器，可能就不是很适用。
我们使用这种方法完成后台程序的运行：
```C
    void SIG_IGN_handler(int sig)
    {
        waitpid(-1, NULL, 0);
        return;
    }
```
在主程序中install这个handler：
>signal(SIGCHLD, SIG_IGN_handler);

这样就完成了后台进程的功能。

## history功能实现
查找前n个命令是比较简单的功能，我们可以使用队列进行实现，在这里笔者就稍微偷懒一点，直接使用定长的字符串数组进行。
```C
/*update command history*/
void UpdateHistory()
{
    char *temp;
    if (strcmp(buf, "\n") == 0)
        return;
    if (HistoryIndex > MAXLINE)
        HistoryIndex = 0;
    temp = (char *)malloc(sizeof(buf));
    strcpy(temp, buf);
    CommandHistory[HistoryIndex++] = temp;
    return;
}


/*print the command with n lines*/
void PrintCommand(int n)
{
    int i,j=0;
    if (n == -1) {
        for (i = 0 ; i < HistoryIndex; i++)
            printf("the %d command: %s\n", i, CommandHistory[i]);
    }
    else {
        if (n > HistoryIndex) {
            printf("Warning: the argument is too large.\n");
            return;
        }
        for (i = HistoryIndex - n; i < HistoryIndex; i++)
            printf("the %d command: %s\n", ++j, CommandHistory[i]);
    }
}

```







# 参考资料
1. [linux信号函数signal][3]
2. Operating System:Design and Implementation,Third Edition 
3. Computer Systems: A Programmer's Perspective, 3/E

[1]: http://zealscott.com/2018/03/06/%E7%AE%A1%E9%81%93%E7%9A%84%E7%90%86%E8%A7%A3%E4%B8%8E%E5%AE%9E%E7%8E%B0/
[2]: http://www.cplusplus.com/reference/cstring/strtok/
[3]: http://blog.csdn.net/u013246898/article/details/52985739