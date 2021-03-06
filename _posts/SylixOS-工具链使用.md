title: SylixOS-工具链使用
date: 2015-06-26 11:44:21
tag: 
- SylixOS 
categories: 
- SylixOS 
---

##三个概念##

首先我们区分以下三个概念：
1. 代码
2. 可执行文件
3. 进程

我们在开发环境或编辑器编写的叫代码，如：
```
#include <stdio.h>

int main (int argc, char *argv[])
{
	printf("Hello World!\n");
	sleep(100);
	return 0;
}
```

将其编译后得到可执行文件，如：

![可执行文件.png](/img/工具链使用/可执行文件.png "")

我们将可执行文件下载到目标机的文件系统里，然后执行一条shell命令将其执行，那么它将”变为“一个操作系统的进程，如：

```
[root@sylixos_station:/apps]# ./sample &
[root@sylixos_station:/apps]# Hello World!

[root@sylixos_station:/apps]# ps

      NAME             FATHER        PID   GRP    MEMORY    UID   GID   USER
----------------- ----------------- ----- ----- ---------- ----- ----- ------
kernel            <orphan>              0     0      36864     0     0 root
sample            <orphan>              2     2      45056     0     0 root

total vprocess : 2
```

流程如下图：

{% plantuml %}

@startuml
digraph G {
    code[label="代码", shape=ellipse];
    elf[label="可执行文件", shape=parallelogram];
    proecss[label="进程", shape=box];
    code->elf[label="编译"];
    elf->proecss[label="操作系统加载"];
}
@enduml

{% endplantuml %}

##编译的流程##

将C代码编译成可执行文件，其细节流程如下：

{% plantuml %}

@startuml
digraph G {
    c_code[label="C代码", shape=ellipse];
    cpp_code[label="预处理过的C代码",  shape=ellipse];
    as_code[label="汇编代码", shape=ellipse];
    object[label="*.o目标文件", shape=box];
    elf[label="可执行文件", shape=box];
    lds[label="链接脚本", shape=parallelogram];
    libraries[label="C库、数学库、GCC库", shape=parallelogram];
    c_code->cpp_code[label="预处理器CPP"];
    cpp_code->as_code[label="纯C编译器CC1"];
    as_code->object[label="汇编器AS"];
    object->elf[label="链接器LD"];
    lds->elf[label="链接器LD"];
    libraries->elf[label="链接器LD"];
}
@enduml

{% endplantuml %}

预处理器CPP、编译器CC1、汇编器AS、链接器LD等等构成了一条链条（上一条命令的输出作为下一条命令的输入），所以常被称为“**工具链toolchain**”。

在计算机（如x86+windows)上编译另一个目标系统（如arm+sylixos，处理器和操作系统与主机的不同）程序的方式，被称为“**交叉编译cross compile**”。

此时“**工具链**”称为“**交叉工具链cross toolchain**”或“**交叉编译器cross compiler**”。

预处理步骤实验命令：
```
arm-sylixos-eabi-cpp  -DSYLIXOS -I"D:\workspace\SylixOS_Base/libsylixos/SylixOS" -I"D:\workspace\SylixOS_Base/libsylixos/SylixOS/include" -I"D:\workspace\SylixOS_Base/libsylixos/SylixOS/include/inet" -mcpu=arm920t  -O0 -g3 -gdwarf-2 -Wall -fmessage-length=0 -fsigned-char -fno-short-enums -fPIC -E sample.c -o sample_cpp.c
```

纯C编译步骤实验命令：
```
D:\ACOINFO\arm-sylixos-toolchain\lib\gcc\arm-sylixos-eabi\4.9.3\cc1.exe -mcpu=arm920t -O0 -g3 -gdwarf-2 -Wall -fmessage-length=0 -fsigned-char -fno-short-enums -fPIC sample_cpp.c -o sample.s
```

汇编步骤实验命令：
```
arm-sylixos-eabi-as -mcpu=arm920t -c sample.s -o sample.o
```

链接步骤实验命令：
```
arm-sylixos-eabi-ld  -nostdlib -fPIC -shared -o sample sample.o -L"D:\workspace\SylixOS_Base/libsylixos/Debug" -L"D:\ACOINFO\arm-sylixos-toolchain\arm-sylixos-eabi\lib" -L"D:\ACOINFO\arm-sylixos-toolchain\lib\gcc\arm-sylixos-eabi\4.9.3" -lvpmpdm -lm -lgcc
```

