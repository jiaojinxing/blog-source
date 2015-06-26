title: SylixOS for ARMv7-M
date: 2015-06-19 11:22:13
tag: 
- SylixOS 
- ARMv7-M
categories: 
- SylixOS 
---

最初 SylixOS 在 ARM7 上跑，后来更高级的 ARM 处理器开始变得流行，这些高级的 ARM 处理器一般带有 MMU(内存管理单元)和 Cache(高速缓存)。

SylixOS 为了发挥这些高级处理器特性(如 Cache)，支持 MMU 变得迫切，2008 年 SylixOS 开始支持页式虚拟内存管理和 MMU 及 Cache。

在很长的一段时间， SylixOS 都跑在这些带有 MMU 和 Cache 的 ARM 处理器，反而不能在不带有 MMU 的 ARM 处理器上跑。

随着 ARM 公司的 Cortex-M4 和 Cortex-M7 核心的发布，各大 MCU 芯片公司也推出了很多高性能的基于 Cortex-M4 和 Cortex-M7 核心的 MCU 芯片。

虽然这些 MCU 芯片不像 Cortex-A 系列处理器那样性能强大和带有 MMU，但它们的性能已经能满足一般的嵌入式应用。并且它们一般带有 MPU(内存保护单元)、VFP(矢量浮点处理单元)、大容量的片内 Flash、外扩内存(如 SDRAM)接口、DSP(数字信号处理器），甚至它们还带有 Cache。

SylixOS 组织意识到这可能是个趋势，2014 年年末我们重新检查了 SylixOS VMM(虚拟内存管理) 与 MMU 的配置宏，SylixOS 终于又支持不带 MMU 的处理器了。

由于 Cortex-M 系列核心的架构(ARMv7-M)和普通的 ARM 处理器有所差异，所以现有的 SylixOS ARM ARCH 层的代码并不能直接用于 Cortex-M 系列处理器上。

经过两天的努力，终于让 SylixOS ARM ARCH 层支持 ARMv7-M 架构，支持 ARMv7-M 架构的 SylixOS Base 工程放在这里：

https://github.com/jiaojinxing/sylixos-base-ARMv7m

有了支持 ARMv7-M 架构的 SylixOS Base 工程，剩下的工作就是针对一款 Cortex-M 系列的 MCU 芯片编写 BSP(板载支持包)了，

我们选择的是 Qemu 虚拟机支持的 Olimex STM32 p103 Dev Board(简称stm32-p103 开发板)，

Qemu 的地址：

https://github.com/beckus/qemu_stm32

BSP 的地址：

https://github.com/jiaojinxing/bspstm32

由于 SylixOS 默认的配置对内存的消耗比较大，所以我们修改了 Qemu 的代码，调整 stm32-p103 开发板的 Flash 空间大小为 8 MByte、SDRAM 空间大小为 64 MByte。

同时我们也优化了 SylixOS 的配置，极大地降低了 SylixOS 对内存和 Flash 的需求。

现在 Release 版本的 stm32-p103 开发板 BSP 生成的操作系统镜像其节区大小信息如下：
```
   text	   data	    bss	    dec	    hex	filename
1477064	 151188	1043428	2671680	 28c440	./Release/bspstm32.elf
```

也就是说，我们可以在只有 2MB Flash 和 2MB SDRAM 的 Cortex-M 系列处理器的目标系统运行功能完整的 SylixOS 操作系统了。

如果我们继续裁减掉一些不必要的组件，应该能将 Flash 和 SDRAM 的需求降低到 1MB 以内，甚至更少。

启动后，256 KB 大小的内核堆使用情况：
```
[unknown@sylixos_station:/]# free
heap show >>

     HEAP         TOTAL      USED     MAX USED  SEGMENT USED
-------------- ---------- ---------- ---------- ------- ----
kersys             262144     120856     160336     740  46%
 ```

堆栈消耗情况：
```
[unknown@sylixos_station:/]# ss
thread stack usage show >>

       NAME        TID   PRI STK USE  STK FREE USED
---------------- ------- --- -------- -------- ----
t_idle           4010000 255      144      880  14%
t_itimer         4010001  20      272     1776  13%
t_except         4010002   0      336     1712  16%
t_log            4010003  60      424     1624  20%
t_power          4010004 254      264     1784  12%
t_hotplug        4010005 250      344     3752   8%
t_reclaim        4010007 253      304     3792   7%
t_netjob         4010008 110      328     3768   8%
t_netproto       4010009 110      448     3648  10%
t_tftpd          401000a 160     2004     6188  24%
t_ftpd           401000b 160     1724    10564  14%
t_telnetd        401000c 160     1708     4436  27%
t_tshell         401000e 150     2408     5784  29%

interrupt stack usage show >>

CPU STK USE  STK FREE USED
--- -------- -------- ----
  0      228     1820  11%
```

内核对象消耗情况：
```
[unknown@sylixos_station:/]# cat /proc/kernel/objects
object      total    used     max-used
event       128      83       84
eventset    4        0        0
heap        4        1        1
msgqueue    10       6        6
partition   10       5        5
rms         4        1        1
thread      64       13       14
threadvar   4        0        0
timer       4        1        1
dpma        1        0        0
threadpool  1        0        0
```

来张 Qemu 和 Shell 的爆照：
![SylixOS-shell.png](/img/SylixOS-for-ARMv7-M/SylixOS-shell.png "")

下一步业余时间计划是移植到 STMicroelectronics 公司的 STM32F429I-DISCO 开发板，
在其上运行 JavaScript 脚本引擎，并尝试跑一些 IoT 协议接入到国内的物联云，这一定比较新奇和有趣，愿意一起折腾的可以联系我:-)

STM32F429I-DISCO 开发板用户手册：
http://www.st.com/st-web-ui/static/active/cn/resource/technical/document/user_manual/DM00092920.pdf

2015/6/20 日追加：

SylixOS 创始人韩辉也表示了极大的关注！SylixOS Lite 已经着手开发，SylixOS Lite 直接在 SylixOS 主线的基础上进行裁减，移除 GDB 调试、动态装载、符号表、SIGNAL 信号等对内存和 Flash 消耗较大的功能特性，但保留 POSIX 层，其它功能特性（如 IO 系统、文件、TCP/IP 网络）依然齐全。SylixOS Lite 的 .text 段会降低到 700K 以内，.data + .bss 段则会降低到 400K 以内，也就是说，在资源丰富的 Cortex-M4 和 M7 芯片上不外扩 SDRAM 也能跑 SylixOS 了！
