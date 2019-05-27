# 构建根文件系统

## 1. 配置并编译 busybox

### 1.1 配置busybox

如果是使用make 3.82 编译开发板配的busybox 1.17，会提示makefile错误，这是新旧makefile规则不兼容导致的。修改方式：

将类似目标"config %config"中的第一个config去掉即可。

然后执行 make menuconfig 进行配置，关键配置点：

- Busybox settings中使能 “Tab completeion”
- Build Option中，选择编译为动态连接，将静态链接的选项disable掉
- Archival Utilities 中使能tar命令全部配置项
- Linux Module Utilities 中使能insomd， lsmod及rmmod全部配置
- Linux System Utilities 中使能mdev，mount，umount全部配置
- Networking Utilities 中使能ifconfig全部配置

### 1.2 Makefile修改

变量赋值改为以下值：

ARCH						?= arm

CROSS_COMPILE	?= arm-linux-

### 1.3 编译

make && make CONFIG_PREFIX=<安装rootfs的目标目录>  install

## 2. 安装 glibc 库

第一步中编译出的busybox是动态链接的bin，所以需要将相应的动态库也复制到rootfs目录中。动态库在编译工具的以下目录：

\<GCC top>/arm-linux/lib/

### 2.1 Glibc组成

1. 加载器 ld-2.3.6.so, ld-linux.s0.2

   动态连接bin启动前，它们被用来加载动态库。

2. 目标文件(.o)

   如 crt1.o， crti.o， crtn.o， gcrt1.o， Mcrt1.o，Scrt1.o。在生成应用程序时，这些文件像一般的目标文件一样被链接。

3. 静态库(.a)

   如libc.a， libm.a， libstdc++.a等，编译静态bin时会被链接到其中。

4. 动态库(.so, .so.[0-9]*)

   如libc.so， libm.so等，编译动态bin时会使用到，但不会链接它们，只有程序运行时才会连接。

5. libtool库

   在链接库文件时会用到这些文件，比如它们列出当前库文件所依赖 的其他库文件。程序运行时无需使用这些文件。

6. gconv目录

   包含有头字符集的动态库，比如IS8859-1.so，GB18030.so等

7. ldscripts目录

   包含各种链接脚本，编译应用程序时，它们被用于指定程序的运行地址、各段的位置等。

### 2.2 安装Glibc

如果不想麻烦，可以直接将上述目录中的加载器和所有动态链接库直接安装到rootfs目录。

如果想裁剪，则可以查看动态编译的busybox 所依赖的动态，然后将必要的动态库安装到目标rootfs。

在x86上查看交叉编译后的bin所依赖的动态库，可以使用uclibc中的ldd.host查看，编译方法

1. cd  uclibc/utils
2. make  ldd.host
3. ldd.host  busybox

## 3. 创建 /etc 目录

### 3.1  /etc/inittab

Busybox的init进程会根据 /etc/inittab 来创建其他子进程。

#### inittab规则

> \<id>:\<runlevel>:\<action>:\<process>

例如

> ttySAC0::askfirst:-/bin/sh

各字段含义

1. id：表示这个子进程要使用的控制台，如果省略，则使用与init进程一样的控制台
2. runlevel：对于busybox init进程，这个字段没有意义，可以省略
3. action：表示init进程如何控制这个子进程，有下表8种取值：

| action名称 | 执行条件                                                     | 说明                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| sysinit    | 系统启动后最先执行                                           | 只执行一次，init进程等待它结束才继续执行其他动作             |
| wait       | 系统执行完sysinit进程后                                      | 只执行一次，init进程等待它结束才继续执行其它动作             |
| once       | 系统执行完wait进程后                                         | 只执行一次，init进程不等待它结束                             |
| respawn    | 启动完once进程后                                             | init进程检测发现子进程推出时，重新启动它                     |
| shutdown   | 当系统关机时                                                 | 即执行重启，关闭系统命令时                                   |
| askfirst   | 启动完respawn进程后                                          | 与respawn类似，不过init进程先输出“Please press Enter to activate this console”，等待用户输入回车键后才启动子进程 |
| restart    | Busybox中配置了CONFIG_FEATURE_USE_INITTAB，并且init进程接收到SIGHUP信号时 | 先重新读取，解析/etc/inittab文件，在执行restart程序          |
| ctrlaltdel | 按下 Ctrl + Alt + Del 组合键时                               |                                                              |

4. process：要执行的程序或者脚本。如果该字段前有“-”，这个程序被称为可交互的。

**总结**： 

