---
title: Tmux使用
date: 2023-07-20
tags: [Linux, Tmux]

---

## 会话

新开启会话

```
$ tmux 或 tmux new -s <name>
```

### 分离会话

```
$ tmux detach
```

查询会话窗口

```
$ tmux ls
```

### 重接会话

```
$ tmux attach -t <name or index>
```

### 杀死会话

```
$ tmux kill-session -t <name or index>
```

### 切换会话

```
tmux switch -t <name or index>
```

### 其他命令

```
# 列出所有快捷键，及其对应的 Tmux 命令
$ tmux list-keys

# 列出所有 Tmux 命令及其参数
$ tmux list-commands

# 列出当前所有 Tmux 会话的信息
$ tmux info

# 重新加载当前的 Tmux 配置
$ tmux source-file ~/.tmux.conf
```

### 安装插件管理器tpm

```
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

 使用快捷键 `prefix + I`  安装插件
 
 ## 窗格(Pane)操作

- % 左右平分出两个窗格
- " 上下平分出两个窗格
- x 关闭当前窗格
- { 当前窗格前移
- } 当前窗格后移
- ; 选择上次使用的窗格
- o 选择下一个窗格，也可以使用上下左右方向键来选择
- space 切换窗格布局，tmux 内置了五种窗格布局，也可以通过 ⌥1 至 ⌥5来切换
- z 最大化当前窗格，再次执行可恢复原来大小
- q 显示所有窗格的序号，在序号出现期间按下对应的数字，即可跳转至对应的窗格

### tmux个人配置.tmux.conf

```
# Set prefix key to Ctrl-a
# unbind-key C-b
# set-option -g prefix C-a
# bind-key C-a last-window # 方便切换，个人习惯
# bind-key a send-prefix
# shell下的Ctrl+a切换到行首在此配置下失效，此处设置之后Ctrl+a再按a即可切换至shell行首
set-option -g prefix2 M-`  # 第二个快捷键为ALT+` 

# 重新读取加载配置文件
bind r source-file ~/.tmux.conf \; display-message "Config reloaded..."

# 不使用prefix键，使用Ctrl和左右方向键方便切换窗口
# bind-key -n "M-Left" select-window -t :-
# bind-key -n "M-Right" select-window -t :+

# displays 
setw -g mouse on

set -g default-terminal "screen-256color"   # use 256 colors
set -g display-time 5000                    # status line messages display
set -g history-limit 100000                 # scrollback buffer n lines
setw -g mode-keys vi                        # use vi mode

# vim ESC 键延迟时间
set -s escape-time 50

unbind-key j
bind-key -n M-j select-pane -D
unbind-key k
bind-key -n M-k select-pane -U
unbind-key h
bind-key -n M-h select-pane -L
unbind-key l
bind-key -n M-l select-pane -R

# 绑定Ctrl+↑↓←→键为面板上下左右调整边缘的快捷指令
bind-key -n "C-Up" resizep -U 3
bind-key -n "C-Down" resizep -D 3
bind-key -n "C-Left" resizep -L 10
bind-key -n "C-Right" resizep -R 10

# 使窗口从1开始，默认从0开始 
set -g base-index 1
# 自动重新编号 window
set -g renumber-windows on

# 关闭rename机制
setw -g automatic-rename off
setw -g allow-rename off

unbind %
bind -n M-- splitw -v -c '#{pane_current_path}'  # 垂直方向新增面板，默认进入当前目录
unbind '"'
bind -n M-= splitw -h -c '#{pane_current_path}' # 水平方向新增面板，默认进入当前目录

# 设置自动刷新的时间间隔
set -g status-interval 1
# 状态栏左对齐
set -g status-justify left
# 状态栏左侧宽度
set -g status-left-length 20
# 状态栏右侧宽度
set -g status-right-length 50

# 状态栏背景颜色
set -g status-bg '#333333'
# 状态栏前景颜色
set -g status-fg '#ffffff'
# 状态栏左侧显示 session 的名字
set -g status-left '#[bg=#00bb00] [#S] #[default] '
# 状态栏右侧显示时间
set -g status-right '#[fg=white,bg=#444444] [#h] #[fg=white,bg=#666666] %Y-%m-%d #[fg=white,bg=#888888] %H:%M:%S '

# 当前激活窗口在状态栏的展位格式
setw -g window-status-current-format '#[bg=#00bbbb, fg=#ffffff, bold]*[#I] #W*'
# 未激活每个窗口占位的格式
setw -g window-status-format '#[bg=#00bbff, fg=#ffffff] [#I] #W '

# 插件列表
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'christoomey/vim-tmux-navigator'
set -g @plugin 'tmux-plugins/tmux-yank'

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run -b '~/.tmux/plugins/tpm/tpm'
```

