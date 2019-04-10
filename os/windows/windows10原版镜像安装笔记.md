# windows10原版镜像安装笔记

# 安装环境

Lenovo E430: I5-3210 + 6G DDR3 + 60G SSD + 500 HDD

系统版本：windows 10 x64  1803

使用量产工具将从 MSDN I tell you上下载的原版系统镜像烧到U盘，主控慧荣SM3267AB

原有disk上都是mbr分区，计划转换成GPT分区，并使用EFI启动。

# 安装流程

## 1. BIOS设置

* 开机时快速按F1进入BIOS
* 在 startup tab下选择UEFI启动
* 选择USB启动顺序为第一

## 2. 分区创建

从U盘启动后 ，一路前进到选择安装分区的时候，发现之前在win8 PE下使用diskpart分好的分区，在选择安装时，系统提示  *无法创建分区* 的错误。

没办法，只好这时按 shift+F10 弹出命令行，重新使用diskpart进行分区，这时留了个心眼：

因为系统要装到SSD上，所以我旨在SSD上进行了分区，HDD上分区被清除了。分区步骤：

进入diskpart环境

```shell
diskpart
```

列出当前磁盘，以便选择后续命令操作的焦点磁盘

```shell
list disk
```

选择你要操作的磁盘，后续的分区创建以及删除都在该磁盘上进行，比如我的是磁盘0

```shell
select disk 0	#选择磁盘0
clean			#清除disk0上的所有分区信息
convert gpt		#创建GPT分区表
create partition efi size=256		#创建EFI分区
create partition msr size=128		#创建微软保留分区，推荐系统分区大于16GB时，保留分区128MB
create partition pri			#创建主分区，不指定size，则使用剩余空间来创建该分区

select partition 1		#选择EFI分区
format fs=fat32 quick	#快速格式化EFI分区，EFI分区指定使用FAT32格式
select partition 3		#选择刚才创建的主分区
format fs=ntfs quick	#快速格式化为NTFS
```

## 3. 开始安装

以上第二步已经准备好分区，推出cmd后刷新一下disk列表，就可以开始安装了

