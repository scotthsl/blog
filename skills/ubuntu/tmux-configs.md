# tmux配置

## 常用配置

#解除Ctrl+b 与前缀的对应关系
unbind C-b

#设置前缀为Ctrl + x
set -g prefix C-x

#将r 设置为加载配置文件，并显示"reloaded!"信息
bind r source-file ~/.tmux.conf \; display "Reloaded!"

#up
bind-key k select-pane -U
#down
bind-key j select-pane -D
#left
bind-key h select-pane -L
#right
bind-key l select-pane -R

#copy-mode 将快捷键设置为vi 模式
setw -g mode-keys vi

#resize pane

bind -r ^k resizep -U 10 # upward (prefix Ctrl+k)
bind -r ^j resizep -D 10 # downward (prefix Ctrl+j)
bind -r ^h resizep -L 10 # to the left (prefix Ctrl+h)
bind -r ^l resizep -R 10 # to the right (prefix Ctrl+l)

#paste buffer (prefix Ctrl+p)

bind ^p pasteb

#select (v)

bind -t vi-copy v begin-selection

#copy (y)

bind -t vi-copy y copy-selection

## 常用按键

C-b ? 显示快捷键帮助
    C-b C-o 调换窗口位置，类似与vim 里的C-w
    C-b 空格键 采用下一个内置布局
    C-b ! 把当前窗口变为新窗口
    C-b " 模向分隔窗口
    C-b % 纵向分隔窗口
    C-b q 显示分隔窗口的编号
    C-b o 跳到下一个分隔窗口
    C-b 上下键 上一个及下一个分隔窗口
    C-b ALT-方向键 调整分隔窗口大小
    C-b c 创建新窗口
    C-b 0~9 选择几号窗口
    C-b c 创建新窗口
    C-b n 选择下一个窗口
    C-b l 切换到最后使用的窗口
    C-b p 选择前一个窗口
    C-b w 以菜单方式显示及选择窗口
    C-b t 显示时钟
    C-b ; 切换到最后一个使用的面板
    C-b x 关闭面板
    C-b & 关闭窗口
    C-b s 以菜单方式显示和选择会话
    C-b d 退出tumx，并保存当前会话，这时，tmux仍在后台运行，可以通过tmux attach进入 到指定的会话