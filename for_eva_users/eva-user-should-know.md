# Eva-User Should Know

> 登录以及使用eva服务器，应该知道什么？

- 当我们开始之前，我们假定你已经阅读完了本repo的readme和login相关文档，并对linux系统以及命令有了一些初步的了解，如果没有，请加以了解。
- 这个repo的目的在于介绍一些基础常用的知识，以及杜绝一些危险操作，毕竟即使是一个container中也会有合作同学，虽然网管能够将你的container进行重置，但是可能丢失的环境以及数据难以找回。请大家**注意定期保存实验数据**并**杜绝危险操作**。

## 你需要的一些LInux Basic Knowledge

- linux基础操作:
    - 在linux下 `ctrl+s` 下屏幕会freeze住，输入的指令**实际上是起效的**，只是不会显示，按q可以退出该模式
    - 在linux下`ctrl+c`是**结束指令**，而不是复制，大部分的terminal使用shift+鼠标拖动可以选中，用`ctrl+shift+c` `ctrl+shift+v` 可以复制或者粘贴，或者鼠标右键点击复制；对于mac系统`command+c` 仍然可以作为复制使用。
- vim的使用:  (为了便于在服务器上修改文件)
    - vim有三种模式，常规模式，编辑模式（insert mode），命令模式，打开文件后默认是常规模式
    - 常规模式下`h j k l`四个字母移动光标，无法输入
    - 常规模式下`i`进入编辑模式，进行输入
    - 编辑模式下`esc`退出，进入常规模式
    - 常规模式下`:`进入命令模式，在窗口底部可输入命令，输入完` Enter`执行命令，并回到常规模式
    - 最常用的命令`:wq` 保存并退出vim，w是保存，q退出
- 软连接
- bashrc


## 常用操作与技巧

1. 允许他人登录你的Container并给予sudo权限: 
     - 编辑准许登陆名单文件(注意需要sudo) `sudo vim /etc/login.user.allow` 将他人的**实验室账号用户名**添加到该文件中并保存。对于sudo权限，同样使用sudo编辑`sudo vim /etc/sudoers`参照里面已有的模板修改
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210923194748.png)
     - 关于sudo权限，网管的建议是谨慎给与，仅较相关且长期(频繁)使用Container sudo权限的才给
     - 另外，实验室**严令禁止**随意共享实验室账号密码的行为，基本的切换用户需求有其它办法做到(参考本部分第二条)

2. 同一个container访问他人目录/使用他人账号
    - `sudo su` 进入sudo用户, 然后再`su xxx`（他人的用户名）

3. Container空间不足？
    - 将一些实验记录以及checkpoint文件，移动到`/data/eva_share_users` 当中，注意建立自己用户名命名的文件夹
        - 因为一些服务器的区别，可能会叫 `/opt/eva_share_users` 或者是 `/data1/eva_share_users`或者是`/home/eva_share_users`。运行`df -h`可以查看。
    - 如果有数据集，可以考虑放到`/data/eva_share`
    - 其它数据如conda文件，甚至你自己的整个home目录，其实也都可以移走，只需要移动后在原地创建一个同名软连接即可，这样一切访问和功能都正常，但空间占用换了地方。
        - 使用软连接是一个不错的习惯，这也有助于我们代码可迁移性，比如不同机器上实际的数据集可能存在不同的位置，但是在不同机器上跑实验我们不想反复修改代码中访问数据集的路径，则可以在代码中统一写作从`/home/xxx/datasets`访问数据集，在所有机器`/home/xxx/datasets`都建立软连接指向实际数据存储的位置即可，保存实验checkpoint时当然也是如此。

## 从0开始配置服务器

> 遇到问题了请大家善用搜索

