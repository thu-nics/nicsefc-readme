# 配置一个新服务器

# -1. 硬件配置以及远程连接准备

1. 将服务器setup成能正常用的状态(在楼下机房能够正常查看到ip，并且准入了之后能够用)
    - 查看文件系统挂载以及IP
    - 没有ifconfig用 ip addr
    - 修改账户的默认密码防止被黑入


2. 网络相关

通过`ifconfig`能够看到IP(如果还没有安装这个命令用`ip addr`)
并且能够用校园网准入这个IP，并且正常登录服务器。
并且能够ping通`101.6.64.67`的205跳板机

# 0. Unpack 下一些必要的文件

> 可能后续会整理一个packge，打包上传，内部主要包含了base-rootfs，各种驱动以及常用包等内容

2. 拷贝一系列文件 (等待整理成一系列文件进行备份)
    - auth-thu脚本，用于校外准入
    - 新的source.list以换源: (按照ubuntu版本定，可以在tuna上进行查看与复制)

# 1. 网络相关(Tinc/HostName)

> tinc: 用作VPN工具，建立了一个虚拟局域网10.4.205.x网段(其实是一个组网mesh)。以205主机(`IP: 101.4.205.*`)为中心，构建起一个mesh组网，保证我们的任意两台服务器之间都可以通过这个内网进行访问，用处是用来传输动态的IP，用于DNS域名服务。

## 1.1 修改hostname，给服务器一个名字:

不用重启的改域名方法`hostnamectl set-hostname eva2`，用`hostname`来确定是否修改成功了
查看`/etc/hosts`中有没有原本的域名，记得也需要一起改过来

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917221004.png)

##  1.2 设置tinc实现局域网

> (编辑了一个setup-tinc.sh的脚本，输入好该服务器的`域名`和`IP名`)，下面拆解具体流程

1. 预先准备的用apt安装一些库 `tinc，net-tools`
2. 拷贝公钥和配置文件(optional), `mkdir -p /etc/tinc/nics;  scp -r -P22222 zhaotianchen@101.6.64.67:/etc/tinc/nics/hosts /etc/tinc/nics/hosts`
    - 注意`scp -P`来指定端口，是大写的P哦，而ssh用的是小写的p。
    - 这里的22222端口表示登录的是205主机，只有网管有权限登录，其他用户的42222端口登录跳板机，本质上是登录上了205机器的一个container
3. 创建本机tinc config文件在`/etc/tinc/nics/tinc.conf`
    - 与模板不同的是只需要修改Name为这台服务器的名字就可以了

``` bash
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

``` bash
#!/bin/sh
ip link set $INTERFACE up
ip addr add 10.4.205.<NUMBER>/24 dev $INTERFACE
ip route add 10.4.205.0/24 dev $INTERFACE
```
tinc-down

``` bash
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


## 1.3 设置dns系统

> 实验室使用的是Cloudfare的域名服务，在205主机上向cloudfare的DNS服务器定期更新域名。购买了`**.nics.cc`的域名，由于学校的服务器ip是由dhcp得到的，是动态更新的。所以上文中利用tinc搭建起了虚拟的nics局域网以控制固定IP(`10.4.205.xx`)。在服务器上需要做的是将自己的IP地址定期往205主机进行传输

1. 安装一些软件 `apt-get install redis-tools, lxc, lxc-utils`
2. 在新机器上编辑上述文件，命名为ip-update.sh，修改上面的网口名<NET_INTERFACE_NAME>为新机器的网口如eno1（ifconfig可以查到），编辑好后注意修改可执行权限，并放到/usr/local/bin/下等待执行。这一文件的作用是将本机的名称和公网ip发送给10.4.205.1；( 结果应该是有几个有效的container的IP就会显示几个ok )
3. 然后将该脚本加入crontab进行定时执行`crontab -e`进入编辑器之后使用`*/5 * * * * /bin/bash /usr/local/bin/ip-update.sh > /home/ubuntu/crontab-ip-update.log 2>&1` （请执行一次测试一下，有些服务器上没有ubuntu这个用户了，可能会导致因为没有这个log路径而导致dump失效的问题，所以需要check一下，比如改成`/home/work`）

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917214455.png)

