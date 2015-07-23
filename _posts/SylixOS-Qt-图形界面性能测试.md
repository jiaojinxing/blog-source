title: SylixOS Qt 图形界面性能测试
date: 2015-07-17 21:18:29
tag: 
- SylixOS 
- 测试
categories: SylixOS
---

##测试目的
测试 SylixOS 上 Qt 的性能。

##测试环境
###硬件平台
硬件平台：飞凌嵌入式 OK335xS

处理器：AM335x(Cortex-A8, 800MHz)

L1-Cache：32KB I-Cache/32KB D-Cache

L2-Cache：256KB

内存：512MB

###操作系统
SylixOS + bspam335x + 按《SylixOS ARMv7A 处理器性能测试与改进》和《SylixOS C 库、数学库性能测试》两文优化后的进程补丁 libvpmpdm：2015/7/20 

对比测试操作系统 Linux：3.2.0（厂家配套的）

###编译器
Linux：

arm-arago-linux-gnueabi-gcc： gcc version 4.5.3 20110311 (prerelease) (GCC)（厂家配套的）
 
SylixOS：

arm-sylixos-eabi-gcc: gcc version 4.9.3 20150303 (release) [ARM/embedded-4_9-branch revision 221220] (
SylixOS Toolchain for ARM Embedded Processors) 

###Qt 库
Linux：

Qt-4.8.5（厂家配套的），编译选项：默认的 cpu 和 arch + -O2
 
SylixOS：

1. arm-sylixos-qt-4.8.6 编译选项： -march=armv4 -mno-unaligned-access -O2
2. armv7-sylixos-qt-4.8.6 编译选项： -march=armv7-a -mfloat-abi=softfp -mfpu=neon -mno-unaligned-access -O2

##测试软件

