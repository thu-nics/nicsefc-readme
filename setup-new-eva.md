# 配置一个新服务器

## -1. 硬件配置以及远程连接准备

1. 将服务器setup成能正常用的状态(在楼下机房能够正常查看到ip，并且准入了之后能够用)
    - 查看文件系统挂载以及IP
    - 没有ifconfig用 ip addr
    - 修改账户的默认密码防止被黑入


2. 网络相关

通过`ifconfig`能够看到IP(如果还没有安装这个命令用`ip addr`)
并且能够用校园网准入这个IP，并且正常登录服务器。
并且能够ping通`101.6.64.67`的205跳板机

## 0. Unpack 下一些必要的文件

> 可能后续会整理一个packge，打包上传，内部主要包含了base-rootfs，各种驱动以及常用包等内容

2. 拷贝一系列文件 (等待整理成一系列文件进行备份)
    - auth-thu脚本，用于校外准入
    - 新的source.list以换源: (按照ubuntu版本定，可以在tuna上进行查看与复制)

## 1. 网络相关(Tinc/HostName)

> tinc: 用作VPN工具，建立了一个虚拟局域网10.4.205.x网段(其实是一个组网mesh)。以205主机(`IP: 101.4.205.*`)为中心，构建起一个mesh组网，保证我们的任意两台服务器之间都可以通过这个内网进行访问，用处是用来传输动态的IP，用于DNS域名服务。

### 1.1 修改hostname，给服务器一个名字:

不用重启的改域名方法`hostnamectl set-hostname eva2`，用`hostname`来确定是否修改成功了
查看`/etc/hosts`中有没有原本的域名，记得也需要一起改过来

###  1.2 设置tinc 

> (编辑了一个setup-tinc.sh的脚本，输入好该服务器的`域名`和`IP名`)，下面拆解具体流程

1. 预先准备的用apt安装一些库 `tinc，net-tools`
2. 拷贝公钥和配置文件(optional), `mkdir -p /etc/tinc/nics;  scp -r -P22222 zhaotianchen@101.6.64.67:/etc/tinc/nics/hosts /etc/tinc/nics/hosts`
    - 注意`scp -P`来指定端口，是大写的P哦，而ssh用的是小写的p。
    - 这里的22222端口表示登录的是205主机，只有网管有权限登录，其他用户的42222端口登录跳板机，本质上是登录上了205机器的一个container
3. 创建本机tinc config文件在`/etc/tinc/nics/tinc.conf`
    - 与模板不同的是只需要修改Name为这台服务器的名字就可以了

```
Name = {SERVER_NAME}
ConnectTo = nics205
ConnectTo = nicsblack
DeviceType = tap
Mode = switch
```

4. 生成密钥,存储文件按默认直接回车即可  `tincd -n nics -K`，默认应该会出现在`/etc/tincs/nics/eva2`（以eva2是hostname为例子）

5. [关键] 将该密钥拷贝回205服务器 `scp -P 22222 /etc/tinc/nics/hosts/$HOST zhaotianchen@101.6.64.67:/tmp/$HOST`
    - (由于205主机的etc目录需要root权限，也不适合给与远程对205主机进行root操作的权限，所以先复制到一个目录，再手动登上205进行复制)
    - **注意：**setup-tincs脚本只执行到copy到tmp目录这一步，下面还需要**手动登录上205进行sudo的复制操作**

6. 配置tinc-up与tinc-down,按如下编辑两个文件并放在`/etc/tinc/nics`文件夹下
    - 其中的number有一些配置规则，需要预先分配(如果使用setup-tincs脚本的话在一开始定义好NUMBER变量)

> number的命名规则：number表示了在nics的虚拟tinc网络中分配给当前服务器的固定IP，为了便于区分，故作以下规定：正统Eva服务器从10开始命名到23(eva0为10，eva1为11，eva10为20，最多支持到eva13)；报废之后使用的备用老eva(EOE)从50开始(eoe0为50)，以此类推，其他类型的服务器规则后续补上。

tinc-up

```
#!/bin/sh
ip link set $INTERFACE up
ip addr add 10.4.205.<NUMBER>/24 dev $INTERFACE
ip route add 10.4.205.0/24 dev $INTERFACE
```
tinc-down

```
#!/bin/sh
ip route del 10.4.205.0/24 dev $INTERFACE
ip addr del 10.4.205.<NUMBER>/24 dev $INTERFACE
ip link set $INTERFACE down
```

7. 编辑启动文件`/etc/tinc/nets.boot`中加入`nics`(这文件里就这几个字母)，表示启动时候启动nics网络

8. 启动tinc (最容易出问题的一步)
    - ( 由于ubuntu20.04之后需要手动enable(在nets.boot文件的注释中有)，需要执行 `systemctl enable tinc@nics`（只需要运行一次）)
    - 执行`tincd -n nics`

- **Checkpoint:** 如果满足:
    - `service tinc status` 会显示active(exited),但是其实ok
    - 使用`ifconfig`能够看到nics的子网络，自己的ip应该是`101.4.205.$NUMBER`
    - 能够ping通205主机 `ping 10.4.205.1`

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917213509.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917213219.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917174342.png)





## 2. 网络相关(Tinc/HostName)


`service nslcd/nscd restart`
nslcd是为nsswitch这种机制提供ldap的backend的服务，ldap的服务本身由slapd管理

主机上的`/etc/nslcd.conf`

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917182035.png)
 -
container中的 

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917182120.png)


> 采购清单: 1）T5 SSD备份重要信息    2) 小屏和线(VGA)    


# 相关素材

1. [yuque - zhongkai的网管教程](https://www.yuque.com/doctor-kaizhong/lab-network/zs137p)