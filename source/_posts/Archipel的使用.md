---
title: Archipel的使用
date: 2016-01-27 05:15:09
tags:
 - Archipel
---

在我的上一篇文章之后，现在你的Archipel GUI应该如下：

![Ａｒｃｈｉｐｅｌ](http://upload-images.jianshu.io/upload_images/1113810-5b7ef3fe012172a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图有两个主要联系人（无视分组），那是两台物理机。在上一篇文章的基础上，我另添加了一台服务器。

#####左边联系人框
        在这里，你可以看到所有的物理机以及运行在其上的虚拟机。当然，最好分好组，方便管理，分组方法\
    ，就是左下的“＋”号，点击添加Group
 ![联系人](http://upload-images.jianshu.io/upload_images/1113810-fc543a3b26f17f04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####物理机
    Health: 选中一个物理机，中间大窗口会显示物理机此时的Health, 以及实时的CPU、Disk、NetWork工作

    Health: 选中一个物理机，中间大窗口会显示物理机此时的Health, 以及实时的CPU、Disk、NetWork工作
            曲线。
    Virtual Machines: 在这里,你可以看到该物理机下所有的虚拟机,Archipel VMs指的是已经让Archipel
                      接管的虚拟机，Other VMs指的是未让Archipel接管的虚拟机，你可以在此窗口,对
                      虚拟机做出调整。 左下的“＋”号可以创建虚拟机，但要事先把相应的ISO文件放进指定
                      目录，该目录默认/vm/iso,需要事先创建，并给予777权限。
    NetWorks：显示该物理机的网络信息
    VMCasts：暂未测试，稍后更新
    Scheduler：设定一系列命令，命令会在你设定的时间执行
    Chat：与其进行交互，如下，更多交互方式查看官网

![Chat](http://upload-images.jianshu.io/upload_images/1113810-a8fb10a5b8dc0655.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    Permissions：Archipel当然不是只有admin才能登陆，你可以在XMPP服务器上创建注册更多的用户，用户
    之间在Archipel也可以互相添加联系人。具有admin权限的用户，可以在这里调整其他用户对与某台物理机的
    操作权限。

#####虚拟机:
    Controls：这里进行一些虚拟机的基本控制
    Definition：在虚拟机关机状态下，设置虚拟机的属性，当然你也可以点击中间下边的“</>”按钮来直
                接编辑该虚拟机的xml文档，编辑完成后点击“define”即可生效，该按钮旁边的按钮功
                能是“undefine the vm”。
    Appliances：稍后更新
    Disks：编辑Disk文件等等
    Snapshots：在这里，拍摄快照，编辑快照等等
    VNC Console：如下图，相当于一个vnc screen

![选区_004.png](http://upload-images.jianshu.io/upload_images/1113810-037db50a50b5e10b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    Scheduler：设定一系列命令，命令会在你设定的时间执行
    Chat：与其进行交互
    Permissions：Archipel当然不是只有admin才能登陆，你可以在XMPP服务器上创建注册更多的用户，用户
    之间在Archipel也可以互相添加联系人。具有admin权限的用户，可以在这里调整其他用户对与某台虚拟机的
    操作权限。


######注：页面头像右边的控制按钮时用来控制虚拟机的
