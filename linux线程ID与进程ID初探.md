---
title: linux 线程ID与进程ID 初探  
tags:
- linux
- 线程ID
- 进程ID
categories:
- Hulinan
- linux
- 进程
---
# 背景
关于Linux的进程ID，线程ID，提供了好些个接口，比如getpid()，比如pthread_self()，再比如网上描述的gettid，搞的感觉很复杂的样子，于是兴起了研究看看到底这些都代表什么意义的念头。
# 代码段
```C++
//本代码用于测试线程ID
//代码结果：主线程main和子线程PrintPid
//分别打印主线程和子线程的getpid、syscall(__NR_gettid) 和 pthread_self
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <sys/types.h>
#include <sys/syscall.h>

//子线程函数，用于打印子线程的资源
void* PrintPid(void * param)
{
	printf("child getpid() = %d, syscall(__NR_gettid) = %ld, pthread_self() = %ld\n",
	    		getpid(), (long int)syscall(__NR_gettid), pthread_self());
	while (1)
	{   
	        sleep(5);
	}   
}

int main()
{
	//打印主线程资源
	printf("main getpid() = %ld, syscall(__NR_gettid) = %ld, pthread_self() = %ld\n",
				 getpid(), (long int)syscall(__NR_gettid), pthread_self());
	pthread_t tid = 0;
	//创建子线程
	int ret = pthread_create(&tid, NULL, PrintPid, NULL);   
	if (ret == 0)
	{   
	        printf("child tid = %ld\n", tid);
	}   
	else
	        printf("create PrintPid thread failed!");
	while (1)
	{   
	        sleep(5);
	}    
	return 0;
}
```
<!-- more -->
上面这段代码保存为**_pid.cpp_**，并在linux下进行编译,生成**_pid_**的可执行文件：  

![image](http://ww3.sinaimg.cn/mw1024/825b97ebjw1fa1utct3jkj20e601ka9z.jpg)

我们来看一下执行结果：  
![image](http://ww3.sinaimg.cn/mw690/825b97ebjw1fa1ux1itz4j20j8027q30.jpg)  

从上图我们可以看出主线程和子线程调用的**getpid**返回的值相同，而**syscall(__NR_gettid)**的系统调用结果不同，**pthread_self**的调用结果也不相同。而子线程下的**pthread_self**返回值和主线程的**tid**值相同。  
我们通过 **top -Hp 23659** 命令查看该进程的具体线程PID:  
![image](http://ww2.sinaimg.cn/mw690/825b97ebjw1fa1vb1i8aoj20ez02baa2.jpg)  
我们看到上图得到的两个**PID**，与**syscall(__NR_gettid)**得到的主线程和子线程的返回值相同

# 详细解释

**getpid**  	函数原型： pid_t getpid(void)   
-- 解释：该函数返回当前进程的进程ID，即pid_t(int)型的进程识别码，通常情况下，我们使用此函数在当前进程下无论哪条线程下执行，都获取的是该进程的进程ID，同时它也是主线程的线程ID。此ID值也是我们采用 top 命令看到的主线程ID.  

**syscall(__NR_gettid)** 	系统调用，同syscall(SYS_gettid)  
-- 解释：该系统调用用于获取当前线程的线程ID，该线程ID也是我们使用 __top -Hp 进程ID__ 命令看到的线程PID值。 该系统调用在底层使用的是**gettid()**函数，而该函数在glibc下是不提供的。  

**pthread_self** 	函数原型： pthread_t pthread_self(void);
-- 解释：该函数获取线程自身的ID。通过使用man命令查看：  
``` The  pthread_self() function returns the ID of the calling thread.  This is the same value that is returned in *thread in the pthread_create(3) call that
 created this thread ```  
 以上的意思是pthread_self的返回值同pthread_create函数调用返回的pthread_t的值是一样的。这个我们从程序执行结果中可以得到验证(即 子线程下的**pthread_self**返回值和主线程的**tid**值相同)。  

# 疑问解答
1. 为什么syscall(__NR_gettid)和pthread_self()的返回值都是线程ID，但结果不同？  
回答：因为线程库实际由两部分组成：内核的线程支持+用户态的库支持(glibc), Linux早期内核不支持线程的时候glibc就在库中（用户态）以纤程（即用户态线程）的方式支持多线程了，POSIX thread只要求了用户编程的调用接口对内核接口没有要求。
linux上的线程实现就是在内核支持的基础上以POSIX thread的方式对外封装了接口，所以才会有两个ID的问题。

# 扩展阅读：   
<a href="http://www.cnblogs.com/0616--ataozhijia/p/4015465.html">1. pstree 以树状图显示进程间的关系</a>  
<a href="http://baike.baidu.com/link?url=gzfA36tkplxui_Rik9SKD4QGZlPyY9A1xxkJkrFXOHO6s4qPbrzNEwbtA2mSM40radgZVvVTYlui1y-Qi0exF_">2. getppid() 获取父进程ID</a>  
