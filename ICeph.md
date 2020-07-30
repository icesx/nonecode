ceph
===

## install

> 1. 准备服务器
>
>    ```
>    bjrdc208
>    bjrdc209 
>    bjrdc210
>    ```
>
> 2. 免秘登录
>
>    ```
>    ssh-copy-id bjrdc210
>    ```
>
> 3. 安装 ceph-deploy
>
>    ```
>    wget -q -O- 'https://download.ceph.com/keys/release.asc' | apt-key add -
>    echo deb https://download.ceph.com/debian-octopus/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list
>    apt update
>    apt install ceph-deploy
>    ```
>
> 4. new
>
>    ```
>    cd /cloud/ceph
>    ceph-deploy new bjrdc208
>    ```
>
> 5. install
>
>    ```
>    export CEPH_DEPLOY_REPO_URL=https://mirrors.aliyun.com/ceph/debian-octopus
>    #使用阿里云源
> ceph-deploy install bjrdc208 bjrdc209 bjrdc210
>    ```
>    
>    > 出现如下问题
>    >
>    > ```
>    > sudo: no tty present and no askpass program specified
>    > ```
>    >
>    > 解决办法，在目标主机bjrdc209上执行如下命令
>    >
>    > ```
>    > echo "bjrdc ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers.d/bjrdc
>    > ```
>    >
>    > 

###  配置

> 1. Deploy Ceph Cluster Monitor
>
>    Ceph cluster monitor has been initialized above. To deploy it, execute the command below on Ceph Admin node.
>
>    ```
>    ceph-deploy mon create-initial
>    ```
>
>    
>
> 2. Copy the Ceph Configuration files and Keys
>
>    ```
>    ceph-deploy admin bjrdc208 bjrdc209 bjrdc210
>    ```
>
> 3. Deploy Ceph Manager Daemon
>
>    ```
>    ceph-deploy mgr create bjrdc208
>    ```
>
> 4. deploy mds
>
>    ```
>    ceph-deploy --overwrite-conf mds create bjrdc208 bjrdc209 bjrdc210
>    ```
>    
>    
>    
> 5. Attach Logical Storage Volumes to Ceph OSD Nodes
>
>    ```
>    ceph-deploy osd create --data /dev/sdb1 bjrdc208
>    ceph-deploy osd create --data /dev/sdb1 bjrdc209
>    ceph-deploy osd create --data /dev/sdb1 bjrdc210
>    ```



### 使用

> 1. health
>
>    ```
>    ceph health
>    ```
>
>    > Degraded data redundancy: 1 pg undersized
>    >
>    > 
>
> 2. state
>
>    ```
>    sudo ceph -s
>    ```
>
> 3. add more mon
>
>    add public network = 192.168.2.0/24
>
>    ```
>    vim ceph.conf
>    ```
>
>    ```
>    [global]
>    fsid = ecc4e749-830a-4ec5-8af9-22fcb5cadbca
>    mon_initial_members = ceph-osd01
>    mon_host = 192.168.2.114
>    auth_cluster_required = cephx
>    auth_service_required = cephx
>    auth_client_required = cephx
>    public network = 192.168.2.0/24
>    ```
>    
>    ​       
>    
>    ```
>    ceph-deploy mon add bjrdc210
>    ```

### and new node

> 1. installl to node
>
>    ```
>    export CEPH_DEPLOY_REPO_URL=https://mirrors.aliyun.com/ceph/debian-octopus
>    #使用阿里云源
>    ceph-deploy install  bjrdc211
>    ```
>
> 2. on node server
>
>    ```
>    echo "bjrdc ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers.d/bjrdc
>    ```
>
>    
>
> 3. create osd
>
>    ```
>    ceph-deploy osd create --data /dev/sdb1 bjrdc211
>    ```
>
>    

### dashboard