##可执行文件的段##

一个可执行文件包含相当多的段，不同的段存放不同的内容。

查看可执行文件的段信息可以使用工具链里的**readelf**工具。

实验命令：
```
arm-sylixos-eabi-readelf -S sample
```

输出如下：
```
There are 24 section headers, starting at offset 0x1d63c:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .hash             HASH            00000094 000094 000054 04   A  2   0  4
  [ 2] .dynsym           DYNSYM          000000e8 0000e8 000100 10   A  3   3  4
  [ 3] .dynstr           STRTAB          000001e8 0001e8 000082 00   A  0   0  1
  [ 4] .rel.plt          REL             0000026c 00026c 000010 08   A  2   5  4
  [ 5] .plt              PROGBITS        0000027c 00027c 00002c 04  AX  0   0  4
  [ 6] .text             PROGBITS        000002a8 0002a8 000044 00  AX  0   0  4
  [ 7] .rodata           PROGBITS        000002ec 0002ec 000010 00   A  0   0  4
  [ 8] .dynamic          DYNAMIC         000082fc 0002fc 000088 08  WA  3   0  4
  [ 9] .got              PROGBITS        00008384 000384 000014 04  WA  0   0  4
  [10] .data             PROGBITS        00008398 000398 000008 00  WA  0   0  4
  [11] .comment          PROGBITS        00000000 0003a0 000078 01  MS  0   0  1
  [12] .debug_aranges    PROGBITS        00000000 000418 000020 00      0   0  1
  [13] .debug_info       PROGBITS        00000000 000438 0000f5 00      0   0  1
  [14] .debug_abbrev     PROGBITS        00000000 00052d 000084 00      0   0  1
  [15] .debug_line       PROGBITS        00000000 0005b1 001c32 00      0   0  1
  [16] .debug_frame      PROGBITS        00000000 0021e4 000034 00      0   0  4
  [17] .debug_str        PROGBITS        00000000 002218 015526 01  MS  0   0  1
  [18] .debug_loc        PROGBITS        00000000 01773e 000044 00      0   0  1
  [19] .debug_macro      PROGBITS        00000000 017782 005db1 00      0   0  1
  [20] .ARM.attributes   ARM_ATTRIBUTES  00000000 01d533 00002f 00      0   0  1
  [21] .shstrtab         STRTAB          00000000 01d562 0000da 00      0   0  1
  [22] .symtab           SYMTAB          00000000 01d9fc 0002e0 10     23  33  4
  [23] .strtab           STRTAB          00000000 01dcdc 0000ae 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

操作系统动态装载器loader将可执行文件装载运行，上面的段实际上不全出现在对应进程的内存视图上，进程的内存视图往往只有如下几个段：

* .text段：
代码段（codesegment/textsegment）通常是指用来存放可执行代码的一块内存区域。通常这部分内存区域的属性是只读。

* .rodata段：
只读数据段（readonly datasegment），用来存放程序中的字符串和#define定义的常量。

* .data段：
数据段（datasegment）通常是指用来存放程序中已初始化的全局变量的一块内存区域。

* .bss段：
bss段（bsssegment）通常是指用来存放程序中未初始化的全局变量的一块内存区域。BSS是英文BlockStarted by Symbol的简称。

bss段并不会记录在可执行文件里，因为其存放未初始化的全局变量，可执行文件只需要记录该段的大小，loader动态装载可执行文件时，根据其大小在内核堆里分配出来并清零就可以了。

##可执行文件的符号##

一个可执行文件包含相当多的符号。

查看可执行文件的符号信息可以使用工具链里的**nm**工具。

实验命令：
```
arm-sylixos-eabi-nm -D sample
```

输出如下：
```
000083a0 D __bss_end__
000083a0 D __bss_start
000083a0 D __bss_start__
00008398 D __data_start
000083a0 D __end__
00008398 V __sylixos_version
000083a0 D _bss_end__
000083a0 D _edata
000083a0 D _end
00080000 N _stack
000002a8 T main
         U puts
         U sleep
```

查看可执行文件的**未定义符号**，使用如下实验命令：
```
arm-sylixos-eabi-nm -u sample
```

输出如下：
```
         U puts
         U sleep
