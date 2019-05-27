# cscope笔记

## 建立数据库

### 命令

#cscope  -Rbq *

### 选项说明

* -R     ：表示包含此目录的子目录，而非仅仅是当前目录
* -b     ：此参数告诉cscope生成数据库后就自动退出
* -q     ：生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度
* -k     ：在生成索引时，不搜索/usr/include目录
* -i      ：如果保存文件列表的文件名不是cscope.files时，需要加此选项告诉cscope到哪里去找源文件列表
* -I dir ：在-I选项指出的目录中查找头文件
* -u     ：扫描所有文件，重新生成交叉索引文件
* -C     ：在搜索时忽略大小写
* -P path：在以相对路径表示的文件前加上的path，这样你不用切换到你数据库文件的目录也可以使用它了

> 说明：要在VIM中使用cscope的功能，需要在编译Vim时选择”+cscope”。Vim的cscope接口会先调用cscope的命令行接口，然后分析其输出结果找到匹配处显示给用户

## 在vim中使用cscope

### 命令leader

* :cscope  或者  :cs
* :scscope  或者  :scs 和上述相同，但会横向分隔窗口

### 可用的命令

* add: 增加一个新的cscope数据库/链接库

  * :cs add {file|dir} [pre-path] [flags]

    [pre-path] 就是以-p选项传递给cscope的文件路径，以相对路径表示的文件前加上的path。

    [flags] 你想传递给cscope的额外标志

    :cscope   add   cscope.out   /usr/local/vim   –C

* find: 查询symbol

  * :cs find {querytype} {name}
  * 

  

