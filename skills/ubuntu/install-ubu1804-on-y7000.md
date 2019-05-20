# How to install ubuntu 18.04 LTS on LEGION Y7000

## 环境

i7-8750 + 16G DDR + 512G SSD + 1T HDD

SSD上是出厂配置，为GPT分区表，包含UEFI分区，win10已装好，uefi引导。

1T HDD是自己装上去的，刚分区的时候使用fdisk分的区，使用了默认的MBR分区表，被这个坑了。

为了保护SSD寿命，又想ubuntu能快点，所以本人只是把boot和root挂载点做在SSD上，tmp以及var和home都在机械硬盘上。

本以为引导和启动使用的挂载点都在SSD上，而其他目录在MBR分区的硬盘上，这样应该没有问题。。。。

### 以下为试错过程

### 过程1

正常安装，正常分区，boot和root都选在SSD上，引导记录放在boot目录的分区。启动时UEFI启动菜单有ubuntu选择，可是选择之后进入grub。到此放弃，没有继续使用grub手动引导内核的方式。

### 过程2 

正常安装，正常分区，boot和root都选在SSD上，引导记录放在UEFI分区上。启动时UEFI启动菜单有ubuntu选择，可是选择之后进入grub。到此放弃，没有继续使用grub手动引导内核的方式。

### 过程3

正常安装，正常分区，boot和root都选在SSD上，引导记录放在boot分区上。启动时UEFI启动菜单有ubuntu选择，可是选择之后进入grub。到此没放弃，进入liveCD后按照网上教程使用boot-repair尝试修复，失败告终，现象和上述一样。

### 过程4

正常安装，正常分区，boot和root都选在SSD上，引导记录放在boot分区上。启动时UEFI启动菜单有ubuntu选择，可是选择之后进入grub。到此没放弃，重启选择进入win10，安装easyUEFI，尝试新增一个启动项指向boot分区，最后失败告终，现象和上述一致。

### 过程5

经过上述折腾，享受了一轮从入门到放弃的过程。。。休息两天后，手又痒了。这回终于想到是不是HDD不是GPT分区的原因？不管了，死马当死马医吧。尝试在liveCD模式下使用parted命令分了几个区。然后开始正常安装，最后重启的时候，又等了半个小时，才把sda的cache回写完，强制重启，这回终于正常进入登陆界面了。但是发现无法调节亮度。。。

*至此安装过程走完。*

### 正确安装方法总结

1. 从启动盘启动时编辑Ubuntu的启动参数，在splash 后添加 nomodeset

2. 另一块硬盘使用GPT分区

   使用parted命令进行分区

   * sudo parted /dev/sda  #这里我的机械硬盘为 /dev/sda
     * mklabel
       * type: gpt
     * mkpart
       * name: boot
       * start: 1049KiB
       * end: 513MiB
     * mkpart
       * name:
       * start: 513MiB
       * end: 1000GB
     * set 2 lvm #设置分区2为lvm flag

3. 配置lvm分区

   1. 创建pv

      * sudo pvcreate /dev/sda2

      * sudo pvdisplay

   2. 创建vg

      * sudo vgcreate vg-0 /dev/sda2
      * sudo vgdisplay

   3. 创建lv

      * sudo lvcreate  -L 30G  -n lv-root  vg-0
      * sudo lvcreate  -L 10G  -n lv-var  vg-0
      * sudo lvcreate  -L  800G  -n lv-home  vg-0

   4. 然后在试用的桌面里选择开始安装

   5. 安装完成后，点击重启，如果等待许久还是没办法重启，则强制关机

   6. 再次启动后，在启动项选择页，编辑启动参数，在splash后加入 nomodeset，然后按F10启动

   7. 进入桌面后，无WiFi，需要先移除ideapad_laptop驱动：

      * sudo modprobe -r ideapad_laptop

      永久解决办法：

      * sudo  vi  /etc/modprobe.d/blacklist.conf

      * 在末尾添加：

        blacklist  ideapad_laptop

   8. 显卡问题

      开机联网后，先更新一轮软件包数据库，然后启动“附加驱动”这个应用，然后查找可用驱动，点击安装NVIDIA 驱动，安装完成后重启，就可以不用添加 nomoeset参数，并且已经能调节亮度。

