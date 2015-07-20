title: SylixOS 线程间通信-SylixOS信号量
date: 2015-06-25 17:10:03
tag: 
- SylixOS 
- 线程间通信
- 书稿
categories: 
- SylixOS 
---

SylixOS信号量有两种类型：二进制信号量和计数型信号量。

二进制信号量的取值范围为FALSE或TRUE；计数型信号量的最小取值为0，而最大取值在创建计数型信号量时决定。

二进制信号量主要应用在以下场合：
* 有允许线程访问的一个资源，使用二进制信号量作为互斥手段；
* 线程或中断通知另一个线程某种事件发生。
 
计数型信号量主要应用在以下场合：
* 有允许线程访问的n个资源，使用计数型信号量作为资源剩余计数；
* 线程或中断通知另一个线程某种事件发生，使用计数型信号量作为事件计数。

##SylixOS二进制信号量的操作函数##
###SylixOS二进制信号量的创建和删除###
```
LW_OBJECT_HANDLE Lw_SemaphoreB_Create(CPCHAR             pcName,
                                      BOOL               bInitValue,
                                      ULONG              ulOption,
                                      LW_OBJECT_ID     *pulId);
```
函数Lw_SemaphoreB_Create原型分析：
* 此函数返回二进制信号量的句柄，失败时为NULL；
* 参数pcName是二进制信号量的名字；
* 参数bInitValue是二进制信号量的初始值（FALSE或TRUE）；
* 参数ulOption是二进制信号量的创建选项；
* 输出参数pulId用于接收二进制信号量的ID。

```
ULONG               Lw_SemaphoreB_Delete(LW_OBJECT_HANDLE  *pulId);
```
函数Lw_SemaphoreB_Delete原型分析：
* 此函数返回错误号；
* 参数pulId是二进制信号量的ID。

二进制信号量的创建选项可以使用以下宏的组合：

|宏名	                    |含义|
|---                        |:---|
|LW_OPTION_WAIT_PRIORITY	|按优先级顺序等待|
|LW_OPTION_WAIT_FIFO	    |按先进先出顺序等待|
|LW_OPTION_OBJECT_GLOBAL	|全局对象|
|LW_OPTION_OBJECT_LOCAL	    |本地对象|

需要注意的是LW_OPTION_WAIT_PRIORITY和LW_OPTION_WAIT_FIFO只能二取一，同样LW_OPTION_OBJECT_GLOBAL和LW_OPTION_OBJECT_LOCAL也只能二取一。

###SylixOS二进制信号量的等待###
```
ULONG            Lw_SemaphoreB_Wait(LW_OBJECT_HANDLE  ulId, 
                                    ULONG               ulTimeOut);
ULONG            Lw_SemaphoreB_TryWait(LW_OBJECT_HANDLE  ulId); 
```
以上两个函数原型分析：
* 两个函数均返回错误号；
* 参数ulId是二进制信号量的ID；
* 参数ulTimeOut是等待的超时时间，单位为时钟嘀嗒Tick。

Lw_SemaphoreB_TryWait和Lw_SemaphoreB_Wait的区别在于，如果二进制信号量当前的值为FLASE，Lw_SemaphoreB_TryWait会立即退出，并返回
ERROR_THREAD_WAIT_TIMEOUT，而Lw_SemaphoreB_Wait则会阻塞直至被唤醒。

参数ulTimeOut除了可以使用数字外还可以使用以下的宏：

|宏名	                    |含义|
|---                        |:---|
|LW_OPTION_NOT_WAIT	        |不等待立即退出|
|LW_OPTION_WAIT_INFINITE	|永远等待|
|LW_OPTION_WAIT_A_TICK	    |等待一个时钟嘀嗒|
|LW_OPTION_WAIT_A_SECOND	|等待一秒|

Lw_SemaphoreB_Get和Lw_SemaphoreB_Take函数与Lw_SemaphoreB_Wait函数的原型及功能一致，这里不再详述。
Lw_SemaphoreB_TryGet和Lw_SemaphoreB_TryTake函数与Lw_SemaphoreB_TryWait函数的原型及功能一致，这里不再详述。

