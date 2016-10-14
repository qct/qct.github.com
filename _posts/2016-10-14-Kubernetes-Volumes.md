---
layout: post
title: "Kubernetes Volume"
description: "kubernetes 1.4 中支持的 volume 简介"
category: 技术
tags: [技术, kubernetes]
---
{% include JB/setup %}


## 1.分类：
* emptyDir
* hostPath
    * Example pod
    * Example pod
* gcePersistentDisk
    * Creating a PD
    * Example pod
* awsElasticBlockStore
    * Creating an EBS volume
    * AWS EBS Example configuration
* nfs
* iscsi
* flocker
* glusterfs
* rbd
* cephfs
* gitRepo
* secret
* persistentVolumeClaim
* downwardAPI
* FlexVolume
* AzureFileVolume
* AzureDiskVolume
* vsphereVolume
    * Creating a VMDK volume
    * vSphere VMDK Example configuration
* Quobyte

## 2.详细介绍

### 1.emptyDir: 

如果Pod配置了emptyDir类型Volume， Pod 被分配到Node上时候，会创建emptyDir，只要Pod运行在Node上，emptyDir都会存在（容器挂掉不会导致emptyDir丢失数据），但是如果Pod从Node上被删除（Pod被删除，或者Pod发生迁移），emptyDir也会被删除，并且永久丢失。

### 2.hostPath:

hostPath允许挂载Node上的文件系统到Pod里面去。如果Pod有需要使用Node上的东西，可以使用hostPath，不过不过建议使用，因为理论上Pod不应该感知Node的，类似docker自身的-v参数。

### 3.gcePersistentDisk:
挂载 Google Compute Engine (GCE) Persistent Disk，要求两点：   

* 运行pod的node必须是GCE上的VM
* 这些VM必须和Persistent Disk 在同一个GCE项目和区域

### 4.awsElasticBlockStore:
挂载 Amazon Web Services (AWS) EBS Volume，要求三点：   

* 运行pod的node必须是AWS EC2 instances
* 这些 instances必须和EBS volume在同一个 region 和 availability-zone
* EBS只支持一个 EC2 实例挂载一个volume

### 5.nfs:
NFS 是Network File System的缩写，即网络文件系统。Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保存的，同时NFS支持同时写操作。

### 6.iscsi
挂载 iSCSI (SCSI over IP) volume，iscsi可以同时被多个consumer以只读方式挂载，同时只能被一个consumer以读写模式挂载，不支持多consumer 以 读写方式同时挂载

### 7.flocker
flocker是一个为docker应用设计的开源数据卷管理工具，通过提供数据迁移，flocker为运维团队在容器中运行有状态服务成为可能，不像docker数据卷一样捆绑在一台主机上，一个flocker数据卷称为一个便携式的数据集合，可以在你集群中的任何一台容器中运行。   
flocker将docker容器和数据卷目录整合在一起，当你用flocker管理你的有状态服务时，数据卷目录将会跟随你的容器在不同的主机上运行。   

flocker volume 允许把Flocker dataset 挂载到pod，如果dataset在Flocker中不存在，要先用 Flocker CLI或者 Flocker API创建dataset，如果dataset已经存在，Flocker会把dataset重新附到运行这个pod的node上。

### 8.glusterfs
GlusterFS是一个开源的分布式文件系统，具有强大的横向扩展能力，通过扩展能够支持数PB存储容量和处理数千客户端。同样地，Kubernetes支持Pod挂载到GlusterFS，这样数据将会永久保存。
GlusterFS支持多个consumer以读写方式挂载

### 9.rbd
rbd volume允许把 [Rados](http://ceph.com/docs/master/rbd/rbd/) 块设备挂载到pod，相对emptyDir，rbd将持久存储你的数据，即使设备卸载，pod删除等，数据都会被保存。rbd其实是由ceph分布式存储提供的，他本身提供对象存储，块存储，文件系统存储。
同一个rbd volume可以被多个客户端以只读的方式挂载，但是只允许一个客户端以读写的方式进行挂载。

### 10.cephfs
CephFS volume运行把一个已经存在的volume挂载到pod，支持同时多个consumer以读写方式挂载。

### 11.gitRepo
gitRepo volume 是一个可以作为volume插件的例子。它在你的pod中挂载一个空目录，并且在它里面克隆一个git仓库。
在将来，这种volume可能会和kubernetes解耦，成为一个松耦合的模型。

### 12.secret
secret volume是用来传递敏感信息（像密码等）到pod，你可以通过kubernetes API保存secret，然后把他们像文件一个挂载到pod，secret后面的文件系统是内存文件系统，不会持久化。

### 13.persistentVolumeClaim
PersistentVolume 子系统，对存储的供应和使用做了抽象，以 API 形式提供给管理员和用户使用。要完成这一任务，我们引入了两个新的 API 资源：PersistentVolume（持久卷） 和 PersistentVolumeClaim（持久卷申请）。   

PersistentVolume（PV）是集群之中的一块网络存储。跟 Node 一样，也是集群的资源。PV 跟 Volume (卷) 类似，不过会有独立于 Pod 的生命周期。这一 API 对象包含了存储的实现细节，例如 NFS、iSCSI 或者其他的云提供商的存储系统。   

PersistentVolumeClaim (PVC) 是用户的一个请求。他跟 Pod 类似。Pod 消费 Node 的资源，PVCs 消费 PV 的资源。Pod 能够申请特定的资源（CPU 和 内存）；Claim 能够请求特定的尺寸和访问模式（例如可以加载一个读写，以及多个只读实例）   

persistentVolumeClaim volume允许把一个persistent volume挂载到pod。

### 14.downwardAPI
downwardAPI是用来使 downward API 数据对程序可用，它挂载一个目录并且把请求数据写入文件。
容器可以通过环境变量来消费downward API，而且downward API允许容器通过被注入的方来式使用所在pod的name和namespace等信息

### 15.FlexVolume
FlexVolume可以让用户把vendor的volume挂载到pod，它认为vendor的驱动已经在所有node的volume插件路径中安装

### 16.AzureFileVolume
挂载 Microsoft Azure File Volume (SMB 2.1 and 3.0)到pod

### 17.AzureDiskVolume
挂载Microsoft Azure [Data Disk](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/)到pod

### 18.vsphereVolume
挂载 vSphere VMDK Volume 到Pod

### 19.Quobyte
挂载一个已经存在的Quobyte volume到pod   

Quobyte是Quobyte公司推出的分布式文件系统，Quobyte提供了一个特性：fixed-user挂载，这个新特性允许所有Quobyte卷都会被挂载到一个路径下面，同时又可以按照不同用户来进行分别使用。