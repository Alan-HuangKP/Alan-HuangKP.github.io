---
layout:     postacg   				    # 使用的布局（不需要改）
title:      在Ubuntu下配置CUDA环境及pytorch框架 				# 标题 
subtitle:   本文基于Ubuntu 20.04 以及CUDA11.3进行编写
date:       2023-1-6 				# 时间
author:     Hernandez						# 作者
header-img: 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 环境配置
---
# 1.Ubuntu换源

1.打开终端，输入以下命令备份原来的源

    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

2.再输入以下命令打开sources.list配置文件更换源

    sudo vim/etc/apt/sources.list

配置内容如下：

    deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse 
    deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse 
    deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse 
    deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse 
    deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse 
    deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse 
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse 
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse 
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse 
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse



按`ESC`后输入`:wq`保存退出
再到终端中输入

    sudo apt-get update
    sudo apt-get upgrade

至此换源操作已完成

# 2.开启SSH服务

## 2.1 查看ssh情况
在Ubuntu系统中，通过`ctrl+alt+T`打开终端，输入

    dpkg -l | grep ssh

查看现在是否有`ssh-server`服务

## 2.2 安装`ssh-server`服务
在终端中输入以下命令：
    
    sudo apt-get install openssh-server
    sudo /etc/init.d/ssh start


安装好后验证ssh服务是否启动

    ps -e | grep ssh

如果只有`ssh-agen`t那`ssh-server`还没有启动，需要`/sudo /etc/init.d/ssh start`或`sudo service ssh start`，如果看到`sshd`那说明`ssh-server`已经启动了。

`ssh-server`配置文件位于`/etc/ssh/sshd_config`，在这里可以定义SSH的服务端口，默认端口是22，你可以自己定义成其他端口号，如222。（或把配置文件中的`PermitRootLogin without-password`加一个`#`号,把它注释掉，再增加一句`PermitRootLogin yes`）

    #PermitRootLogin without-password

    PermitRootLogin yes

然后重启SSH服务：

    sudo /etc/init.d/ssh stop
    sudo /etc/init.d/ssh start


## 2.3连接ssh

    ssh 用户名@IP地址
    eg:
    ssh Hernandez@192.168.50.233

然后需要你输入密码，输入密码期间，终端不会显示任何字符，输入完成后，按回车键即可。

## 2.4 配置免密登录

添加HOME变量
打开环境变量属性页面，在用户变量部分点击新建，变量为HOME，值为 `C:\Users\name` 其name为用户名，可以去查看自己电脑的用户名，之后生成的密钥对默认保存在这个目录下。

生成密钥对
打开cmd命令行，在ssh程序所在目录运行，或者添加系统环境之后随处运行: `ssh-keygen -t rsa `这条命令用来生成密钥，随后一路回车，当看到一幅矩形图生成，那么密钥生成成功。

将公钥添加到linux账户
同样的打开cmd命令行，输入 `ssh username@host “cat >> ~/.ssh/authorized_keys” < C:\User\name\.ssh\id_rsa.pub `这条命令是首先登录linux，然后将本机即win下的公钥添加到账户个人目录下，从而实现免密码登录。注意这一步需要输入账户的密码。

验证是否添加成功
cmd下输入 `ssh username@host uname `如果看到输出Linux 表示添加成功。
同样可以直接输入 `ssh username@host` 这时可以看到不用输入密码即可登录linux了。

依然无法免密码登录
添加key之后还是需要输入密码,主要排查方法是` ssh username@host -vvv `输出debug信息,或者remote机器使用 `cat /var/log/secure `查看日志

可能的原因是:

1.目录权限问题. 远程目录.ssh 和.ssh 目录下的文件的权限需要分别设置700和600

2.找不到密钥. ssh登录命令添加参数` -i C:\User\name\.ssh\id_rsa `手动指定密钥文件


现在你可以实现用Windows端VScode等软件连接该电脑进行远程开发


