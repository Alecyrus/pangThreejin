---
layout: post
title: OpenStack Ubuntu镜像制作
date: 2016-12-3 05:10:54
tags:
 - OpenStack
---

>##### **制作之前**
>
* 在主机创建一个**kvm**虚拟机**VM**(本文使用Ubuntu 12.04LTS)，使用**默认**的分区方式
* 你可以**在VM中配置好你想要的环境**
* 本文中将使用VM的磁盘文件**VM.img**制作一个在**OpenStack**(测试环境为kilo)中可用的镜像

##### 1.在虚拟机中的配置
*  安装cloud-init，openssh软件包
>  $ sudo apt-get install cloud-init openssh-server
* 配置元数据源
> $ sudo dpkg-reconfigure cloud-init

  > 选中**EC2**数据源，保存退出

* 修改**cloud-init**使用的账户名为admin
> $ sudo vi **/etc/cloud/cloud.cfg**

  >修改user的参数为admin，如下
>     user:admin

* 关闭虚拟机
> $ sudo shutdown -h now

##### 2.在主机上的操作
* 清除VM镜像中的MAC地址相关的信息
>  $ sudo virt-sysprep -d **VM**

* 删除主机中的**VM**虚拟机定义
> $ sudo virsh undefine **VM**

* 上传镜像至**Glance**服务
> $ glance image-create --name "**VM-templet**" --file  **VM.img** --disk-format **qcow2** --container-format bare --visibility public --progress

##### Done!
