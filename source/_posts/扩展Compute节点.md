---
title: 扩展Compute节点
date: 2016-09-20 05:03:07
tags:
 - OpenStack
---

## 准备工作

### 计算节点设置apt-cacher代理
>`# echo 'Acquire::http::Proxy "http://{proxy_server_ip}:{port}";' > /etc/apt/apt.conf`

### 若之前执行过apt-get update命令， 则运行一下命令
>`# rm /var/lib/apt/lists/partial/security.ubuntu.com_ubuntu_dists_trusty-security_main_binary-i386_
Packages
`

### 修改软件源为阿里云
>`# vim /etc/apt/source.list`
>`:%s/cn.archive.ubuntu/mirrors.aliyun/g`
>`# apt-get update`

### 安装KVM
>`# apt-get install qemu-kvm qemu-system libvirt-bin  bridge-utils`

### 配置语言
>`# apt-get -y install language-pack-zh-hans`
### 修改/etc/hosts文件
>`# scp {username}@{ip}:/etc/hosts /etc/hosts`
>此时用`ping`验证一下当前节点与其他节点的互联性

### 修改主机名
>我们以节点ip的最后一位作为节点序号
>>`# hostname compute$n`
>`# echo "compute$n"  > /etc/hostname`

>将$n替换为对应的节点序号

### 重新登录使主机名更改生效
>这里exit后再次ssh连接即可

>![](http://upload-images.jianshu.io/upload_images/1113810-1319a3036e9167db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 配置OpenStack Provider Network Interface
>`# vim /etc/network/interfaces`
> 在末尾添加以下内容:

>---
>>`# The provider network interface
>auto em3
>iface  em3 inet manual
>up ip link set dev $IFACE up
>down ip link set dev $IFACE down`

>---
> 启动Provider Interface
> `# ifup em3`
>如果em3启动失败，则通过ip link检查是否存在em3网卡

## 开始部署

### 配置NTP服务
>`# apt-get install chrony -y`
> 编辑 /etc/chrony/chrony.conf， 修改相关内容为
>> `server controller iburst`
![](http://upload-images.jianshu.io/upload_images/1113810-cf638a891f14fe98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>验证ntp服务
>>`# service chrony restart`
>>`# chronyc sources`
>> ![](http://upload-images.jianshu.io/upload_images/1113810-440082195c472e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 配置OpenStack 软件源以及相关依赖
> 设置软件源
>>`# apt-get install software-properties-common && add-apt-repository cloud-archive:mitaka`

>更新本地仓库以及安装OpenStackClient
>>`# apt-get update && apt-get dist-upgrade &&  apt-get install python-openstackclient`

>---
> 如果出现网络错误，重新执行相关命令

### 配置服务
####安装软件包
>`# apt-get install -y nova-compute`
`# apt-get install -y neutron-linuxbridge-agent`
>`获取启动脚本文件`:`# scp {username}@compute1:/home/hit/compute_service ./`


#### 配置Nova



>备份
>>`# cp /etc/nova/nova.conf /etc/nova/nova.conf.bak`

>获取配置文件
>>`# scp root@192.168.100.1:/etc/nova/nova.conf  /etc/nova/nova.conf`
>修改配置文件，修改ip为本节点ip
>>`[DEFAULT]
...
my_ip = {ip}
`
>![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1113810-d3c90aef01632d14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 配置Neutron
>备份以下两个文件
>>`#  cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak`
`# /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak`


>获取配置文件
>>`# scp root@192.168.100.1:/etc/neutron/neutron.conf /etc/neutron/neutron.conf`
`# scp root@192.168.100.1:/etc/neutron/plugins/ml2/linuxbridge_agent.ini  /etc/neutron/plugins/ml2/linuxbridge_agent.ini`



>修改配置文件linuxbridge_agent.ini,修改provider Interface为之前配置OpenStack Provider Network Interface，修改ip为本节点ip:`192.168.100.X`

>>`[DEFAULT]
>...
>physical_interface_mappings = provider:em3`

>---
>>`[vxlan]
...
enable_vxlan = True
local_ip = {ip}
l2_population = True`

>---
>![](http://upload-images.jianshu.io/upload_images/1113810-14ae21842a7324ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>![](http://upload-images.jianshu.io/upload_images/1113810-05c83c0cabe0ce37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>重启计算节点服务
>`#./compute_service`

#### 验证
> 在控制节点运行，查看新添加的节点是否在线，当前新添加的为`compute2`
>>`#. admin-openrc`
>>`# nova service-list`
>>`# neutron agent-list`


>![](http://upload-images.jianshu.io/upload_images/1113810-19012f6fe28ec51a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>![](http://upload-images.jianshu.io/upload_images/1113810-0d8cc4c55db44884.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
