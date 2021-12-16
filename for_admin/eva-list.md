# Workstations-list

> 服务器军需储备, 以及一些硬件调试指南

## 正在服役的主机

| 名称  | tinc IP     | 容器mac | CPU                  | GPU                       | 内存 | 其它     |
| ----- | ----------- | ------- | -------------------- | ------------------------- | ---- | -------- |
| 205   | 10.4.205.1  | -       | -                    | -                         | -    | -        |
| eva0  | 10.4.205.10 | e0:xxx  | 256 AMD EPYC 7H12    | 8*GeForce RTX 3090        | 500G | 万兆网卡 |
| eva1  | 10.4.205.11 | e1:xxx  | 256 AMD EPYC 7H12    | 8*GeForce RTX 3090        | 500G | 万兆网卡 |
| eva2  | 10.4.205.12 | e2:xxx  | 256 AMD EPYC 7H12    | 8*GeForce RTX 3090        | 500G | 万兆网卡 |
| eva3  | 10.4.205.13 | e3:xxx  | 256 AMD EPYC 7H12    | 8*GeForce RTX 3090        | 500G | 万兆网卡 |
| eva4  | 10.4.205.14 | -       | 8 Intel i7-9700K     | 1*GeForce RTX 2080        | 60G  | -        |
| eva7  | 10.4.205.17 | e7:xxx  | 40 Intel Silver 4210 | 8*GeForce RTX 3090        | 500G | -        |
| eva8  | 10.4.205.18 | e8:xxx  | 40 Intel E5-2630 v4  | 8*GeForce RTX 2080 Ti     | 500G | -        |
| eva9  | 10.4.205.19 | e9:xxx  | 40 Intel E5-2630 v4  | 8*GeForce RTX 2080 Ti     | 500G | -        |
| eva10 | 10.4.205.30 | f0:xxx  | 40 Intel E5-2630 v4  | 8*GeForce RTX 2080 Ti     | 500G | -        |
| eva11 | 10.4.205.21 | f1:xxx  | 128 AMD Threadripper | 1*A100-PCIE-80GB          | 250G | -        |
| eva12 | 10.4.205.22 | f2:xxx  | 128 AMD Threadripper | 4*GeForce RTX 3090        | 250G | -        |
| eva13 | 10.4.205.33 | -       | 64 Intel Gold 6226R  | 2*2080/3090/V100/AMD9100  | 250G | -        |
| eoe0  | 10.4.205.50 | d0:xxx  | 24 Intel E5-2643 v4  | 4*GeForce GTX 1080 Ti     | 90G  | -        |
| eoe1  | 10.4.205.51 | d1:xxx  | 16 Intel i7-6900K    | 4*TITAN X (Pascal)        | 40G  | -        |
| fpga1 | 10.4.205.41 | -       | 20 Intel i9-7900X    | -                         | 60G  | -        |
| fpga2 | 10.4.205.42 | -       | 32 Intel Silver 4208 | -                         | 40G  | -        |
| fpga3 | 10.4.205.43 | -       | 32 Intel Silver 4208 | -                         | 40G  | 2*U200   |
| fpga4 | 10.4.205.44 | -       | 32 Intel Silver 4208 | -                         | 40G  | 2*U200   |

# 调试技巧

## 0. 重启服务器之后

1. 需要重新准入当前的IP
2. 需要在主机上执行deviceQuery
3. 对于某些服务器重启之后所有container都会启动，需要手动关闭

## 1. 硬件相关调试指南

1. 由于config文件中一般默认选取下面一个网口，config文件中默认写的位置，如果换网口会导致container拿不到IP
     - eva0/1/2/3比较特殊，他们有4个物理网口，其中`eno0/1`是显卡中间的两个网口，`ensp`是主机上的
     - 对于网口规则，对于老的的主机式EOE，接上面的网口； 对于eva服务器，遵从左上原则(目前只有eva7和fpga是用的左边的网口)

## 2. 软件相关调试指南

