# u-boot 打补丁、编译及烧录

## 一、打补丁

1. 进入要打补丁的目标目录

2. 执行命令：

   patch  -p \<n>  < xxxxx.patch

   n为要去掉的前级目录层次，具体要去掉多少层，依赖于当前工作目录，以及patch中包含的目录层级。

## 二、编译

### 配置

make  100ask24x0_config

### 编译

make

## 三、烧录

Ubuntu下使用oflash烧录：

sudo oflash u-boot.bin 

然后根据提示选择即可，注意需要先给开发板上电并连接好PC。u-boot推荐烧写到nor flash，后续的其他实验性固件烧写到NAND flash，速度较快。