1. 系统启动时，init进程首先启动action为 sysinit、wait、once 3类子进程。
2. 系统正常运行时，init进程首先启动action为 respawn、askfirst 这两类子进程，并监视它们，发现某个子进程退出时重新启动它。
3. 在系统退出时，执行action为 shutdown、restart、crtlaltdel 这3类子进程。
4. 如果文件系统中没有  /etc/inittab 文件，则busybox init进程执行以下默认inittab条目

```shell
::sysinit:/etc/init.d/rcS
::askfirst:/bin/sh
tty2::askfirst:/bin/sh
tty3::askfirst:/bin/sh
tty4::askfirst:/bin/sh
::ctrlaltdel:/sbin/reboot
::shutdown:/sbin/swapoff -a
::shutdown:/sbin/umount -a -r
::restart:/sbin/init
```

#### S3C2440的 /etc/inittab

~~~shell
#/etc/inittab
::sysinit:/etc/init.d/rcS
ttySAC0::askfirst:-/bin/sh
::ctrlaltdel:/sbin/reboot
::shutdown:/sbin/umount -a -r
~~~

### 3.2  /etc/init.d/rcS

这是一个有可执行权限的脚本文件，一般需要开机启动执行的命令放在此文件中。在此我们只是添加网口IP配置并挂载fstab指定的所有文件系统：

~~~shell
#!/bin/sh
ifconfig eth0 192.168.0.11
mount -a
~~~

### 3.3 /etc/fstab

/etc/fstab文件被用来定义文件系统的“景泰信息”，这个信息被用来控制mount命令的行为。JZ2440上使用的文件如下：

~~~shell
# device	mount-point		type		options		dump	fsck order
proc		/proc			proc		defaults	0		0
tmpfs		/tmp			tmpfs		defaults	0		0
~~~

其他字段含义一目了然，这里记录options字段后的含义：

1. options：挂载参数，以逗号分隔

   | 参数名           | 说明                                                         | 默认值 |
   | ---------------- | ------------------------------------------------------------ | ------ |
   | auto<br />noauto | 决定执行mount -a时是否自动挂载。<br />auto: 自动挂载，noauto：不挂载 | auto   |
   | user<br/>nouser  | user：允许普通用户挂载设备 <br/>nouser：不允许               | nouser |
   | exec<br/>noexec  | exec：允许运行所挂载设备上的程序 <br/>noexec：不允许         | exec   |
   | ro               | 只读方式挂载                                                 |        |
   | rw               | 读写方式挂载                                                 |        |
   | sync<br/>nosync  | sync：修改文件时，同时写入设备中 <br/>async：不会同步写入    | sync   |
   | default          | rw，suid，dev，exec，auto，nouser，async等组合               |        |

2. dump&fsck order：

- dump：0 = dump命令不会备份改文件系统， 1 = 备份。
- fsck：决定磁盘的检查顺序。 0 = 忽略该文件系统的检查，如要检查，一般root设为1，其他设为2

## 4. 创建 /dev 目录及相关设备文件

1. 板子上将通过使用mdev来自动创建相关设备文件，但由于mdev时通过busybox的init进程启动的，在启动mdev之前，init进程需要用到两个设备文件  /dev/console 以及  /dev/null，所以这两个设备文件需要动态创建：

~~~shell
cd <rootfs_top>/dev/
sudo mknod console c 5 1
sudo mknod null c 1 3
~~~

2. 板子上，通过mdev自动生成的设备中，S3C2410/S3C2440的串口名是s3c2410_serial0，s3c2410_serial1.s3c2410_serial1，s3c2410_serial2。所以需要修改 /etc/inittab 文件：

   ~~~shell
   s3c2410_serial0::askfirst:-/bin/sh
   ~~~

3. 要使用mdev，内核需要支持sysfs文件系统，为了减少对flash的读写，还要支持tmpfs文件系统。所以首先需要确保内核编译选项已经配置了CONFIG_SYSFS和CONFIG_TMPFS。然后 /etc/fstab中需要设置以上文件系统的自动挂载：

   ~~~shell
   # device	mount-point		type		options		dump	fsck order
   proc		/proc			proc		defaults	0		0
   tmpfs		/tmp			tmpfs		defaults	0		0
   sysfs		/sys			sysfs		defaults	0		0
   tmpfs		/dev			tmpfs		defaults	0		0
   ~~~

4. 还需要在启动脚本  /etc/init.d/rcS 中加入以下命令配置mdev并启动mdev

   ~~~shelll
   mkdir /dev/pts
   mount -t devpts devpts /dev/pts
   echo /sbin/mdev > /proc/sys/kernel/hotplug
   mdev -s
   ~~~

## 5. 制作yaffs2镜像