# 3.安装显卡驱动
## 3.1 检查是否有驱动
输入`nvidia-smi` 若有界面出现则有驱动

## 3.2 安装NVIDIA显卡驱动
1.在官网 <https://www.nvidia.cn/Download/index.aspx?lang=cn> 选择你的显卡型号以及系统版本.

然后依次点击`搜索`、`下载`和`同意并开始下载`。此时你的浏览器就应该在下载驱动了。

2.禁用默认显卡驱动
安装.run文件之前，禁用Ubuntu20.04默认的系统显卡驱动，命令如下：

    // 修改属性
    sudo chmod 666 /etc/modprobe.d/blacklist.conf  
    //用vim打开
    sudo vim /etc/modprobe.d/blacklist.conf
    // 在最后一行加入以下几句，保存退出
    blacklist vga16fb
    blacklist nouveau
    blacklist rivafb
    blacklist rivatv    
    blacklist nvidiafb
    // 对刚才修改的文件进行更新
    sudo update-initramfs -u
    // 最好重启一下服务器
    sudo reboot
    //若执行完该句，没有任何输出，则nouveau被成功禁用
    lsmod | grep nouveau 

3.卸载旧版本的nvidia驱动
因为我的系统不带nvidia驱动，可以不卸载，但如果已经有nvidia驱动，执行以下语句卸载：

    sudo apt-get remove --purge nvidia*

4.安装gcc编译
如果没有安装，或者版本不对应，有可能会导致驱动安装失败。

    sudo apt-get  install  build-essential 
    gcc --version 

如果有提示g++或cmake没安装，执行以下语句：

    sudo apt-get install g++
    sudo apt-get install make

5.安装新的nvidia驱动
有资料说需要关闭图形界面，我并没有操作这一步。直接执行以下命令，并按提示安装。

    //进入文件所在目录下
    cd 
    //修改权限
    sudo chmod a+x NVIDIA-Linux-x86_64-xxx.run
    //执行安装
    sudo ./NVIDIA-Linux-x86_64-xxx.run

安装成功后，输入：`nvidia-smi`,显示以下界面便是成功

更新内核，并重新启动计算机（这一步非必须，但最好有）

    //更新内核
    sudo update-initramfs -u
    //重启服务器
    sudo reboot now

至此NVIDIA显卡驱动已经安装完成

# 4.配置CUDA和cuDNN环境
## 4.1 CUDA 11.3的配置

先去官网 <https://developer.nvidia.com/cuda-11.3.0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local> 选择适合的CUDA安装版本，点击runfile文件下载，等待下载完成，注意这里CUDA的驱动版本要小于等于前面的驱动版本。

如果按照上述流程安装了最新的NVIDIA Driver，runfile安装过程中会提醒是否继续(Existing package manager installation of the driver found. It is strongly recommended that you remove this before continuing.)，选择继续​`​continue​​`，在后续选择安装内容时去除驱动选择。这一点一定要注意。

运行成功后看见输出`Summary`，根据提示需要进行​​环境变量的设置​​，在.`bashrc`(ZSH使用者对应.zshrc)末尾追加如下文字。
```
if [ $LD_LIBRARY_PATH ]; then
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
else
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64
fi

if [ $PATH ]; then
    export PATH=$PATH:/usr/local/cuda/bin
else
    export PATH=/usr/local/cuda/bin
fi

if [ $CUDA_HOME ]; then
    export CUDA_HOME=$CUDA_HOME:/usr/local/cuda
else
    export CUDA_HOME=/usr/local/cuda
fi
```

最后测试以下CUDA是否成功安装，运行`​​nvcc -V​`​:
如下图所示，即为安装完成。

