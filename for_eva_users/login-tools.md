# 登录工具

> 考虑到大家可能使用不同平台，不同登录工具进行ssh。

- 在阅读本章之前，请确保自己已经阅读并理解了[./login.md](./login.md)中的相关内容。各种软件图形界面所做的其实只是帮助你更高雅/便利的使用ssh命令
- 对不同平台大家常用的登录工具进行了一个简单梳理, 大家可以依据自己所习惯的开发环境进行选择
    - Linux： Terminal，VSCode
    - Windows: Terminal, MobaXTerm. VSCode
    - Mac: Terminal, VSCode

# Terminal

> 这是最本质的登录方式

- 如果你选择了这种方式，那么恭喜你，在[./login.md](./login.md)中的相关内容已经够用了。无论在哪种平台，你都可以用Terminal直接登录服务器，并且在Terminal之中进行服务器的各种操作。   
    - 一个可能存在的问题是，在服务器上进行coding时，没有图形界面，网管建议使用Vim/Emacs，但是对于一些年轻同学来说可能有些困难，如果想使用图形界面进行coding的，可以使用下面的vsode
- Mac与Linux中的Terminal都比较优秀，而Windows中无论是cmd还是powershell都存在(丑且不方便的问题)，这里，网管的方案是[安装WSL并使用WindowsTerminal获得类似Linux的开发体验 - 博客](http://a-suozhang.xyz/2020/04/21/winSetup/)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920113803.png)


# VSCode

> 大多数人使用的方式，戴导倾心推荐

- 相信大家在开发中都经常用到Vscode，vscode原生插件支持了远程登录功能，提供了"在服务器上就好像在本地coding"一样的体验。以下是教程：(网络上也有很多相关教学，由于网管太久不用vscode，这里可能会稍简略一些)
1. 在Extension中安装Remote SSH
![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920112701.png)

2. 使用快捷键`ctrl+shift+p`输入”ssh“索引到ssh相关的操作, 选择"Connected to Host"

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920112958.png)

3. 输入`username@host`，输入密码进行登录（中间可能会有确定框或者是选择平台 - Linux）

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920113250.png)

4. 登录成功之后左下角会显示新的域名：

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920113516.png)

5. 点击这两个位置可以呼出Terminal，在Vscode里也可以使用terminal

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920113555.png)

6. 使用OpenFolder功能之后，可以直接在该窗口中打开文件夹，并进行类似本地的开发

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920113635.png)

7. 进阶技巧：免密码登陆

如果你不希望每次打开 vscode 的时候都需要输入密码，可以参考 [官方说明](https://code.visualstudio.com/docs/remote/troubleshooting) 中的 `Quick start: Using SSH keys` 章节配置 SSH Keys 来验证自己的身份：

`Create your local SSH key pair` 用于在本地生成 SSH Key；

根据本机器的系统版本阅读 `Authorize your XXX machine to connect` 来授权使得本地的 SSH Key 公钥可以登陆服务器

# MobaXTerm

> 有一些out-dated，这里暂时不做推荐，也没有教程

1. 注意如果你使用FPGA服务器(fpga1/2)，可能会存在需要图形界面转发的需求； 那么你可以直接使用MobaXTerm来便捷的进行使用。

2. MobaXterm使用跳板机
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211014221742.png)
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211014221839.png)