>安装dasnboard
>
>```
>sudo apt install ceph-mgr-dashboard
>```
>
>enable
>
>```
>sudo ceph mgr module enable dashboard
>```
>
>如果出现如下问题
>
>`module 'dashboard' reports that it cannot run on the active manager daemon: No module named 'distutils.util'`
>
>```
>sudo apt install python3-distutils
>sudo systemctl restart ceph-mgr@bjrdc208.service
>sudo ceph mgr module enable dashboard
>```
>
>关闭 ssl
>
>```
>ceph config set mgr mgr/dashboard/ssl false
>```
>
>帐号密码
>
>```
>sudo ceph dashboard ac-user-create bjrdc xxx administrator
>```
>
>
>
>访问
>
>`http://bjrdc208:7000`



## RBD

Ceph可以同时提供对象存储RADOSGW、块存储RBD、文件系统存储Ceph FS。 RBD即RADOS Block Device的简称，RBD块存储是最稳定且最常用的存储类型。RBD块设备类似磁盘可以被挂载。 RBD块设备具有快照、多副本、克隆和一致性等特性，数据以条带化的方式存储在Ceph集群的多个OSD中。

> 创建rbd
>
> ```
> sudo ceph osd pool create rdb_pool_01 128 128
> sudo rbd pool init rbd_pool_01
> sudo ceph osd pool set-quota rdb_pool_01 max_bytes $((10 * 1024 * 1024 * 1024))
> ```
>
> 创建块存储
>
> ```
> sudo rbd create rdb_pool_01/volume01 --size $((2*1024))
> sudo rbd --image rdb_pool_01/volume01 info
> sudo rados -p rdb_pool_01 ls
> sudo rbd info rdb_pool_01/volume01
> ```
>
> 挂载
>
> 1. 准备工作
>
>    ```
>    cat >/etc/ceph/ceph.conf <<EOF 
>    [global]
>    fsid = d84f5d5b-f8d5-42c6-ab8f-e1240e9bcf78
>    mon_initial_members = bjrdc208
>    mon_host = 172.16.15.208
>    auth_cluster_required = cephx
>    auth_service_required = cephx
>    auth_client_required = cephx
>    public network=172.16.15.0/24
>    
>    [mgr]
>    debug mgr balancer = 1/20
>    
>    EOF
>    ```
>
>    copy keyring
>
>    ```
>    scp **/ceph.client.admin.keyring xxx:/home/bjrdc/
>    sudo mv ceph.client.admin.keyring  /etc/ceph/
>    ```
>
>    
>
> 2. 挂载
>
>    ```
>    sudo rbd map rdb_pool_01/volume01
>    sudo mkfs.ext4 /dev/rbd/rdb_pool_01/volume01 /moa-rbd
>    ```
> ```
>    
>    
> 
> 
> ```

## cephfs

ceph application not enabled，toenable cephfs

```
sudo ceph osd pool application enable bjrdc-pool cephfs
```

create cephfs

```
sudo ceph osd pool create bjrdc-pool 128
sudo ceph osd pool create bjrdc-pool-metadata 128
sudo ceph fs new cephfs bjrdc-pool-metadata bjrdc-pool
```



mount

```
sudo mount -t  ceph bjrdc208:/ /ceph-data/ -o name=admin,secret=xxxxx==
```

mount target

```
sudo mount -t ceph bjrdc208:/some/directory/in/cephfs /mnt/mycephfs
```



secret从ceph配置文件中获取

```
cat ceph.client.admin.keyring 
[client.admin]
       key = xxxxx==
       caps mds = "allow *"
       caps mgr = "allow *"
       caps mon = "allow *"
       caps osd = "allow *"
```

mount on fstab

ceph_secretfile 为secret内容保存成的文件

```
bjrdc208:/mysql-root     /moa-ceph    ceph    name=admin,secretfile=/home/bjrdc/ceph_secretfile,noatime,_netdev    0       2
```





## 基本命令

