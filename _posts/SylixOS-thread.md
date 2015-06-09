title: SylixOS线程
tag: SylixOS
categories: SylixOS
---
##线程
线程是程序中的一条独立的控制流，线程是SylixOS调度器进行任务调度的基本单位，每个线程都有一个独一无二的ID号、栈、线程控制块（TCB）。

为了管理线程，SylixOS会为每一个线程分配一个线程控制块，用于记录该线程的信息，如线程入口函数、线程优化级、线程名字、线程当前的堆栈指针等。

线程入口函数可以调用其它函数，函数内定义的局部变量将从它自己的栈里分配，所以不同的线程可以有相同的线程入口函数，但又独立运行。

线程栈可以由用户自己分配，也可以操作系统代为分配，线程栈的大小不应改过少（容易造成栈溢出），也不应该过大（造成浪费）。

线程ID在线程被创建时决定，是线程的标识符。

线程优先级和调度策略是SylixOS调度器进行任务调度的参数，SylixOS是一个实时操作系统（RTOS），始终运行一个优先级最高的就绪态任务。

##线程的基本状态
线程的基本状态有如下几种：

1. 初始化          创建线程过程中线程的状态
2. 就绪            线程可以运行的状态
3. 等待            线程需要等行某些事件而不能继续的状态
4. 僵死状态        线程退出后处理的状态

![threadstatus.png](/img/SylixOS-thread/threadstatus.png "")

##线程优先级
线程优先级的范围处决于操作系统的配置，默认是0~255，0是最高优先级，255是最低优先级:

```
#define LW_PRIO_HIGHEST         0                                       /*  SylixOS 最高优先级          */
#define LW_PRIO_LOWEST          255                                     /*  SylixOS 最低优先级          */

/*********************************************************************************************************
    优先级 (一般应用的最高优先级不能高于 LW_PRIO_CRITICAL 最低不能低过 LW_PRIO_LOW)
*********************************************************************************************************/
     
#define LW_PRIO_EXTREME         LW_PRIO_HIGHEST                         /*  最高优先级                  */
#define LW_PRIO_CRITICAL        50                                      /*  关键处理任务                */
#define LW_PRIO_REALTIME        100                                     /*  实时处理任务                */
#define LW_PRIO_HIGH            150                                     /*  高优先级任务                */
#define LW_PRIO_NORMAL          200                                     /*  正常优先级                  */
#define LW_PRIO_LOW             250                                     /*  低优先级                    */
#define LW_PRIO_IDLE            LW_PRIO_LOWEST                          /*  最低优先级                  */
```

##线程调度策略
SylixOS支持同优先级线程，同优先级线程的调度策略取决于当前线程的调度策略，如果当前线程的调度策略为先来先服务（FIFO），

那么必须等当前线程的阻塞时，同优先级的其它线程才有机会运行；如果当前线程的调度策略为轮转（RR），

则当当前线程的时间片用完时或当前线程的阻塞时，同优先级的其它线程才有机会运行。

```
#define  LW_OPTION_SCHED_FIFO                           0x01            /*  调度器 FIFO                 */
#define  LW_OPTION_SCHED_RR                             0x00            /*  调度器 RR                   */
```

##线程相关 API
SylixOS 提供了一套 POSIX 线程 API，其接口定义和行为完全符合 POSIX 线程标准，方便了现有 POSIX 程序移植到 SylixOS 上，

关于 POSIX 线程的编程，市面上有很多书籍，网上也已经有很多资料，这里不再详述，POSIX 线程的编程可以参考这篇文章 https://computing.llnl.gov/tutorials/pthreads/

SylixOS 系统内部线程 API 的命名一般以 API_Thread 开始，API 的声明位于 libsylixos/SylixOS/kernel/include/k_api.h 文件，

为了适应不同语言习惯的人，也提供了一套以 Lw_ 开头的 API，请查看 libsylixos/SylixOS/api 目录下的头文件。

##内部线程 API 相关的数据类型

```
线程句柄  LW_OBJECT_HANDLE
线程 ID   LW_OBJECT_ID
线程属性  LW_CLASS_THREADATTR，*PLW_CLASS_THREADATTR
线程函数  PTHREAD_START_ROUTINE
```

其中 线程函数 PTHREAD_START_ROUTINE 的类型需要理解，它被定义为如下的函数指针：

```
typedef PVOID       (*PTHREAD_START_ROUTINE)(PVOID pvArg);              /*  系统线程类型定义            */
```

所以线程函数一般形如：

```
PVOID  thread_func (PVOID pvArg)
{
    // do thing...
    return  NULL;
}
```

##一分钟实验

```

#include <SylixOS.h> // 包含 SylixOS.h 头文件
  
/*
  * 测试用线程函数
 */
PVOID  TestThread (PVOID  pvArg)
{
    while (1) {
     		printf("hello in thread\n");
     		sleep(1);
    }
}
     
LW_OBJECT_HANDLE  hThread; // 定义线程句柄
      
hThread = API_ThreadCreate("t_test", TestThread, NULL, NULL); // 创建一个名为 t_test 的线程，线程函数为 TestThread，使用默认的线程属性，不接收线程 ID
```