###SylixOS二进制信号量的释放###
```
ULONG            Lw_SemaphoreB_Post(LW_OBJECT_HANDLE  ulId);
```
函数Lw_SemaphoreB_Post原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID。

Lw_SemaphoreB_Give和Lw_SemaphoreB_Send函数与Lw_SemaphoreB_Post函数的原型及功能一致，这里不再详述。

```
ULONG            Lw_SemaphoreB_Release(LW_OBJECT_HANDLE  ulId, 
                                       ULONG               ulReleaseCounter, 
                                       BOOL                *pbPreviousValue);
```
函数Lw_SemaphoreB_Release原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 参数ulReleaseCounter是释放信号量的次数；
* 输出参数pbPreviousValue用于接收原先的信号量状态，可以为 NULL。

###SylixOS二进制信号量的清除###
```
ULONG            Lw_SemaphoreB_Clear(LW_OBJECT_HANDLE  ulId); 
```
函数Lw_SemaphoreB_Clear原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID。

###释放等待SylixOS二进制信号量的所有线程###
```
ULONG            Lw_SemaphoreB_Flush(LW_OBJECT_HANDLE  ulId, 
                                     ULONG            *pulThreadUnblockNum);
```
函数Lw_SemaphoreB_Flush原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 输出参数pulThreadUnblockNum用于接收被解锁（解除阻塞，下同）的线程数量，可以为NULL。

###获得SylixOS二进制信号量的状态###
```
ULONG            Lw_SemaphoreB_Status(LW_OBJECT_HANDLE   ulId,
                                      BOOL                 *pbValue,
                                      ULONG                *pulOption,
                                      ULONG                *pulThreadBlockNum);
```
函数Lw_SemaphoreB_Status原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 输出参数pbValue用于接收二进制信号量当前的值（FALSE或TRUE）；
* 输出参数pulOption用于接收二进制信号量的创建选项；
* 输出参数pulThreadBlockNum用于接收当前阻塞在该二进制信号量的线程数。

Lw_SemaphoreB_Info和Lw_SemaphoreB_Status函数的原型及功能一致，这里不再详述。

###获得SylixOS二进制信号量的名字###
```
ULONG  Lw_SemaphoreB_GetName(LW_OBJECT_HANDLE  ulId, PCHAR  pcName);
```
函数Lw_SemaphoreB_GetName原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 输出参数pcName是二进制信号量的名字，pcName应该指向一个大小为LW_CFG_OBJECT_NAME_SIZE的字符数组。

###带消息传递的SylixOS二进制信号量的等待与释放###
```
ULONG            Lw_SemaphoreB_WaitEx(LW_OBJECT_HANDLE  ulId, 
                                      ULONG             ulTimeOut, 
                                      PVOID            *ppvMsgPtr);
```
函数Lw_SemaphoreB_WaitEx原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 参数ulTimeOut是等待的超时时间，单位为时钟嘀嗒Tick；
* 输出参数ppvMsgPtr用于接收Lw_SemaphoreB_PostEx函数传递的消息。

Lw_SemaphoreB_GetEx和Lw_SemaphoreB_TakeEx函数与Lw_SemaphoreB_WaitEx函数的原型及功能一致，这里不再详述。

```
ULONG            Lw_SemaphoreB_PostEx(LW_OBJECT_HANDLE  ulId, 
                                      PVOID                pvMsgPtr); 
```
函数Lw_SemaphoreB_PostEx原型分析：
* 此函数返回错误号；
* 参数ulId是二进制信号量的ID；
* 参数pvMsgPtr是消息指针（一个空类型的指针，可以指向任意类型的数据），该消息将被传递到Lw_SemaphoreB_WaitEx函数的输出参数ppvMsgPtr。

Lw_SemaphoreB_GiveEx和Lw_SemaphoreB_SendEx函数与Lw_SemaphoreB_PostEx函数的原型及功能一致，这里不再详述。
由于Lw_SemaphoreB_WaitEx和Lw_SemaphoreB_PostEx函数组合已经起到传统RTOS的邮箱的作用，所以SylixOS没有提供邮箱的API。
