# Eva-list

> eva的军需储备, 以及一些硬件调试指南

## 正在服役的Eva

1. Eva0 （all set）
2. Eva1
3. Eva2
4. Eva3: 空的，还未配置
5. 新的两台FPGA服务器



## 硬件相关调试指南

1. 由于config文件中一般默认选取下面一个网口，config文件中默认写的位置，如果换网口会导致container拿不到IP
     - eva0/1/2/3比较特殊，他们有4个物理网口，其中`eno0/1`是显卡中间的两个网口，`ensp`是主机上的
     - 对于网口规则，对于老的的主机式EOE，接上面的网口； 对于eva服务器，遵从左上原则(目前只有eva7和fpga是用的左边的网口)


## 软件相关调试指南



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

### Trouble Shooting for admin

1. ping得捅，登录不上
     - 大概率是因为在罗姆楼所以能ping通，但是登录不上
2. 登录的上205，但是ping服务器完全不出东西
     - 大概是205正在进行域名更新，稍等几min
3. 在服务器主机上进行了网络相关操作之后发现ping不通过各种东西
     - 不用急着重启，可能只是**准入被搞坏了，重新准入一下**