![V0lYyy.png](https://i.imgloc.com/2023/06/02/V0lYyy.png)

## 4.2 安装cuDNN驱动程序

cuDNN其实不用安装，是把文件下载下来放在cuda指定的目录下。
1.下载cuDNN
下载地址：<https://developer.nvidia.com/rdp/cudnn-archive>
下载界面如下，也是按照版本选择的，下载后是一个压缩包：

2.安装

    //用tar命令解压，得到cuda的文件夹：
    tar -zxvf cudnn-11.2-linux-x64-v8.1.0.77.tgz
    //拷贝相关文件到上面安装的cuda文件中，并赋予权限：
    sudo cp cuda/include/cudnn.h /usr/local/cuda-11.3/include
    sudo cp cuda/lib64/libcudnn* /usr/local/cuda-11.3/lib64
    sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda-11.3/lib64/libcudnn*

到这里，cuDNN的安装已经完成。

# 5.配置Anaconda环境
## 5.1下载Anaconda
先去官网 <https://www.anaconda.com/distribution/#download-section> 找到合适版本对应的下载网址，如`https://repo.anaconda.com/archive/Anaconda3-2022.10-Linux-x86_64.sh`

然后输入命令
    cd ~
    wget https://repo.anaconda.com/archive/Anaconda3-2022.10-Linux-x86_64.sh
即开始下载

## 5.2 安装Anaconda
输入命令：

    sudo bash

验证安装，输入

    python

则可确认安装以及安装完成
## 5.3 conda换源
若conda下载慢，可更换清华源

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/fastai/

    conda config --set show_channel_urls yes

再删除 `.condarc`下的`-default`这一行。

安装pytorch 

    conda create --name torch python=3.8

激活环境 

    conda activate torch
    
安装pytorch 

    conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch

验证是否能用GPU 

    python

    import torch 
    
    torch.cuda.is_available()
    
如下图所示，显示True即为成功 

假装有个图


# 6.CUDA多版本切换/管理方式
当本机上安装有多个版本cuda时可以通过一下步骤进行管理/版本切换，比如我们要切换使用cuda11.6时：
`sudo vim ~/.bashrc`加入以下的内容：
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
export PATH=$PATH:/usr/local/cuda/bin
export CUDA_HOME=$CUDA_HOME:/usr/local/cuda

```
修改完保存，`source ~/.bashrc`
```
sudo rm -rf /usr/local/cuda # 删除旧版本的软连接
sudo ln -s /usr/local/cuda-11.6 /usr/local/cuda # 建立新版本的软连接
                                              # 前面的路径是需要的版本的cuda的安装路径。

```

之后再使用nvcc -V查看cuda版本，如
假装还有个图

想要查看 Pytorch 实际使用的运行时的 cuda 目录，可以直接输出之前介绍的 `cpp_extension.py` 中的 `CUDA_HOME` 变量。
    
    >>> import torch
    >>> import torch.utils
    >>> import torch.utils.cpp_extension
    >>> torch.utils.cpp_extension.CUDA_HOME        #输出 Pytorch 运行时使用的 cuda 

至此结束了pytorch更换CUDA版本的工作

# 7.配置结尾阶段

## 7.1 远控
进入官网下载向日葵Linux版本（这里我选择的是图形版本）
<https://sunlogin.oray.com/download?categ=personal>
下载完毕后应得到一个deb包

进入安装包所在的文件夹，输入指令

    sudo dpkg -i SunloginClient-10.1.1.38139_amd64.deb
    # 报错缺少依赖的话，执行下一步再尝试安装
    sudo apt-get -f -y install

安装lightdm
因为向日葵不支持ubuntu的原始桌面，所以需要更换为lightdm

    sudo apt-get install lightdm



## 7.2 Clash
在Clash github仓库中，下载`LInux`版本
<https://github.com/Fndroid/clash_for_windows_pkg/releases>


下载好后，解压文件

    tar -zxvf 文件名字

解压完成后，进入文件中，执行`cfw`文件

    ./cfw

在Proxies中导入，再到Ubuntu的设置中开启`Networks Proxies`，选择为`Manual`，现在即可像Windows中使用Clash一样使用Ubuntu的Clash。