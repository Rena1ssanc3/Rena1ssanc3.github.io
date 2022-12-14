---
title: HomeLab系列（三） - 存储（中）
date: 2022-11-19 21:30:10
tags: HomeLab
---

# HomeLab存储如何搭建（中）

　　上一篇讲了一些存储底层的东西，这一篇讲一下文件系统。不妨假设我们现在买了一车HDD，插进电脑里。此时我们拥有了几个块设备，但真正能让我们使用的则是文件系统。


## 文件系统类型

### 日志文件系统

　　日志文件系统是基于对每个存储在存储设备上的文件进行索引节点，即每个存放在虚拟目录下的文件都有与之对应的索引节点，从而形成一个索引节点表，每个索引节点包含文件信息及其物理设备上的文件位置指针。

　　日志文件系统在数据还没有写入磁盘之前，会先写入日志文件，然后再写入磁盘。这样做的好处是，如果在写入磁盘的过程中出现了问题，那么就可以通过日志文件来恢复数据。

### 写时复制文件系统

　　所谓写时复制，即每次写磁盘数据时，先将更新数据写入一个新的 block，当新数据写入成功之后，再更新相关的数据结构指向新 block 。一旦系统突然断电，重启之后不需要做Fsck。

## 单机文件系统选择

### NTFS

　　NTFS是Windows的默认文件系统。因为这货可不是开源的，所以你在Linux上是用不了的……当然，你可以用FUSE来挂载NTFS，但是这样的话你就失去了NTFS的一些特性，比如说NTFS的压缩功能。

### EXT4

　　EXT4是第四代扩展文件系统（英语：Fourth extended filesystem，缩写为 ext4）是Linux系统下的日志文件系统，是ext3文件系统的后继版本。ext4支持最大1EB分区、最大16TB文件和64000子目录。通过Extents替代Block Mapping，这种方式可以增加大型文件的效率并减少分裂文件。

### XFS

　　XFS 与非 ext 文件系统在 Linux 中的主线中的地位一样。它是一个 64 位的日志文件系统，自 2001 年以来内置于 Linux 内核中，为大型文件系统和高度并发性提供了高性能（即大量的进程都会立即写入文件系统）。不同于Ext4，它通过B+树来索引inode和数据块。用树结构的文件系统通常相比Ext4用表结构，如链表、直接/间接Block以及extent，能更好地支持大文件，如视频/数据库文件等。另外其元数据规模少，使得硬盘可用空间更多，实测XFS、Btrfs多平均至少1.5%以上的可用空间。

### F2FS

　　F2FS (Flash Friendly File System) 是专门为基于 NAND 的存储设备设计的新型开源 flash 文件系统。特别针对NAND 闪存存储介质做了友好设计。F2FS 于2012年12月进入Linux 3.8 内核。F2FS仅支持Linux操作系统。通常你能在手机上看到他。

### BTRFS

　　BTRFS（通常念成Butter FS），由Oracle于2007年宣布并进行中的写时复制文件系统。目标是取代Linux的ext3文件系统，改善ext3的限制，特别是单一文件大小的限制，总文件系统大小限制以及加入文件校验和特性。加入ext3/4未支持的一些功能，例如可写的磁盘快照，以及支持递归的快照，内建磁盘阵列支持，支持子卷的概念，允许在线调整文件系统大小。 Extent，B-Tree 和动态 inode 创建等特性保证了 btrfs 在大型机器上仍有卓越的表现，其整体性能而不会随着系统容量的增加而降低。

### ZFS
　　ZFS是一款128bit的写时复制文件系统，它还拥有自优化，自动校验数据完整性，存储池/卷系统易管理等诸多优点。较ext3系统有较大运行速率，提高大约30%-40%。  
　　ZFS使用可变大小的块，最大可至128KB。现有的代码允许管理员调整最大块大小，这在大块效果不好的时候有用。未来也许能做到自动调整适合工作量的块大小。ZFS的可变大小的块与BtrFS和Ext4的extent不同。在ZFS中，在一个文件中所有数据块的逻辑长度必须是相同的，不同文件之间的块大小可以不同，因此ZFS可以用直接映射的方式来来搜索间接块的数据指针数组。BtrFS和Ext4的extent方式在同一个文件中每个数据快的大小都可以不相同，因此需要用B+ Tree或者类B Tree的方式来组织间接块的数据。

## 分布式文件系统

　　上面我们聊到的都是本地的文件系统。云原生年代，基于网络的存储也是大势所趋。分布式文件系统的优点主要在于高扩展性、高可用性、高性能、可横向扩展等特点。依托于网络，分布式文件系统可以突破单机存储能力的上限，无论是容量还是带宽，同时具有很好的容灾能力。缺点却也很明显，网络制约了分布式文件系统的延迟表现，而且网络的不稳定性也会影响到分布式文件系统的性能抖动。

### Ceph

　　Ceph是一种为优秀的性能、可靠性和可扩展性而设计的统一的、分布式文件系统。作为分布式文件系统，其能够在维护 POSIX 兼容性的同时加入了复制和容错功能。从 2010 年 3 月底，您可以在Linux 内核（从2.6.34版开始）中找到 Ceph 的身影，作为Linux的文件系统备选之一，Ceph.ko已经集成入Linux内核之中。虽然目前Ceph 可能还不适用于生产环境，但它对测试目的还是非常有用的。  
　　Ceph 生态系统架构可以划分为四部分：
- Clients：客户端（数据用户）
- mds：Metadata server cluster，元数据服务器（缓存和同步分布式元数据）
- osd：Object storage cluster，对象存储集群（将数据和元数据作为对象存储，执行其他关键职能） 
- mon：Cluster monitors，集群监视器（执行监视功能）
    
### GlusterFS

　　GlusterFS系统是一个可扩展的网络文件系统，相比其他分布式文件系统，GlusterFS具有高扩展性、高可用性、高性能、可横向扩展等特点，并且其没有元数据服务器的设计，让整个服务没有单点故障的隐患。  
　　GlusterFS系统架构可以划分为三部分：
- glusterd:是一个管理模块，处理gluster发过来的命令，处理集群管理、存储池管理、brick管理、负载均衡、快照管理等。集群信息、存储池信息和快照信息等都是以配置文件的形式存放在服务器中，当客户端挂载存储时，glusterd会把存储池的配置文件发送给客户端。
- glusterfsd：是服务端模块，存储池中的每个brick都会启动一个glusterfsd进程。此模块主要是处理客户端的读写请求，从关联的brick所在磁盘中读写数据，然后返回给客户端。
- glusterfs：是客户端模块，负责通过mount挂载集群中某台服务器的存储池，以目录的形式呈现给用户。当用户从此目录读写数据时，客户端根据从glusterd模块获取的存储池的配置文件信息，通过DHT算法计算文件所在服务器的brick位置，然后通过Infiniband RDMA 或Tcp/Ip 方式把数据发送给brick，等brick处理完，给用户返回结果。存储池的副本、条带、hash、EC等逻辑都在客户端处理。

