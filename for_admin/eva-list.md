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
| eva13 | 10.4.205.33 | -       | 64 Intel Gold 6226R  | 2\*2080/3090/V100/AMD9100 | 250G | -        |
| eoe0  | 10.4.205.50 | d0:xxx  | 24 Intel E5-2643 v4  | 4*GeForce GTX 1080 Ti     | 90G  | -        |
| eoe1  | 10.4.205.51 | d1:xxx  | 16 Intel i7-6900K    | 4*TITAN X (Pascal)        | 40G  | -        |
| fpga1 | 10.4.205.41 | -       | 20 Intel i9-7900X    | -                         | 60G  | -        |
| fpga2 | 10.4.205.42 | -       | 32 Intel Silver 4208 | -                         | 40G  | -        |
| fpga3 | 10.4.205.43 | -       | 32 Intel Silver 4208 | -                         | 40G  | 2*U200   |
| fpga4 | 10.4.205.44 | -       | 32 Intel Silver 4208 | -                         | 40G  | 2*U200   |

## 硬件相关调试指南

1. 由于config文件中一般默认选取下面一个网口，config文件中默认写的位置，如果换网口会导致container拿不到IP
     - eva0/1/2/3比较特殊，他们有4个物理网口，其中`eno0/1`是显卡中间的两个网口，`ensp`是主机上的
     - 对于网口规则，对于老的的主机式EOE，接上面的网口； 对于eva服务器，遵从左上原则(目前只有eva7和fpga是用的左边的网口)


## 软件相关调试指南

1. 使用Top命令，以及`ps auxw|head -1;ps auxw|sort -rn -k4|head -10 `
     - （改成k3就是CPU，k4是内存，改成k5就是虚拟内存）
     - 有时候登录不上container，可能有原因是某个container出现了内存泄露，导致整个服务器被挤满了，这个时候只能上主机找到程序之后kill掉。(2021-10-14)

2. 检查用户登录记录
     - `last`: 直接查看该服务器的登录纪录

3. 检查某个PID是否在Container内
     - 进入`/proc/$pid/cgroups` 查看，如果在container中里面会写 `/lxc/xxx/...` 

4. 查看某个进程的路径  `pwdx $PID`
     - 该命令不能区分是否在container内，所以参考上面方法先查看pid的来源
     - 查看某个用户的使用 `top -u $USERNAME`

5. 查看磁盘IO
   * 现象: vim保存文件卡顿，但是tmux切换窗口不卡顿
   * 使用`iotop`命令，可能需要手动安装一下，在container内部是会报OSError，需要主机上的sudo权限

### 修改container的rootfs大小

``` bash

lxc-stop -n $ROOTFS_NAME
truncate -s ${NEW_SIZE}G $ROOTFS_PATH
e2fsck -f $ROOTFS_PATH
resize2fs $ROOTFS_PATH ${NEW_SIZE}G
lxc-start -n $ROOTFS_NAME

```

- **注意！** 执行truncate的时候**一定不能忘记File的Size中的G**，如果失败了，不要进行额外的操作！否则可能会导致container坏掉！应该在e2fsck之后直接将其truncate回原本的大小！
     - 为了避免把它变小，可以将`truncate -s+300k $FILEPATH`

### 0. LDAP php前端

1. 搭建在205主机的80端口上，目前个人的解决方法是映射到本机的`12888`端口上，并在Chorme浏览器中下载插件`Modheader`插件，添加`ldap.nics.cc`,应该能够打开一个php的界面(注意不要挂梯子)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923174940.png)

2. 查找最新的用户，进入search，将uidNumber加入到`Show Attributes / Order by`当中，应该就会按照创建用户的顺序进行排列，注意下面的`Search Results`要设置的大一些

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923175458.png)

3. 进入用户界面，注意可以修改用户名密码，但是不能看到密码
      - 修改用户名的注意cn，username以及home目录位置都需要修改   

### 1. 修改的IP route的优先度

* 在ubuntu18.04之后由于route规则的改变，route规则会将`10.0.3.1`的一条放在首条，导致在外面无法登录
* 通过修改不同ip route规则的metric可以解决:
* 实现方式(目前已经在我们eva7上的base-rootfs中修改完成了)
  * 在`/etc/networkd-dispatcher/routable.d`中添加了一个脚本，内容为`ifmetric eth1 200` (通过ifmetric库实现，需要加到我们的默认rootfs里)
  * 这个脚本将eth1(主机上由lxc-bridge所产生的虚拟网卡的route)优先级变低，metric的值越高就越不优先
  * 另外还需要安装`ifmetric这个package`

#### 2. 修改rootfs-template

* 由于lxc可以在`/var/lib/lxc`中直接修改配置以及rootfs
* 所以直接在该目录下新建一个sandbox文件夹，并且拷贝一份其他container的config文件过来(需要修改name，rootfs的位置，以及网卡的hwaddr，否则会报错address already in use)，并且将需要修改的rootfs软连接到这里来

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210909125956.png)

* lxc-ls查看应该能够看到新的sandbox container了，尝试start它，(可能会报错所以可以用 `lxc-start sandbox --logfile ./logs` 将报错信息打印到log文件中)
  * 如果启动遇到了一些问题需要改动这个rootfs，那么就将这个文件给挂载到某个目录下`mount rootfs /media/rootfs`,然后chroot到`chroot /media/rootfs`,进行修改文件系统
* lxc-attach进去，做改动，然后也不用保存，自动就改了

- 在eva1上进行测试搭建了mark0 container，绑定了eva1上的新rootfs，以后请从这里进行cp

### 3. 自动化保持准入状态 - (Crontabs)