### 2-1: 服务器维护相关(程序)

> 出现过许多次某个程序卡死 / 挖矿程序等情况，导致服务器不正常

1. 使用Top命令确定当前服务器上进程情况， Example: `ps auxw|head -1;ps auxw|sort -rn -k4|head -10 `
     - （改成k3就是CPU，k4是内存，改成k5就是虚拟内存）
     - 📆: 登录不上container(从主机也attach不进去)，可能有原因是某个container出现了内存泄露，导致整个服务器被挤满了，这个时候只能上主机找到程序之后kill掉。(2021-10-14)

2. 检查用户登录记录
     - `last`: 直接查看该服务器的登录纪录

3. 检查某个PID是否在Container内
     - 进入`/proc/$pid/cgroups` 查看，如果在container中里面会写 `/lxc/xxx/...` 

4. 查看某个进程的路径  `pwdx $PID`
     - 该命令不能区分是否在container内，所以参考上面方法先查看pid的来源

### 2-2: 服务器维护相关(登录)

-  ping得通，但是ssh登录不上： **准入问题**
- ping得通，甚至能够现实输入密码，但是始终Perimssion Denied，确认不是输错密码的情况下，可能是原本的container网络fail掉了，这个IP被更新给了其他的服务器，而我们的DNS由于没有更新仍然显示老IP，这种情况大概率是主机fail掉了，要确认主机是否fail掉了，登陆上其他服务器，通过tinc内网来尝试ssh上去，比如`ssh 10.4.205.1*`,确认服务器是否物理网络fail掉，还是校园网抽风
- 准入出现问题，诸如不在DHCP表中等等，大概率都是校园网抽风了，等待一段时间

### 2-3: 修改container的rootfs大小

``` bash

lxc-stop -n $ROOTFS_NAME
truncate -s +${NEW_SIZE}G $ROOTFS_PATH
e2fsck -f $ROOTFS_PATH
resize2fs $ROOTFS_PATH 
lxc-start -n $ROOTFS_NAME

```

- 注意这里的**G是一定要加的**否则truncate的命令可能会有问题，可以使用`truncate -s +100G /data/xxx`这样的方式，会更安全一些。之后的一步也改成 `resize2fs /data/xxx`
     - 📆: 2021-11, ztc因为在扩容Container的时候没有用加号和G，导致将container变成了100B，并且没有及时fix磁盘问题，导致container报废。


### 2-4: 修改base的rootfs template

* 简单的方式：
     - 就将这rootfs文件挂载到某个目录下`mount rootfs /media/rootfs`,然后chroot到`chroot /media/rootfs`,进行修改文件系统，然而该种方式可能存在不能使用apt安装库的方式

- 合理的方式：
     * 由于lxc可以在`/var/lib/lxc`中直接修改配置以及rootfs,所以直接在该目录下新建一个sandbox文件夹，并且拷贝一份其他container的config文件过来(修改name，rootfs的位置，以及网卡的hwaddr，否则会报错address already in use)，并且将需要修改的rootfs软连接到这里来。直接lxc-attach进这个container进行修改。

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210909125956.png)

### 2-5：服务器之间迁移Container

- 直接拷贝整个`/var/lib/lxc/xxx` 整个文件夹(包括了config以及rootfs文件)到新的服务器，并**修改config**
     - 检查虚拟的网卡HW ADDR，如`lxc.net.0.hwaddr = 00:15:04:25:e7:47`,其中最后两位有意义，e7表示了eva7(d表示了eoe)，而47表示了在eva7上被构建的第47个container，当迁移到新服务器中**这两条都要修改**
     - 进入Container，**修改hostname**，确认`/etc/hosts; /etc/hostname; hostnamectl status`,更新container的hostname，否则可能会导致域名服务的混乱（当出现了两个ztc.eva7的时候，dns解析只会对应到一个）

### 2-6：修改服务器Route规则

