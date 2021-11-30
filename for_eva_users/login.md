# 如何登录服务器

## 在我们开始之前，请首先熟悉几个概念

1. SSH: Secure Shell是一种加密的网络传输协议, 我们将通过SSH协议在本地机器(你的笔记本电脑)，登录到服务器，在服务器上进行操作。SSH本身只是一种协议，实际你将通过各种支持SSH访问的各种软件(如terminal/MobaXTerm/VSCode等)，关于SSH工具的选择与教程，请参考[./login-tools.md](./login-tools.md)，关于ssh的基础知识和使用方法，请自行查询网络寻找。

2. Linux:  服务器大多基于Linux系统，它是与Windows，MacOS并列的一种操作系统(OS)，我们的服务器使用的具体系统是Ubuntu。不同于你笔记本电脑的图形界面，服务器主要交互方式是基于SHELL(类似命令行的操作)。你通过SSH登录上服务器之后将通过命令行进行各种操作。建议预先学习Linux操作的基础知识。(对于常用操作，可以参考[网管的博客-Linux操作相关](http://a-suozhang.xyz/2019/09/06/Linux-Commands/)  )

3. Linux Container (LXC): 环境隔离的工具，类似Docker或虚拟机，但是更为轻量级，实际上是Docker早期的底层技术。计算资源以Container（容器）为单位进行管理和使用，每一个Container都是一个独立的Linux系统，环境不会互相影响，你可以在其中拥有sudo权限。(如果你把container搞崩了，可以联系网管删除并重建)

4. 网络的基础知识：首先让我们明确几个概念，每个服务器(以及你的笔记本电脑)在网络环境中是用IP地址所标识的(相当于他们的ID)
    - IP: 比如`101.6.64.67`，ssh中通过ip来确认需要登录哪个服务器。
    - 域名: IP的别名，如`ztc.eva7.nics.cc`，可以认为是方便大家记忆IP的方式，比如`www.baidu.com`也是一个域名。(从域名获取IP的方式，执行`ping`命令)
    - 初代网管为我们实验室的服务器搭建了域名服务，其中诸如`eva*.nics.cc`的域名，是服务器主机，只有网管可以登录。而类似`ztc.eva7.nics.cc`的是eva7主机上名为ztc的Container，用户登录时用这个IP，就可以直接登录进你的Container(自己的linux系统)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919202335.png)

5. 校园网：因为各种不可抗力，这个步骤居然是我们登录服务器**最困难**的部分(也是最为个性化的部分)。由于校园网需要**准入IP**才能让服务器可以被访问，所以与一般的登录服务器不同，在登录校园网内的服务器的时候，需要**先准入IP，再正常使用ssh登录服务器**。准入中可能会遇到一些问题，请参考[../qa.md](../qa.md)
    - 具体的准入方法,登录[usereg.tsinghua.edu.cn](http://usereg.tsinghua.edu.cn/), 登录自己的账号，并且选中"准入代认证"，填入你需要准入的服务器IP与密码(你的校园网账户的密码)，选择“校内”或“校外”，进行登录。如果显示“登录成功”，那么说明此操作成功了。如果显示**空白**或是**IP不在DHCP表中**，说明是报错信息，可查看[../qa.md](../qa.md)
    - 我们先重申几个概念，请记住他们的含义，方便大家的交流与沟通：
        - 准入:  对应着“准入待认证”中选择“校内”选项，表示将对应的IP **加入校园网，使其可以在校园网内网被访问**，注意：**此操作是登录上服务器的必要条件**，且**不代表服务器能够连接外网(这个是准出所作的事情)**。一个校园网账户理论上可以同时准入无上限个IP。
        - 准出： 对应着“准入代认证”中的“校外”选项，表示**允许对应的IP访问外网**，也就是服务器可以登录外网。注意：**校园网账号的准出IP数目有上限的，包括个人手机电脑等设备，超出之后，之前的IP会被挤掉**
        - 一个IP的状态可以是：(未准入准出) -> (准入) -> (准出)；对已经“准入”的（同一账号），或者是“未准入准出”的服务器，在usereg上用同一账号执行“准出”都可以让它直接进入准出状态。
        - 如果你想要下线一个IP，进入“准入在线”之后，可以看到你的校园网账号所准入的所有IP，选择其并下线。
        - 判断服务器准入准出是否成功的方式：
            - 准入： 在本地/跳板机(之后会讲到)使用ping命令，`ping`你需要准入的IP，如果能够ping通，那么说明准入成功 (随着学校网络政策的不断更新，该种方式可能过时，即未准入状态也能ping通，所以如果能ping通但是不能登录服务器，还是要尝试使用usereg进行准入)  
            - 准出： 在登录服务器之后，`ping www.baidu.com`

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919201319.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919201428.png)

- 上述知识图示: (如果你完全理解了上面的部分，并查询了相关网络资料，我们可以进行下一步了。)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920172607.png)