``` bash
#!/bin/bash

REDIS_HOST="10.4.205.1"
REDIS_PREFIX="DNS:nics.info.:"
CURRENT_HOST=`hostname`
IFACE=<NET_INTERFACE_NAME>

for name in $(lxc-ls -1 --running); do
    for ip in $(lxc-info -iH --name ${name}); do
        if [[ ${ip} == "166.111"* ]]; then
            record_value="{\"value\":\"${ip}\"}"
            key="${REDIS_PREFIX}${name}.${CURRENT_HOST}:A"
            echo ${key}, ${record_value}
            printf $record_value | redis-cli -h ${REDIS_HOST} -x set $key
        fi
        if [[ ${ip} == "101.6."* ]]; then
            record_value="{\"value\":\"${ip}\"}"
            key="${REDIS_PREFIX}${name}.${CURRENT_HOST}:A"
            echo ${key}, ${record_value}
            printf $record_value | redis-cli -h ${REDIS_HOST} -x set $key
        fi
    done
done

IP4=`ip addr show dev $IFACE | grep -m 1 'inet\ ' | sed -e 's/^.*inet \([^ \\]\+\)\/.*$/\1/'`

key="${REDIS_PREFIX}${CURRENT_HOST}:A"
record_value="{\"value\":\"${IP4}\"}"
echo ${key}, ${record_value}
printf $record_value | redis-cli -h ${REDIS_HOST} -x set $key         
```
-  **Checkpoint:**:
    - 执行脚本之后能看到ok
    - 查看crontab使用情况，最好也查看一下crontab的status  `service cron status`

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917215310.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917215521.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917215553.png)

注意本脚本对于eva1做了一些修改，本身的逻辑是"扫网卡0/1是否有连接，有的话把IP丢出去(默认是0优先)"，由于我们的eva1的0网口上搭了网络存储服务，所以有限扫1之后递IP了

> 完成了该脚本之后，将新的服务器的IP信息从服务器上推送到了205的主机上，这时候登录[实验室网站查看DNS](https://nicsefc.ee.tsinghua.edu.cn/internal/dns/)应该可以看到了，但是这时候域名服务还没有正式生效，需要将205主机上的信息推送到cloudfare上，大概过几分钟才会生效；

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917220413.png)

4. (**以下操作是在205主机上进行操作的**) crontab定时执行```python /usr/local/bin/cloudflare_dns.py --zone-id 5bxxxxxxxxxxxxxxx030 --auth-key 3exxxxxxxxxxxxxxxxxxxxxxx8d >>/var/log/cloudflare_dns.log 2>&1```
    - 这一串命令中的有些是密钥，需要代代流传，205主机上不出意外应该不会停crontab，直接从那里复制下来测试以下即可

# 2. 用户系统(LDAP)相关