-  在ubuntu18.04之后由于route规则的改变，route规则会将`10.0.3.1`的一条放在首条，导致在外面无法登录，甚至完全无法ping通(即使准入)
     * 通过修改不同ip route规则的metric可以解决:，在`/etc/networkd-dispatcher/routable.d`中添加了一个脚本，内容为`ifmetric eth1 200` (通过ifmetric库实现，需要加到我们的默认rootfs里)，这个脚本将eth1(主机上由lxc-bridge所产生的虚拟网卡的route)优先级变低，metric的值越高就越不优先，需要安装`ifmetric这个package`（通过修改rootfs安装）

### 2-7： 磁盘相关的问题

- 我们的Conatiner磁盘规划有一个问题是：构建container的时候其实没有限制，有的时候会忘记而导致**磁盘超发**，
     - 在`/var/lib/lxc/xxx` 中看rootfs的文件的size，是container实际被使用的大小，而不是rootfs大小的上限，所以如果超发了，到某个时刻才用满
     - 特定container的大小，得进入Container执行`df -kh`才能知道
- 📆：由于**磁盘超发**，导致某container的磁盘写错误而导致变成readonly filesystem,(会直接显示整块磁盘满了，但是实际上它没满)，直接停止该硬盘上的所有container，并umount且运行磁盘检(`fsck`)，并进行搬运。

### 2-8: 开机自启动

> 在ubuntu 16以及18以及20都采用了不同的自启动方式，20.04用systemd，需要改动的内容较多，在此记录

1. 修改systemd中的rc-local的服务 `vim /lib/systemd/system/rc-local.service`，在末尾加上

```
[Install]
WantedBy=multi-user.target
Alias=rc-local.service
```

2. 创建rc.local文件并给其可执行权限 `vim /etc/rc.local; chomd a+x /etc/rc.local`

3. 加入需要启动的东西，记得要加上`!#/bin/bash`,然后最好加一个文件流重定向来检查是否成功 `echo "XXX" /usr/local/autostart.log`, 注意最后一定要加`exit 0`

4. 创建软连接(enable服务) `ln -s /lib/systemd/system/rc-local.service /etc/systemd/system`
     - 这个步骤和`systemctl enable rc-local.service`等效

## 3. 其他Tool的使用

### 3-1: LDAP php前端

1. 搭建在205主机的80端口上，目前个人的解决方法是映射到本机的`12888`端口上，并在Chorme浏览器中下载插件`Modheader`插件，添加`ldap.nics.cc`,应该能够打开一个php的界面(注意不要挂梯子)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923174940.png)

2. 查找最新的用户，进入search，将uidNumber加入到`Show Attributes / Order by`当中，应该就会按照创建用户的顺序进行排列，注意下面的`Search Results`要设置的大一些

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923175458.png)

3. 进入用户界面，注意可以修改用户名密码，但是不能看到密码
      - 修改用户名的注意cn，username以及home目录位置都需要修改   

### 3-2: 网站相关

- 更方便的方式直接使用 `https://nicsefc.ee.tsinghua.edu.cn/nicsadmin/signup.html`
     - 输入注册
     - 默认密码 **口口相传**
     - 验证问题 **口口相传**
- 网站的后台权限控制，进入 `Authorization`，搜索用户修改权限
     - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211107205735.png)

## -1. 强制调试

> 2021-0928，在调试nslcd时候出现了大问题，将用户系统直接搞崩溃，让服务器完全无法登录，主机的sudo也被破坏

1. 使用u盘插入系统进行强制修复，开机F11进Boot选项，从u盘进行登录
2. 登录之后try ubuntu系统，找到Linux主文件系统所在的盘符(这次是一个400个G的，它的硬盘上还有其他类似swap之类的空间分区，将主要的分区给挂载上)
3. `chroot /media/mount_point`, 就可以以sudo身份进入系统，然后执行操作(我的case是修正了一个文件`/etc/nsswitch.conf`), 之后登录系统就正常了

# Trouble Shooting for admin

## 服务器登录不上了