## 使用SSH登录服务器

1. 我们假定你已经对ssh命令如何使用已经有所了解了。请注意：下面的部分中，请**不要使用SSH软件，而使用Terminal/SHELL来完成**，通过这种方式了解ssh的基本原理，这样你才能更好的理解ssh的软件使用方式。(请不要抱着侥幸心理直接一开始就照搬学长学姐的配置方式，将来的科研生活离不开ssh，如果不清楚原理，你将**无法debug登录问题以及尝试各种炫酷的功能**)
    - 如果你不知道如何使用terminal，在ubuntu/shell中直接使用系统terminal，在windows中请使用cmd/powershell（可能用户体验会差一点，为了照顾你们，网管下面以windows为例）（如果你连怎么打开cmd都不知道，那么你可能有很多知识可以学习，使用win+R打开“运行”界面，并输入cmd）

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919203414.png)

2. 我们这里会用到的ssh登录的几个要素： 用户名(你的实验室账户名字)，IP（或者是域名，设定上两者等效，使用域名时主机会先从互联网DNS服务器查询该域名的IP，这里建议先用IP登陆），端口(Optional)。ssh命令的使用方式是`ssh username@IP -v -p port`,在大部分时候，`-p port`的部分可以省略(注意一定要是小写的p，大写P有另外的含义)，`-v`是显示详细信息的意思，建议加上。假设你已经完成了**对你需要登录的服务器进行usereg准入**，那么你可以尝试执行`ssh username@IP`了。不出意外，系统会提示你输入密码，正确输入后回车，就可以正常登录上服务器了，是不是很简单？

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919204048.png)

如果你还记得我上面说的，域名是IP的另外一种方便记的方式，现在试着不用刚才准入的IP，而用域名登录试试？

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919204219.png)

一样可以登录，这样就不用记一串没有含义的数字了。且需要注意的是，服务器的IP可能会动态改变，但是域名不会改变，改变后的IP会被我们实验室的域名服务自动映射，所以除了Container刚建立时域名还没有同步的时候采用IP登陆以外，其他时候采用类似`ztc.eva7.nics.cc`这样的域名登陆即可。**只要服务器处于准入状态，那么你只要一直使用这个命令就可以登录了**)

3. 看，登录服务器就是这么简单！直接执行`ssh username@xxx.evax.nics.cc`就可以登录，发现登录不上，就去用`ping`找一下IP，然后在usereg中准入一下就可以了呢！其实你使用其他ssh软件，本质上就是以不同的GUI执行了这个操作，由于terminal是最为简单且快捷的方式，所以之后在**登录不上debug的时候，请使用terminal进行测试**

   ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919204734.png)

## 在本地与服务器之间传输文件

1. scp命令: 
    - 格式如 `scp ./something zhaotianchen@ztc.eva7.nics.cc:~/`
        - scp需要与ssh类似的用户名+域名(IP)的登录方式
        - 冒号后面的内容是服务器上的路径
        - 注意如果要指定端口的话，用scp命令的`-P `大写(比如你要往205跳板机上scp，就需要使用`scp -P 42222 zhaotianchen@101.6.64.67:/`)
        - scp支持批量拷贝文件夹，使用`-r`指令
        - scp也支持下面所介绍的ssh config的功能哦，你可以直接`scp ./something ztc-eva7:~/`

2. 如Vscode等软件可能可以支持直接拖拽上传下载

---

## Trouble Shooting

> 如果登录服务器真的像上面这么简单的话，网管就不需要写这个文档了，在其中各个环节都有可能出现问题，下面将进行错误拆解、分析，并提供解决方案。

- 这一部分目前挪到[qa.md](../qa.md)了

## 向网管汇报问题

> 能走到这一步，大概率是前面的部分都测试过，可能由于校园网的管理问题以及服务器的配置问题出现了无法登录的情况