* 入网的脚本auth-linux我放在205的`/home/eva_share`下了，之后可以从这里scp，以及跳板机上`scp -P 42222 zhaotianchen@101.6.64.67:/home/eva_share/auth-thu.linux.x86_64 ./`

* 在我的服务器的home目录下写了一个config，然后只要直接执行./auth就可以了，需要写一个crontab
  * [19. crontab 定时任务 — Linux Tools Quick Tutorial (linuxtools-rst.readthedocs.io)](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html)
  * still not safe enough, since should save passwd

* 写一个crontab脚本 `ping info.tsinghua.edu.cn` 让它一直在内网中, and use the ztc.eva7.nics.cc to login

* 创建一个脚本之后，`crontab $FILENAME`,之后`contab -l`查看什么正在执行
  * `crontab -e `直接编辑
  * `crontab -r` 删除

### 4. 迁移老服务器成EOE

> revive的老服务器将以EOE命名，参考量产机，又名End Of Evagelion，表示退役之后重新加入使用的Eva
> EOE系列的container的Mac地址中的倒数第二个字节(2个16进制数)，为`dN`的形式，为了与eva的`eN`区分开，否则会造成dhcp错误
> EOE系列的tinc地址，将从`10. 4.205.50`开始进行排列(tinc-up与tinc-down中)

* [2021-09-11] 尝试将老eva7(现EOE-0)进行复活的时候，由于没有修改lxc config，导致container与新eva7的container中share了硬件hwaddr，从而导致DHCP步骤出错，让同一个ip地址被两个虚拟lxc环境，登录时出错： 
  * ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210911195914.png)
* 需要修改lxc-config中的两个位置的hw-addr:
  * ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210911202514.png)
  * 我们一般的rule是e7表示eva7； f0表示eva0； 对eoe0，我暂时改成了d0。
  * `for name in `lxc-ls`; do sed -i "s/e5/d1/g" ./$name/config; done`
  * 完成了这一步理论上就可以正常登录了(用绝对IP)
* **目前对于eoe0只对了最新的wcy container进行了修改，如果老container需要起来的话，也需要做类似的操作，但是目前还没做**
* 为了让域名等生效，还需要配置好tinc以及dns，参考[凯哥的网管总结 - 网络配置](https://www.yuque.com/doctor-kaizhong/lab-network/lo5q51)中的整个流程
  * 修改hostname: 需要修改 `/etc/hostname` 以及 `/etc/hosts` ，使用hostname命令就可以看到hostname，由于实验室用到了tinc服务，在`/etc/tinc/tinc.conf`中也做修改，通过tinc将新的域名组推到205中，便于205上的DNS服务器更新域名解析。
* 即便最后`service tinc status`显示tinc-up有`exit with error code2`，但是能够正常的ping通`10.4.205.1`(205主机的ip)，暂时没有问题了；以及执行过了crontab脚本中将信息从tinc推送到205部分逻辑，也是ok的。
* 已经建立好的container的hostname，进入文件系统直接修改`/etc/hostname`这个是建立container的时候就确定的，在`fork_lxc_new.py`文件中，使用了`socket.gethostname()来获取本机的hostname`，下面一段逻辑是创建container的时候，将这些信息写入到了template_rootfs的`/etc`所对应的文件中
  * ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210911203937.png)


## -1. 强制调试

> 2021-0928，在调试nslcd时候出现了大问题，将用户系统直接搞崩溃，让服务器完全无法登录，主机的sudo也被破坏

1. 使用u盘插入系统进行强制修复，开机F11进Boot选项，从u盘进行登录
2. 登录之后try ubuntu系统，找到Linux主文件系统所在的盘符(这次是一个400个G的，它的硬盘上还有其他类似swap之类的空间分区，将主要的分区给挂载上)
3. `chroot /media/mount_point`, 就可以以sudo身份进入系统，然后执行操作(我的case是修正了一个文件`/etc/nsswitch.conf`), 之后登录系统就正常了

# Trouble Shooting for admin

1. ping得通，登录不上
     - 大概率是因为在罗姆楼所以能ping通，但是登录不上
2. 登录的上205，但是ping服务器完全不出东西
     - 大概是205正在进行域名更新，稍等几min
3. 在服务器主机上进行了网络相关操作之后发现ping不通过各种东西
     - 不用急着重启，可能只是**准入被搞坏了，重新准入一下**
4. 登录不上container
     * 有时候可能是某个container出现了内存泄露，导致整个服务器被挤满了，这个时候只能上主机找到程序之后kill掉。(2021-10-14)
5. 磁盘问题：
     * 由于磁盘超发，导致某container的磁盘写错误而导致变成readonly filesystem,(会直接显示整块磁盘满了，但是实际上它没满)，直接运行磁盘检查，并进行搬运。


# 其他管理 - 网站

## 注册新用户

- Win下： 修改host文件，加上这一条
     - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211107205317.png)
- 在插件中添加一条request head(注意之后与LDAP的服务进行切换)
     - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211107205430.png)
- (由于我配置了端口转发，访问这个url，下面这个更稳定)
     - `http://nicsefc_old.ee.tsinghua.edu.cn:12888/internal/auth/login/?next=/internal/`
     - `http://nicsefc_old.ee.tsinghua.edu.cn:12888/internal/auth/signup/`

---

- 更方便的方式直接使用 `https://nicsefc.ee.tsinghua.edu.cn/nicsadmin/signup.html`
     - 输入注册
     - 默认密码 **口口相传**
     - 验证问题 **口口相传**

## 新网站后台权限控制

- 进入 `Authorization`，搜索用户修改权限
     - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211107205735.png)
