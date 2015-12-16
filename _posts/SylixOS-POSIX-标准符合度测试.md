title: SylixOS POSIX 标准符合度测试
date: 2015-07-17 21:17:37
tag: 
- SylixOS 
- 测试
categories: SylixOS
---

##测试目的
测试 SylixOS 对 IEEE1003（POSIX）标准的符合度。

##测试环境
###主机操作系统
Ubuntu-12.04

###硬件平台
硬件平台：飞凌嵌入式 OK335xS

处理器：AM335x(Cortex-A8, 800MHz)

L1-Cache：32KB I-Cache/32KB D-Cache

L2-Cache：256KB

内存：512MB

###操作系统
SylixOS + bspam335x

对比测试操作系统 Linux：3.2.0（厂家配套的）

###编译器
Linux：

arm-arago-linux-gnueabi-gcc： gcc version 4.5.3 20110311 (prerelease) (GCC) 
 
SylixOS：

arm-sylixos-eabi-gcc: gcc version 4.9.3 20150303 (release) [ARM/embedded-4_9-branch revision 221220] (
SylixOS Toolchain for ARM Embedded Processors) 

##测试软件

测试软件使用 posixtestsuite 

posixtestsuite 的官网: http://sourceforge.net/projects/posixtest/files/?source=navbar

posixtestsuite 测试分为三类：标准符合度测试 conformance、功能测试 functional、压力测试 stress。

移植好的 posixtestsuite 放在：

##SylixOS测试结果
第一次测试命令：

```
make -f Makefile 1>sylixos.log 2>sylixos_err.log 
```

将符合度结果记录在 sylixos.log 文件，将编译出错的信息记录在 sylixos_err.log 文件。

测试完毕后，查看 sylixos_err.log 文件，分析编译出错的原因，找出 posixtestsuite 的不合理测试所引致的问题，然后修改 posixtestsuite 的代码。

第二次测试命令：

```
make 1>sylixos_new.log 2>sylixos_err_new.log 
```

查看 sylixos_new.log 文件，统计通过测试的总数和不通过测试的总数，计算 SylixOS 对 IEEE1003（POSIX）标准的符合度。

备注：跳过链接的测试一般用于检测头文件和类型是否有定义，跳过链接的测试如果成功，则算为通过的测试，否则算为不通过的测试。

```
Search "PASS" (1728 hits in 1 file)
  D:\workspace_opensource\posixtestsuite\sylixos_new.log (1728 hits)
  
Search "SKIP" (233 hits in 1 file)
  D:\workspace_opensource\posixtestsuite\sylixos_new.log (233 hits)

Search "FAILED" (21 hits in 1 file)
  D:\workspace_opensource\posixtestsuite\sylixos_new.log (21 hits)
```

SylixOS 对 IEEE1003（POSIX）标准的符合度 = (1728 + 233) / (1728 + 233 + 21) = 0.9894046417759839 = 99%






##Linux测试结果