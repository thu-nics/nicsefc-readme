# 申请服务器资源邮件格式

网管好！

我是： (你的姓名，年级，以及研究方向)
负责研究生：(你的负责研究生的名字)
我的实验室账号已经完成注册，用户名为: ()

我需要申请服务器资源的原因： (你所参与的工作的内容)
我的硬件需求有： (使用GPU/CPU, 如果你对CUDA版本以及GPU卡的版本有要求，请一定预先查询清楚，这与网管为你在哪台服务器上建container有关)

我已经阅读过了 nicsefc-readme (https://github.com/A-suozhang/nicsefc-readme) 中的readme以及login的相关部分

(将此邮件发送给网管，并抄送你的负责研究生)

宁雪妃 (foxfi)   foxdoraame@gmail.com
钟凯                zhongk19@mails.tsinghua.edu.cn
赵天辰             suozhang1998@gmail.com

---

# 网管回复邮件格式

> 我自己也用这个来备份呢

同学你好

你申请的计算服务器资源如下：

域名：zhuyu.eva7.nics.cc 
    目前IP: 101.6.69.101 (域名服务可能目前还没跟上，先可以用静态IP登录，大概半小时之后就能使用域名登录)

请务必查看git-repo：nicsefc-readme(https://github.com/A-suozhang/nicsefc-readme)  中的login以及readme中的相关内容，并联系你的负责研究生将你拉进nicsefc-server的群。
并阅读完以下内容(如果有不明白的，请参考上面的git-repo中的其他帖子)。
    
目前学校的管理模式下，需要对服务器（各位使用的container在学校网络看来是独立的ip）进行账号准入才能使用（学校要确认每一台机器是谁在用）
基本准入方式：在usereg.tsinghua.edu.cn上的“准入待认证”功能下，输入想“准入”的IP和你的tsinghua账号密码，即可准入。选择“校内”的意思是该IP将只能从校内地址访问，只能访问校内地址；选择校外的意思是该IP可以同时访问校内外地址。
服务器目前地址可以用ping域名的方式查看，如果不对，可能是最新的dns没有更新，半小时会更新一次。如果还不行就是服务器存在硬件问题，请联系网管。
准入后流量不足可能会下线，建议加一条定时执行指令。比如运行crontab -e编辑定时执行指令，添加：*/5 * * * * /bin/ping -c 20 166.111.8.28 > /home/crontab-ping.log 2>&1。意思是每5分钟运行一次ping指令，结果保存到log里。
选择一种准入方式后，只能先“准入下线”才能换成另一种。如果是自己准入的，可以在usereg网站下线，如果目前是别的同学准入的，可以联系他下线，或者在服务器内拷贝/opt/auth-thu程序，运行进行查看和下线，下线后再在usereg上用自己账号准入（因为在服务器上用auth-thu“准入下线”后你跟服务器的连接就断了）
我们的container管理模式大家的权限很高，但是尽量不要操作除了/home文件夹以外的其他文件夹，尤其是/opt（保存了cuda和一些常用软件）、/usr/local/nvidia（保存了nvidia相关程序）等。不要自己装gpu驱动（本质是任何硬件驱动都不行）。
/home目录下空间默认50G，此外所有该eva主机上的同学可以共享使用的存储空间有：
/home/eva_share，一般只用来存数据集。
/opt，可以用来分享一些常用软件的安装包，比如conda。
/home/eva_share_users，可以用来保存其他个人文件，可以在不同人的环境间共享文件，空间较大，请在该文件夹下新建一个自己用户名命名的文件夹，然后将自己的文件放进去，非个人用户名的文件夹会被删除；此外如果整个共享空间占用量超过上限，会从占用空间最大的人的文件夹开始提醒删除。

常见场景：
如果想让别人也能登陆你的机器，可以在/etc/login.users.allow里加入该同学的账号，想给他增加sudo权限，可以在/etc/sudoers里添加。
如果GPU运行遇到类似找不到GPU，或驱动版本不符等问题，可以先用sudo运行一下/opt/cuda-XX.X/samples/1_Utilities/deviceQuery/deveceQuery  以及/usr/local/nvidia/bin/nvidia-smi。

服务器会坏，网管会删。注意随时保存自己的工作进度和数据。

网管部：
宁雪妃 (foxfi)   foxdoraame@gmail.com
钟凯                zhongk19@mails.tsinghua.edu.cn
赵天辰             suozhang1998@gmail.com