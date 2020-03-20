Ihyperledger
===============

## 基本概念

### peer

一个区块链网络主要包含了一堆对等节点（peer node）。对等节点是网络中最基本的元素，因为它们包含了账本以及智能合约

### orderer

Orderer 节点（Ordering Service Node，OSN）在网络中起到代理作用，多个 Orderer 节点会连接到 Kafka 集群，利用 Kafka 的共识功能，完成对网络中交易的排序和打包成区块的工作

### chaincode

智能合约,上文已提到。每个chaincode可提供多个不同的调用命令

### channel

私有的子网络,事实上是为了隔离不同的应用,一个channel可含有一批chaincode。

### endorsing

背书。金融上的意义为:指持票人为将票据权利转让给他人或者将一定的票据权利
授予他人行使,而在票据背面或者粘单上记载有关事项并签章的行为。通常我们引申为对某个
事情负责。在我们的共识机制的投票环节里,背书意味着参与投票。

### transaction

交易,每条指令都是一次交易。

### PKI

Public Key Infrastructure,一种遵循标准的利用公钥加密技术为电子商务的开展提供一
套安全基础平台的技术和规范

### MSP

Membership Service Provider,联盟链成员的证书管理,它定义了哪些RCA以及ICA
在链里是可信任的,包括定义了channel上的合作者

### org

orginazation,管理一系列合作企业的组织

## install（V2.0.0）

### Sample安装

1. For new Droplets, always set the locale (choose en_US.UTF-8 if in doubt) and do apt update/upgrade

```
sudo dpkg-reconfigure locales
sudo apt-get update && sudo apt-get upgrade
```

2. Setup a new user, it’s good practice not to use root to install these

   ```
   sudo adduser dora
   sudo usermod -aG sudo dora
   #Switch to the newly created user
   su - dora
   ```

3. Set up the Prerequisites, logout and log back in as dora

   ```
   sudo apt-get install curl git docker.io docker-compose nodejs npm python
   #Updating npm to 5.6.0
   sudo npm install npm@5.6.0 -g
   #Setting up docker configuration
   sudo usermod -a -G docker $USER
   sudo systemctl start docker
   sudo systemctl enable docker
   #Installing golang
   wget https://dl.google.com/go/go1.13.6.linux-amd64.tar.gz
   tar -xzvf go1.13.6.linux-amd64.tar.gz
   sudo mv go/ /usr/local
   #edit gopath in .bashrc
   pico ~/.bashrc
   #(add these 2 lines to end of .bashrc file)
   export GOPATH=/usr/local/go
   export PATH=$PATH:$GOPATH/bin
   #exit and log back in as dora
   exit
   su - dora
   ```

4. This is the magic step to setup all the images needed for Hyperledger Fabric v2.0.

   ```
   curl -sSL https://bit.ly/2ysbOFE | bash -s 2.0.0
   ```

   如上命令将下载一个bootstrap.sh 文件

5. That’s it. You have done it. 

6. start 

   ```
   git clone https://github.com/hyperledger/fabric.git
   cd fabric
   git checkout v2.0.0
   git clone -b master https://github.com/hyperledger/fabric-samples.git
   cd fabric-samples
   git checkout v2.0.0-beta
   cd first-network
   ./byfn.sh generate
   ./byfn.sh up
   docker ps -a
   ./byfn.sh down
   ```

#### 问题处理

1. error getting chaincode bytes: failed to calculate dependencies: incomplete package

   可能是fabric-samples 和fabirc的版本不匹配



## 多节点安装

