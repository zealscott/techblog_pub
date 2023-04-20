
# 竞争条件

协同进程可能共享一些彼此都能够读写的公用存储区（例如打印机脱机系统），也可能是一些共享文件，当两个或多个进程读写某些共享数据，而最后的结果却取决于进程运行的精确时序，就称为**竞争条件（race condition）**。

如果程序之间有竞争条件，也许大部分的运行结果都很好，但在极少数情况下会发生一些难以解释的事情。

## 互斥（mutual exclusion）

要避免这种错误，关键是要找出某种途径防止多个进程在使用一个共享变量或文件时，其他进程不能做同样的事。

互斥的实现有很多种，在UNIX编程中，总体来说有两种大的方案：

1. **忙等待形式的互斥**
   - 优势在于被等待的进程（线程）不需要context switch（no extra overhead），提高了效率
   - 但若等待时间较长，会浪费CPU资源。
   - 会造成优先级反转问题（priority inversion problem）
2. **睡眠等待**
   - CPU利用率较高，但会造成context switch的overhead。

## 临界区

**把对共享内存进行访问的程序片段称为临界区或者临界段（critical region）。**

如果能够进行适当安排，使得两个进程不可能同时处于临界区，则能够避免竞争条件。

我们认为一个好的方案应该能解决竞争条件的同时，依然高效地进行操作，满足以下四个条件：

1. 任何两个进程不能同时处于临界区。
2. 不应该对CPU的速度和数目做任何假设。
3. 临界区外的进程不得阻塞其他进程。
4. 不能使进程在临界区外无休止的等待。

# 忙等待的互斥

## 关闭中断

这是最简单也最直接的方案，**使得每个进程在进入临界区后先关闭中断，在离开之前再打开中断。**

中断被关闭后，时钟中断也会关闭。因此CPU在做完临界区之前都不会发生进程切换。

**缺点**：

1. 把关闭中断的权利交给用户进程是不明智的。可能会造成系统终止。
2. 不适用于多CPU情形。
3. 在实际中很少采用。

关闭中断对于操作系统是一项很有用的技术，但对于用户进程不是一种合适的通用互斥机制。

## 锁变量

设想有一个共享锁变量，在进程想要进入临界区时，先测试这把锁。

但可以想象，如果锁变量依然是普通类型（不是原子类型），则依然会发生竞争条件。

## 严格交替法

首先看示意代码：

```C
/// process 0 ////
while(true){  
    while(turn!=0);  
    critical_region();  
    turn=1;  
    noncritical_region();  
}  
/// process 1 ////
while(true){  
    while(turn!=1);  
    critical_region();  
    turn=0;  
    noncritical_region();  
}  
```

process 0 必须在 turn 变量等于0时才会进入临界区，process 1 必须在 turn 变量等于1时才能进入临界区。

假设turn变量初始化为0，则process 1会一直持续地检测一个变量，直到为1才执行下面的代码。这种等待为**忙等待**。一个适用于忙等待的锁称为**自旋锁（spin lock）**。

仔细观察以上代码，两个进程互相依靠对方提供的turn变量才能继续下去。若一个进程的noncritical_region() 很长，另一个必须等它完成后才停止while循环。因此，**即使其代码在非临界区中，也会阻塞其他进程。**

因此，该方案违反了条件3：进程被一个临界区之外的进程阻塞，所以不能作为一个很好的方案。

## peterson解法

结合了锁变量和轮换法的思想：

```C
#define N 2  
#define FALSE 0  
#define TRUE 1  
int turn;  
int interest[N]={FALSE};//所有值初始为0   
void enter_region(int process){  
    interest[process] = TRUE;  
    turn = process;  
    int other = 1-process;  
    while(turn == process && interest[other] == TRUE);  
}  
void leave_region(int process){  
    interest[process] = FALSE;  
}  
```

我们主要关注enter_region可能发生的情况：

1. 一个进程进入后，没有进程中断它，那么一直执行，不会发生空转；
2. 若在`turn = process;  `之后执行了下一个线程，那么下一个线程不会空转，直接执行；

可以这样理解条件判断语句：

- 若`turn == process`不满足，则说明另一个进程一定处于等待中（本进程的interest为TRUE），因此可以进入临界区。
- 若`turn == process `满足而 `interest[other] == TRUE`不满足，则说明这时候没有其他进程在等待进入，因此可以进入临界区。

## TSL上锁

如果能有一种硬件解决方案，使得我们拿到了变量值，就不间断的更改这个值（原子操作），那将会是更有效的方法。

现在的计算机都支持这种方式，并有一个特殊的指令`TSL`。

> TSL RX, LOCK

将一个存储器字读到寄存器中，然后在该内存的地址上存一个非零值。

**读数和写数操作保证是不可分割的，即该指令结束之前其他处理机均不允许访问该存储器字。**

使用这条指令来防止两个进程进入临界区的方案如下：

![race1](/images/race1.png)

程序将LOCK原来的值复制到寄存器中，并将LOCK值置为1，随后这个原先的值与0比较。若非0，则说明之前已经上锁，从而程序一直空转。

并且，在现代操作系统中规定，**拿到锁的进程即使没有时间片也要立即执行**，这样会防止CPU的效率进一步下降。

## 优先级反转问题

例如H进程优先级高，L进程优先级低 ，假设L处于临界区中，H这时转到就绪态想进入临界区，H需要忙等待直到L退出临界区，但是H就绪时L不能调度，L由于不能调度无法退出临界区，所以H永远等待下去。 

因此，我们想知道，是否有其他方法，既能使CPU效率提高，也能解决优先级反转问题。

