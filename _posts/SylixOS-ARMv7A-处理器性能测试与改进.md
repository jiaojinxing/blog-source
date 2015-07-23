title: SylixOS ARMv7A 处理器性能测试与改进
date: 2015-07-17 21:24:31
tag: 
- SylixOS 
- 测试
categories: SylixOS
---

##测试目的
验证 SylixOS 是否发挥 ARMv7A Cache、VFP、NEON、分支预测性能，验证 BSP 是否在内存控制器、CPU 主频设置方面存在不正确的地方。

找出 SylixOS 实时性远优于 Linux 和 RT-Linux（见《SylixOS实时性测试报告》，但 Qt 性能测试 [qtperf](https://github.com/jiaojinxing/qtperf "") 不如 Linux 的原因，并提出解决办法。

##测试环境
###硬件平台
硬件平台：飞凌嵌入式 OK335xS

处理器：AM335x(Cortex-A8, 800MHz)

L1-Cache：32KB I-Cache/32KB D-Cache

L2-Cache：256KB

内存：512MB

###操作系统
SylixOS + bspam335x：2015/7/15 

对比测试操作系统 Linux：3.2.0（厂家配套的）

###编译器
Linux：

arm-arago-linux-gnueabi-gcc： gcc version 4.5.3 20110311 (prerelease) (GCC)（厂家配套的）
 
SylixOS：

arm-sylixos-eabi-gcc: gcc version 4.9.3 20150303 (release) [ARM/embedded-4_9-branch revision 221220] (
SylixOS Toolchain for ARM Embedded Processors) 

##测试软件
[nbench](http://www.tux.org/~mayer/linux/bmark.html "") 是一个简单的用于测试处理器、存储器性能的基准测试程序，即著名的 BYTE Magazine 杂志的 BYTEmark benchmark program。

nbench 在系统中运行并将结果和一台运行 Linux 的 AMD K6-233 电脑比较，得到的比值作为性能指数。

由于是完全开源的，爱好者可以在各种平台和操作系统上运行 nbench，并进行优化和测试，是一个简单有效的性能测试工具。

nbench 的结果主要分为 MEM、INT 和 FP，其中 MEM 指数主要体现处理器总线、CACHE 和存储器性能，INT 当然是整数处理性能，FP 则体现双精度浮点性能（大多数嵌入式处理器都没有强大的双精度浮点能力）。

```
Numeric sort - Sorts an array of long integers.

String sort - Sorts an array of strings of arbitrary length.

Bitfield - Executes a variety of bit manipulation functions.

Emulated floating-point - A small software floating-point package.

Fourier coefficients - A numerical analysis routine for calculating series approximations of waveforms.

Assignment algorithm - A well-known task allocation algorithm.

Huffman compression - A well-known text and graphics compression algorithm.

IDEA encryption - A relatively new block cipher algorithm.

Neural Net - A small but functional back-propagation network simulator.

LU Decomposition - A robust algorithm for solving linear equations.
```

使用版本：2.2.3

##Linux测试结果

###arm-arago-linux-gnueabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O3
```
root@ok335x:/home/forlinx# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :           395.2  :      10.14  :       3.33
STRING SORT         :          40.032  :      17.89  :       2.77
BITFIELD            :      1.3728e+08  :      23.55  :       4.92
FP EMULATION        :            67.8  :      32.53  :       7.51
FOURIER             :          1324.1  :       1.51  :       0.85
ASSIGNMENT          :          5.2366  :      19.93  :       5.17
IDEA                :           840.3  :      12.85  :       3.82
HUFFMAN             :          514.44  :      14.27  :       4.56
NEURAL NET          :            1.42  :       2.28  :       0.96
LU DECOMPOSITION    :          55.316  :       2.87  :       2.07
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 17.524
FLOATING-POINT INDEX: 2.143
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 4.129
INTEGER INDEX       : 4.565
FLOATING-POINT INDEX: 1.189
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.

```


###arm-arago-linux-gnueabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=neon -O3
```
root@ok335x:/home/forlinx# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          395.52  :      10.14  :       3.33
STRING SORT         :          34.784  :      15.54  :       2.41
BITFIELD            :      1.3741e+08  :      23.57  :       4.92
FP EMULATION        :          67.746  :      32.51  :       7.50
FOURIER             :          1324.1  :       1.51  :       0.85
ASSIGNMENT          :          5.3171  :      20.23  :       5.25
IDEA                :             840  :      12.85  :       3.81
HUFFMAN             :          514.24  :      14.26  :       4.55
NEURAL NET          :          1.4205  :       2.28  :       0.96
LU DECOMPOSITION    :          54.778  :       2.84  :       2.05
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 17.213
FLOATING-POINT INDEX: 2.136
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 3.961
INTEGER INDEX       : 4.564
FLOATING-POINT INDEX: 1.185
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

###arm-arago-linux-gnueabi-gcc -mcpu=cortex-a8 -O3
```
root@ok335x:/home/forlinx# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          394.88  :      10.13  :       3.33
STRING SORT         :          39.968  :      17.86  :       2.76
BITFIELD            :      1.3734e+08  :      23.56  :       4.92
FP EMULATION        :          67.773  :      32.52  :       7.50
FOURIER             :          1324.1  :       1.51  :       0.85
ASSIGNMENT          :          5.2469  :      19.97  :       5.18
IDEA                :             840  :      12.85  :       3.81
HUFFMAN             :          514.44  :      14.27  :       4.56
NEURAL NET          :          1.4015  :       2.25  :       0.95
LU DECOMPOSITION    :          55.196  :       2.86  :       2.06
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 17.522
FLOATING-POINT INDEX: 2.132
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 4.130
INTEGER INDEX       : 4.563
FLOATING-POINT INDEX: 1.183
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

###arm-arago-linux-gnueabi-gcc -mcpu=arm920t -O3
```
root@ok335x:/home/forlinx# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.08  :       9.90  :       3.25
STRING SORT         :          39.306  :      17.56  :       2.72
BITFIELD            :      1.1958e+08  :      20.51  :       4.28
FP EMULATION        :          61.975  :      29.74  :       6.86
FOURIER             :          1315.8  :       1.50  :       0.84
ASSIGNMENT          :          5.5866  :      21.26  :       5.51
IDEA                :           654.5  :      10.01  :       2.97
HUFFMAN             :          501.35  :      13.90  :       4.44
NEURAL NET          :          1.4306  :       2.30  :       0.97
LU DECOMPOSITION    :          55.835  :       2.89  :       2.09
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 16.361
FLOATING-POINT INDEX: 2.151
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 4.005
INTEGER INDEX       : 4.142
FLOATING-POINT INDEX: 1.193
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.

```

###arm-arago-linux-gnueabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O2
```
root@ok335x:/home/forlinx# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          356.64  :       9.15  :       3.00
STRING SORT         :          37.949  :      16.96  :       2.62
BITFIELD            :      8.5789e+07  :      14.72  :       3.07
FP EMULATION        :          32.934  :      15.80  :       3.65
FOURIER             :          1303.5  :       1.48  :       0.83
ASSIGNMENT          :          5.2346  :      19.92  :       5.17
IDEA                :           646.3  :       9.88  :       2.93
HUFFMAN             :          490.39  :      13.60  :       4.34
NEURAL NET          :          1.5157  :       2.43  :       1.02
LU DECOMPOSITION    :          55.698  :       2.89  :       2.08
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 13.826
FLOATING-POINT INDEX: 2.184
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 3.467
INTEGER INDEX       : 3.437
FLOATING-POINT INDEX: 1.211
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.

```

##SylixOS测试结果

###arm-sylixos-eabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O3
```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.53  :       9.91  :       3.26
STRING SORT         :          12.447  :       5.56  :       0.86
BITFIELD            :      1.4006e+08  :      24.03  :       5.02
FP EMULATION        :          88.258  :      42.35  :       9.77
FOURIER             :          1591.8  :       1.81  :       1.02
ASSIGNMENT          :          6.4511  :      24.55  :       6.37
IDEA                :          958.43  :      14.66  :       4.35
HUFFMAN             :          619.13  :      17.17  :       5.48
NEURAL NET          :          1.6131  :       2.59  :       1.09
LU DECOMPOSITION    :          59.127  :       3.06  :       2.21
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 16.595
FLOATING-POINT INDEX: 2.431
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 3.019
INTEGER INDEX       : 5.249
FLOATING-POINT INDEX: 1.348
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.

```

###arm-sylixos-eabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=neon -O3
```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.79  :       9.92  :       3.26
STRING SORT         :           12.32  :       5.50  :       0.85
BITFIELD            :      1.4009e+08  :      24.03  :       5.02
FP EMULATION        :          84.482  :      40.54  :       9.35
FOURIER             :          1633.5  :       1.86  :       1.04
ASSIGNMENT          :          6.7122  :      25.54  :       6.62
IDEA                :          958.12  :      14.65  :       4.35
HUFFMAN             :          615.79  :      17.08  :       5.45
NEURAL NET          :          1.6196  :       2.60  :       1.09
LU DECOMPOSITION    :          59.124  :       3.06  :       2.21
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 16.549
FLOATING-POINT INDEX: 2.455
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 3.049
INTEGER INDEX       : 5.185
FLOATING-POINT INDEX: 1.362
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

###arm-sylixos-eabi-gcc -mcpu=cortex-a8 -O3
```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.91  :       9.92  :       3.26
STRING SORT         :          12.429  :       5.55  :       0.86
BITFIELD            :      1.4009e+08  :      24.03  :       5.02
FP EMULATION        :          89.179  :      42.79  :       9.87
FOURIER             :          424.47  :       0.48  :       0.27
ASSIGNMENT          :          6.5648  :      24.98  :       6.48
IDEA                :          958.47  :      14.66  :       4.35
HUFFMAN             :           565.9  :      15.69  :       5.01
NEURAL NET          :         0.53259  :       0.86  :       0.36
LU DECOMPOSITION    :          16.094  :       0.83  :       0.60
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 16.448
FLOATING-POINT INDEX: 0.701
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 3.035
INTEGER INDEX       : 5.147
FLOATING-POINT INDEX: 0.389
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

###arm-sylixos-eabi-gcc -mcpu=arm920t -O3
```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          400.44  :      10.27  :       3.37
STRING SORT         :          12.422  :       5.55  :       0.86
BITFIELD            :      1.3164e+08  :      22.58  :       4.72
FP EMULATION        :          81.987  :      39.34  :       9.08
FOURIER             :          411.76  :       0.47  :       0.26
ASSIGNMENT          :          6.3944  :      24.33  :       6.31
IDEA                :          1027.1  :      15.71  :       4.66
HUFFMAN             :          559.44  :      15.51  :       4.95
NEURAL NET          :         0.52203  :       0.84  :       0.35
LU DECOMPOSITION    :          15.716  :       0.81  :       0.59
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 16.258
FLOATING-POINT INDEX: 0.684
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 2.946
INTEGER INDEX       : 5.157
FLOATING-POINT INDEX: 0.379
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

###arm-sylixos-eabi-gcc -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O2
```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          335.32  :       8.60  :       2.82
STRING SORT         :          11.952  :       5.34  :       0.83
BITFIELD            :      1.4737e+08  :      25.28  :       5.28
FP EMULATION        :          55.498  :      26.63  :       6.15
FOURIER             :          1555.4  :       1.77  :       0.99
ASSIGNMENT          :          6.5063  :      24.76  :       6.42
IDEA                :          1042.9  :      15.95  :       4.74
HUFFMAN             :          612.91  :      17.00  :       5.43
NEURAL NET          :          1.5783  :       2.54  :       1.07
LU DECOMPOSITION    :          59.134  :       3.06  :       2.21
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 15.422
FLOATING-POINT INDEX: 2.395
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 3.038
INTEGER INDEX       : 4.596
FLOATING-POINT INDEX: 1.328
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

##Linux SylixOS 测试结果汇总与对比

|Linux AM335x | cortex-a8 vfpv3 -O3 | cortex-a8 neon -O3 | cortex-a8 -O3 | arm920t -O3 | cortex-a8 -vfpv3 -O2 |
|------------ |:------------------- |:------------------ |:------------- |:----------- | :------------------- |
|MEMORY INDEX        | 4.129 | 3.961 | 4.130 | 4.005 | 3.467 |
|INTEGER INDEX       | 4.565 | 4.564 | 4.563 | 4.142 | 3.437 |
|FLOATING-POINT INDEX| 1.189 | 1.185 | 1.183 | 1.193 | 1.211 |

不同 FPU 编译选项，浮点性能差异极少，怀疑 arm-arago-linux-gnueabi-gcc 默认就打开了硬件浮点。

|SylixOS AM335x | cortex-a8 vfpv3 -O3 | cortex-a8 neon -O3 | cortex-a8 -O3 | arm920t -O3 | cortex-a8 -vfpv3 -O2 |
|-------------- |:------------------- |:------------------ |:------------- |:----------- | :------------------- |
|MEMORY INDEX        | 3.019 | 3.049 | 3.035 | 2.946 | 3.038 |
|INTEGER INDEX       | 5.249 | 5.185 | 5.147 | 5.157 | 4.596 |
|FLOATING-POINT INDEX| 1.348 | 1.362 | 0.389 | 0.379 | 1.328 |

不同 FPU 编译选项，浮点性能差异较大，说明 arm-sylixos-eabi-gcc 默认不打开了硬件浮点。

SylixOS 的 INTEGER INDEX （定点处理性能）与 FLOATING-POINT INDEX （浮点处理性能）要优于 Linux，而 MEMORY INDEX （内存性能）要低于 Linux。

**值得注意：使用 -mfpu=neon 编译选项不见得比 -mfpu=vfpv3 要好，频繁地切换 ARM/NEON 指令集反而会使性能有所损耗。
在 Cortex-A8 处理器上 NEON 单元是强制的，但在 Cortex-A9 处理器上 NEON 单元却是可选的，使用 -mcpu=cortex-a8 -mfpu=vfpv3 -O3 编译选项可以获得最优化性能和兼容性。
当然 -O3 优化可能也会带来一些问题，如果 -O2 优化已经足够，那么建议使用 -O2 优化选项，这也是 SylixOS Release 版本使用的优化选项。**

##Linux SylixOS 测试结果对比分析
下面的对比分析基于 -mcpu=cortex-a8 -mfpu=vfpv3 -O3 编译选项。

Linux：
```
NUMERIC SORT        :           395.2  :      10.14  :       3.33
STRING SORT         :          40.032  :      17.89  :       2.77
BITFIELD            :      1.3728e+08  :      23.55  :       4.92
FP EMULATION        :            67.8  :      32.53  :       7.51
FOURIER             :          1324.1  :       1.51  :       0.85
ASSIGNMENT          :          5.2366  :      19.93  :       5.17
IDEA                :           840.3  :      12.85  :       3.82
HUFFMAN             :          514.44  :      14.27  :       4.56
NEURAL NET          :            1.42  :       2.28  :       0.96
LU DECOMPOSITION    :          55.316  :       2.87  :       2.07
```

SylixOS：
```
NUMERIC SORT        :          386.53  :       9.91  :       3.26
STRING SORT         :          12.447  :       5.56  :       0.86
BITFIELD            :      1.4006e+08  :      24.03  :       5.02
FP EMULATION        :          88.258  :      42.35  :       9.77
FOURIER             :          1591.8  :       1.81  :       1.02
ASSIGNMENT          :          6.4511  :      24.55  :       6.37
IDEA                :          958.43  :      14.66  :       4.35
HUFFMAN             :          619.13  :      17.17  :       5.48
NEURAL NET          :          1.6131  :       2.59  :       1.09
LU DECOMPOSITION    :          59.127  :       3.06  :       2.21
```

SylixOS 在 NUMERIC SORT 测试中小幅落后于 Linux，而 STRING SORT 测试中大幅落后于 Linux，其它测试 SylixOS 均优于 Linux。

MEMORY INDEX （内存性能）低于 Linux 的原故是 STRING SORT 测试中大幅落后于 Linux。

STRING SORT 测试是 Sorts an array of strings of arbitrary length，落后的原因可能有三：

1. SylixOS 相关字符串函数的执行效率不高；
2. SylixOS 未能发挥 ARMv7A Cache 性能；
3. BSP 在内存控制器设置方面存在不正确的地方。

##在 Linux 测试 SylixOS 的 lib_memcpy 函数性能

通过分析 nbench 的代码，发现 nbench 调用了 memmove 函数，在 SylixOS 上该函数使用 lib_memcpy 函数实现。

在 Linux 上，使用 SylixOS 的 lib_memcpy 函数替换 memmove 函数，以测试 SylixOS 的 lib_memcpy 函数的性能，编译选项为： -mcpu=cortex-a8 -mfloat-abi=softfp -mfpu=vfpv3 -O3。

我们只需要使用 nbench 测试 NUMERIC SORT 和 STRING SORT，nbench 支持执行时指定配置文件：
```
./nbench -CCOM.DAT
```

COM.DAT文件内容：
```
ALLSTATS=F
DONUMSORT=T
DOSTRINGSORT=T
DOBITFIELD=F
DOEMF=F
DOFOUR=F
DOASSIGN=F
DOIDEA=F
DOHUFF=F
DONNET=F
DOLU=F
```

执行后输出：
```
root@ok335x:/home/forlinx# ./nbench -CCOM.DAT

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          394.56  :      10.12  :       3.32
STRING SORT         :          9.8736  :       4.41  :       0.68
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 1.721
FLOATING-POINT INDEX: 1.000
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : Linux 3.2.0
C compiler          : arm-arago-linux-gnueabi-gcc
libc                : static
MEMORY INDEX        : 0.881
INTEGER INDEX       : 1.350
FLOATING-POINT INDEX: 1.000
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

STRING SORT 测试性能评分下降严重（从 40.032 降低到 9.8736，比 SylixOS 的 12.447 还低），可见 SylixOS 的 lib_memcpy 函数在性能上远不如 Linux 的 memmove 函数。

同时我们可以得出如下结论：
1. SylixOS 的 lib_memcpy 函数在性能上远不如 Linux 的 memmove 函数；
2. SylixOS 正常发挥 ARMv7A Cache 性能；
3. BSP 正确设置了内存控制器参数。

##使用 gprof 分析 nbench

我们可以使用 gprof 工具分析 nbench，得到其运行时代码性能和调用栈。

nbench 工程编译和链接时加入 -pg 选项，gcc 会在每一个函数入口处调用 gnu_mcount_nc 函数，gnu_mcount_nc 函数将记录该父子函数的调用次数和关系。

同时启动 ITIMER_PROF 定时器（频率为1000Hz）和绑定 SIGPROF 信号，SIGPROF 信号处理函数获得当前线程的 PC 值，传入 profil_count 函数，profil_count 函数对 nbench 内的函数进行有损精度的耗时统计。

nbench 退出时会生成 gmon.out 文件，将其上传到计算机，计算机执行 arm-sylixos-eabi-gprof，输入文件为 gmon.out 和 nbench，arm-sylixos-eabi-gprof 将生成函数耗时、调用次数、调用栈报告。

使用 [gprof2dot](https://pypi.python.org/pypi/gprof2dot/ "") 可以该报告转换为 .dot 文件，使用 dot（[graphviz](http://www.graphviz.org/ "") 安装目录内）可将其转换为 png 图片或 svg 矢量图。

执行后输出：
```
[root@sylixos_station:/apps]# ./nbench -CCOM.DAT

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :           60.56  :       1.55  :       0.51
STRING SORT         :          3.3703  :       1.51  :       0.23
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 1.129
FLOATING-POINT INDEX: 1.000
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 0.615
INTEGER INDEX       : 0.845
FLOATING-POINT INDEX: 1.000
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
save to libvpmpdm_gmon.out success
save to gmon.out success
```

这里性能参数不具参考价值，我们只关心 libvpmpdm_gmon.out 和 gmon.out。

libvpmpdm_gmon.out 是 libvpmpdm.so 的性能文件，gmon.out 是 nbench 的性能文件，由于 glibc 自带的 libgmon 只支持静态链接的应用程序，所以我们在移植 libgmon 到 SylixOS 时做了大量的修改，让其支持动态链接的应用程序，每一个链接库和应用程序均生成一个 gmon.out 文件。

libvpmpdm.so 对应的调用栈图：
![libvpmpdm_gprof.svg](/img/使用-gprof-分析代码性能和调用栈/libvpmpdm_gprof.svg "")

可以看到 libvpmpdm.so 最耗时的是 lib_memcpy 函数（44.28%）。

nbench 对应的调用栈图：
![nbench_gprof.svg](/img/使用-gprof-分析代码性能和调用栈/nbench_gprof.svg "")

可以看到 nbench 最耗时的是 NumSift 函数（35.05%）。

可以得出，如果我们想提升的 nbench 的 MEM 性能，必须优化 lib_memcpy 函数。

##lib_memcpy 函数优化

lib_memcpy 函数是一个编译进操作系统镜像的“内核函数”，操作系统的编译参数为 -mcpu=cortex-a8 -O2，没有使用 VFP 或 NEON 及 -O3，尝试把 lib_memcpy 函数放入在 libvpmpdm 工程，应用程序 nbench 将优先使用 libvpmpdm.so 的 lib_memcpy 函数。

优化最简单的方法就是修改 libvpmpdm 工程的编译选项，使用 -mcpu=cortex-a8 -mfpu=vfpv3 -O3 编译选项。

```
[root@sylixos_station:/apps]# ./nbench -CCOM.DAT

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.85  :       9.92  :       3.26
STRING SORT         :          11.008  :       4.92  :       0.76
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 1.743
FLOATING-POINT INDEX: 1.000
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 0.913
INTEGER INDEX       : 1.344
FLOATING-POINT INDEX: 1.000
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

结果有点出乎意料，STRING SORT 测试性能评分不升反降，只能用其它优化方法了。

##mem 类函数移植与测试

优化方法要么重新实现 lib_memcpy 函数，要么使用现成的 mem 类函数（memcpy、memmove等），我倾向于使用现成的 mem 类函数，

mem 类函数来源有以下几种：

* FreeBSD 的 C 库里的 mem 类函数
* Android 的 bionic C 库里的 mem 类函数
* 编译器内建的 newlib C 库里的 mem 类函数
* glibc C 库里的 mem 类函数

以上四种 mem 类函数均使用汇编语言实现。

FreeBSD 的 C 库里的 mem 类函数，容易移植，但没有对 VFP 和 NEON 的优化，实测 nbench 运行到 STRING SORT 测试时直接崩溃，PASS 掉。

glibc C 库里的 mem 类函数过于复杂，有对 VFP 和 NEON 的优化，但不容易移植，最后才考虑。

编译器内建的 newlib C 库里的 mem 类函数，有对 VFP 和 NEON 的优化，不需要移植，但 SylixOS 不能使用内建的 newlib C 库，最后才考虑。

Android 的 bionic C 库里的 mem 类函数，有对 VFP 和 NEON 的优化，同时针对不同的 Cortex-Ax 进行优化，移植还算容易，性能应该有保证，下面尝试之。

```
[root@sylixos_station:/apps]# ./nbench -CCOM.DAT

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :           386.9  :       9.92  :       3.26
STRING SORT         :          85.406  :      38.16  :       5.91
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 2.335
FLOATING-POINT INDEX: 1.000
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 1.808
INTEGER INDEX       : 1.344
FLOATING-POINT INDEX: 1.000
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

STRING SORT 测试性能评分提升极大 （从 12.447 提升到 85.406，比 Linux 的 40.032 高出一倍多）。

阅读 bionic C 库里的 mem 类函数，发现其极大地发挥了 Cache 预取特性（通过 PLD 指令）。

将 Android 的 bionic C 库里的 mem 类函数替换掉 SylixOS 的 mem 类函数，文件主要有如下几个：
```
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/memcpy.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/memset.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/stpcpy.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/strcat.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/__strcat_chk.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/strcmp.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/strcpy.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/__strcpy_chk.S \
SylixOS/lib/bionic/arch-arm/cortex-a15/bionic/strlen.S \
SylixOS/lib/bionic/arch-arm/generic/bionic/memcmp.S \
SylixOS/lib/bionic/arch-arm/denver/bionic/memmove.S \
```

完成一次完整的测试：

```
[root@sylixos_station:/apps]# ./nbench

BYTEmark* Native Mode Benchmark ver. 2 (10/95)
Index-split by Andrew D. Balsa (11/97)
Linux/Unix* port by Uwe F. Mayer (12/96,11/97)

TEST                : Iterations/sec.  : Old Index   : New Index
                    :                  : Pentium 90* : AMD K6/233*
--------------------:------------------:-------------:------------
NUMERIC SORT        :          386.91  :       9.92  :       3.26
STRING SORT         :          85.585  :      38.24  :       5.92
BITFIELD            :      1.4014e+08  :      24.04  :       5.02
FP EMULATION        :          88.262  :      42.35  :       9.77
FOURIER             :          1591.9  :       1.81  :       1.02
ASSIGNMENT          :          6.5071  :      24.76  :       6.42
IDEA                :          958.89  :      14.67  :       4.35
HUFFMAN             :          620.88  :      17.22  :       5.50
NEURAL NET          :          1.6134  :       2.59  :       1.09
LU DECOMPOSITION    :          59.122  :       3.06  :       2.21
==========================ORIGINAL BYTEMARK RESULTS==========================
INTEGER INDEX       : 21.899
FLOATING-POINT INDEX: 2.431
Baseline (MSDOS*)   : Pentium* 90, 256 KB L2-cache, Watcom* compiler 10.0
==============================LINUX DATA BELOW===============================
CPU                 :
L2 Cache            :
OS                  : sh: command not found.
C compiler          :
libc                :
MEMORY INDEX        : 5.758
INTEGER INDEX       : 5.255
FLOATING-POINT INDEX: 1.348
Baseline (LINUX)    : AMD K6/233*, 512 KB L2-cache, gcc 2.7.2.3, libc-5.4.38
* Trademarks are property of their respective holder.
```

##Linux SylixOS 测试结果汇总与对比

|      AM335x | Linux cortex-a8 vfpv3 -O3 | 优化前 SylixOS cortex-a8 vfpv3 -O3 | 优化后 SylixOS cortex-a8 neon -O3 |
|------------ |:------------------- |:------------------ |:------------------ |
|MEMORY INDEX        | 4.129 | 3.019 | 5.758 |
|INTEGER INDEX       | 4.565 | 5.249 | 5.255 |
|FLOATING-POINT INDEX| 1.189 | 1.348 | 1.348 |

SylixOS 的 INTEGER INDEX （定点处理性能）与 FLOATING-POINT INDEX （浮点处理性能）及 MEMORY INDEX （内存性能）均要优于 Linux。

SylixOS 优化前后 INTEGER INDEX 和 FLOATING-POINT INDEX 性能基本没有发生变化，但 MEMORY INDEX 性能提升较大（接近一倍的性能提升）。

##测试结论
1. 优化后的 mem 类函数的性能比 Linux 的要好；
2. SylixOS 正常发挥 ARMv7A Cache、VFP、NEON、分支预测性能，比 Linux 的还要好；
3. BSP 正确设置了内存控制器参数和处理器主频。
4. SylixOS 使用的编译器(4.9.4)比 Linux 使用的编译器(4.5.3)更能发挥 ARMv7A 性能。





