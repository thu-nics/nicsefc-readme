# 申请服务器资源邮件格式

> 请编辑如下邮件，将下面模板中（）处的说明内容替换成自己的或直接删去
>
> 然后将此邮件发送给网管，并抄送你的负责研究生
>
> 宁雪妃 (foxfi)   foxdoraame@gmail.com
> 钟凯                  zhongk19@mails.tsinghua.edu.cn
> 赵天辰              suozhang1998@gmail.com



邮件主题: Container申请-(xxx)



网管好！

我是： (你的姓名，年级，以及研究方向)

负责研究生：(你的负责研究生的名字)

我的实验室账号已经完成注册，用户名为: (zhaotianchen)

我希望的服务器名字: (比如说 （ztc).evax.nics.cc )


我需要申请服务器资源的原因： 

* (目前container的申请相对严格，只有**你独立有一个单独的项目需要开发，和组内其他同学以及负责研究生的任务差距较大，或者某子任务有非常特殊的硬件需求**才会单开container，如果想要开启自己的container，请详细说明原因，如果单纯只是一个"跑实验"的理由，网管不会受理)
* (如果你与你的负责研究生研究内容类似(which is often the case)，网管很建议**联系你的负责研究生，并申请加入他的Container中**，这样数据搬运备份交互等都会方便许多)
* (如果你只是想学习测试，网管会建议你申请sandbox container)

我的软硬件需求有： 
  - (是否是内存占用较大或频繁读写磁盘的任务)
  - (是否需要GPU，一般用多少)
  - (是否需要FPGA)
  - (是否需要特定的操作系统版本，CUDA驱动版本，GPU型号；如果有特定要求，请一定预先查询清楚，这与网管为你在哪台服务器上建Container有关，默认Container系统是Ubuntu 18.04, 驱动版本及GPU型号不尽相同）

我已经阅读过了 nicsefc-readme (https://github.com/thu-nics/nicsefc-readme) 中的readme以及login的相关部分

祝好！

（XXX）



---

# 网管回复邮件格式

> 我自己也用这个来备份呢

同学你好

你申请的计算服务器资源如下：

域名：xx.eva7.nics.cc 

目前IP: 101.6.69.101 (请准入该域名后登录，域名服务可能目前还没跟上，先可以用IP登录，大概半小时过半小时，以后就能使用域名登录了，**登录前记得先准入！**)


* 使用说明：
  * 请务必查看github repo nicsefc-readme(https://github.com/thu-nics/nicsefc-readme)  中的相关内容，尤其是for_eva_users目录下的内容。
  * 联系你的负责研究生将你拉进nicsefc-server的群。
* 准入说明（下面为要点概述，更多信息请参考github repo中相关文档）：
  *  目前学校的管理模式下，需要对服务器（各位使用的container在学校网络看来是独立的ip）进行账号准入才能使用（学校要确认每一台机器是谁在用）。
  * 基本准入方式是登陆usereg.tsinghua.edu.cn，在“准入待认证”功能下，输入想“准入”的IP和你的tsinghua账号的密码，即可准入。选择“校内”的意思是该服务器将只能从校内地址ssh，也只能访问校内地址；选择校外的意思是该IP可以同时访问校内外地址。
  * 对于一个tsinghua账号来说，同时准入“校外”的IP存在限制，最多5个，达到5个后继续给新的IP准入“校外”，会把之前的挤掉。同时准入“校内”的则不存在数量限制。
  * 服务器目前地址可以用ping域名的方式查看，如果不对，可能是最新的dns没有更新，半小时会更新一次。如果还不行就是服务器存在问题，请联系网管。
  * 准入后一段时间不用会自动下线，建议加一条定时执行指令。请了解一下crontab的相关基础知识，运行crontab -e编辑定时执行指令（第一次打开时要选择编辑器，建议选择vim），添加 */5 * * * * /bin/ping -c 20 166.111.8.28 > /home/your_username/crontab-ping.log 2>&1 这样的一行。意思是每5分钟ping一次学校的网关，结果保存到你自己home目录下的crontab-ping.log文件里。
  * 选择一种准入方式后，只能先“准入下线”才能换成另一种。如果是自己准入的，可以在usereg网站下线，如果目前是别的同学准入的，可以联系他下线，或者在服务器内拷贝/opt/auth-thu程序，运行该程序进行查看和下线，下线后再在usereg上用自己账号准入（因为在服务器上用auth-thu执行“deauth”后你跟服务器的ssh连接就断了）
* Container环境说明
  * 我们的container管理模式大家的权限很高，但是尽量不要操作除了/home文件夹以外的其他文件夹，尤其是/opt（保存了一些cuda的链接库和一些常用软件包）、/usr/local/nvidia（保存了cuda驱动相关程序）等。不要自己装GPU驱动（本质是任何硬件驱动都不行）。
  * /home目录下空间默认100G，此外所有同一个eva主机上的同学可以共享使用的存储空间有：
    * /data/eva_share，一般只用来存数据集。
    * /opt，可以用来分享一些常用软件的安装包，比如conda。
    * /data/eva_share_users，可以用来保存其他个人文件，可以在不同人的环境间共享文件，要使用这个空间，请在该目录下新建一个自己用户名命名的目录，然后将自己的文件放进去，非个人用户名的目录会被删除；此外如果整个共享空间占用量超过上限，会从占用空间最大的人的目录开始提醒删除。
* 常见场景：
  * 如果想让别人也能登陆你的Container，可以在/etc/login.users.allow里加入该同学的账号，想给他增加sudo权限，可以在/etc/sudoers里添加。
  * 如果GPU运行遇到类似找不到GPU，或驱动版本不符等问题，可以先用sudo运行一下/opt/cuda-XX.X/samples/1_Utilities/deviceQuery/deveceQuery以及/usr/local/nvidia/bin/nvidia-smi。
* 【重要】：服务器会坏，网管会删。注意随时保存自己的代码和数据。

网管部：
宁雪妃 (foxfi)   foxdoraame@gmail.com
钟凯                  zhongk19@mails.tsinghua.edu.cn
赵天辰              suozhang1998@gmail.com
