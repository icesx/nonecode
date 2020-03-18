Ihyperledger
===============

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
   cd fabric-samples/first-network
   ./byfn.sh generate
   ./byfn.sh up
   ```

#### 问题处理

1.  error getting chaincode bytes: failed to calculate dependencies: incomplete package
2. 



### 多节点安装

