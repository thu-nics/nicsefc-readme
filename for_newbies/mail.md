# 申请服务器资源邮件格式

同学您好：
    你的NICS实验室账号如下（实验室的资源都用这一套账号密码登录）：
    账号: 
    初始密码: 

先阅读一下: 实验室资源使用方法指南： http://nicsefc.ee.tsinghua.edu.cn/wiki/nerv:workstations

实验室资源：
* 实验室主页：https://nicsefc.ee.tsinghua.edu.cn/
* 实验室WIKI（上有各种信息，不一定及时更新）：https://nicsefc.ee.tsinghua.edu.cn/wiki； 新入实验室建议阅读：http://nicsefc.ee.tsinghua.edu.cn/wiki/nerv:101
* 账号管理系统（你可以在这里修改你的密码）：https://ldap.nics.cc (ICP备案之前需要用IP登陆: https://101.6.64.67)
* 实验室服务器：
    * 需要额外申请权限，步骤如下：
        * 先咨询带你的研究生学长/学姐，是否需要自己独立的container，还是只需要他们给你开个登陆他们已有container的权限（很容易，数据可共享，且不需要再联系网管）。
        * 如果确定需要自己独立的container，邮件联系 foxdoraame@gmail.com 和 zhongk19@mails.tsinghua.edu.cn 并抄送你的负责研究生，注明你需要的资源:
            * 任务类型: GPU计算/CPU计算/简单学习为主
            * 硬盘大小: 默认独立空间只有50G, 以及有所有用户shared的空间可以放。一些数据集在服务器share（或eva_share、/dataset等共享的）文件夹已经有了，可以先问一下你的负责研究生。
            * 其它需求
    * 资源概况：http://nicsefc.ee.tsinghua.edu.cn/wiki/op:servers
* 跳板机（在校外如果要访问服务器资源，需要准入+上线，比较麻烦，可以使用跳板机再访问）：跳板机地址: 205.nics.cc (ICP备案之前需要用IP登陆:101.6.64.67) 端口: 42222（ssh xxxx@101.6.64.67 -p42222）

网管部：
宁雪妃 (foxfi)     foxdoraame@gmail.com
钟凯              zhongk19@mails.tsinghua.edu.cn

BANNER：如果有网络、驱动等问题可以联系网管部。但网管部不负责配环境。

---

# 网管回复邮件格式

> 我自己也用这个来备份呢

同学你好

	你申请的计算服务器资源如下：

	域名：zhuyu.eva7.nics.cc 
        目前IP: 101.6.69.101 (域名服务可能目前还没跟上，先可以用静态IP登录，大概半小时之后就能使用域名登录)

       【登录说明太长不看版本】
        在校内环境ping你的域名得到静态IP，在usereg.tsinghua.edu.cn中准入待认证之后，就可以使用命令： ssh 你的账户id@域名的方式进行登录。

	一些说明：	
        （登录eva服务器指南，请查看 login [NICS Wiki] (tsinghua.edu.cn)）
    
目前学校的管理模式下，需要对服务器（各位使用的container在学校网络看来是独立的ip）进行账号准入才能使用（学校要确认每一台机器是谁在用）。
基本准入方式：在usereg.tsinghua.edu.cn上的“准入待认证”功能下，输入想“准入”的IP和你的tsinghua账号密码，即可准入。选择“校内”的意思是该IP将只能从校内地址访问，只能访问校内地址；选择校外的意思是该IP可以同时访问校内外地址。
服务器目前地址可以用ping域名的方式查看，如果不对，可能是最新的dns没有更新，半小时会更新一次。如果还不行就是挂了。
准入后流量不足可能会下线，建议加一条定时执行指令。比如运行crontab -e编辑定时执行指令，添加：*/5 * * * * /bin/ping -c 20 166.111.8.28 > /home/crontab-ping.log 2>&1。意思是每5分钟运行一次ping指令，结果保存到log里。
选择一种准入方式后，只能先“准入下线”才能换成另一种。如果是自己准入的，可以在usereg网站下线，如果目前是别的同学准入的，可以联系他下线，或者在服务器内拷贝/opt/auth-thu程序，运行进行查看和下线，下线后再在usereg上用自己账号准入（因为在服务器上用auth-thu“准入下线”后你跟服务器的连接就断了）。
我们的container管理模式大家的权限很高，但是尽量不要操作除了/home文件夹以外的其他文件夹，尤其是/opt（保存了cuda和一些常用软件）、/usr/local/nvidia（保存了nvidia相关程序）等。不要自己装gpu驱动（本质是任何硬件驱动都不行）。
/home目录下空间默认50G，此外所有该eva主机上的同学可以共享使用的存储空间有：
/home/eva_share，一般只用来存数据集。
/opt，可以用来分享一些常用软件的安装包，比如conda。
/home/eva_share_users，可以用来保存其他个人文件，可以在不同人的环境间共享文件，空间较大，请在该文件夹下新建一个自己用户名命名的文件夹，然后将自己的文件放进去，非个人用户名的文件夹会被删除；此外如果整个共享空间占用量超过上限，会从占用空间最大的人的文件夹开始提醒删除。
常见场景：
如果想让别人也能登陆你的机器，可以在/etc/login.users.allow里加入该同学的账号，想给他增加sudo权限，可以在/etc/sudoers里添加。
如果GPU运行遇到类似找不到GPU，或驱动版本不符等问题，可以先用sudo运行一下/opt/cuda-XX.X/samples/1_Utilities/deviceQuery/deveceQuery  以及/usr/local/nvidia/bin/nvidia-smi。
如果找不到驱动，可以尝试运行ldconfig。
服务器会坏，网管会删。注意随时保存自己的工作进度和数据。

网管部：
宁雪妃 (foxfi)   foxdoraame@gmail.com
钟凯                zhongk19@mails.tsinghua.edu.cn
赵天辰             suozhang1998@gmail.com