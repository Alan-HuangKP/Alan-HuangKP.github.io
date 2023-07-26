---
layout:     postacg   				    # 使用的布局（不需要改）
title:      双系统设置默认启动系统为Windows 				# 标题 
#subtitle:   
date:       2023-1-10 				# 时间
author:     Hernandez						# 作者
header-img: 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 环境配置
---

问题：双系统的引导菜单是Ubuntu的引导其grub的菜单，所以默认启动项是ubuntu系统。我们可以在ubuntu中修改grub的默认项，让它默认启动Windows。

本文提出一个解决办法，实现双系统中默认启动项为Ubuntu

1. 在Ubuntu下打开一个终端，操作为`crtl+alt+t`，使用文本编辑器gedit打开grub的配置文件。输入命令如下：
```
sudo gedit /etc/default/grub
```
其界面类似这样：

![](https://img-blog.csdnimg.cn/cf903d8a8ebe42dd920e63c06f2eb653.jpeg)

当我们开机时，选择系统的界面，类似这样：

![](https://img-blog.csdnimg.cn/d9c1687f4826483b8489f5833e4a615d.jpeg)

这四个启动项从上到下依次是0-3，第一项Ubuntu的启动项是序号0，而Windows的启动项是2，而`GRUB_DEFAULT`的默认数值是0，也就是我们现在看到的默认启动项是第一项ubuntu的启动项，所以我们只需要将`GRUB_DEFAULT`的数值改成2，就可以实现开机默认启动Windows。

修改完保存。注意：每个人的开机启动项顺序可能不一样，需要看清楚Windows启动项的编号，就将`GRUB_DEFAULT`的默认值改成对应值。

（3）然后在终端程序里执行下面命令,更新一下设置：

    sudo update-grub

现在即可实现Ubuntu和Windows双系统开机即进入Windows系统了