> 我们的用户系统使用了LDAP(LightweightDIrectoryAccessProtocol)为核心进行管理维护，这样大家就可以直接使用实验室网站上的用户名和密码来登录服务器。其有几个部分组成: 
> 1) Slapd服务: 在我们的系统中负责传输与更新用户名称密码对，在205主机上搭建Slapd master server，用来维护最新版本的用户信息； 服务器主机中搭建起Slapd的slave server，负责从主机抓取，并且通过lxc-bridge(10.0.3.1)给各个container同步ldap数据。
> 2) nslcd服务(nscd服务是它的辅助备份服务): 为nsswitch这种机制提供ldap的backend， 负责从ldap server(主机上)将数据抓给nsswitch
> 3) linux的nswitch(name service switch): 本质上是linux系统层级将各种数据库做名称解析，可以认为是完成了前置服务的数据解析成了linux用户并加以创建
> 我后来查询网络找到了做类似事情的教程[csdn文章](https://blog.csdn.net/sssssuuuuu666/article/details/109738921)以及一个[Chen Jiehua's Blog](https://chenjiehua.me/linux/ldap-auth.html)

## 2.1 Slapd服务相关配置

1. 安装配置slapd的软件安装 `sudo apt-get install ldap-utils, slapd`
    - 在安装slapd的时候，会跳出设置密码，这个密码需要记住，最好和root密码保持一致
    - 注意将原本的slapd的**配置删除**，`/etc/ldap/slapd.d` 
        - [debug 2021-09-28] 由于没有删除原本的配置，导致本机slapd没有获取到205上的用户信息，而导致后续的报错
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210928201608.png)
2. 修改slapd的配置，`/etc/ldap/slapd.conf` 标准的配置文件如下 (也可以去到其他服务器进行拷贝，一样可能因为权限问题不能直接拷贝到当前目录，所以只能先拷贝到`/tmp`)
    - 在新版本中，可能会出现没有这个文件存在的情况，在eva1，2上测试了，直接将其他地方的复制过来能用(despite slapd的版本以及ubuntu系统版本都不一样)
    - 需要修改的部分:
        - 把 @BACKEND@ 替换成 hdb
        - 把 @SUFFIX@ 替换成 dc=nics,dc=info
        - 把 @ADMIN@ 替换成 cn=admin,dc=nics,dc=info
        - 找到 # rootdn 那一行，把注释去掉
        - 在 moduleload back_hdb 之后加一行 moduleload memberof.la
        - 以及在最后还需要加入

``` bash
#Overlay
overlay memberof
memberof-group-oc       groupOfNames
memberof-member-ad      member
memberof-memberof-ad    memberOf

# Consumer
syncrepl rid=123
        provider=ldap://10.4.205.1:389/
        type=refreshAndPersist
        retry="60 10 300 +"
        searchbase="dc=nics,dc=info"
        attrs="*,+"
        bindmethod=simple
        binddn="cn=ldapslave,dc=nics,dc=info"
        credentials="@@@@"
```

3. 启动Slapd服务 `service slapd start`

> slapd: Standalone ldap Daemon

- **checkpoint**：
    - 查看slapd的service情况，应该是active的
    - 配置好后，本机就跟ldap://10.4.205.1:389/一样可以提供ldap的服务了，本机上的container并不连接205，都是通过本机的slave服务器获取的（在container的nslcd配置中显示为ldap://10.0.3.1）

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917223406.png)

## 2.2 nslcd服务配置

> 相当于客户端接入的配置

1. 安装 `apt-get install nslcd`，安装中需要做几个配置(注意这个需要等超级久，大概几分钟)，其中需要进行几个配置
    - LDAP服务器地址填 ldapi:///  ，可以将 ldap://10.4.205.1/作为第二 LDAP 服务器 
        - 注意这里的第一个表示从本地的ldap服务器抓取，后者则是从205上的ldap抓取(当205 ping不通的时候不可用)
    - base 填 dc=nics,dc=info
    - passwd, group, shadow 安装时的界面用空格选中
2. 安装成功后，上述配置保存在 /etc/nsswitch.conf 文件中(之前有一次网管记录，需要将passwd，group，shadow都设置成`compact`现在看起来好像不一定必要)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917224337.png)

* **Checkpoint:** 
    - 执行 `getent passwd`,查看是否是最新的
    - `su xxx`到某些用户看一下(不一定需要知道密码，但是只要不报用户不存在就是成功)
    - 之前曾经报错过getent正常，但是su没人的情况，重启了nsclcd和nsld服务之后修正

3. 确认修改一下 `/etc/nslcd.conf`

主机上的，表示是从本地的slapd slave server来获得信息

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917182035.png)

container中的，表示从10.0.3.1的lxc bridge中来获取信息

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210917182120.png)


## 2.3 管理用户登录权限