提问模板: 

```
我登录服务器遇到了问题，我的域名是 [`xxx.evax.nics.cc`]，我的登录平台和工具是: [VScode windows]
我用ping的方法查询我的服务器当前IP是 [xxx.xxx.xx.xx], 该IP已经准入校园网 (如果出错，描述报错信息)， 且[是/否]可以ping通
目前的错误的现象是 [登录没有反应/报错Connection Refused at Port 22/输入密码之后报Permission Denied/其他]
麻烦网管帮忙看一下，感谢
```

---



# 如何优雅的登录服务器

> 首先请确保你掌握了上述的朴素登录服务器方式，下面让我们来试试一些fancy的操作。以下的一些trick的教程在网络上都可以找到，如果网管这里有写的不全的部分，请自行ref网络资料


### 服务器连外网

1. 从上面部分我们知道，usereg中对IP有准入和准出两种状态；只有准入了才能登录服务器，只有准出了才能让服务器登陆外网*
2. 


### 使用跳板机

- 上文中我们提到过，我们的服务器都在清华内网，如果我们想要在校外网络环境 (校外WiFi - 比如你的手机热点，或者是你回家了)，也想登录服务器，那该怎么办呢？我们提供了一台**跳板机**(如果对这个概念感兴趣可以查询其意义)服务器。叫做**205**(因为它的物理位置放在4-205)，通过它作为中继，可以在校外登录服务器，它的IP是`101.6.64.67`，请记住这个IP。

- 登录205的方式为：`username@101.6.64.67 -p 42222`
    - 这个命令的含义是，从42222端口(**这个点非常关键，如果不这么做会登录不上**)，用你的实验室账户用户名和密码，去登录跳板机
    - 我们一般登录服务器不需要指定port的原因是，ssh服务的默认端口是22，而一般我们登录服务器就是走22端口，但是205服务器必须从42222端口进行登录，如果不填写port，ssh会尝试从22端口登录205，会登录不上
- 成功登录之后，你就从校外环境，进入了校内环境，现在你可以在205上进行正常的ssh登录，比如`ssh zhaotianchen@ztc.eva7.nics.cc`,就可以登录了


### 撰写ssh config并快速登录

> 上面的这种登录方式太麻烦了！每次登录的时候都需要写好长一条命令！我能否以`ssh ztc-eva7`来替代`ssh zhaotianchen@ztc.eva7.nics.cc`呢？如果我想要使用跳板机，我是否可以把两次ssh的登录，当成一次登录呢？

- (网管不会再在这里提供ssh config模板，而请你自己编辑一份，为了防止大家照搬而完全不理解里面内容的含义 - happened before)

- 当然可以！这就是ssh config文件的作用！通过撰写一个config文件，去简化命令。
- 对于Linux系统，编辑`~/.ssh/config`文件(如果没有，就自己创建一个)
    - 如果你不知道`~`这个目录是`/home/$username`的意思
- 我们以登录205服务器为例，撰写一条config，并尝试`ssh 205`.
    - Host： 可以任意定义，这台服务器的别名，我们这里是205，所以你的测试命令将会是`ssh 205`
    - HostName: 域名或者是IP，这里是`101.6.64.67`对于其他服务器，可以是`ztc.eva7.nics.cc`
    - User: 表示登录服务器所用的用户，是你的服务器账户名字
    - Port: 从哪个端口登录，默认是22，这里指定为42222，对于其他服务器，这条配置可以空着，或者是填22
- 我们下面尝试一下通过205登录其他服务器给merge成一条config，注意把域名和用户名替换成你自己的
    - 如下添加一条config
    - 主要起效的是ProxyCommand这一条，表示了**通过205作为proxy来登录**
    - 下面在terminal中`ssh ztc-eva7`,那么即使在校园网环境外，也可以一步到位了
    - (这种proxyCommand可以叠加，你甚至可以A-B-C-D-E这样连环跳）

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920120249.png)

---

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920115229.png)

- 对于VSCode，一样可以撰写ssh config (如果你还不知道vscode正常怎么使用ssh，参考[./login-tools.md](./login-tools.md))
    - 使用`ctrl+shift_p`，键入`ssh`,选择`Open Configuration File`
    - 选择正确的ssh config file的位置，对于Windows来说一般是截图中的这个
    - 按照上述的方式去撰写ssh config file，在登录ssh的时候，就可以选择Host了

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920115702.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920115807.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920120037.png)

