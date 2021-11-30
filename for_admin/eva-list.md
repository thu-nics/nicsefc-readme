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

### 修改container的rootfs大小

``` bash

lxc-stop -n $ROOTFS_NAME
truncate -s ${NEW_SIZE}G $ROOTFS_PATH
e2fsck -f $ROOTFS_PATH
resize2fs $ROOTFS_PATH ${NEW_SIZE}G
lxc-start -n $ROOTFS_NAME

```

### LDAP php前端

1. 搭建在205主机的80端口上，目前个人的解决方法是映射到本机的`12888`端口上，并在Chorme浏览器中下载插件`Modheader`插件，添加`ldap.nics.cc`,应该能够打开一个php的界面(注意不要挂梯子)

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923174940.png)

2. 查找最新的用户，进入search，将uidNumber加入到`Show Attributes / Order by`当中，应该就会按照创建用户的顺序进行排列，注意下面的`Search Results`要设置的大一些

![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923175458.png)

3. 进入用户界面，注意可以修改用户名密码，但是不能看到密码
      - 修改用户名的注意cn，username以及home目录位置都需要修改   


## 强制调试

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