> 关于用户登录权限的配置，详细可以参考pam_listfile 的手册。我们在主机上和container里都选用白名单的方式控制。[linux-pam-list](https://www.docs4dev.com/docs/zh/linux-pam/1.1.2/reference/sag-pam_listfile.html)

1. 首先在 /etc/pam.d/common-session 中加入一行： `session required	pam_mkhomedir.so	skel=/etc/skel	umask=0022`
2. 然后在 /etc/pam.d/sshd 的@include common-auth后的一行加入如下脚本 `auth required pam_listfile.so onerr=fail item=user sense=allow file=/etc/login.user.allow`
    - 表示用白名单文件控制能够ssh登陆的用户，只有用户名在白名单文件中的才能登陆 
3. 创建/etc/login.user.allow文件，里面填用户名的白名单，注意将**改了密码的ubuntu也填入**
4. 编辑/etc/sudoers，admin那一行下面加入：  `%srvAdmin ALL=(ALL) ALL`
    - 将sudoer组中的人都加入

# 3. Nvidia-Driver以及CUDA相关

> 需要安装cuda以及driver

1. 整理成了一个脚本，在eva2上的`/home/work/nv`中，`get_nv.sh`，从eva0上拷贝对应文件，执行脚本安装，并软连接

2. 由于链接会导致一些奇怪的问题:
     - 目前我们的eva01237的配置会将`/home/work/opt`链接到container中的opt目录，这一步支持软连接(一般会将`ln -s /opt /home/work`)，但是如果主机的/opt中本身已经有了软连接，在container中将不能看到，所以需要**make sure /opt当中的cuda是文件而不是软连接**(上面的脚本中直接进行了复制)
     - 另外`/usr/local/nvidia`也被挂载到了Container当中
     - 总结一下，安装的位置, 在container中所visible的:
         - CUDA - `/opt/cuda` - （包含了deviceQuery）- 主机上的/opt被连接到了`/home/work/opt`
         - Nvidia-Driver - `/usr/local/nvidia` - (在bin中有nvidia-smi) - 主机上的相同位置

# 4. 启动Container

> 这一部分整理成了一个简单的bash脚本在eva1上`/home/work/setup-lxc.sh` 由于各台服务器上config有微妙的区别，所以我们还是以eva1为准。

1. 从各种地方(其他eva服务器上)，scp各种文件,
    - `/opt`
    - `/home/work/lxc`： 包含了启动脚本以及各种base-rootfs
    - 这一步保证你拷贝的启动脚本和condfig所对应的服务器上的`/home/work`目录下的一些软连接是一致的(否则可能会报错mount不上的问题，由于rootfs中已经有了这些地址)
        - 在目录下新建`/data1/eva_share_users`，并软连接到`/home/work`，`ln -s /data1/eva_share_users`
        - 在`/home/work`下新建`eva_share`以及`opt`
2. 安装python以及jinja  `sudo apt-get install python  python-jinja2`
3. 修改config，在`/home/work/lxc/templates/config`中，主要需要修改的是网络相关的部分(以及mount部分如果想要修改也可以)
    - 'e2'的位置是因服务器而异的，e2表示eva2，d0表示eoe0
    - 框子里的部分因服务器而异，是你的主机上拿到IP的那个网卡

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210922153231.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210922155931.png)

4. 执行`python fork_lxc_new.py`脚本，并且指定硬盘位

---


# 相关素材

1. [yuque - zhongkai的网管教程](https://www.yuque.com/doctor-kaizhong/lab-network/zs137p)
2. 物理/网络，服务器结构图:

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920172513.png)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210920172525.png)

---

# 知识点理解

1. LDAP: 我目前是把它理解为了一个轻量化的可联网的数据库
     - phpLDAPadmin: 我们登录所用的网页端操作
2. NSS (Name Service Switch)： 为通用数据库和名称解析提供了数据来源
    - 源文件包括: `/etc/passwd  & groups & hosts` 或是dns以及Ldap服务
    - 配置文件为`/etc/nsswitch.conf`: 规定了查找特定信息的顺序，以及采取的动作