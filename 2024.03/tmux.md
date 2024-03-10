# Tmux

> 命令行的典型使用方式是，打开一个终端窗口（terminal window，以下简称"窗口"），在里面输入命令。用户与计算机的这种临时的交互，称为一次"会话"（session） 。
> 会话的一个重要特点是，窗口与其中启动的进程是连在一起的。打开窗口，会话开始；关闭窗口，会话结束，会话内部的进程也会随之终止，不管有没有运行完。
> 一个典型的例子就是，SSH 登录远程计算机，打开一个远程窗口执行命令。这时，网络突然断线，再次登录的时候，是找不回上一次执行的命令的。因为上一次 SSH 会话已经终止了，里面的进程也随之消失了。
> 为了解决这个问题，会话与窗口可以"解绑"：窗口关闭时，会话并不终止，而是继续运行，等到以后需要的时候，再让会话"绑定"其他窗口。

## 基本使用

现在的服务器似乎都自带

``` bash
tmux  # 直接创建
tmux new -s <session-name>  # 创建并且起名字

ctrl + b , d  # 退出,会话继续运行
tmux detach  # 同上
```

### 接入

``` bash
tmux ls  # 查看现在有哪些 也可以tmux list-session
tmux attach -t <session-name>

# 如果正在tmux里也可以
ctrl + b, s  # 查看现在有哪些session, 显示更详尽
tmux switch -t <session-name> 切换会话
```

### 其他

``` bash
ctrl + d  # 结束这个session
exit  # 同上

tmux kill-session -t <session-name>  # 杀死会话

tmux rename-session -t <old-session-name> <session-name>
ctrl+b, $  # 重命名当前会话
```

## 窗格
有时候一个服务需要多个窗格协同,所以tmux提供窗格功能,把界面分成多个区域

``` bash
tmux split-window  # 默认划分成上下
tmux split-window -h

tmux select-pane -U/D/L/R  # up/down/left/right

tmux swap-pane -U/D  # 移动窗格
```


### 其他快捷键

```
Ctrl+b %：划分左右两个窗格。
Ctrl+b "：划分上下两个窗格。
Ctrl+b <arrow key>：光标切换到其他窗格。<arrow key>是指向要切换到的窗格的方向键，比如切换到下方窗格，就按方向键↓。
Ctrl+b ;：光标切换到上一个窗格。
Ctrl+b o：光标切换到下一个窗格。
Ctrl+b {：当前窗格与上一个窗格交换位置。
Ctrl+b }：当前窗格与下一个窗格交换位置。
Ctrl+b Ctrl+o：所有窗格向前移动一个位置，第一个窗格变成最后一个窗格。
Ctrl+b Alt+o：所有窗格向后移动一个位置，最后一个窗格变成第一个窗格。
Ctrl+b x：关闭当前窗格。
Ctrl+b !：将当前窗格拆分为一个独立窗口。
Ctrl+b z：当前窗格全屏显示，再使用一次会变回原来大小。
Ctrl+b Ctrl+<arrow key>：按箭头方向调整窗格大小。
Ctrl+b q：显示窗格编号。
```

## 窗口
又有的时候有很多个相似的服务(比如都是streamlit的服务),这个时候不能够放到窗格里

``` bash
tmux new-window  # 创建窗口
tmux new-window -n <window-name>  # 加名字
ctrl + b, c  # 快捷创建

ctrl + b, p/n  # 前后切换窗口
ctrl + b, <number>  # 按照编号切换窗口
ctrl + b, w  # 从列表中选择窗口

tmux rename-window <new-name>  # 窗口重命名
ctrl + b, ,  # 窗口重命名 就是逗号

```
