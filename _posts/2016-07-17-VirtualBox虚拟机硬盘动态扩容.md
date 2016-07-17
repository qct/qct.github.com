---
layout: post
title: "VirtualBox虚拟机硬盘动态扩容"
description: "它是容器运行时为各个主机提供子网互通的虚拟网络。  
像Google的Kubernetes这样的平台，它假设在集群内部，每个容器（pod）有一个唯一、可路由的IP。这种模型的好处是它减少了端口映射的复杂性。"
category: 技术
tags: [技术, virtualbox, 虚拟机]
---
{% include JB/setup %}

***这篇文章目的是扩展虚拟机的根目录，比如原来根目录只有8G，那么把它扩展到15G***  
1. 如果你的硬盘格式是vmdk，先用virtualbox自带的管理工把硬盘转换成vdi格式：

```
VBoxManage.exe clonehd "C:\Users\alex\VirtualBox VMs\ubuntu14.04-k8s1\ubuntu14.04-base-1.0-disk1.vmdk" "C:\Users\alex\VirtualBox VMs\ubuntu14.04-k8s1\ubuntu14.04-base-1.0-disk1.vdi" --format VDI
```
2. 再用virtualbox自带的管理工具把本地磁盘扩大：
```
VBoxManage.exe modifyhd "C:\Users\alex\VirtualBox VMs\ubuntu14.04-k8s1\ubuntu14.04-base-1.0-disk1.vdi" --resize 15000
```
上面的意思是把硬盘扩大到15G  

如果是从vmdk转换成vdi格式的硬盘，那么去虚拟机设置里把原来的vmdk硬盘删除，挂上新转换的vdi格式硬盘。

下面虚拟机开机，去虚拟机里操作。（前提是使用了LVM）

3. 使用fdisk建立分区：  
不管你是新挂了硬盘，还是像前面的操作一样扩大了原来的硬盘，都要使用fdisk建立新的lvm分区。
如果是扩大了原来的硬盘，那么建立分区的时，选择柱面起始位置的候要注意，默认的不一定是对的，目的是把新扩展的空间全部分配给新分区。   
剩下的步骤不说了，就是 n, p, ....，建立完成之后使用 t 把分区类型改成8e，即LVM。下面假设我们已经建立好了/dev/sda3的LVM分区。

4. 建立物理卷：  
```
pvcreate /dev/sda3
```

5. 扩展卷组：  
```
vgextend ubuntu-vg /dev/sda3
```
这里 ubuntu-vg是卷组的名字，可以通过 vgdisplay查询：
```
root@k8s1:~# vgdisplay 
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               14.41 GiB
  PE Size               4.00 MiB
  Total PE              3688
  Alloc PE / Size       3688 / 14.41 GiB
  Free  PE / Size       0 / 0   
  VG UUID               8vE2Im-8C0s-3HDo-j4gA-zVEC-lgX5-gHIw0n
```

6. 扩展逻辑卷：  
```
lvextend -l +100%FREE /dev/ubuntu-vg/root
```
这里 /dev/ubuntu-vg/root 是逻辑卷的 path，可以通过lvdisplay查询：
```
root@k8s1:~# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/root
  LV Name                root
  VG Name                ubuntu-vg
  LV UUID                c7eEoR-5E1n-eaCA-QIAv-T3W8-2jb0-DpQXBz
  LV Write Access        read/write
  LV Creation host, time ubuntu, 2016-07-11 18:46:03 +0800
  LV Status              available
  # open                 1
  LV Size                13.66 GiB
  Current LE             3497
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
```

7. 最后一步，调整大小：  
```
resize2fs /dev/ubuntu-vg/root
```

如果是centos，最后一步则用 `xfs_growfs /dev/ubuntu-vg/root`。  

什么系统不重要，不论centos还是ubuntu，只要理解的扩容的过程，还是很简单的，总结一下：  

***先通过虚拟化工具扩大虚拟磁盘， 然后用 fdisk 分区，转成 LVM， 建立物理卷，扩展根目录所在卷组，扩展根目录所在逻辑卷，调整文件系统大小。***