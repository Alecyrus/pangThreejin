---
title: 使用Guestfish给虚拟磁盘扩容
date: 2016-10-11 05:09:51
tags:
 - Linux
 - GuestFish
---

##### 问题背景：
使用openstack dashboard 创建虚拟机时，有时创建虚拟机会出现如下错误

  >**Error: Flavor's disk is too small for requested image**

>   这种情况是由于flavor的配置不够导致的，将flavor的配置弄好后，再次创建，出现以下错误： **No Space Left To Write**, 这是因为我只有一个compute1节点，而且磁盘只有10G，但我创建的分配给实例的是20G。

 *  我们知道openstack的实例时创建在**compute节点**的**/var/lib/nova/instances/**里的，再挂载一块磁盘到这个目录显然不是一个很好的解决方式，最好是直接给compute节点的硬盘扩容，本文采用**Guestfish**给虚拟磁盘扩容。

1.安装**Guestfish**
> $ sudo apt-get install libguestfs-tools

2.查看虚拟硬盘分区表
>  $ sudo virt-filesystems --long --parts --blkdevs -h -a VM.img

>     Name       Type       MBR  Size   Parent
    /dev/sda1  partition  83   9.7G   /dev/sda
    /dev/sda2  partition  05   1.0K   /dev/sda
    /dev/sda5  partition  82   1022M  /dev/sda
    /dev/sda   device     -    11G    -   
  那么，**/dev/sda1** 就是我们要扩容的分区

3.创建一个更大的磁盘文件
  >磁盘文件格式与之前的虚拟磁盘格式保持一致

> $ qemu-img create -f qcow2 new_disk.qcow2 50G 

>     Formatting 'new_disk.qcow2', fmt=qcow2 size=53687091200 encryption=off 
    cluster_size=65536 lazy_refcounts=off


4.使用guestfish中的virt-*命令进行扩容
 > $ virt-resize --expand /dev/sda1 VM.img new_disk.qcow2 

>     Examining VM.img ...
    **********
    Summary of changes:
    /dev/sda1: This partition will be resized from 9.7G to 49.0G.  The 
    filesystem ext4 on /dev/sda1 will be expanded using the 'resize2fs' 
    method.
    /dev/sda2: This partition will be left alone.
    **********  
    Setting up initial partition table on com.qcow2 ...
    Copying /dev/sda1 ...
    ...
    $<2> 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
    Copying /dev/sda2 ...
    ...
    $<2> 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
    Expanding /dev/sda1 using the 'resize2fs' method ...
    Resize operation completed with no errors.  Before deleting the old disk,
    carefully check that the resized disk boots and works correctly.

5.查看分区表
>$  sudo virt-filesystems --long --parts --blkdevs -h -a new_disk.qcow2

>     Name       Type       MBR  Size   Parent
    /dev/sda1  partition  83   49G    /dev/sda
    /dev/sda2  partition  05   1.0K   /dev/sda
    /dev/sda5  partition  82   1022M  /dev/sda
    /dev/sda   device     -    50G    -

>$ virt-filesystems --long --parts --blkdevs -h -a VM.img 

>     Name       Type       MBR  Size   Parent
    /dev/sda1  partition  83   9.7G   /dev/sda
    /dev/sda2  partition  05   1.0K   /dev/sda
    /dev/sda5  partition  82   1022M  /dev/sda
    /dev/sda   device     -    11G    -
此时，源磁盘VM.img里面的文件系统已经完整的被拷贝到new_disk.qcow2

 5.删除源磁盘，并将new_disk.qcow2 改名为 VM.img
> $ sudo rm VM.img 
> $ sudo mv new_disk.qcow2 VM.img

###### DONE！
