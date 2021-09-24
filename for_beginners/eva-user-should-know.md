# Eva-User Should Know

> 登录以及使用eva服务器，应该知道什么？

- 当我们开始之前，我们假定你已经阅读完了本repo的readme以及for_newbies的部分，并对linux系统以及命令有了一些初步的了解，如果没有，请加以了解。
- 这个repo的目的在于介绍一些基础常用的知识，以及杜绝一些危险操作，毕竟即使是一个container中也会有合作同学，虽然网管能够将你的container进行重置，但是可能丢失的环境以及数据难以找回。请大家**注意定期保存实验数据**并**杜绝危险操作**。


## 从0开始配置服务器

0. 你需要的一些LInux Background Knowledge

1. 登录上服务器之后，如果你有CUDA使用的需求，那么首先我们需要查看cuda以及显卡情况是否正常
    - 记住两个关键文件夹，分别是**Nvidia-Driver**以及**CUDA**, 注意，对这两个文件夹**仅进行执行操作，而不要修改**
    - nvidia-driver，在`/usr/local/nvidia`中，其bin文件夹中有一个`nvidia-smi`的命令，是用来查询显卡情况的常见命令
        - 为了方便你随时进行使用该命令查询，可以考虑将这个路径加入到环境变量中，加入bashrc(如果你不熟悉linux的环境变量以及bashrc的原理，请自行查询)，`export PATH=$PATH:/usr/local/nvidia/bin` 
        - 之后随时在终端中都可以进行`nvidia-smi`进行查询了 (网管的建议是写一个即时刷新的命令，`alias nv="watch -n 0.002 -d nvidia-smi"`也加入到bashrc中)
    - CUDA，在`/opt/cuda/samples/1_Utilities/deviceQuery`位置中有一个deviceQuery的文件，有时候经常需要执行该命令用来执行检错与cuda的debug，注意执行的时候一定要**sudo**
        - 同理也可以加一个alias来快捷执行它`alias deviceQuery="sudo /opt/cuda-11.1/samples/1_Utilities/deviceQuery/deviceQuery"`
        - 正常情况应该是显示pass
        - 显示error code 999 - 去群里提醒网管帮你在主机上执行该操作，并附上截图
        - 显示error code 100 - 执行`sudo ldconfig`
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210924103422.png)
    - CUDA标准化，我们服务器上的CUDA的位置与默认位置有所不同，可能有时候会出现因为网上其他教程按照默认位置编写导致的问题，所以，这里采用软连接的方式**尽量做到CUDA位置的标准化**
        - `ln -s /usr/local/nvidia/lib/* /usr/local/lib/`
        - `ln -s /opt/cuda-11.1/ /usr/local/cuda`
        - ` export CUDA_HOME="/opt/cuda/"`



## 常用操作与技巧

1. 允许他人登录你的Container并给予sudo权限: 
     - 编辑(注意需要sudo) `sudo vim /etc/login.user.allow` 将他人的**实验室账号用户名**添加到该文件中并保存。对于sudo权限，同样使用sudo编辑`sudo vim /etc/sudoers`参照里面已有的模板修改
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923194748.png)
     - 关于sudo权限，网管的建议是谨慎给与，仅较相关且长期(频繁)使用Container的才给与sudo权限。
     - 另外，实验室**严令禁止**随意共享实验室账号密码的行为，你基本的切换用户需求都有办法做到(参考本部分第二条)

2. 同一个container访问他人目录/使用他人账号
    - `sudo su` 进入sudo目录, 然后再`sudo xxx`（你的linux用户名）




## 危险操作

> 由于我们的container中大家一般都会有sudo权限，可能就会做出一些危险操作，在此进行列举。

1. `sudo apt-get upgrade`命令，会按照对应的源，将你能够更新的包全部更新到最新版本，需要花很长时间，并且**很容易出现环境不兼容问题，甚至让Container崩溃**，所以**即使在很多教程中让你执行这步操作**，请跳过他(大多数时候这条操作是不必要的)
2. 请对cuda以及nvidia-driver仅使用，不要**尝试修改或者重装**，由于cuda是整台eva(AcrossContainer)所共用的，重装所造成的问题影响范围广，且在Container内部重装可能会出更多的问题。所以，**如果你有特殊的使用需求(如需要使用旧版本的CUDA)**，请联系网管(在nics-server群/邮件)发出诉求。对于兼容性问题，网管的个人建议是先做尝试，毕竟很多库都具有尚可的向前兼容能力。实在有需求，再提出申请(因为网管能做的也只是为你在老服务器上开一个Container)
3. xxx