>```
>sudo ceph health detail
>```
>
>**rbd** 
>
>```
>sudo rbd create k8s_pool_01/volume01 --size $((5* 1024)) 
>sudo rbd resize --size $((5*1024*1024)) k8s_pool_01/volume01
>sduo rbd rm {pool-name}/{image-name}
>sudo rbd list k8s_pool_01
>sudo rbd map k8s_pool_01/k8s_v1
>sudo rbd mv k8s_pool_01/k8s_v1 k8s_pool_01/k8s-v1
>```
>
>
>
>**osd** 
>
>objrdc storage daemon
>
>```
>ceph osd lspools
>rados df
>ceph osd pool create bjrdc-pool 128 128
>ceph osd pool rename bjrdc-pool bjrdc-pool-new
>ceph osd pool set-quota bjrdc-pool max_bytes $((100 * 1024 * 1024 * 1024))
>ceph osd pool get-quota bjrdc-pool
>ceph osd pool mksnap bjrdc-pool bjrdc-pool-snapshot
>ceph osd pool rmsnap bjrdc-pool bjrdc-pool-snapshot
>sudo ceph osd tree
>```
>
>**文件操作**
>
>```
>rados -p bjrdc-pool put testfile /etc/hosts
>rados -p bjrdc-pool ls
>rados -p bjrdc-pool rm testfile
>```
>
>**mon**
>
>```
>sudo ceph mon dump
>```
>
>**pg**
>
>```
>ceph pg stat
>```
>
>**fs**
>
>```
>sudo ceph fs reset cephfs --yes-i-really-mean-it
>
>```
>
>[参考]: https://docs.ceph.com/docs/jewel/cephfs/administration/
>
>**df**
>
>```
>sudo ceph df
>```
>
>**log**
>
>```
>sudo ceph log
>```
>
>
>
>**mgr**
>
>```
>sudo ceph mgr module ls
>sudo ceph mgr services
>sudo ceph mgr module enable dashboard
>```
>
>

## 性能测试

### IO



## 基本概念

### 存储

#### 对象存储（RGW:RADOS gateway）

Ceph 对象存储服务提供了 REST 风格的 API ，它有与 Amazon S3 和 OpenStack Swift 兼容的接口。也就是通常意义的键值存储，其接口就是简单的GET、PUT、DEL和其他扩展;

#### 块存储（RBD：RADOS block device）

RBD 是通过librbd库对应用提供块存储，主要面向云平台的虚拟机提供虚拟磁盘；RBD类似传统的SAN存储，提供数据块级别的访问；

目前 RBD 提供了两个接口，一种是直接在用户态实现， 通过 QEMU Driver 供 KVM 虚拟机使用。 另一种是在操作系统内核态实现了一个内核模块。通过该模块可以把块设备映射给物理主机，由物理主机直接访问。

#### 文件存储

Ceph 文件系统服务提供了兼容 POSIX 的文件系统，可以直接挂载为用户空间文件系统。它跟传统的文件系统如Ext4是一个类型，区别在于分布式存储提供了并行化的能力；

### 组件

- Monitor：一个 Ceph 集群需要多个 Monitor 组成的小集群，它们通过 Paxos 同步数据，用来保存 OSD 的元数据。

  ### OSD

  OSD 全称 Object Storage Device，也就是负责响应客户端请求返回具体数据的进程，一个Ceph集群一般有很多个OSD。

  *可以理解为一个节点*

- CRUSH：CRUSH 是 Ceph 使用的数据分布算法，类似一致性哈希，让数据分配到预期的位置。

  #### PG

  PG全称 Placement Groups，是一个逻辑的概念,一个PG 包含多个 OSD 。引入 PG 这一层其实是为了更好的分配数据和定位数据。

  每个OSD上分布很多PG, 并且每个PG会自动散落在不同的OSD上。如果扩容那么相应的PG会进行迁移到新的OSD上，保证PG数量的均衡

  

- Object：Ceph 最底层的存储单元是 Object对象，每个 Object 包含元数据和原始数据。

- RADOS：实现数据分配、Failover 等集群操作。

- Libradio：Libradio 是RADOS提供库，因为 RADOS 是协议，很难直接访问，因此上层的 RBD、RGW和CephFS都是通过libradios访问的，目前提供 PHP、Ruby、Java、Python、C 和 C++的支持。

  ### MDS

  MDS全称Ceph Metadata Server，是CephFS服务依赖的元数据服务。

  

- RBD：RBD全称 RADOS Block Device，是 Ceph 对外提供的块设备服务。

- RGW：RGW全称RADOS gateway，是Ceph对外提供的对象存储服务，接口与S3和Swift兼容。

- CephFS：CephFS全称Ceph File System，是Ceph对外提供的文件系统服务。