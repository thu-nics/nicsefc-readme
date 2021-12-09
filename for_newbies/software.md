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
 
- apt安装软件，参考了[Ubuntu的官方tutorial]
    - 由于我们使用的是清华tuan源，所以理论并不需要准出也可以进行apt的软件安装
    - `sudo apt-get update` 最常见的操作，在更新当前源下库的情况(但不会安装)
    - `sudo apt-get upgrade`: **危险操作**不要执行，等效将当前源下的所有包更新到最新版本！会打破很多依赖
    - `sudo apt-get check` 查看当前apt的情况
    - `apt-get -f install` fix当前的umet dependencies(有一定危险性，建议查看存在umet的库是否涉及到比较通用的)
    - `apt-get autoclean` 删除掉已经不用的deb包（默认位置在 `/var/cache/apt/archives`），可能可以清理出一些空间
        - **注意：** 与之类似的`apt-get clean`是全部删除包
    - `apt-get remove <package_name>` 清除某个包(remove换成purge则会递归删除掉所有它依赖的包，有一定危险性)
    - `apt-get autoremove` 自动清除某些不被依赖的包
    - `apt list -a $name` 列举出某个包的所有版本
    - `apt-cache search <search_term> /  dpkg -l *<search_term>*` 查找某个包, 比较常用，因为linux下的软件包的名字一般和我们想象的不太一样，可以通过模糊搜索来找到我们需要去install啥
    - `apt-cache show <package_name>` 展示某个包的详细信息以及版本等等
    - `dpkg-reconfigure <package_name>` 重新配置某个包


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

2. (Tianchen Zhao)的Vim配置方法:

安装vim以及Vundle(vim插件管理器)

```
sudo apt-get install vim git
mkdir -p ~/.vim/bundle 
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim  # vim 插件管理器
```

获取[.vimrc](https://github.com/A-suozhang/WhatIveRead/blob/master/bak/vim/.vimrc)，放在`~`目录下。(该配置文件只配置了高亮颜色以及便捷注释插件(nerdcommenter)）
更多方式可以进一步探索，并且在能够联网的环境下进入Vim，进入命令模式，输入`:PluginInstall`，执行完之后有报错，不用管，到下一步fix


(fix一个语法高亮上的小bug： `~/.vim/bundle/molokai/colors/molokai.vim:line 132` 将"none"改成None，就正常了！)



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