```

所谓**未定义符号**，是指不在该可执行文件里定义的符号，符号可以是函数名、变量名。如可执行文件sample的未定义符号是puts和sleep，这两个都是操作系统的API。

*因为GCC的优化，所以可执行文件sample并不调用复杂的printf，而是更快速的puts函数。*

##可执行文件依赖的动态库##

链接可执行文件的流程：

{% plantuml %}

@startuml
digraph G {
    object[label="*.o目标文件", shape=box];
    elf[label="可执行文件", shape=box];
    lds[label="链接脚本", shape=parallelogram];
    libraries[label="C库、数学库、GCC库", shape=parallelogram];
    object->elf[label="链接器LD"];
    lds->elf[label="链接器LD"];
    libraries->elf[label="链接器LD"];
}
@enduml

{% endplantuml %}

链接步骤实验命令：
```
arm-sylixos-eabi-ld  -nostdlib -fPIC -shared -o sample sample.o -L"D:\workspace\SylixOS_Base/libsylixos/Debug" -L"D:\ACOINFO\arm-sylixos-toolchain\arm-sylixos-eabi\lib" -L"D:\ACOINFO\arm-sylixos-toolchain\lib\gcc\arm-sylixos-eabi\4.9.3" -lvpmpdm -lm -lgcc
```

链接可执行文件sample时链接了三个库：libvpmpdm、libm、libgcc。

其中libvpmpdm是一个动态库，文件为libvpmpdm.so；而libm和libgcc是静态库，文件分别是libm.a和libgcc.a。

GCC对库的顺序有要求，比如libvpmpdm.so用到了libgcc和libm的符号，libm也用到了libgcc的符号，链接命令应该使用：
```
-lvpmpdm -lm -lgcc
```
而不能够是：
```
-lgcc -lm -lvpmpdm 
```
否则，可执行文件sample装载失败。

查看可执行文件依赖的动态库信息可以使用工具链里的**readelf**工具和GNU的grep工具。

实验命令：
```
arm-sylixos-eabi-readelf -a sample | grep NEEDED
```

输出如下：
```
 0x00000001 (NEEDED)                     Shared library: [libvpmpdm.so]
```

因为可执行文件sample依赖于动态库libvpmpdm.so，所以libvpmpdm.so应该存放在目标机的文件系统的/lib目录里：
```
[root@sylixos_station:/lib]# ls
libvpmpdm.so    libcextern.so   modules
```

实际上并非一定要将动态库存放到目标机的文件系统的/lib目录里，loader动态装载可执行文件时，会在目标机的环境变量LD_LIBRARY_PATH指定的路径里查找该可执行文件依赖的动态库。

```
[root@sylixos_station:/lib]# echo $LD_LIBRARY_PATH
/qt/lib:/usr/lib:/lib:/usr/local/lib
```

可执行文件sample运行时，我们也可以输入**modules**命令查看进程依赖的动态库：

```
[root@sylixos_station:/apps]# modules

            NAME                HANDLE   TYPE  GLB   BASE     SIZE    SYMCNT
------------------------------ -------- ------ --- -------- -------- --------
VPROCESS: kernel               pid:   0 TOTAL MEMORY: 36864
+ xsiipc.ko                    30c74b68 KERNEL YES c0019000     427c       14
+ xinput.ko                    30c74cb8 KERNEL YES c000f000     16c0        1
VPROCESS: sample               pid:   3 TOTAL MEMORY: 45056 <vp ver:1.3.4>
+ sample                       30c78dc0 USER   YES c0020000     8394        2
+ libvpmpdm.so                 30c79250 USER   YES c0030000     bc70       70

total modules : 4
```

##反汇编可执行文件##

反汇编可执行文件可以使用工具链里的**objdump**工具。

实验命令：
```
arm-sylixos-eabi-objdump -d sample
```

输出如下：
```

sample:     file format elf32-littlearm


Disassembly of section .plt:

0000027c <.plt>:
 27c:	e52de004 	push	{lr}		; (str lr, [sp, #-4]!)
 280:	e59fe004 	ldr	lr, [pc, #4]	; 28c <main-0x1c>
 284:	e08fe00e 	add	lr, pc, lr
 288:	e5bef008 	ldr	pc, [lr, #8]!
 28c:	000080f8 	.word	0x000080f8
 290:	e28fc600 	add	ip, pc, #0, 12
 294:	e28cca08 	add	ip, ip, #8, 20	; 0x8000
 298:	e5bcf0f8 	ldr	pc, [ip, #248]!	; 0xf8
 29c:	e28fc600 	add	ip, pc, #0, 12
 2a0:	e28cca08 	add	ip, ip, #8, 20	; 0x8000
 2a4:	e5bcf0f0 	ldr	pc, [ip, #240]!	; 0xf0

Disassembly of section .text:

000002a8 <main>:
 2a8:	e92d4800 	push	{fp, lr}
 2ac:	e28db004 	add	fp, sp, #4
 2b0:	e24dd008 	sub	sp, sp, #8
 2b4:	e50b0008 	str	r0, [fp, #-8]
 2b8:	e50b100c 	str	r1, [fp, #-12]
 2bc:	e59f3024 	ldr	r3, [pc, #36]	; 2e8 <main+0x40>
 2c0:	e08f3003 	add	r3, pc, r3
 2c4:	e1a00003 	mov	r0, r3
 2c8:	ebfffff0 	bl	290 <main-0x18>
 2cc:	e3a00064 	mov	r0, #100	; 0x64
 2d0:	ebfffff1 	bl	29c <main-0xc>
 2d4:	e3a03000 	mov	r3, #0
 2d8:	e1a00003 	mov	r0, r3
 2dc:	e24bd004 	sub	sp, fp, #4
 2e0:	e8bd4800 	pop	{fp, lr}
 2e4:	e12fff1e 	bx	lr
 2e8:	00000024 	.word	0x00000024
```

##查看可执行文件的大小##

查看可执行文件的大小可以使用工具链里的**size**工具。

实验命令：
```
arm-sylixos-eabi-size sample
```

输出如下：
```
   text	   data	    bss	    dec	    hex	filename
    614	    164	      0	    778	    30a	sample
```

##裁剪可执行文件##

一个简单的hello world程序，其大小就达到了120KB：
```
2015/06/26  14:46           122,250 sample
```

显然就有点占目标机的磁盘空间了。

裁剪可执行文件可以使用工具链里的**strip**工具。

实验命令：
```
arm-sylixos-eabi-strip sample
```

裁剪后，可执行文件sample的大小降低到1.7KB左右：
```
2015/06/26  16:01             1,756 sample
```

*注意：裁剪后的可执行文件不能同于调试，因为去掉了调试信息。*

##制作静态库##

制作静态库可以使用工具链里的**ar**工具。

实验命令：
```
arm-sylixos-eabi-ar -r libsample.a sample.o
```

*注意：GCC对库的名字有要求，库的名字前缀必须是lib，扩展名为.a代表这是个静态库（*.o的集合），扩展名为*.so代表这是个动态库。*

生成的静态库：
![静态库.png](/img/工具链使用/静态库.png "")

这个静态库并没有什么实际意义，这里只是为了演示。

##调试程序##

当我们觉得程序有问题时，我们可以使用工具链里的**gdb**工具进行调试。

*注意：strip过的可执行文件不能调试，因为不带调试信息，此外，编译时用参数 -O0 -g3 -gdwarf-2，即不优化代码并带调试信息。*

* 先在主机启动gdb
实验命令：
```
arm-sylixos-eabi-gdb sample
```

* 然后在目标机启动debug server：
```
[root@sylixos_station:/apps]# debug :1234 /apps/sample &
[root@sylixos_station:/apps]# [GDB]Waiting for connect...
```

* 主机的gdb连接目标机的debug server
```
(gdb) target remote 192.168.7.30:1234
Remote debugging using 192.168.7.30:1234
```
如果能连接上，目标机应该会输出：
```
[root@sylixos_station:/apps]# [GDB]Connected. host : 192.168.7.20
```

* 列出代码
```
(gdb) l
1       /*
2        * sample.c
3        *
4        *  Created on: 2015-06-26
5        *      Author: Administrator
6        */
7
8       #include <stdio.h>
9
10      int main (int argc, char *argv[])
```

* 在main函数入口处下个断点
```
(gdb) b main
Breakpoint 1 at 0xc00202bc: file sample.c, line 12.
```

* 恢复运行
```
(gdb) c
Continuing.
Breakpoint 1, main (argc=1, argv=0x30c78d14) at sample.c:12
12              printf("Hello World!\n");
```
程序遇到第一个断点，将停在main函数的入口处。

