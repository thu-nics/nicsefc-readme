# 新入组必备技能之 常用工具

> 必先利其器

## 参考课程

1. [MIT - The Missing Semester of Your CS Education](https://missing-semester-cn.github.io/)
    - 非常经典的Intro Class，很建议至少看一遍文字资料
2. [IPADS新人培训第一讲：Shell-哔哩哔哩](https://b23.tv/8DIwrX)
    - 上交IPADS组的一个shell教程，非常推荐看一下，包含了一些command-line tools以及bash-scripts的使用; 在该b站账号上有一系列后续教程


## 0. Linux 相关

> 常用命令，常用软件

- SHELL就是传统意义上的 ”Linux命令行“(CLI - command line interface)，使用linux，请首先学会命令行
    - [IPADS新人培训第一讲：Shell-哔哩哔哩](https://b23.tv/8DIwrX)， 上交IPADS组的一个shell教程，非常推荐看一下，包含了一些command-line tools以及bash-scripts的使用
 
- apt安装软件
    - 由于我们使用的是清华tuan源，所以理论并不需要准出也可以进行apt的软件安装

### Tmux

- tmux是linux下的多窗口工具，可以将多个Terminal排列在屏幕中，且**可以避免因为SSH登录终端而导致信息丢失的情况** (比如你直接连接上服务器跑一个程序，然后因为网断了，ssh session断了，于是程序就fail了，但是如果使用tmux的话，可以理解为**程序跑在tmux**里面，而tmux的server一直在服务器上跑着，所以程序不会掉，你只需要重新登录并执行 `tmux-attach即可`)
- 安装,使用 `sudo apt-get install tmux` 即可，使用`tmux`新建一个tmux session并且进入tmux界面；更常用的则是`tmux attach`，直接进入tmux界面，而不新建
- ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211017120718.png)

- 使用说明: tmux的操作基本是在`<prefix>`之后加上一个操作(prefix默认为`CTRL+B`)
    - `<Prefix>+"` 垂直分屏幕
    - `<Prefix>+%`水平分屏幕
    - `<Prefix>+方向键`在屏幕之间移动
    - `<Prefix>+S` 在多个session之间切换
    - `<Prefix>+Z` focus到某个panel
    - `<Prefix>+:` 打开命令行输入
    - `<Prefix>+[` 进入Scroll模式，在此模式下用`CTRL+r` 可以搜索

- 撰写tmux config，一般放在`~/.tmux.conf`中，为了确保config生效，可以在tmux中用`<Prefix>+:`在命令行输入`source-file ~/.tmux.conf`

- tmux的sessions会在服务器重启之后消失，如何保存？**使用tmux-resurrect插件**
    - 下载包管理器  `git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm`
    - 将一下内容加入到tmux的config中,source一下config
    - 然后用`<Prefix+I>` 安装插件,成功之后会显示
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211017133813.png)
    - 然后可以用`<Prefix> +CTRL+C` 以及 `<Prefix> +CTRL+V` 来存储与restore

```
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect' 

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run -b '~/.tmux/plugins/tpm/tpm'
```

## 1. 编辑器

> 因为本届网管主要用vim，大部分服务器也预装vim，所以在这里大概会介绍vim的使用

#### Vim

1. [最强Vim新手指南，手把手教你打造只属于自己的代码编辑器！-哔哩哔哩](https://b23.tv/AQTlVZ)
    - 有一些标题党和浮躁，但是内容还比较适合入门

## 2. Git(版本控制)

> git非常常用！学会一次，受用终身！

## 3. Python相关

> 我们小组主要就是写python

- MiniConda
- Pip
- PyTorch/Numpy

## 4. 文献管理

> 因人而异，欢迎补充

### Zotero

- [github-一些zotero使用说明](https://github.com/redleafnew/Chinese-STD-GB-T-7714-related-csl/stargazers)

## 42. 一些零零散散的小工具

1. [command-not-found](https://command-not-found.com/)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211031102116.png)