# 睡眠等待

考虑通信原语（primitive）：sleep 和 wakeup。

- sleep系统调用会引起进程阻塞，直到另一进程将其唤醒。
- wakeup调用即将被唤醒的进程。

## 生产者消费者问题

两个进程共享一个公共的固定大小的缓冲区，其中一个是生产者，负责将信息放入缓冲区；一个是消费者，负责从缓冲区中读取信息。但如果我们使用常规的count变量记录缓冲区数量时，还是会出现两个进程永远睡眠的情况：

```C
#define N 100  
int count = 0;  
void producer(void)  
{  
    int item;  
    while(TRUE)  
    {  
        item = produce_item();  
        if(count == N)                  //如果缓冲区满就休眠  
        sleep();  
        insert_item(item);  
        count = count + 1;              //缓冲区数据项计数加1  
        if(count == 1)  
        wakeup(consumer);  
    }  
}  
  
void consumer(void)  
{  
    int item;  
    while(TRUE)  
    {  
        if(count == 0)              //如果缓冲区空就休眠  
            sleep();  
        item = remove_item();  
        count = count - 1;          //缓冲区数据项计数减1  
        if(count == N - 1)  
            wakeup(producer);  
        consume_item(item);  
    }  
```



一种解决方案是增加一个**唤醒等待位**，当一个清醒的进程发送一个唤醒信号时，将该位置设为1；当程序要睡眠时，如果唤醒等待位为1，则清零，但不会睡眠。

### 信号量

引入一个整型变量来累计唤醒次数，称为信号量（semaphore）。一个信号量为非负数。

常用的为binary semaphore：down 和 up。

- down 操作是检查其值是否大于0，若为真，则减一，若为0，则进程将睡眠，并且，**检查数值，改变数值以及可能发生的睡眠操作是单一的原子操作（atomic action）**。
- up 操作是递增信号量的值。对于一个进程，若有睡眠的进程，则信号量执行一次up操作后，信号量依然为0，但在其上的睡眠进程减少一个（唤醒一个）。

```C
#define N 100
typedef int semaphore;
semaphore mutex = 1;
semaphore empty = N;
semaphore full = 0;
void producer(void)
{
	int item;
	while(TRUE)
	{
		item = produce_item();
		down(&empty);				//空槽数目减1，相当于P(empty)
		down(&mutex);				//进入临界区，相当于P(mutex)
		insert_item(item);			//将新数据放到缓冲区中
		up(&mutex);				//离开临界区，相当于V(mutex)
		up(&full);				//满槽数目加1，相当于V(full)
	}
}
void consumer(void)
{
	int item;
	while(TRUE)
	{
		down(&full);				//将满槽数目减1，相当于P(full)
		down(&mutex);				//进入临界区，相当于P(mutex)
		item = remove_item();	   		 //从缓冲区中取出数据
		up(&mutex);				//离开临界区，相当于V(mutex)		
		up(&empty);				//将空槽数目加1 ，相当于V(empty)
		consume_item(item);			//处理取出的数据项
	}
}
```

我们使用两种不同的方法来使用信号量。

- 信号量mutex用于互斥。保证任意时刻只有一个进程读写缓冲区和相关的变量
- 信号量full与empty用于保证一定的事件顺序发生或不发生。用于**同步**。

## 互斥（mutex）

若不需要信号量的计数能力，可以用于互斥：是一个处于两种变量之间（解锁和加锁）的变量。

适用于两个过程：

1. 当一个进程需要进入临界区时，调用mutex_lock，如果此时互斥是解锁的，那么调用进程可以进入临界区。
2. 若该互斥已经加锁，调用者被阻塞，等待在临界区中的进程完成操作并调用mutex_unlock退出为止。

# 多线程实现

这里以银行汇款为例子，讲解在竞争条件或互斥下的不同状态。若A和B都有10000元，考虑同时汇款的情形（thread1、thread2），我们可以写出以下代码：

```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

int sharedi = 0;
int A = 10000;
int B = 10000;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void thread1()
{
    int i = 0;
    for (i = 0; i < 10; i++)
    {
        //pthread_mutex_lock(&mutex);
        A += 100;
        sleep(1);
        B -= 100;
        printf("thread1: A  = %d , B = %d\n", A, B);
        //pthread_mutex_unlock(&mutex);
    }
}

void thread2()
{
    int i = 0;
    for (i = 0; i < 10; i++)
    {
        //pthread_mutex_lock(&mutex);
        A += 100;
        sleep(1);
        B -= 100;
        printf("thread2: A  = %d , B = %d\n", A, B);
        //pthread_mutex_unlock(&mutex);
    }
}

int main()
{
    int ret;
    pthread_t thrd1, thrd2;

    ret = pthread_create(&thrd1, NULL, (void *)thread1, NULL);
    ret = pthread_create(&thrd2, NULL, (void *)thread2, NULL);
    pthread_join(thrd1, NULL);
    pthread_join(thrd2, NULL);

    return 0;
}
```

若直接运行，我们会得到很奇怪的答案：

![race2](/images/race2.png)

会发现每一次汇款后，钱的总数不再等于20000，因此发生了竞争条件。

我们使用POSIX提供的mutex函数加锁后：

![race3](/images/race3.png)

这样就得到了我们想要的答案。



# 参考资料

1. [忙等待的互斥](https://blog.csdn.net/gettogetto/article/details/50594357)
2. Operating System:Design and Implementation,Third Edition 
3. [多线程的同步和互斥](https://www.cnblogs.com/fuyunbiyi/p/3475602.html)