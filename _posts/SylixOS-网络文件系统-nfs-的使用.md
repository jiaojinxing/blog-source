title: SylixOS 网络文件系统 nfs 的使用
date: 2015-07-31 11:16:24
tag: 
- SylixOS 
categories: 
- SylixOS 
---

SylixOS 支持网络文件系统 nfs，在开发阶段，当工程文件相当多并修改频繁时，使用 nfs 可以免去频繁下载这些工程文件的麻烦，从而提高开发效率，下面介绍 nfs 的使用方法。

##确保 SylixOS 编译了 nfs 组件

默认情况 SylixOS 开启了 nfs 的支持，但 nfs 可以裁减，查看 sylixos-base/libsylixos/config/fs/fs_cfg.h 文件，找到 LW_CFG_NFS_EN 的定义，确保 LW_CFG_NFS_EN 被定义为 1，如下：
```
#define LW_CFG_NFS_EN                       1                           /*  是否使能 NFS 文件系统服务   */
```

此外，nfs 依赖于如下组件：

1. 网络
2. RPC
3. I/O 系统

需要确保以上组件均已经使能。

##主机运行 nfs 服务器

双击 FreeNFS.exe 运行 nfs 服务器，FreeNFS.exe 运行后会退到系统托盘，在系统托盘选中 FreeNFS 的图标，并右键打开快捷菜单，点击 "settings..." 菜单打开设置对话框。

切换到 Server 页面：

![图像 1.png](/img/SylixOS-网络文件系统-nfs-的使用/图像 1.png "")

Path 输入框输入主机用于 nfs 的目录路径。

切换到 Clients 页面：

![图像 2.png](/img/SylixOS-网络文件系统-nfs-的使用/图像 2.png "")

Allowed host 输入允许的开发板的 IP 地址，使用空格分隔多个 IP 地址。

切换到 Filenames 页面：

![图像 2.png](/img/SylixOS-网络文件系统-nfs-的使用/图像 3.png "")

Codepage 选择 “20936 (简体中文 GB2312)"。

最后点击 OK 按钮完成设置。

##开发板挂载 nfs

使用网线连接开发板与主机（或确保开发板与主机在同一网段并可相连)。

在开发板的 shell 执行如下命令：
```
 mount -t nfs 192.168.1.10:/posixtestsuite /mnt/nfs
```

mount 是挂载命令；

-t 指定了文件系统的类型为 nfs；

192.168.1.10:/posixtestsuite 是主机的路径，其中 192.168.1.10 是主机的 IP 地址，**而 /posixtestsuite 是主机的 D:\workspace_opensource\posixtestsuite 目录下存在的子目录**; 

/mnt/nfs 是需要挂载到路径，一般情况下我们使用 /mnt 的一个子目录用于挂载，/mnt/nfs 目录在挂载时被创建，所以无需事先创建。

挂载成功后，进入 /mnt/nfs/ 目录，ls 可查看主机 D:\workspace_opensource\posixtestsuite\posixtestsuite 目录下的内容：
```
[root@sylixos_station:/]# cd /mnt/nfs/
[root@sylixos_station:/mnt/nfs]# ls
AUTHORS         BUILD           ChangeLog       conformance     COPYING
Documentation   exec-func.sh    execute.sh      functional      include
INSTALL         LDFLAGS         locate-test     logfile         Makefile.sylixos
NEWS            posixtestsuite_run_test         QUICK-START     README
run_tests       sed.exe.stackdump               stress          t0.c
```

showmount 命令可以查看当前系统挂载的文件系统：
```
[root@sylixos_station:/]# showmount
all mount point show >>
       VOLUME                    BLK NAME
-------------------- --------------------------------
/mnt/nfs             192.168.1.10:/posixtestsuite
/ramdisk             0
```

df 命令可以查看文件系统的大小、空闲空间、使用百分比及类型：
```
[root@sylixos_station:/]# df /mnt/nfs/
    VOLUME         TOTAL        FREE     USED RO       FS TYPE
-------------- ------------ ------------ ---- -- --------------------
/mnt/nfs/          443.22GB     359.69GB  18% n  NFSv3 FileSystem
```

umount 命令可以取消挂载文件系统：
```
[root@sylixos_station:/]# umount /mnt/nfs/
[root@sylixos_station:/]# showmount
all mount point show >>
       VOLUME                    BLK NAME
-------------------- --------------------------------
/ramdisk             0
```