1. 登录上服务器之后，如果你有CUDA使用的需求，那么首先我们需要查看cuda以及显卡情况是否正常
    - 记住两个关键文件夹，分别是**Nvidia-Driver**以及**CUDA**, 注意，对这两个文件夹**仅进行执行操作，而不要修改**
    - nvidia-driver，在`/usr/local/nvidia`中，其bin文件夹中有一个`nvidia-smi`的命令，是用来查询显卡情况的常见命令
        - 为了方便你随时进行使用该命令查询，可以考虑将这个路径加入到环境变量中，加入bashrc(如果你不熟悉linux的环境变量以及bashrc的原理，请自行查询)，`export PATH=$PATH:/usr/local/nvidia/bin` 
        - 之后随时在终端中都可以进行`nvidia-smi`进行查询了 (网管的建议是写一个即时刷新的命令，`alias nv="watch -n 0.002 -d nvidia-smi"`也加入到bashrc中)
    - CUDA，在`/opt/cuda/samples/1_Utilities/deviceQuery`位置中有一个deviceQuery的文件，有时候经常需要执行该命令用来执行检错与cuda的debug，注意执行的时候一定要**sudo**
        - 同理也可以加一个alias来快捷执行它`alias deviceQuery="sudo /opt/cuda/samples/1_Utilities/deviceQuery/deviceQuery"`
        - 正常情况应该是显示pass，对于显卡后面信息，如果显示NO是正常现象(这个指示的是各个gpu之间的互联，我们并没有nvlink，都是走的pcie)
        -  ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20211014115151.png)
        - 显示error code 999 - 去群里提醒网管帮你在主机上执行该操作，并附上截图
        - 显示error code 100 - 执行`sudo ldconfig`
        - ![](https://github.com/A-suozhang/MyPicBed/raw/master//img/20210924103422.png)
        - 在opt目录下，你可能看到多个cuda版本，比如`/opt/cuda`和一个`/opt/cuda-11.1`, 注意区分文件是真实文件，还是软连接
    -  (如果你安装的库不报错，这一步理论上不需要做) CUDA标准化，我们服务器上的CUDA的位置与默认位置有所不同，可能有时候会出现因为网上其他教程按照默认位置编写导致的问题，所以，这里采用软连接的方式**尽量做到CUDA位置的标准化**，这一步需要sudo
        - `ln -s /usr/local/nvidia/lib/* /usr/local/lib/`
        - `ln -s /opt/cuda/ /usr/local/cuda`
        - `export CUDA_HOME="/opt/cuda/"`
    - 有的时候由于cuda位置不在标准位置，可能会报错: `ImportError: libcusparse.so.11: cannot open shared object file: No such file or directory`
        - 需要将cuda的对应库的位置加到LD_LIBRARY的环境变量中 `export LD_LIBRARY_PATH="/opt/cuda/lib64:$LD_LIBRARY_PATH"`
    - 如果你有特殊的CUDA-DRIVER的需求，请联系网管协助安装

2. 可以从/opt目录拷贝一些常用文件过来，注意该目录的scp需要sudo权限

3. 你可能需要自己使用apt安装一些基本库,比如:
     - gcc
     - g++
     - htop

- (optional) 配置vim
    - apt安装git以及vim  `sudo apt-get install vim git`
    - (ztc的个人方案) - 想要使用可以直接拷贝zhaotianchen目录下的 `~/.vim/bundle` 以及`~/.vimrc`
        - clone Vundle as vim的包管理器(follow了初代目网管bigeagle的方案)  `mkdir -p ~/.vim/bundle;   git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim `
        - 从其他地方拷贝`~/.vimrc`，进入，输入`:PluginInstall`, 等待它安装好
        - molokai配色有一个bug，`~/.vim/bundle/molokai/colors/molokai.vim:line` 132行，从none->NONE
    - (zk的个人方案) - 可以参考https://github.com/I-Doctor/my_configs
        - 里面不仅有vim配置，还有bashrc和tmux的配置(实现了tmux基本个性化和一个tmux窗口保存与恢复功能，用的一个叫resurrect插件)

## 从老服务器迁移

> 如果你之前已经有了一个container或者自己的环境配置了，希望迁移到新的服务器，那么不必从0开始

- **注意:** 对于新的服务器，CUDA等相关系统配置仍然需要自己check一下，上一章的CUDA-check仍然需要做

0. 拷贝你的各种配置
 - `~/.bashrc` 等
 - `~/.vimrc` 等
 - `~/.tmux.conf` 等

1. 迁移整个conda环境
    - 对于python环境的迁移，网络上的教程很多诉诸于`pip freeze / conda list --export > requirements.txt`,但是这种方式经常可能因为dependancy问题导致失败，这里建议： **使用conda-pack**打包整个conda环境。[教程参考](https://conda.github.io/conda-pack/)

- 如果你的container环境非常复杂，且所占用空间不大，有一种特别一劳永逸的方法，联系网管**直接拷贝你i的整个lxc Container**
    - 如果选择这种方式，请**邮件向网管发起申请**，并清理你container中的checkpoint等大文件
    - 由于这种方式本质上违背了我们“建立container以完成环境隔离”，即“一个Container做一件事情”，所以如果做此申请，请**确保有明确的理由**，如果只是懒得再配置一遍环境，网管不会受理这种请求


## 没什么鸟用的操作

1. neofetch:
    - 个人的一点恶趣味，登录时候显示一些额外的信息，配置文件在`~/.config/neofetch`中

## 危险操作

> 由于我们的container中大家一般都会有sudo权限，可能就会做出一些危险操作，在此进行列举。

1. `sudo apt-get upgrade`命令，会按照对应的源，将你能够更新的包全部更新到最新版本，需要花很长时间，并且**很容易出现环境不兼容问题，甚至让Container崩溃**，所以**即使在很多教程中让你执行这步操作**，请跳过他(大多数时候这条操作是不必要的)
2. 不要**尝试修改或者重装nvidia驱动**，由于nvidia驱动是整台eva(Across Container)所共用的，重装所造成的问题影响范围广，且在Container内部重装完你自己大概率也用不了，可能会出更多的问题。所以，**如果你有特殊的使用需求(如需要使用旧版本的驱动)**，请联系网管(在nics-server群/邮件)发出诉求。对于兼容性问题，网管的个人建议是先做尝试，毕竟很多库都具有尚可的向前兼容能力。实在有需求，再提出申请(因为网管能做的也只是为你在老服务器上开一个Container)。如果有升级需求，也联系网管，网管会评估新驱动的稳定性和大家的使用情况，找时间升级。
3. `/opt/`和`/usr/local/nvidia/bin`中的文件是大家共用的，尤其不要尝试修改或不小心给改了。`/opt/`中的cuda是cuda链接库，并非cuda driver，但改了也有可能影响别人。不过如果你是用conda装pytorch等软件时，选了cuda版本，提示下载安装某个版本的cudatoolkit，则是另外下载一份pytorch自己用的链接库，与`/opt/`里的其实没有关系。