### 免密码登录

> 每次登录输密码太麻烦了！有的时候因为VSCode的bug甚至要输入两次密码！我要点开就登！

- 当然可以！可以通过**拷贝公钥**的形式来实现"免密码ssh登录"

- 如果你是Windows系统，且没有安装WSL(用wsl的terminal中的ssh登录相当于是用linux了)，那么请执行以下操作
    - follow了[知乎文章](https://zhuanlan.zhihu.com/p/117292835)
    1. 在windows中使用cmd，在上面ssh conifg的目录中打开，执行`ssh-keygen -t rsa -b 4096`
    2. 会在这个目录下生成文件`.ssh\id_rsa.pub`
    3. 将这个文件里的内容，复制到远程Linux 中的`~/.ssh/authorized_keys` (如果没有就新建)。
    4. 继续登录，应该就可以不用密码了。
    - 另外一种方式，你也可以不手动复制id_rsa.pub中的内容，而是在ssh config中也指定id_rsa文件的位置

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920121848.png)

- 对于Linux或者是Mac， 你可以直接在Terminal中完成以上操作：
    - (本质上是因为windows的cmd下的ssh并没有提供`ssh-copy-id`命令，所以需要手动处理)
    - `ssh-keygen -t rsa -b 4096`
    - `ssh-copy-key -i id_rsa.pub ztc-eva7` 并遵从引导，输入密码，并且再次尝试登录


### ssh转发图形界面

1. Ubuntu/Linux上的配置
    - 网上的资料较多，可以自行查询
2. Mac上可以尝试一些图形界面转发软件
    - 网管并不用Mac，不清楚怎么使用
3. Windows: 如果你使用了WSL，可以安装xlaunch工具，来将wsl的图形转发映射到你的windows主机

- 常用的调试手段，先用apt安装`x11-apps`，然后使用`xclock`看有没有相关的
- MobaXTerm自带图形界面转发，如果使用fpga服务器有开发软件图形界面需求的同学们，可以尝试直接使用此工具，运行如vivado，即可在本机弹出一个窗口
- 也可以采用vnc的方式连接图形界面，vnc需要配置服务，对于fpga服务器，没有container，不要自己到服务器上配置，请联系服务器负责人操作开通


### 让服务器用上本机的梯子

> 写这个post的motivation是github经常被墙，给我们sync code带来了麻烦，因此需要让服务器用上梯子

- 如果只是为了使用 github ，也可以在 github 上配置 ssh 下载（而不是使用 https ），一般来说不用起飞就可以正常 clone
    - 配置 ssh 的方式可以参考 github 的官方文档 https://docs.github.com/en/authentication/connecting-to-github-with-ssh

- 如果要使用此功能，你的梯子软件**需要能够配置http服务开在某个端口**，如果你是买的那种别人自己host的小飞机，可能就不一定支持这个功能；对于这种情况，网管也没有太好的办法；网管自己对梯子的解决方案是购买JustMySocks的梯子，并且使用V2ray/SS等程序使用。（与小飞机的关系你可以理解为，小飞机的host自己买了很多服务器并且又包装了一层卖）
    - 比如我们的V2ray客户端中就可以设置http-proxy开在本地的10809端口
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211013132726.png)

1. 确定你的梯子的服务开在哪个端口，比如我的http-10809端口,socks5 - 10808端口
    - 在bashrc中添加 `alias setproxy="export http_proxy=http://127.0.0.1:10809; export https_proxy=$http_proxy; echo 'HTTP Proxy on';"`
2. 在ssh登录config中指定remoteForward
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211013133103.png)
    - 这条指令的目的是将你的本地10809端口映射到远端的10809端口，让服务器端用本地梯子
3. 配置你的git，编辑 `~/.gitconfig`中加上
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211013133237.png)
    - 指定了git所使用的proxy端口就是(服务器)本地的10809端口

- debug此过程是否成功
    - 使用wget(命令行下载器)，编辑它的配置文件 `~/.wgetrc` 如下图，也是使用10809端口作为proxy
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211013133336.png)
    - 直接下载google的网页作为测试，如果成功了，现象应该是这样
    - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211013133443.png)

- 为特定的软件设置代理 / 设置linux全局代理，参考[Zhihu文章](https://zhuanlan.zhihu.com/p/58690128)
