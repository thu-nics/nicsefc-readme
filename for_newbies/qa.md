# Trouble-Shooting

## [1] -  CUDA相关

> 有的时候很容易出现cuda没了的情况，有时候smi不出东西，有时候`torch.cuda.is_available()`报错false，记住执行**deviceQuery**是解决问题的主要途径！(下面的1)

### 0. 发现nvdiia-smi没这个命令

- 把`/usr/local/bin`加到环境变量PATH中 

### 1. 发现了Pytorch啊以及各种程序(nvidia-smi)报错 'cuda error'或者cuda不available

- 首先进入`/opt/cuda-x.x/samples/1_Utiliies/deviceQuery`(cuda版本以你具体用到的cuda版本一致)**以sudo形式**执行deviceQuery，观察是否PASS，如果PASS了，那大概率是可以ok了
- 如果报错了`ERROR_CODE=100`的话，先执行`sudo ldconfig`，再执行上面的deviceQuery
- 如果报错了`ERROR_CODE=999`的话，截图并联系网管(在nicsefc-server群里)，帮你在主机上执行一次deviceQuery(理论上重启之后会需要这么做)
- 如果再出现了别的错误，可以先google一下报错指令，如果没有解答的话联系网管

### 2. deviceQuery能够PASS，但是之后nvidia-smi和启动都很慢

- sudo运行`/usr/local/nvidia/bin/nvidia-persistenced`

## [2] - 我怎么登陆不上服务器了？

- 注意，**当你使用ssh软件发现登录不上服务器的时候**，请第一时间，使用terminal进行同样的操作进行登录，检查**是否是ssh软件(ssh config撰写错误)的原因，还是ssh本身的原因**(这是一个好的debug习惯与思路，希望大家牢记)
我们知道，登录服务器(大象装冰箱)，总共分三步: 下面将对以下环节可能出现的问题进行列举，当出现问题时，请**遵循以下步骤，逐步进行排查**
    - 1） 确定你要登录服务器的IP
    - 2）准入对应IP
    - 3）使用ssh登录服务器

### 0. 使用Terminal来进行ssh登录

1. 不用你的诸如vscode等的ssh工具，直接使用terminal手动输入ssh命令进行登录，如果能够正常登录，那么说明**是你SSH工具的问题**，请参考[./login-tools.md](./login-tools.md)的trouble-shooting部分来寻找原因。

### 1. 确定你的服务器IP

1. 使用`ping 域名`的方式，查看你的服务器的IP，如上面所说的，基本都可以看到对应的类似`101.6.68.180`的IP，这一步很可能你是ping不通这个IP的，请查看下一部分，这一步**几乎不可能失败**，如果失败了(在2021-09-18早上，网管发现了这个问题并debug了2h，最后发现是学校的校园网崩了)。大概率是你的网络或者本机的DNS服务出了问题，没有简单解决方案，如果真的是这样，请汇报给网管，或者将本地笔记本电脑切换至手机热点进行测试。

### 2. 确定你的准入状态

2. (这是最大概率的错误情况，现象是输入ssh命令之后没有反应)，上一步你已经知道了域名，首先尝试能否ping通这个域名(如果没有出现`64 bytes from 101.6.68.180: icmp_req=1 ttl=62 time=0.311 ms`这类的，就是ping不通)。如果不是你自己本机网络挂了的话，应该就是服务器的准入掉了，登录usereg，参考上面的部分对步骤1得到的IP进行准入。如果显示“登录成功”，那么大概率，你的问题就已经解决。
    - 如果，本来应该显示"登录成功"的位置一片空白，或者显示“地址不在DHCP表中“，有两种可能，一种是”学校的DHCP表更新延迟了“，建议是**稍等10min**，如果问题还没有解决，请把问题报告给网管(包括你的IP，以及usereg登录失败的截图)，可能是服务器有了硬件问题。
    - 如果准入之后显示登录成功，但是不能ping通，那么，*将问题汇报给网管*
    - 如果准入之后显示登录成功，且能ping通，但是登录的时候不显示东西，那么，**检查你的登录命令！**，大概率**是你登录命令的问题**，检查用户名和IP，如果通过了跳板机等其他中继节点，检查对应的IP以及端口是否正确。

### 3. ssh登录服务器

3. ssh时候提示输入密码(那就一定不是准入的问题)，输入了之后显示Permission Denied，如下图:
    - 可能是你的密码输错了！检查有没有打错，以及**大写锁定**
    - 确定你所登录的域名/IP是否正确，是不是登录错服务器了
    - 你所登录的服务器可能你的用户并没有权限，如果你是登录到其他人的container，请让其他人检查`/etc/login.user.allow`中是否有你的用户名(或者是他们加的时候是不是把你的用户名打错了)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210919205015.png)


## [3] - 服务器如何联网？

- 当你已经能够登录服务器之后，如果你需要访问外网，有以下几种操作方式:

1. 如果是你自己在usereg上"准入"了这个IP，那么你可以自己去将其**下线**，并再选择"校外准入"一次。
    - 有一个很愚蠢的事情是，如果你的这个IP是被别人的校园网账号所准入了，那你无法在usereg中将其准出 (学校的网络usereg服务在这里很愚蠢)，为了解决上面的这种问题，请参考下面的auth-thu的方式
2. 使用auth-thu进行准入准出之间的切换:
    - [auth-thu脚本的官方repo](https://github.com/z4yx/GoAuthing)
    - [auth-thu的使用指南](https://thu.services/services/)
    - 去[release页面](https://github.com/z4yx/GoAuthing/releases)下载对应平台的脚本可执行文件，并且通过scp(或者其他方式)将这个脚本上传到服务器(你只要能登录服务器就能scp)
    - 在服务器上调用`./auth-thu.linux.x86_64 $COMMAND`
        - $COMMAND如果是auth/deauth 表示准入上线、下线
        - 如果是login/logout则表示准出上线、下线
        - [方案1] 假设你现在登录上服务器，你大概率处于准入但非准出的状态，这个时候你可以直接执行`./auth-thu.linux.x86_64 login`,那么就可以完成准出
        - [方案2] 或者你可以先进行`./auth-thu.linux.x86_64 deauth`, 将该IP的准入也下线(不出意外你的ssh登录也会断掉)，然后再在usereg上进行"校外"登录，再登录服务器
            - 由于方案1有的时候可能会报错…这才催生了方案2这种不太高雅的实现方式。
        - 现在试试`ping baidu.com`，看看是不是能ping通了？
            - 注意`sudo apt-get update`命令不能检验是否准出成功，因为如果你换了tuna的清华源的话，那么你安装包的时候仍然走的是清华内网，即使你能成功update，也不说明你能够访问外网了

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920134145.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920134859.png)

