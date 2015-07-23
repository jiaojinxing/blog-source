title: SylixOS C 库、数学库性能测试
date: 2015-07-17 21:19:24
tag: 
- SylixOS 
- 测试
categories: SylixOS
---

##测试目的
验证 SylixOS 的 C 库和数学库的某些函数是否存在性能问题。

找出 SylixOS 实时性远优于 Linux 和 RT-Linux（见《SylixOS实时性测试报告》，和 SylixOS 在 ARMv7A 性能优于 Linux （见[《SylixOS ARMv7A 处理器性能测试与改进》](http://jiaojinxing.github.io/2015/07/17/SylixOS-ARMv7A-%E5%A4%84%E7%90%86%E5%99%A8%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E4%B8%8E%E6%94%B9%E8%BF%9B/ "")），但 Qt 性能测试 [qtperf](https://github.com/jiaojinxing/qtperf "") 中部分项目不如 Linux 的原因，并提出解决办法。

##测试环境
###硬件平台
硬件平台：飞凌嵌入式 OK335xS

处理器：AM335x(Cortex-A8, 800MHz)

L1-Cache：32KB I-Cache/32KB D-Cache

L2-Cache：256KB

内存：512MB

###操作系统
SylixOS + bspam335x + 按《SylixOS ARMv7A 处理器性能测试与改进》一文优化后的进程补丁 libvpmpdm：2015/7/15 

对比测试操作系统 Linux：3.2.0（厂家配套的）

###编译器
Linux：

arm-arago-linux-gnueabi-gcc： gcc version 4.5.3 20110311 (prerelease) (GCC) 
 
SylixOS：

arm-sylixos-eabi-gcc: gcc version 4.9.3 20150303 (release) [ARM/embedded-4_9-branch revision 221220] (
SylixOS Toolchain for ARM Embedded Processors) 

##测试软件

测试软件使用 glibc-2.21 中的 benchtests 程序。

benchtests 有如下三个方面的测试：
1. 字符串类函数性能测试
2. 多线程 malloc 函数性能测试
3. 数学类函数性能测试

benchtests 覆盖测试了常用的 C 库和数学库函数。

由于厂家配套的 Linux 使用的 glibc 的版本为 2.12.2（不是 2.21），及 SylixOS 使用的是自已的实现 C 库，所以需要移植 benchtests，屏蔽了若干个 glibc-2.12.2 和 SylixOS 不支持的函数的测试。

benchtests 的编译选项统一使用 -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O2，由于 Linux 的文件系统已经放置了 C 库和数学库的动态库，为了避免使用这些可能不是 VFP 的动态库，所以 Linux 的 benchtests 使用静态链接（加入 -static 选项）。

移植好的 benchtests 放在 https://github.com/jiaojinxing/benchtest

##测试结果

Linux 性能测试结果放在 https://github.com/jiaojinxing/benchtests/tree/master/linux

SylixOS 性能测试结果放在 https://github.com/jiaojinxing/benchtests/tree/master/sylixos

##测试结果对比

Linux 与 SylixOS 字符串类函数性能测试结果对比放在 https://github.com/jiaojinxing/benchtests/tree/master/images

字符串类函数性能测试结果显示，SylixOS 和 Linux 各有千秋，我们将根据这个结果，对 SylixOS 的 C 库进行优化。

从完成数学类函数性能测试所耗时来看，SylixOS 远优化于 Linux，本质是 newlib 的 libm 的性能比 glibc 的 libm 的性能要好得多。

从完成多线程 malloc 函数性能测试所耗时来看，Linux 要优于 SylixOS，本质是 Linux 的 glibc 使用的内存分配算法要优化于 SylixOS 使用的，我们将使用更优秀的内存分配算法和更精细、快速的内存分配锁。

##SylixOS 的 C 库优化

规划中，暂略。

##SylixOS 内存分配算法优化

这篇文章做了一个常用内存分配算法的对比: http://webkit.sed.hu/blog/20100324/war-allocators-tlsf-action

常用内存分配算法有：

1. ptmalloc
2. tcmalloc
3. dlmalloc
4. jemalloc
5. hoard
6. tlsf
7. nedmalloc

[dlmalloc](http://g.oswego.edu/dl/ "") 由 Doug Lea 在 1987 年开发完成，这是 Android 系统中使用的内存分配器。dlmalloc 使用单 C 文件实现，约 5000 行，可移植性比较好。

[ptmalloc](http://www.malloc.de/ "") 在 dlmalloc 早期的基础上进行了改进，以更好地适应多线程和 SMP，这是 Linux 系统的 glibc 中使用的内存分配器。ptmalloc 与 glibc 绑定比较紧，可移植性比较差。

[nedmalloc](http://www.nedprod.com/programs/portable/nedmalloc/ "") 在 dlmalloc 的基础上加入线程 cache，实现了大多数情况下的无锁内存分配。nedmalloc 使用单 C 文件实现，约 7000 行，可移植性比较好。
 
[tcmalloc](https://github.com/gperftools "") 是 google 开发并开源出来的一款专为高并发而优化的内存分配器。tcmalloc 的 tc 含义是 thread cache，tcmalloc 正是通过 thread cache 这种机制实现了大多数情况下的无锁内存分配。tcmalloc 属于 google-perftools 性能分析工具中的一元，使用 C++ 实现，源代码有点多，可移植性一般。

[tlsf](http://tlsf.baisoku.org/ "") （Two Level Segregated Fit）优势是用两个位图优化合适内存的查找，实现时间复杂度为常数的内存分配，非常适合在实时系统中使用。使用单 C 文件实现，约 1000 行，可移植性非常好。tlsf 的不足之处在于在 32 位机器上，malloc 返回的指针是 4 字节对齐，而非 8 字节对齐。

[jemalloc](http://www.canonware.com/jemalloc/ "") 是 Jason Evans （FreeBSD 很有名的开发人员）在 2006 年为提高低性能的 malloc 而写的内存分配器。核心算法和 tcmalloc 类似，但 jemalloc 针对多核多线程进行了优化，在多核上的性能表现应该会 tcmalloc 更好。jemalloc 使用 C 实现，源代码有点多，可移植性一般。

将 dlmalloc、nedmalloc、tlsf、jemalloc 移植到 SylixOS 的进程补丁 libvpmpdm.so 上。

tlsf 并没有规定使用那种锁机制，可以使用自旋锁 spinlock 或互斥锁 mutex。

dlmalloc 和 nedmalloc 可以使用自旋锁 spinlock 或互斥锁 mutex。

jemalloc 使用互斥锁 mutex。

实测（客户的应用程序和 qtperf）四种内存分配算法速度性能表现如下：

dlmalloc（spinlock) > tlsf（spinlock) > tlsf（mutex) > dlmalloc(mutex) > nedmalloc(spinlock) > nedmalloc(mutex) > jemalloc > SylixOS 原生内存分配算法

所以将使用 dlmalloc（spinlock) 内存分配算法完成后续测试。