测试软件使用 [qtperf](https://github.com/jiaojinxing/qtperf "") 程序。

使用 Qt-Creator 编译 Release 版本的 qtperf 并部署到目标板的文件系统里。

##Linux 测试结果
qtperf 编译选项：
```
-mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O2
```

```
root@ok335x:/home/forlinx/qtperf# ./qtperf4 -qws &
[1] 1545
root@ok335x:/home/forlinx/qtperf# QLineEdit - 0.755 s
QComboBox - 4.08 s
QComboBoxEntry - 4.397 s
QSpinBox - 0.472 s
QProgressBar - 0.754 s
QPushButton - 0.333 s
QCheckbox - 0.218 s
QRadioButton - 0.564 s
QTextEdit add text - 2.98 s
QTextEdit scroll - 0.462 s
QPainter lines - 5.997 s
QPainter circles - 5.947 s
QPainter text - 6.363 s
QPainter pixmap - 6.224 s
Total: 39.546001 s
```

##SylixOS测试结果
###arm-sylixos-qt-4.8.6
qtperf 编译选项：
```
-march=armv4 -mno-unaligned-access -O2
```

```
[root@sylixos_station:/apps/qtperf]# ./qtperf4 -qws &
[root@sylixos_station:/apps/qtperf]# Connected to SylixOS FrameBuffer server: 800 x 480 x 32 282x169mm (72x72dpi)
Corrupt calibration data
QSylixOSInputMouseHandler: connected.
QSylixOSInputKeyboardHandler: connected.
QLineEdit - 0.943 s
QComboBox - 5.179 s
QComboBoxEntry - 5.723 s
QSpinBox - 0.649 s
QProgressBar - 0.955 s
QPushButton - 0.418 s
QCheckbox - 0.313 s
QRadioButton - 0.781 s
QTextEdit add text - 2.871 s
QTextEdit scroll - 0.482 s
QPainter lines - 4.633 s
QPainter circles - 4.598 s
QPainter text - 5.089 s
QPainter pixmap - 4.677 s
Total: 37.310997 s
```


###armv7-sylixos-qt-4.8.6
qtperf 编译选项：
```
-march=armv7-a -mfloat-abi=softfp -mfpu=neon -mno-unaligned-access -O2
```

```
[root@sylixos_station:/apps/qtperf]# ./qtperf4 -qws &
[root@sylixos_station:/apps/qtperf]# Connected to SylixOS FrameBuffer server: 80                                                                                                                                                             0 x 480 x 32 282x169mm (72x72dpi)
Corrupt calibration data
QSylixOSInputMouseHandler: connected.
QSylixOSInputKeyboardHandler: connected.
QLineEdit - 0.612 s
QComboBox - 3.506 s
QComboBoxEntry - 3.726 s
QSpinBox - 0.367 s
QProgressBar - 0.568 s
QPushButton - 0.258 s
QCheckbox - 0.185 s
QRadioButton - 0.484 s
QTextEdit add text - 2.434 s
QTextEdit scroll - 0.317 s
QPainter lines - 4.325 s
QPainter circles - 4.279 s
QPainter text - 4.7 s
QPainter pixmap - 4.301 s
Total: 30.062002 s
```

##测试结果汇总与对比
测试结果汇总如下表：

|      AM335x | Linux Qt-4.8.5 | arm-sylixos-qt-4.8.6 | armv7-sylixos-qt-4.8.6 |
|------------ |:------------------- |:------------------ |:------------------ |
|QLineEdit        | 0.755 s | 0.943 s | 0.612 s |
|QComboBox       | 4.08 s | 5.179 s | 3.506 s |
|QComboBoxEntry| 4.397 s | 5.723 s | 3.726 s |
|QSpinBox| 0.472 s | 0.649 s | 0.367 s |
|QProgressBar| 0.754 s | 0.955 s |  0.568 s |
|QPushButton| 0.333 s | 0.418 s | 0.258 s |
|QCheckbox| 0.218 s | 0.313 s | 0.185 s |
|QRadioButton| 0.564 s | 0.781 s | 0.484 s |
|QTextEdit add text| 2.98 s | 2.871 s | 2.434 s |
|QTextEdit scroll| 0.462 s | 0.482 s | 0.317 s |
|QPainter lines| 5.997 s |  4.633 s | 4.325 s |
|QPainter circles| 5.947 s |  4.598 s | 4.279 s |
|QPainter text| 6.363 s | 5.089 s | 4.7 s |
|QPainter pixmap| 6.224 s | 4.677 s | 4.301 s |
|Total| 39.546001 s | 37.310997 s | 30.062002 s |

armv7-sylixos-qt-4.8.6 的性能最好，arm-sylixos-qt-4.8.6 的总体性能比 Linux Qt-4.8.5 稍优，但 Linux Qt-4.8.5 大部分测试项目性能要优于 arm-sylixos-qt-4.8.6。

##测试结果对比分析
在测试 armv7-sylixos-qt-4.8.6 和 arm-sylixos-qt-4.8.6 时，使用了相同的运行环境（操作系统镜像、libvpmpdm.so、libcextern.so、xsiipc.ko、xinput.ko 均相同），所以 armv7-sylixos-qt-4.8.6 的性能优于 arm-sylixos-qt-4.8.6 的原因在于 Qt-4.8.6 被编译为更有高级的指令集（ARMv7A 和 NEON）。

同时 armv7-sylixos-qt-4.8.6 的原子量操作性能比 arm-sylixos-qt-4.8.6 的要好，因为 armv7-sylixos-qt-4.8.6 的原子量操作使用机器指令实现，而 arm-sylixos-qt-4.8.6 的使用 SylixOS API 实现。

Linux Qt-4.8.5 大部分测试项目性能要优于 arm-sylixos-qt-4.8.6，说明了 SylixOS 的 C 库尚有优化的空间。

##测试结论
armv7-sylixos-qt-4.8.6 的性能最好，建议客户在 ARMv7A 系列处理器（如 Cortex-A8 等）上使用 armv7-sylixos-qt-4.8.6。


