Kubernetes Doing
=====

## 架构

![架构图](https://jimmysong.io/kubernetes-handbook/images/architecture.png)

- etcd保存了整个集群的状态；
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）的管理；
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
- kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；

![架构图](https://jimmysong.io/kubernetes-handbook/images/kubernetes-high-level-component-archtecture.jpg)
## 云原生

这里我们抛出一个我们自己的理解：云原生代表着原生为云设计。

详细的解释是：应用原生被设计为在云上以最佳方式运行，充分发挥云的优势。

CNCF，全称Cloud Native Computing Foundation（云原生计算基金会），成立于 2015 年7月21日（[于美国波特兰OSCON 2015上宣布](https://www.cncf.io/announcement/2015/06/21/new-cloud-native-computing-foundation-to-drive-alignment-among-container-technologies/)），其最初的口号是**坚持和整合开源技术来让编排容器作为微服务架构的一部分**，其作为致力于云原生应用推广和普及的一支重要力量，不论您是云原生应用的开发者、管理者还是研究人员都有必要了解

https://github.com/cncf/landscape

![](https://landscape.cncf.io/images/serverless.png)

## 体验Docker

> kubernetes 1.20.x后将逐步去掉docker，改用containerd作为ori

1. hello-world

   ```
   docker run hello-world
   ```

2. node.js

   server.js 代码

   ```js
   cat > server.js <<EOF 
   var http = require('http');
   
   var handleRequest = function(request, response) {
   console.log('Received request for URL: ' + request.url);
   response.writeHead(200);
   response.end('Hello World!');
   };
   var www = http.createServer(handleRequest);
   www.listen(8080);
   EOF
   ```

   Dockerfile

   ```dockerfile
   cat > Dockerfile <<EOF
   FROM bjrdc206.reg/library/node:8.10.0
   EXPOSE 8080
   COPY server.js .
   CMD [ "node", "server.js" ]
   EOF
   ```

   制作镜像

   ```sh
   docker build -t bjrdc206.reg/bjrdc-dev/hello-node:v1.0.1 .
   ```

   执行

   ```shell
   docker run -p 8080:8080 bjrdc206.reg/bjrdc-dev/hello-node:v1.0.1
   docker run -p 8080:8080 -d bjrdc206.reg/bjrdc-dev/hello-node:v1.0.1
   ```

   访问

   ```shell
   curl localhost:8080
   ```

   

## harbor

>Harbor 是 Vmware 公司开源的 企业级的 Docker Registry 管理项目
>
>它主要 提供 Dcoker Registry 管理UI，可基于角色访问控制, AD/LDAP 集成，日志审核等功能，完全的支持中文。
>
>在安装kubernetes之前需要安装一个harbor用于本地镜像的服务。

### 安装

#### 1.准备证书

1. Generate a CA certificate private key

   ```sh
   openssl genrsa  -out ca.key 4096
   ```

2. Generate the CA certificate.

   ```sh
   openssl req -x509 -new -nodes -sha512 -days 3650  -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=bjrdc206.reg"  -key ca.key -out ca.crt
   ```

3. Generate a Server Certificate

   ```sh
   openssl genrsa -out bjrdc206.reg.key 4096
   ```

4. Generate a certificate signing request (CSR).

   ```sh
   openssl req -sha512 -new     -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=bjrdc206.reg"     -key bjrdc206.reg.key -out bjrdc206.reg.csr
   ```

   I had the same issue as you on Ubuntu 18.04.x. Removing (or commenting out) `RANDFILE = $ENV::HOME/.rnd` from `/etc/ssl/openssl.cnf` worked for me.

5. Generate an x509 v3 extension file

   ```sh
   cat > v3.ext <<EOF
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
   extendedKeyUsage = serverAuth
   subjectAltName = @alt_names
   
   [alt_names]
   DNS.1=bjrdc206.reg
   EOF
   ```

   

6. Use the `v3.ext` file to generate a certificate for your Harbor host.

   ```sh
   openssl x509 -req -sha512 -days 3650 \
       -extfile v3.ext \
       -CA ca.crt -CAkey ca.key -CAcreateserial \
       -in bjrdc206.reg.csr \
       -out bjrdc206.reg.crt
   ```

7. 可选操作

   1. Copy the server certificate and key into the certficates folder on your Harbor host

      ```
      cp bjrdc206.reg.crt /docker/cert/
      cp bjrdc206.reg.key /docker/cert/
      ```

      

   2. Convert `yourdomain.com.crt` to `yourdomain.com.cert`, for use by Docker.

      ```
      openssl x509 -inform PEM -in bjrdc206.reg.crt -out bjrdc206.reg.cert	
      ```

      

   3. Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host. You must create the appropriate folders first

      ```
      mkdir /etc/docker/certs.d/bjrdc206.reg -p
      cp bjrdc206.reg.cert /etc/docker/certs.d/bjrdc206.reg/
      cp bjrdc206.reg.key /etc/docker/certs.d/bjrdc206.reg/
      cp ca.crt /etc/docker/certs.d/bjrdc206.reg/
      ```

      ```
      cp bjrdc206.reg.crt /usr/local/share/ca-certificates/
      update-ca-certificates
      ```

      

8. 下载ca.crt到本地，添加到浏览器中，并信任后，即可通过浏览器访问

   ```
   https://bjrdc206.reg
   ```

9. 如果客户端pull或者push需要证书的话，需要将bjrdc206.reg.crt和ca.crt复制到对应的主机上

   ```sh
   sudo cp /home/bjrdc/bjrdc206.reg.crt /usr/local/share/ca-certificates/
   sudo cp  /home/bjrdc/ca.crt /etc/ca-certificates/update.d/
   sudo update-ca-certificates
   ```

   

#### 2.使用docker安装

1. 安装docker

   ```sh
   systemctl enable docker.service
   systemctl restart docker
   ```

2. 下载离线安装包

   ```
   wget https://github.com/goharbor/harbor/releases/download/v2.2.3/harbor-offline-installer-v2.2.3.tgz
   ```

   

3. install docker-compose

   ```sh
   sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   
   ```

4. 修改 harbor.yml

   hostname `bjrdc206.reg` 必须和证书的hostname相同

   ```yaml
   hostname: bjrdc206.reg
   
   # http related config
   http:
     # port for http, default is 80. If https enabled, this port will redirect to https port
     port: 80
   
   # https related config
   https:
     # https port for harbor, default is 443
     port: 443
     # The path of cert and key files for nginx
     certificate: /docker/cert/bjrdc206.reg.crt
     private_key: /docker/cert/bjrdc206.reg.key
    ...
   ```

   ```sh
   ./install.sh
   ```


#### 3.自启动

1. 创建service文件

   ```sh
   cat >harbor.service <<EOF
   [Unit]
   Description=Redis
   After=network.target
   
   [Service]
   ExecStart=/usr/local/bin/docker-compose -f /docker/harbor/docker-compose.yml start 
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

   

2. 服务质量harbor.service

   ```sh
   cp harbor/harbor.service /lib/systemd/system/
   ```

   

3. enable service

   ```sh
   systemctl enable harbor
   ```

   

#### 4. 问题处理

1. :failed to connect to tcp://postgresql:5432

   查看日志时,发现错误:failed to connect to tcp://postgresql:5432
   解决办法:

   ```sh
   cd /docker/harbor
   sudo docker-compose down -v
   docker-compose up -d
   ```

   ```
   停止并删除docker容器:docker-compose down -v
   启动所有docker容器:docker-compose up -d
   ```

   

### 界面配置



### push

1. on docker

   修改host 的docker配置，让https生效

   ```sh
   cat > /etc/docker/daemon.json <<EOF
   {
     "graph": "/docker",
     "exec-opts": ["native.cgroupdriver=systemd"],
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "100m"
     },
     "storage-driver": "overlay2",
     "insecure-registries":["bjrdc206.reg"] 
   }
   EOF
   sudo service docker restart
   ```

   >  "insecure-registries":["bjrdc206.reg"],用于告知客户端信任该证书

   ```sh
    docker login bjrdc206.reg
    docker tag hello-world bjrdc206:443/bjrdc-dev/hello-world:v1.0.0
    sudo docker push bjrdc206.reg/bjrdc-dev/hello-world:v1.0.0
   ```

2. on ctr

   

### register

为harbo增加远程源

在harbor的管理界面中，的“registries”中增加`https://registry-1.docker.io`

### 重启

使用docker-compose

```sh
sudo docker-compose down
sudo docker-compose up -d -f /docker/harbor/docker-compose.yml
```

或者直接重启host

## CRI

### docker

目前1.20.x版本后已经不需要安装docker，如果是老版本的，需要安装docker。安装方式如下

```sh
sudo apt install -y docker.io
systemctl enable docker.service
cat > /etc/docker/daemon.json <<EOF
{
"graph": "/docker",
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
 "max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
```



### 操作系统配置

```sh
cat <<EOF | sudo tee /etc/modules-load.d/cri.conf 
overlay 
br_netfilter 
EOF
```



```sh
sudo modprobe overlay 
sudo modprobe br_netfilter
```

```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

```
sudo sysctl --system
```



### containerd（kubernetes>=1.20.0）

 ctr is an unsupported debug and administrative client for interacting
 with the containerd daemon. Because it is unsupported, the commands,
 options, and operations are not guaranteed to be backward compatible or
 stable from release to release of the containerd project

#### 安装

如果是v1.20后版本，需要安装containerd

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```



```sh
sudo apt-get install -y containerd
```

```
sudo systemctl restart containerd
```



#### 配置

1. 非root执行

   config.toml中设置uid和gid

   ```toml
   [grpc]
     address = "/run/containerd/containerd.sock"
     tcp_address = ""
     tcp_tls_cert = ""
     tcp_tls_key = ""
     uid = 1000
     gid = 1000
   ```

   

4. 安装harbor register的证书

   ```sh
   sudo cp /home/bjrdc/bjrdc206.reg.crt /usr/local/share/ca-certificates/
   sudo cp  /home/bjrdc/ca.crt /etc/ca-certificates/update.d/
   sudo update-ca-certificates
   ```

5. 修改本地存储路径

   cat /etc/containerd/config.toml

   ```toml
   version = 2
   root = "/cloud/var/lib/containerd"
   state = "/run/containerd"
   ```

6. 增加register的登录信息

   vi /etc/containerd/config.toml

   ```toml
           [plugins."io.containerd.grpc.v1.cri".registry.mirrors."bjrdc206.reg"]
             endpoint = ["https://bjrdc206.reg"]
           [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."bjrdc206.reg".auth]
          username = "admin"
          password = "Harbor12345"
   ```

   ```
   sudo systemctl restart containerd
   ```
   
   

#### pause

国内有墙的原因，导致google的镜像拿不下来，但是docker.io作了映射如下

在`k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10`可以映射为`mirrorgooglecontainers/kubernetes-zookeeper:1.0-3.4.10`

映射下来的的镜像可以通过docker tag, docker push 到私有的harbor中再进行使用。

特别是在kubernet1.20后默认配置的pause用的是`k8s.gcr.io/pause:3.1`，这个镜像没有下载下来的话，节点无法上线。可以使用如下方式

1. 使用docker push镜像到harbor

   注：最好单独搞一台虚拟机用于docker的操作，不要将docker安装到kubernetes节点上。

   ```sh
   docker pull mirrorgooglecontainers/pause:3.1
   docker tag mirrorgooglecontainers/pause:3.1 bjrdc206.reg/gcr/pause:3.1
   docker push
   ```

2. containerd to harbor

   ```sh
   sudo containerd config default|sudo tee /etc/containerd/config.toml
   ```

3. 修改 pause的register

   ```sh
   sudo sed -i "s/k8s.gcr.io\/pause:3.1/bjrdc206.reg\/gcr\/pause:3.1/g" /etc/containerd/config.toml
   ```

   或者

   ```toml
     [plugins."io.containerd.grpc.v1.cri"]
       disable_tcp_service = true
       stream_server_address = "127.0.0.1"
       stream_server_port = "0"
       stream_idle_timeout = "4h0m0s"
       enable_selinux = false
       sandbox_image = "bjrdc206.reg/gcr/pause:3.1"
   ```

5. 重启服务

   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart containerd
   ```

   



#### ctr 命令

1. pull

   ```sh
   ctr image pull --skip-verify bjrdc206.reg/bjrdc-dev/java:8-jdk-bjrdc-v1.0.1
   ctr images pull hub-mirror.c.163.com/library/redis:alpine
   ctr images pull hub.c.163.com/library/nginx:latest
   ```

   如果没有安装bjrdc206.reg的证书的话会报错误

   `x509: certificate signed by unknown authority`

   解决办法是下载bjrdc206.reg的证书`/etc/docker/certs.d/bjrdc206.reg/ca.crt`，并安装到本地

   ```sh
   cp ca.crt /etc/ca-certificates/update.d
   sudo update-ca-certificates
   ```

   

2. run

   ```sh
   bjrdc@bjrdc105:~$ ctr images ls
   REF                                                             TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                                                                LABELS hub-mirror.c.163.com/library/redis:alpine                       application/vnd.docker.distribution.manifest.list.v2+json sha256:2cd821f730b90a197816252972c2472e3d1fad3c42f052580bc958d3ad641f96 10.1 MiB  linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64/v8,linux/ppc64le,linux/s390x -      hub.c.163.com/library/nginx:latest                              application/vnd.oci.image.manifest.v1+json                sha256:8eeb06742b41fb67514e4b14049f6740dc582520486d7a1612e78c55b1dbe40e 41.2 MiB  linux/amd64 
   ```

   ```sh
   sudo ctr run hub-mirror.c.163.com/library/redis:alpine redis-v11:C 20 Jan 2021 01:56:25.900 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo...
   ```

   

3. ls/rm

   ```
   ctr c ls
   ctr c rm nginx-test-1
   ```

4. task

   ```
   ctr task ls
   sudo ctr task attach redis-v1
   ```
   
   

### CRI-O

#### 安装

| 操作系统     | `$OS`           |
| ------------ | --------------- |
| Ubuntu 20.04 | `xUbuntu_20.04` |
| Ubuntu 19.10 | `xUbuntu_19.10` |
| Ubuntu 19.04 | `xUbuntu_19.04` |
| Ubuntu 18.04 | `xUbuntu_18.04` |

然后，将 `$VERSION` 设置为与你的 Kubernetes 相匹配的 CRI-O 版本。 例如，如果你要安装 CRI-O 1.20, 请设置 `VERSION=1.20`. 你也可以安装一个特定的发行版本。 例如要安装 1.20.0 版本，设置 `VERSION=1.20:1.20.0`

```
export OS=xUbuntu_18.04
export VERSION=1.22
```



```shell
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF
```

```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers-cri-o.gpg add -

```



```
sudo apt-get update
sudo apt-get install cri-o cri-o-runc
```

#### 配置

1. 默认配置

   ```sh
   cat << EOF |sudo tee /etc/crio/crio.conf
   `sudo crio config --default`
   EOF
   ```

2. 修改storage

   ubuntu18，的内核版本低，不支持`metacopy=off`

   ```sh
   sudo sed -i s/nodev\,metacopy=on//g /etc/crio/crio.conf
   ```

   ```toml
   storage_option = [
           "overlay.mountopt=",
   ]
   ```

   ```sh
   sudo systemctl restart crio
   ```

3. 秀爱pause

   vi /etc/crio/crio.conf

   ```
   #pause_image = "k8s.gcr.io/pause:3.5"
   pause_image = "bjrdc206.reg/gcr/pause:3.2"
   ```

   

4. 命令

   ```
   sudo ctictl image 
   ```

   

#### 登录

暂时未找到crio的登录register的配置

#### 启动

```
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```



## Kubernetes install

### 准备工作

1. 确保mac和uuid唯一

   您可以使用命令 `ip link` 或 `ifconfig -a` 来获取网络接口的 MAC 地址

   ```shell
   sudo cat /sys/class/dmi/id/product_uuid
   ```

   

2. 在 Linux 中，nftables 当前可以作为内核 iptables 子系统的替代品。 `iptables` 工具可以充当兼容性层，其行为类似于 iptables 但实际上是在配置 nftables。 nftables 后端与当前的 kubeadm 软件包不兼容：它会导致重复防火墙规则并破坏 `kube-proxy`。

   如果您系统的 `iptables` 工具使用 nftables 后端，则需要把 `iptables` 工具切换到“旧版”模式来避免这些问题。 默认情况下，至少在 Debian 10 (Buster)、Ubuntu 19.04、Fedora 29 和较新的发行版本中会出现这种问题。RHEL 8 不支持切换到旧版本模式，因此与当前的 kubeadm 软件包不兼容。
   
   ```
   update-alternatives --set iptables /usr/sbin/iptables-legacy
   update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
   update-alternatives --set arptables /usr/sbin/arptables-legacy
   update-alternatives --set ebtables /usr/sbin/ebtables-legacy
   ```
   
3. disable swap

   ```
   sudo swapoff -a
   ```

   change /etc/fatab

   reboot

### 安装

#### master

1. disable swap
   
   ```sh
   sudo vi /etc/fstab
   #/dev/mapper/fw--vg-swap_1 none            swap    sw              0       0
   ```
   
   ```sh
   sudo reboot
   ```
   
   ​        
   
2. sysctl

   ```bash
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sudo sysctl --system
   ```

3. 容器引擎

   1. enable docker

      > 1.20.x版本后不需要docker，故不需要安装docker

      
   
4. 安装

   1. （阿里云）

   

   ```sh
   sudo su root
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
   ```

   

   ```sh
   # 添加 k8s 镜像源
   sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   ```

    on master

   ```sh
   sudo apt update
   sudo apt-get install -y kubectl kubeadm kubelet
   ```

   

   1. 按照安装官方教程安装（需要梯子）

   ```bash
   sudo apt-get update && sudo apt-get install -y apt-transport-https curl
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

   

5. 环境变量

   ```sh
   cat <<EOF |sudo tee /var/lib/kubelet/kubeadm-flags.env
   KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf"
   EOF
   ```

6. crictl 配置

   ```
   sudo echo "runtime-endpoint: unix:///var/run/containerd/containerd.sock" |sudo tee /etc/crictl.yaml
   ```

   

7. 重启

   ```
   reboot
   ```

   

8. 初始化

   ```sh
   kubeadm init \
   --apiserver-advertise-address=172.16.15.17 \
   --image-repository registry.aliyuncs.com/google_containers \
   --pod-network-cidr=10.244.0.0/16 \
   --kubernetes-version=v1.21.0\
   --cri-socket=/run/containerd/containerd.sock
   #如果无法下载，需要设置--kubernetes-version为当前registry服务器上有的版本
   ```

   v1.20.x 之后的版本废弃了docker，可以选择新版本安装。

   ```sh
   kubectl get nodes
   ```

   初始化过程中会出现多次找不到镜像的问题，需要人工去下载进行。

   ```
   kubeadm reset -f
   ```

   

   执行完成后，返回内容如下，在slaver节点执行。

   kubeadm join 172.16.15.17:6443 --token 1cx9wb.3bkuu2eq5qh8vn9k  --discovery-token-ca-cert-hash sha256:f680fe2f1575db37e653e2879ded96efc40e5104acd69aa66947b350ec3d35ce

   

9. .kube/config

   ```sh
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

10. 部署 flannel 网络

   > Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务；简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址，并使Docker容器可以互连。
   >
   > ```sh
   > kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   > ```
   >
   > *但是这个网络是做什么的呢？*
   >
   > **打通pod与集群**

   1. 查看pod

      ```sh
      kubectl get pod --all-namespaces
      ```

11. 安装dashborad

    ```sh
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
    ```

#### multi master



#### Node

​	<span id="node_install">node_install</span>

1. 环境准备

   1. disable swap

      ```
      sudo vi /etc/fstab
      #/dev/mapper/fw--vg-swap_1 none            swap    sw              0       0
      ```

   2. sysctl

      ```sh
      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
      sudo sysctl --system
      ```

   3. docker

      详见上文

   4. containerd

      详见上文
   
   5. cri-o
   
      详见上文
   
2. 安装kubectl kubeadm

   

   ```sh
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
   ```

   

   ```sh
   cat <<EOF |sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   ```

   ```sh
   sudo apt update && sudo apt install -y kubelet kubeadm
   ```

   

3. 修改环境变量

   1. contained

      如果默认安装，可能在在node增加到集群后，无法下载需要的pod，需要在`/var/lib/kubelet/kubeadm-flags.env `增加如下内容

      ```sh
      cat <<EOF |sudo tee /var/lib/kubelet/kubeadm-flags.env
      KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf"
      EOF
      ```

      如果是v1.20.0以后版本，使用containerd作为cri，默认是无法下载pause的，详见上文安装pause

   2. cri-o

      

4. 记得重新load，最好重启一下机器

   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

5. crictl 配置

   1. containerd

      ```sh
      sudo echo "runtime-endpoint: unix:///var/run/containerd/containerd.sock" |sudo tee /etc/crictl.yaml
      ```

   2. cri-o

      ```sh
      sudo echo "runtime-endpoint: unix:///run/crio/crio.sock" |sudo tee /etc/crictl.yaml
      ```

      ```sh
      sudo crictl images list
      ```

      

6. join

   1. on master

      ```sh
      kubeadm token list
      kubeadm token create --print-join-command
      
      kubeadm join 172.16.15.17:6443 --token h81gdw.duityezgzrxsl4g7     --discovery-token-ca-cert-hash sha256:18f9acf00a214334c0a8d284e5808a9eec346bfe99bee6b9ebb5b016c9d6ca1f
      ```

      

   2. on node

      ```
      kubeadm join 172.16.15.17:6443 --token h81gdw.duityezgzrxsl4g7     --discovery-token-ca-cert-hash sha256:18f9acf00a214334c0a8d284e5808a9eec346bfe99bee6b9ebb5b016c9d6ca1f
      ```


#### 问题处理

##### registry.aliyuncs.com/google_containers/coredns:v1.8.0: not found

因为阿里云上没有coredns:v1.8.0。

由于crictl没有tag命令，无法将者私有的register上的coredns标记为`registry.aliyuncs.com/google_containers/coredns:v1.8.0`，故需要变通：

1. 通过独立的机器上docker tag registry.aliyuncs.com/google_containers/coredns:v1.8.0，

2. docker save

   ```
   docker save -o coredns1.8.0.tar registry.aliyuncs.com/google_containers/coredns:v1.8.0
   ```

3. scp

4. ctr import

   ```sh
   sudo ctr -n=k8s.io images import coredns1.8.0.tar
   ```

   

##### failed to pull image \"k8s.gcr.io/pause:3.2\"

同上

##### FATA[0010] failed to connect: failed to connect: context deadline exceeded

```sh
cat << EOF|sudo tee /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
debug: false
EOF
```

#####  could not find a JWS signature in the cluster-info ConfigMap for token ID "d4xo0n"

token 过期

### master 上安装其他组件

#### dashboard

1. install

   ```sh
   wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
   kubectl apply -f recommended.yaml
   ```

3. 生成浏览器证书

   此时直接访问dasnboradr，使用如下地址，会forbidden

   ```http
   https://bjrdc17:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
   ```

   需要生成证书

   ```shell
   # 生成client-certificate-data
   grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt
   
   # 生成client-key-data
   grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key
   
   # 生成p12
   openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
   
   ```

4. 访问

   下载生成的kubecfg.p12文件，并导入浏览器
   
   使用浏览器打开
   
   ```http
   https://bjrdc17:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
   ```
   
   在master上通过如下命令获取token
   
   ```shell
   kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
   ```
   
   在登录界面输入token完成登录

5. 角色绑定

   此时登录将会没有权限看到resources，使用如下方式为admin-user用户绑定权限
   
    创建角色admin-user.rbac.yaml
   
    **创建名为admin-user的serviceaccount，放到kube-system namespace下，并将用户绑定到名称为cluster-admin的ClusterRole下**
   
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: admin-user
    namespace: kube-system
    ---
    # Create ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    	name: admin-user
    roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system
    ```
   
    执行该权限
   
    ```sh
    kubectl  create -f admin-user.rbac.yaml 
    ```
   
    查询token
   
    ```sh
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    
    ')
    Name:         admin-user-token-w4knf
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: admin-user
                  kubernetes.io/service-account.uid: 65323ead-467f-448d-b7ee-1c52a002f3c2
    
    Type:  kubernetes.io/service-account-token
    
    Data
    ====
    token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InhZbkI0S001RXlYbXV5UHgwZVBKYzBYMUFUQnF2NFhGUW1iLTlRNW45ZFkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXc0a25mIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2NTMyM2VhZC00NjdmLTQ0OGQtYjdlZS0xYzUyYTAwMmYzYzIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.g4zufIj7ZUUrv9BgtPCd6djE5z7APV6bhE_OchKzczULdbuSkMBrLWwwbHm-0Jg5cUN37fTS-lFsMPxrt2Uw2_m0omx7N47qU-3LBdYxAwiBS-OBUDq6qfyWZoYsQizqdAf1y9kaxUZNbQ1iRMFqyH9-xgp-gk2rbixlOr0ToCOiDC0_FNjJ9bRnhjzQVCXoKQ0XefLuEv21AqeOpaN0U0lP8txziRIOI83grhtbF4RqDHxF0ZoiIakJ5KhKozff29am9lUYScNJpNc6ooqU2wvoNgXHeyODWohXOi9Q1cFPETpA_6kjKYxwpcqsMfJ85lTVPMOCadLV4YJq_h4Kfg
    ca.crt:     1025 bytes
    namespace:  11 bytes
    ```
   
    使用该token登录

#### heapster

> Heapster was initially [deprecated](https://github.com/kubernetes-retired/heapster/blob/master/docs/deprecation.md) in 1.11; users were encouraged to move to the `metrics-server` for similar functionality. With 1.18, the `cluster-monitoring` addons (Heapster, InfluxDB, and Grafana) have been removed from the Kubernetes source tree and therefore removed from the `cdk-addons` snap as well. Customers relying on these addons should migrate to a `metrics-server` solution prior to upgrading. Note: these removals do not affect the Kubernetes Dashboard nor the methods described in

#### metrics-server

> 资源监控，目前官方推荐的是metrics-server，安装方式如下：

1. 下载最新版本

   ```
   wget https://github.com/kubernetes-sigs/metrics-server/archive/v0.3.6.tar.gz
   tar -xzvf v0.3.6.tar.gz
   cd metrics-server-0.3.6/deploy/1.8+
   ```

   

2. 修改配置文件

   ```
   vi metrics-server-deployment.yaml
   ```

   ```
         containers:
         - name: metrics-server
           image: mirrorgooglecontainers/metrics-server-amd64:v0.3.6
           imagePullPolicy: Always
           command:
               - /metrics-server
               - --kubelet-preferred-address-types=InternalDNS,InternalIP,ExternalDNS,ExternalIP,Hostname
               - --kubelet-insecure-tls
   ```

   

3. 安装

   ```
   kubectl apply -f .
   ```

   

4. 验证

   ```
   kubectl top node
   NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
   bjrdc17    188m         2%     1629Mi          16%       
   bjrdc205   41m          0%     944Mi           9%        
   bjrdc81    63m          0%     825Mi           8%  
   ```

   ```
   kubectl top pods -n kube-system
   NAME                              CPU(cores)   MEMORY(bytes)   
   coredns-7ff77c879f-26gn7          5m           8Mi             
   coredns-7ff77c879f-26j8v          4m           7Mi             
   etcd-bjrdc17                      28m          28Mi            
   kube-apiserver-bjrdc17            62m          276Mi           
   kube-controller-manager-bjrdc17   21m          42Mi            
   kube-flannel-ds-amd64-bd5j9       2m           13Mi            
   kube-flannel-ds-amd64-chvrj       5m           14Mi            
   kube-flannel-ds-amd64-zht42       2m           12Mi            
   kube-proxy-6vllz                  1m           17Mi            
   kube-proxy-7zlh2                  1m           12Mi            
   kube-proxy-tn7rg                  1m           12Mi            
   kube-scheduler-bjrdc17            5m           12Mi            
   metrics-server-85b7f6dc48-fnrsw   1m           13Mi   	
   ```

   

#### Ingress

> An API object that manages external access to the services in a cluster, typically HTTP.
>
> Ingress may provide load balancing, SSL termination and name-based virtual hosting.

> 一般在生产环境中使用。推荐使用ingress。
>
> Load balancer的问题是每一个服务都要有一个Load balancer，服务多了之后会很麻烦，这时就会用Ingress，它的缺点是配置起来比较复杂。

> 默认安装中并没有ingress的controller，需要另行安装。

1. 下载yaml，不需要进行修改可以直接使用

   ```sh
   wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
   ```
   
2. 准备image

   由于默认使用的镜像是`quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.46.0`这个镜像在k8s中无法下载，最好在github中找到一个可用的。

   ```
   docker pull bitnami/nginx-ingress-controller:0.46.0
   ```

   修改deploy.yaml 将其中的`quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0`替换为`bitnami/nginx-ingress-controller:0.46.0`

3. 安装ingress-nginx

   ```
   kubectl create -f deploy.yaml
   ```

4. 配置ingress

   安装了ingrss后，需要为服务配置ingress，假设已将安装了hello-node的服务，使用如下yaml，为该服务增加ingress

   ```yaml
   cat > hello-node-ingress.yaml <<EOF
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: hello-node-ingress
     namespace: bjrdc-dev
     annotations:
       nginx.ingress.kubernetes.io/proxy-body-size: "20M"
   spec:
     rules:
     - host: ingress.bjrdc17
       http:
         paths:
         - path: /hello
           backend:
             serviceName: hello-node
             servicePort: 3000
   EOF
   ```

5. ingress rewrite

   ingree 支持rewrite，从而可以将指定的路径映射到目标目录。特别是在和spring-boot进行整合的时候，可以将一个path，映射到/下
   
   如下的配置可以将所有的/sc-gateway开头的路径映射到spring-cloud-k8s-gateway服务8097端口的/下
   
   如`curl ingress.bjrdc17:30080/sc-gateway/consumer/feign/list`请求到达`curl spring-cloud-k8s-gateway.bjrdc-dev.svc.cluster.local:8097/consumer/feign/list`
   
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
   name: sc-gateway-ingress
   namespace: bjrdc-dev
   annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "20M"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
   spec:
   rules:
     - host: ingress.bjrdc17
       http:
         paths:
         - path: /sc-gateway/(.*)
           backend:
             serviceName: spring-cloud-k8s-gateway
             servicePort: 8097
   ```
   
   

6. 验证

   ```shell
   sudo sh -c "echo 172.16.10.17 ingress.bjrdc17 >> /etc/hosts"
   ```

   查看ingress端口

   ```shell
   kubectl get service -n ingress-nginx        
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
   ingress-nginx-controller             LoadBalancer   10.103.204.188   <pending>     80:32628/TCP,443:31687/TCP   3h50m
   ingress-nginx-controller-admission   ClusterIP      10.101.7.151     <none>        443/TCP 
   ```

   访问

   ```shell
   curl ingress.bjrdc17:32628
   ```

7. 配置固定端口

   将ingress的service的type设置为NodePort即可**但是不推荐使用这种方式，因为如此会有大量的nodeport暴露出来，并且nodeport的数量有限**

   ```
   spec:
     type: NodePort
     ports:
       - name: http
         port: 80
         protocol: TCP
         targetPort: http
         nodePort: 30080
   ```

8. **Ingress**的理解

   > ingress其实也是一个service，当然可以配置为loadbanlace或者nodeport两种方式。
   >
   > ingress的意义在于被ingress的service可以设置为clusterip，这样就省去了大连的nodeport，而只需要一个配置给ingress的nodeport。
   >
   > 理论上讲，可以在集群外部搭建一个nginx作为loadbanlce为所有的ingress的nodeip作负载均衡。

9. ingress vs spring-cloud-gateway/zuul

   > **ingress 和spring-cloud-gateway/zuul均可以实现网管的作用。gateway的功能更加强大，ingress如果能够满足需求，那么最好使用ingress（满足云原生架构理念）**

## 网络穿透

### 	in openvpn

如果需要将本机纳入到集群中，可以nsloopup到service，则需要在openvpn的配置`/etc/openvpn/server.conf`中增加dns的push

```
push "route 10.0.0.0 255.0.0.0"
```

如此加入到openvpn网络的机器如果dnsserver配置了10.96.0.10，则可以通过该dnsserver lookup 到service

### 打通虚拟网络

 1. 配置路由

    kubernets 的所有的pod使用的网络为10.0.0.0/8，故不能通过本机与此网络直接打通。打通的办法是，在**vpn的主机**上增加指向10.0.0.0/8的路由

    ```sh
    route add -net 10.0.0.0/8 gw 172.16.15.17
    ```

    172.16.15.17为k8s的master的ip

    或者通过netplan配置，and config to netplan.yaml

    ```yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        ens19:
    ...
          routes:
          - to: 10.0.0.0/8
            via: 172.16.15.17
    ```

    

    但是打通IP由有什么用呢？能够发现service吗？*可以*

### 本地resolv配置

本机dns server 配置使用 systemd-resolved进行，其他的方式都不是最新的了。

vi /etc/systemd/resolved.conf

```
[Resolve]
DNS=10.96.0.10
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#DNSOverTLS=no
#Cache=yes
#DNSStubListener=yes
#ReadEtcHosts=yes
```

```
sudo systemctl daemon-reload
sudo systemctl restart systemd-resolved
```



### 验证



```sh
nslookup es-stateful-2.es-stateful-headless.bjrdc-dev.svc.cluster.local 10.96.0.10
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	es-stateful-2.es-stateful-headless.bjrdc-dev.svc.cluster.local
Address: 10.244.1.101
```

如果无法查找到，则使用如下命令

```
nslookup -norecurse hello-node.bjrdc-dev.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	hello-node.bjrdc-dev.svc.cluster.local
Address: 10.110.0.151

```

```
curl hello-node.bjrdc-dev.svc.cluster.local:3000
```



## DNS

### 基本配置

> This tells dnsmasq that queries for anything in the `cluster.local` domain should be forwarded to the DNS server at 10.96.0.10. This happens to be the default IP address of the `kube-dns` service in the `kube-system` namespace. If your cluster’s DNS service has a different IP address, you’ll need to specify it instead

kubernetes安装的时候，会自动的安装一个kube-dns的服务，该服务用于对service设置域名（因为service的clusterIp是会变化的，在service重启，或者故障的时候），建议使用域名进行service的访问，域名的格式如下

${servicename}.\${namespace}.svc.cluster.local

```sh
ping hello-node.bjrdc-dev.svc.cluster.local
ping mysql.bjrdc-dev.svc.cluster.local
```

dns 其实是配置在/var/lib/kubelet/config.yaml这个文件里的

 ```yaml
 clusterDNS:
 - 10.96.0.10
 clusterDomain: cluster.local
 ```

```
kubectl exec -ti busybox -- nslookup kubernetes.default
```



### IP与网络

service地址和pod地址在不同网段，service地址为虚拟地址，不配在pod上或主机上，外部访问时，先到Node节点网络，再转到service网络，最后代理给pod网络。

#### 服务暴露（expose）

有三种方式暴露服务，NodePort,Loadbanlace,ingress

##### ClusterIP 模式

群内的其它应用都可以访问该服务。集群外部无法访问它.

开启clusterIP后必须使用loadbanlace或者ingress来实现服务的透传

```yaml
apiVersion: v1
kind: Service
metadata:  
	name: my-internal-service
selector:    
	app: my-app
spec:
	type: ClusterIP
```



##### **NodePort**
是引导外部流量到你的服务的最原始方式。NodePort，正如这个名字所示，在所有节点（虚拟机）上开放一个特定端口，任何发送到该端口的流量都被转发到对应服务。

**开启NodePort后，可以通过任何一个NodeIP和nodeport来访问服务**

```yaml
apiVersion: v1
kind: Service
metadata:  
	name: my-nodeport-service
selector:    
	app: my-app
spec:
	type: NodePort
ports:  
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30036
    protocol: TCP
```



##### **Ingress**
通过类似反向代理的方式将集群内service暴露出去，需要咱装ingress-control，本文中安装的是ingress-nginx

详细参见上文



##### **Loadbanlace**
一般是云服务商提供的服务，具体功能尚未明确

TODO

**Node IP**

可以是物理机的IP（也可能是虚拟机IP）。

每个Service都会在Node节点上开通一个端口，外部可以通过NodeIP:NodePort即可访问Service里的Pod,和我们访问服务器部署的项目一样，IP:端口/项目名

```sh
kubectl describe node nodeName
```



**Pod IP**
Pod IP是每个Pod的IP地址，他是Docker Engine根据docker网桥的IP地址段进行分配的，通常是一个虚拟的二层网络

同Service下的pod可以直接根据PodIP相互通信，不同Service下的pod在集群间pod通信要借助于 cluster ip
pod和集群外通信，要借助于node ip

```sh
kubectl get pods
kubectl describe pod podName
```



#### Cluster IP

Service的IP地址，此为虚拟IP地址。外部网络无法ping通，只有kubernetes集群内部访问使用。

在kubernetes查询Cluster IP

```sh
kubectl -n 命名空间 get Service即可看到ClusterIP
```

#### 三种IP网络间的通信

service地址和pod地址在不同网段，service地址为虚拟地址，不配在pod上或主机上，外部访问时，先到Node节点网络，再转到service网络，最后代理给pod网络。



## 集群重启

1. master 重启

   如果因为操作系统更新需要重启，直接重启host,重启前检查系统的ip配置以及网络配置是否正常
   
    ```sh
    sudo reboot
    ```
   
   
	如果重启后k8s未启动通过如下命令查看状态
   
    ```sh
    journalctl -xe kubelet
    ```
   
   确保kubelet开机自启动了
   
   ```
   systemctl enable kubelet
   ```
   
   

2. node 重启

   与master类似

   

## 基本概念与YAML

### image

创建Dockerfile

```dockerfile
cat >Dockerfile <<EOF
FROM node:8.10.0
EXPOSE 8080
COPY server.js .
CMD [ "node", "server.js" ]
EOF
```

构建docker镜像

```sh
sudo docker build -t hello-node:v1 .
```

镜像推送到harbor

```sh
sudo docker tag hello-node:v1 bjrdc206:443/bjrdc-dev/hello-node:v1.0.0
sudo docker push bjrdc206:443/bjrdc-dev/hello-node:v1.0.0
```

### namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
   name: bjrdc-dev
   labels:
     name: bjrdc-dev
```



### deployment

> deployment 创建pod
>
> spec.template.metadata.lables.xxx中描述的就是pod的lable
>
> spec.template.描述的是pod的信息

```yaml
cat >deloyment<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-node
  namespace: bjrdc-dev 
spec:
  selector:
    matchLabels:
      app: hello-node-pod 
  replicas: 2
  template:
    metadata:
      labels:
        app: hello-node-pod #used by service 
    spec:     # pod spec
      containers: 
      - name: hello-node
        image: bjrdc206:443/bjrdc-dev/hello-node:v1.0.0 # image we pushed
        ports:
        - containerPort: 8080 # 容器的服务端口
EOF
```

```
kubectl create -f deployment.yaml
```



### service(svc)

Deployment和Service关联起来只需要Label标签相同就可以关联起来形成负载均衡.

.spec.selector:xxx lable下描述的就是关联的deployment中声明的pod的lable

```yaml
cat >service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: hello-node
  namespace: bjrdc-dev
  labels:
    app: hello-node
spec:
  ports:
  - port: 3000  #对外暴露的端口
    targetPort: 8080
    protocol: TCP
  selector:
    app: hello-node-pod
    
EOF
```

```sh
kubectl create -f service.yaml
```

```shell
kubectl get services -n bjrdc-dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
hello-node   ClusterIP   10.102.118.239   <none>        3000/TCP   39m

curl 10.102.118.239:3000
```



#### targetPort port 

**注意：**  `Service` 能够将一个接收 `port` 映射到任意的 `targetPort`。 默认情况下，`targetPort` 将被设置为与 `port` 字段相同的值。

targetPort:pod 的服务端口

port：service将pod的端口映射为集群的端口。

```sh
kubectl get services --all-namespaces
NAMESPACE              NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
bjrdc-dev              hello-node                  ClusterIP   10.102.118.239   <none>        3000/TCP                 12m

curl 10.102.118.239:3000
```

### 负载均衡

> 经实验通过headless service和普通的service均可以实现负载均衡的效果。

#### 通过service的负载均衡

他哦难过deployment部署了1个pod，通过scale伸缩为2个pod后，状态如下

```
kubectl get pod -n bjrdc-dev|grep spring-cloud-
spring-cloud-k8s-configmap-7f466bcdf5-prjfp   1/1     Running   0          82m
spring-cloud-k8s-consumer-7989fcf49c-h46rw    2/2     Running   0          29m
spring-cloud-k8s-consumer-7989fcf49c-w6pbm    2/2     Running   0          11m
spring-cloud-k8s-gateway-775777c6c7-ptnzn     1/1     Running   0          87m
spring-cloud-k8s-provider-79f4585944-2npmk    1/1     Running   0          33m
```

通过deployment的service访问pod，发现请求会随机发送到2个pod上来。说明service+deployment是具有负载均衡效果的。

```
kubectl logs --tail=20 spring-cloud-k8s-consumer-7989fcf49c-h46rw -c spring-cloud-k8s-consumer-server -n bjrdc-dev -f
```

```
curl spring-cloud-k8s-consumer.bjrdc-dev.svc.cluster.local:8096/sc-k8s-consumer/feign/list
```

通过jmeter测试发现，在一个pod的时候，10000个请求下来，throughtpt=600/sec，scale到2后，可以达到800/sec。

注：**service负载均衡下客户端只知道一个serviceip的**

#### 通过Headless的负载均衡

通过statefuset+headless的方式也可以实现负载均衡，和service的方式不同的是 **headless返回给客户端的是多个ip，由客户端自己决定使用那个ip**

### headless

Headless 也是一种Service，但不同的是会定义`spec:clusterIP: None`，也就是不需要`Cluster IP`的`Service`

 有什么用途呢？

 - 可以通过pod.headless...获取到pod。

 - 在statefulset模式下，多个pod之间要相互访问，需要使用`pod.headless...`

 - 可以通过headless获取到服务的ip列表

   ```
   nslookup kzookeeper-stateful-headless.bjrdc-dev.svc.cluster.local 10.96.0.10
   Server:		10.96.0.10
   Address:	10.96.0.10#53
   
   Name:	kzookeeper-stateful-headless.bjrdc-dev.svc.cluster.local
   Address: 10.244.5.32
   Name:	kzookeeper-stateful-headless.bjrdc-dev.svc.cluster.local
   Address: 10.244.5.24
   Name:	kzookeeper-stateful-headless.bjrdc-dev.svc.cluster.local
   Address: 10.244.5.26
   ```

   

### Configmap

>configmap 是k8s的配置服务，一个简单的配置如下

```
kind: ConfigMap
apiVersion: v1
metadata:
name: spring-cloud-k8s-configmap
namespace: bjrdc-dev
data:
application.yaml: |-
cn.xportal.cs.config.base: base 
---
spring:
profiles: k8s
cn.xportal.cs.config.base: k8s 
---
spring:
profiles: local
cn.xportal.cs.config.base: local 
```



>"application.yaml: |-"可以理解为一个文件段，当然也可以引用外部的文件。
>
>在spring-cloud中使用这个configmap需要

### pod

pod 是container的更高抽象

1. 独立声明pod
    ```yaml
    cat >0-busybox-pod.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
      namespace: bjrdc-dev
      labels: 
       app: busybox 
    spec:
          containers: 
          - name: busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - sleep 10; touch /tmp/healthy; sleep 30000
            readinessProbe:           
              exec:
                command:
                - cat
                - /tmp/healthy
              initialDelaySeconds: 10      
              periodSeconds: 5
    EOF          
    ```

    

 2. 重启pod

    ```sh
    kubectl get pod mysql-on-ceph-01-yyy -o yaml -n bjrdc-dev|kubectl replace --force -f -
    ```

 3. 迁移pod

    先将node设置为不可调度

    ```sh
    kubectl cordon bjrdc81
    ```

    重启pod

    ```shell
    kubectl get pod mysql-on-ceph-01-xxx -o yaml -n bjrdc-dev|kubectl replace --force -f -
    ```

    恢复node

    ```shell
    kubectl uncordon bjrdc81
    ```

 4. 驱逐所有pod

    TODO


### pv and pvc

> pv 是对卷的声明，pvc是对声明的卷的使用，相当与从中再切割一部分出来。
>
> pv 和pvc是一一对应的，如果一个pv要对应多个pvc那是不可以的只能用*storageclass*
>

#### 共享pvc

> 采用depolyment挂载pvc，当deployment中包含多个pod的时候，这些pod将共享pvc

1. 创建pvc

   ```yaml
   cat >0-nginx-pvc.yaml <<EOF
   ---
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: csi-rbd-sc-nginx-pvc
     namespace: bjrdc-dev
   spec:
     accessModes:
       - ReadWriteMany
     volumeMode: Block
     resources:
       requests:
         storage: 200M
     storageClassName: csi-rbd-sc
   ```

   volumeMode为Block，不是Filesystem,Filesystem无法被多个pod共享

   accessModes为ReadWriteMany，可以被多个pod共享

2. 创建deployment

   ```yaml
   cat >1-nginx-storageclass-deployment.yaml <<EOF
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-storageclass
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: nginx 
     replicas: 3
     template:
       metadata:
         labels:
           app: nginx 
       spec:
         containers:
         - name: nginx
           image: nginx:1.19.0
           ports:
           - containerPort: 80
           volumeMounts:
           - name: logs
             mountPath: /var/log/nginx
           volumeDevices:
           - name: www
             devicePath: /dev/xvda
         volumes:
         - name: logs
           emptyDir: {}
         - name: www
           persistentVolumeClaim: 
             claimName: csi-rbd-sc-nginx-pvc
   EOF          
   ```

   注：**暂时未找到如何使用volumeDevices的方法**
   
   rdb的模式下，看材料似乎不支持一个block共享挂载到多个pod上。

#### statefulset pvc

> 采用pv的方式不需要做格外的配置，但是pv的弊端是一个pv只能挂在一个pvc，无法在statefulset模式下使用。安装方式如下：

1. 创建cecret,key密码为base64后的值

   ```yaml
   cat >1-ceph-secret.yaml <<EOF
   apiVersion: v1
   kind: Secret
   metadata:
     name: ceph-normal-secret
     namespace: bjrdc-dev
   data:
     key: QVFCYUZCUmZPVndHQkJBQWJKQ2ZENTZrbGpMaHJ1aURmSW1odlE9PQ==
   EOF
   ```

   其中key通过如下命令获取

   ```sh
   ceph auth get-key client.admin | base64
   ```

2. 创建pv

   ```yaml
   cat >2-pv.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: ceph-normal-pv
     namespace: bjrdc-dev
     labels:
       pv: ceph-normal-pv
   spec:
     capacity:
       storage: 2Gi
     accessModes:
       - ReadWriteOnce 
     rbd:
       monitors:
         - 172.16.15.208:6789
       pool: k8s_pool_01
       image: k8s-v1
       user: admin
       secretRef:
         name: ceph-normal-secret
       fsType: ext4
       readOnly: false
     persistentVolumeReclaimPolicy: Recycle
   EOF
   ```

3. 创建vpc

   ```yaml
   cat >3-pvc.yaml <<EOF
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: ceph-normal-pvc
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         pv: ceph-normal-pv
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 2Gi
   EOF
   ```

4. 创建deployment

   ```yaml
   cat >4-deploy.yaml <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: ceph-normal-demo
     namespace: bjrdc-dev
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: ceph-normal-demo
     template:
       metadata:
         labels:
           app: ceph-normal-demo
       spec:
         containers:
         - name: ceph-normal-demo
           image: bjrdc206.reg/bjrdc-dev/hello-node:v1.0.1
           ports:
           - containerPort: 8080
           volumeMounts:
             - mountPath: "/data"
               name: ceph-normal-pvc-name
         volumes:
           - name: ceph-normal-pvc-name
             persistentVolumeClaim:
               claimName: ceph-normal-pvc
   EOF
   ```

5. 执行

   ```sh
   kubectl apply -f .
   ```



### Statefulset

> [官方文档](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
>
> deployment 部署的pod都是无状态的，所谓的无状态是指其下配置的pod之间是没有连带关系的，没有状态的依存关系。
>
> 如果一个app需要部署的时候各个pod之间是有先后顺序，并且id是全局不变的，则需要使用到statefulset，如*mysql-cluster*,*es*,*redis-cluster*,*kafka*等
>
> **volumeClaimTemplates**: 表示一类PVC的模板，系统会根据Statefulset配置的replicas数量，创建相应数量的PVC。这些PVC除了名字不一样之外其他配置都是一样的

**要使用statefulset,先要让storageclass可用。直接用pv和pvc会出问题，第一个pod可以创建出来，第二个pod就创建不出来了。因为第一个pod占用唯一的一个pv。**

**在statefulset.spec.serviceName需要配置为对外访问的servicename，否则无法通过dns访问到pod**

> 在已经配置好了ceph 的storageclass后，使用如下方式配置一个简单的statefulset

1. 创建secret

   
   
2. 创建storageclass

   ```yaml
   cat >0-ceph-stateful-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-stateful
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

3. 创建statefulset

   ```yaml
   cat >1-statefulset.yaml <<EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-stateful
     namespace: bjrdc-dev
     labels:
       app: nginx-stateful
   spec:
     ports:
     - port: 80
       name: web
     clusterIP: None
     selector:
       app: nginx-stateful
   ---
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: web-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: nginx-stateful # has to match .spec.template.metadata.labels
     serviceName: nginx-stateful
     replicas: 3 # by default is 1
     template:
       metadata:
         labels:
           app: nginx-stateful # has to match .spec.selector.matchLabels
       spec:
         terminationGracePeriodSeconds: 10
         containers:
         - name: nginx-stateful
           image: nginx:1.19.0
           ports:
           - containerPort: 80
             name: web
           volumeMounts:
           - name: www
             mountPath: /usr/share/nginx/html
     volumeClaimTemplates:
     - metadata:
         name: www
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-stateful
         resources:
           requests:
             storage: 500Mi
   EOF          
   ```

4. 查看pod，此时应该创建了多个pod

   ```shell
   kubectl get pod -n bjrdc-dev
   web-stateful-0                      1/1     Running   0          13h
   web-stateful-1                      1/1     Running   0          13h
   web-stateful-2                      1/1     Running   0          13h
   ```

5. 验证

   部署完成后，可以使用service和pod的域名进行访问，访问的模式为

   service：#{service_name}.svc.cluster.local

   pod：#{pod_name}.#{service_name}.svc.cluster.local

   在每个pod上创建index.html，使用如下命令，命令中需要将pod的name修改为对应的。

   ```sh
   kubectl exec -it web-stateful-2 -n bjrdc-dev -- /bin/bash -c "echo web-stateful-2 > /usr/share/nginx/html/index.html"
   ```

   通过service访问，发现有负载均衡的作用**这不就是mysql的读写分离需要的吗？**

   ```sh
   for i in {0..5}; do curl nginx-stateful.bjrdc-dev.svc.cluster.local; done
   web-stateful-2
   web-stateful-1
   web-stateful-0
   web-stateful-2
   web-stateful-2
   web-stateful-2
   ```

   通过pod访问

   ```shell
   for i in {0..5}; do curl web-stateful-0.nginx-stateful.bjrdc-dev.svc.cluster.local; done
   web-stateful-0
   web-stateful-0
   web-stateful-0
   web-stateful-0
   web-stateful-0
   web-stateful-0
   ```

   

下一步将使用同样的原理进行mysql集群的创建。



### health check

> kubernetes 默认的健康检查机制为：每个容器启动时都会执行一个进程，此进程由 Dockerfile 的 CMD 或 ENTRYPOINT 指定。如果进程退出时返回码非零，则认为容器发生故障，Kubernetes 就会根据 `restartPolicy` 重启容器

1. **LivenessProbe**

   容器是否正常执行

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: liveness-exec
   spec:
     containers:
     - name: liveness
       image: tomcagcr.io/google_containers/busybox
       args:
       - /bin/sh
       - -c
       - echo ok > /tmp/health;sleep 10;rm -fr /tmp/health;sleep 600
       livenessProbe:
         exec:
           command:
           - cat
           - /tmp/health
         initialDelaySeconds: 15
         timeoutSeconds: 1
         periodSeconds: 5
   ```

   > `periodSeconds` 规定kubelet要每隔5秒执行一次liveness probe。 `initialDelaySeconds` 告诉kubelet在第一次执行probe之前要的等待5秒钟。探针检测命令是在容器中执行 `cat /tmp/healthy` 命令。如果命令执行成功，将返回0，kubelet就会认为该容器是活着的并且很健康。如果返回非0值，kubelet就会杀掉这个容器并重启它

   http 检测

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     labels:
       test: liveness
     name: liveness-http
   spec:
     containers:
     - name: liveness
       args:
       - /server
       image: gcr.io/google_containers/liveness
       livenessProbe:
         httpGet:
           path: /healthz
           port: 8080
           httpHeaders:
             - name: X-Custom-Header
               value: Awesome
         initialDelaySeconds: 3
         periodSeconds: 3
   ```

   > 任何大于200小于400的返回码都会认定是成功的返回码。其他返回码都会被认为是失败的返回码

   端口检查

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: pod-with-healthcheck
   spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containnerPort: 80
       livenessProbe:
         tcpSocket:
           port: 80
         initialDelaySeconds: 15
         timeoutSeconds: 1
   ```



2. **readinessProbe**

   容器是否可以接受请求。

   有时，应用程序暂时无法对外部流量提供服务。 例如，应用程序可能需要在启动期间加载大量数据或配置文件。 在这种情况下，你不想杀死应用程序，但你也不想发送请求。 Kubernetes提供了readiness probe来检测和减轻这些情况。 Pod中的容器可以报告自己还没有准备，不能处理Kubernetes服务发送过来的流量

   ```yaml
   readinessProbe:
     exec:
       command:
       - cat
       - /tmp/healthy
     initialDelaySeconds: 5
     periodSeconds: 5
   ```

   Readiness probe的HTTP和TCP的探测器配置跟liveness probe一样。

### StorageClass

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent. This concept is sometimes called "profiles" in other storage systems.

而动态供给的关键就是StorageClass，它的作用就是创建PV模板。

*参考 ceph*


### RBAC

> k8s提供的基于角色的权限管理。

```
kubectl auth  can-i get pods --as system:serviceaccount:gitlab-runner:gitlab-ci
kubectl auth  can-i get pods -n bjrdc-dev --as system:serviceaccount:gitlab-runner:default
```



#### **ServiceAccount**

>每个 namespace 中都有一个默认的叫做 `default` 的 service account 资源.
>
>当您（真人用户）访问集群（例如使用`kubectl`命令）时，apiserver 会将您认证为一个特定的 User Account（目前通常是`admin`，除非您的系统管理员自定义了集群配置）
>
>Pod 容器中的进程也可以与 apiserver 联系。 当它们在联系 apiserver 的时候，它们会被认证为一个特定的 Service Account（例如`default`）。
>
>运行在pod里的进程需要调用Kubernetes API以及非Kubernetes API的其它服务。Service Account它并不是给kubernetes集群的用户使用的，而是给pod里面的进程使用的，它为pod提供必要的身份认证
>
>往往出现权限的问题就是这个default捣的鬼，建议自建serviceaccount，并绑定service。

 serviceaccount

 创建一个serviceaccount

 ```yaml
 apiVersion: v1
 kind: ServiceAccount
 metadata:
 name: admin-user
 namespace: kube-system
 ```

 查看serviceaccount

 ```sh
 kubectl get serviceaccounts -n bjrdc-dev
 ```

#### Role

一个 `Role` 只可以用来对某一命名空间中的资源赋予访问权限

定义到名称为 "default" 的命名空间，可以用来授予对该命名空间中的 Pods 的读取权限：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" 指定核心 API 组
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```



#### ClusterRole

定义到集群中的角色。`ClusterRole` 可以授予的权限和 `Role` 相同， 但是因为 `ClusterRole` 属于集群范围，所以它也可以授予以下访问权限：

- 集群范围资源 （比如 nodes）
- 非资源端点（比如 "/healthz"）
- 跨命名空间访问的有名字空间作用域的资源（如 Pods），比如运行命令`kubectl get pods --all-namespaces` 时需要此能力

下面的 `ClusterRole` 示例可用来对某特定命名空间下的 Secrets 的读取操作授权， 或者跨所有命名空间执行授权（取决于它是如何[绑定](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)的）：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # 此处的 "namespace" 被省略掉是因为 ClusterRoles 是没有命名空间的。
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```



#### RoleBinding 

一个 `RoleBinding` 可以引用同一的命名空间中的 `Role`,下面的例子 `RoleBinding` 将 "pod-reader" 角色授予在 "default" 命名空间中的用户 "jane"； 这样，用户 "jane" 就具有了读取 "default" 命名空间中 pods 的权限。

`roleRef` 里的内容决定了实际创建绑定的方法。`kind` 可以是 `Role` 或 `ClusterRole`， `name` 将引用你要指定的 `Role` 或 `ClusterRole` 的名称。在下面的例子中，角色绑定使用 `roleRef` 将用户 "jane" 绑定到前文创建的角色 `Role`，其名称是 `pod-reader`。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定使得用户 "jane" 能够读取 "default" 命名空间中的 Pods
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # 这里的名称必须与你想要绑定的 Role 或 ClusterRole 名称一致
  apiGroup: rbac.authorization.k8s.io
```

#### ClusterRoleBinding



### apiversion

Deployment
 1.6版本之前 apiVsersion：extensions/v1beta1

 1.6版本到1.9版本之间：apps/v1beta1

 1.9版本之后:apps/v1



1. v1 

    Kubernetes API的稳定版本，包含很多核心对象：pod、service等

 2. app/v1 

    在kubernetes1.9版本中，引入apps/v1，deployment等资源从extensions/v1beta1, apps/v1beta1 和 apps/v1beta2迁入apps/v1，原来的v1beta1等被废弃。

> apps/v1代表：包含一些通用的应用层的api组合，如：Deployments, RollingUpdates, and ReplicaSets

### Label

> *Labels* are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.
>
> labels do not provide uniqueness. In general, we expect many objects to carry the same label(s)

### selector

>service选择pod的时候，需要在service的spec.selector:xxx中描述pod的lable



### Container

> 一般情况下一个pod只需要一个container，但是有一些情况需要多个container共同完成一个工作。这种情况，这些container是可以共享configmap，pvc的
>
> 如下案例，创建一个nginx服务，通过一个busybox实现对nginx中的index.html的修改。

1. 创建ceph-pv

   ```yaml
   cat >0-pod-shard-pv.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: ceph-pod-shard-pv
     namespace: bjrdc-dev
     labels:
       pv: ceph-pod-shard-pv
   spec:
     capacity:
       storage: 601Mi
     accessModes:
       - ReadWriteOnce 
     rbd:
       monitors:
         - bjrdc208:6789
       pool: rdb_pool_01
       image: k8s-pod-shard-v1
       user: admin
       secretRef:
         name: ceph-normal-secret
       fsType: ext4
       readOnly: false
     persistentVolumeReclaimPolicy: Recycle
   EOF  
   ```

2. 创建pvc

   ```yaml
   cat >1-pod-shard-pvc.yaml <<EOF
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: ceph-pod-shard-pvc
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         pv: ceph-pod-shard-pv
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 601Mi
   EOF      
   ```

3. 创建depoyment，其中一个pod中包含多个container，并在busybox的container中修改了index.html

   ```yaml
   cat >2-pod-shard-depoyment.yaml<<EOF 
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: pod-shard
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: pod-shard
     strategy:
       type: Recreate
     template:
       metadata:
         labels:
           app: pod-shard
       spec:
         containers:
         - name: pod-shard-nginx
           image: nginx:1.19.0
           ports:
           - containerPort: 80
             name: web
           volumeMounts:
           - name: pod-shard-www
             mountPath: /usr/share/nginx/html 
         - name: busybox
           image: busybox
           args:
           - /bin/sh
           - -c
           - sleep 10; touch /tmp/healthy; echo `hostname` >/html/index.html;sleep 30000
           readinessProbe:
             exec:
               command:
               - cat
               - /tmp/healthy
             initialDelaySeconds: 10
             periodSeconds: 5
           volumeMounts:
           - name: pod-shard-www
             mountPath: /html
         volumes:
         - name: pod-shard-www
           persistentVolumeClaim: 
             claimName: ceph-pod-shard-pvc
   EOF          
   ```

4. 创建service，用于测试访问

   ```yaml
   cat >3-pod-shard-service.yaml <<EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: pod-shard-service
     namespace: bjrdc-dev
     labels:
       app: pod-shard-service
   spec:
     selector:
         app: pod-shard
     ports:
     - protocol : TCP
       port: 80
       targetPort: 80
   EOF    
   ```

5. 测试

   查看pod的状态

   ```sh
   kubectl get pod -n bjrdc-dev|grep pod-shard
   pod-shard-5c7b7f6bd6-2dhdj          2/2     Running   0          18m
   ```

   测试pod的服务，发现能够正常的返回pod的hostname`pod-shard-5c7b7f6bd6-2dhdj`

   ```sh
   curl pod-shard-service.bjrdc-dev.svc.cluster.local
   pod-shard-5c7b7f6bd6-2dhdj
   ```


### env

> 在container的配置中可以设置环境变量，如下

```yaml
  template:
    metadata:
      labels:
        app: es-stateful
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: es-stateful
        image: elasticsearch:6.8.11
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
```

这个POD_NAME可以在其他的yaml文件中通过${POD_NAME}获取

## restful api

> kubernetes 通过restfulapi可以方便的访问集群资源，有如下两种方式

### proxy

1. 启动proxy

   ```
   kubectl proxy --port=8080
   ```

2. 通过curl访问

   ```
   curl http://localhost:8080/api/
   curl http://localhost:8080/api/v1/namespaces/bjrdc-dev/pods
   ```

### no proxy

```sh
# 查看所有的集群，因为你的 .kubeconfig 文件中可能包含多个上下文
kubectl config view -o jsonpath='{"Cluster name\tServer\n"}{range .clusters[*]}{.name}{"\t"}{.cluster.server}{"\n"}{end}'

# 从上述命令输出中选择你要与之交互的集群的名称
export CLUSTER_NAME="some_server_name"

# 指向引用该集群名称的 API 服务器
APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")

# 获得令牌
TOKEN=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 -d)

# 使用令牌玩转 API
curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

### python

>Python 客户端可以使用与 kubectl 命令行工具相同的 [kubeconfig 文件](https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/) 定位和验证 API 服务器。参见这个 [例子](https://github.com/kubernetes-client/python/blob/master/examples/out_of_cluster_config.py)：

```
pip install kubernetes
```



```python
from kubernetes import client, config

config.load_kube_config()

v1=client.CoreV1Api()
print("Listing pods with their IPs:")
ret = v1.list_pod_for_all_namespaces(watch=False)
for i in ret.items:
    print("%s\t%s\t%s" % (i.status.pod_ip, i.metadata.namespace, i.metadata.name))
```



### java

>参阅https://github.com/kubernetes-client/java/releases 了解当前支持的版本。
>
>Java 客户端可以使用 kubectl 命令行所使用的 [kubeconfig 文件](https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/) 以定位 API 服务器并向其认证身份。 参看此[示例](https://github.com/kubernetes-client/java/blob/master/examples/src/main/java/io/kubernetes/client/examples/KubeConfigFileClientExample.java)：

```java
package io.kubernetes.client.examples;

import io.kubernetes.client.ApiClient;
import io.kubernetes.client.ApiException;
import io.kubernetes.client.Configuration;
import io.kubernetes.client.apis.CoreV1Api;
import io.kubernetes.client.models.V1Pod;
import io.kubernetes.client.models.V1PodList;
import io.kubernetes.client.util.ClientBuilder;
import io.kubernetes.client.util.KubeConfig;
import java.io.FileReader;
import java.io.IOException;

/**
 * A simple example of how to use the Java API from an application outside a kubernetes cluster
 *
 * <p>Easiest way to run this: mvn exec:java
 * -Dexec.mainClass="io.kubernetes.client.examples.KubeConfigFileClientExample"
 *
 */
public class KubeConfigFileClientExample {
  public static void main(String[] args) throws IOException, ApiException {

    // file path to your KubeConfig
    String kubeConfigPath = "~/.kube/config";

    // loading the out-of-cluster config, a kubeconfig from file-system
    ApiClient client =
        ClientBuilder.kubeconfig(KubeConfig.loadKubeConfig(new FileReader(kubeConfigPath))).build();

    // set the global default api-client to the in-cluster one from above
    Configuration.setDefaultApiClient(client);

    // the CoreV1Api loads default api-client from global configuration.
    CoreV1Api api = new CoreV1Api();

    // invokes the CoreV1Api client
    V1PodList list = api.listPodForAllNamespaces(null, null, null, null, null, null, null, null, null);
    System.out.println("Listing all pods: ");
    for (V1Pod item : list.getItems()) {
      System.out.println(item.getMetadata().getName());
    }
  }
}
```



## NFS



## ceph



### rbd-provisoner

**可悲的是在k8s 1.10版本的某个版本以（当前文档对应版本1.18.0），将默认的rbd支持去掉了——文档竟然没有更新**

解决办法是安装rbd-provisoner插件，安装方法如下

1. kubernetes官方有安装的地址和方法**[kubenetes官方的部署yaml](https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd/deploy/rbac)**

   ```
   .
   ├── clusterrolebinding.yaml
   ├── clusterrole.yaml
   ├── deployment.yaml
   ├── rolebinding.yaml
   ├── role.yaml
   └── serviceaccount.yaml
   ```

   

2. 首先下载这个目录的文件，在执行之前需要配置namespace和修改权限，具体方法如下

3. 修改namespace，按照官方的的方式修改namespace

   ```sh
   sed -r -i "s/namespace: [^ ]+/namespace: kube-system/g" ./rbac/clusterrolebinding.yaml ./rbac/rolebinding.yaml
   ```

4. 这种方式完了后，发现会漏，继续修改

   在serviceaccount.yaml和deployment.yaml中增加

   ```yaml
     namespace: kube-system
   ```

5. 最后还需要让`rbd-provisioner`具有secret权限

   ```yaml
   vi clusterrole.yaml
     - apiGroups: [""]
       resources: ["secrets"]
       verbs: ["get", "create", "delete"]
   ```

6. 执行所有的yaml应该就可以将rbd-provisioner安装好了

   ```sh
   kubectl apply -f .
   ```

7. 下一步可以使用storageclass来部署statefulset

8. 问题处理

   1. failed to provision volume with StorageClass "ceph-storageclass": failed to get admin secret from ["bjrdc-dev"/"ceph-rbd-secret"]: secrets "ceph-rbd-secret" is forbidden: User "system:serviceaccount:kube-system:rbd-provisioner" cannot get resource "secrets" in API group "" in the namespace "bjrdc-dev"

      ```yaml
      vi clusterrole.yaml
        - apiGroups: [""]
          resources: ["secrets"]
          verbs: ["get", "create", "delete"]
      ```

   2. auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring,/etc/ceph/ceph.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin,: (2) No such file or directory

      **secrte配置错了**

   3. 在测试过程中发现官方的sroageclass的方式有问题，会报`Error creating rbd image: executable file not found in $PATH #38923`的问题 详细见如下issue，需要安装rdb-provisoner

      [https://github.com/kubernetes/kubernetes/issues/38923#issuecomment-315255075 ](https://github.com/kubernetes/kubernetes/issues/38923#issuecomment-315255075 )

      记的在主机上安装ceph-common

      ```
      sudo apt install ceph-common
      ```

      

9. *那么还有一个问题，可以直接从storageclass 声明pvc吗？*可以的

   ```yaml
   cat 2-prometheus-pvc.yaml 
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: prometheus-pvc
     namespace: pro-mon
   spec:
     storageClassName: ceph-storageclass-prometheus 
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 3Gi
   ```

   可直接在deployment中使用

   ```yaml
   cat 4-prometheus-deployment.yaml 
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: prometheus
     namespace: pro-mon 
     labels:
       app: prometheus
   spec:
   ...
       spec:
         containers:
         - image: bjrdc206.reg/library/prometheus:v2.20.1
           name: prometheus
           imagePullPolicy: IfNotPresent
         ...
         - name: data
           persistentVolumeClaim:
             claimName: prometheus-pvc
   ```

   


​      

#### ceph-common

> RBD 模式下可以使用storageclass 和普通的pvc两种模式

记得要先在主机上安装ceph-common

```
sudo apt install ceph-common 
```

#### pool

```
sudo ceph osd pool create k8s_pool_01 128 128
sudo rbd pool init k8s_pool_01
sudo ceph osd pool set-quota k8s_pool_01 max_bytes $((10 * 1024 * 1024 * 1024))
```



#### Storageclass

> 在进行storageclass模式之前需要确保`rbd-provisoner`安装成功。

在ceph master上找到 admin 账号的key，的base64

```
grep key /etc/ceph/ceph.client.admin.keyring |awk '{printf "%s", $NF}'|base64
QVFCR3N1VmdsVkc1RFJBQUJoL2t0SUgrQ1prQ01NUFIzVnZ1T3c9PQ==
```

或者

```
sudo ceph auth get-key client.admin | base64
```



1. 创建cecret，key密码为base64后的值

   ```yaml
   cat >1-ceph-secret.yaml <<EOF
   apiVersion: v1
   kind: Secret
   metadata:
     name: ceph-rbd-secret
     namespace: bjrdc-dev
   data:
     key: QVFCR3N1VmdsVkc1RFJBQUJoL2t0SUgrQ1prQ01NUFIzVnZ1T3c9PQ==
   EOF  
   ```

2. 创建storageclass实例

   adminSecretName，userSecretName:为secret的name

   ```yaml
   cat >2-ceph-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

3. 创建pvc

   执行成功后应该可以通过`kubectl get pvc -n bjrdc-dev`查看到

   ```yaml
   cat >3-ceph-pvc.yaml <<EOF
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: ceph-storageclass-pvc
     namespace: bjrdc-dev
   spec:
     storageClassName: ceph-storageclass
     accessModes:
       - ReadWriteOnce 
     resources:
       requests:
         storage: 800Mi
   EOF      
   ```

4. 挂载到pod

   ```yaml
   cat >4-ceph-deploy.yaml <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: ceph-rbd-demo
     namespace: bjrdc-dev
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: ceph-rbd-demo
     template:
       metadata:
         labels:
           app: ceph-rbd-demo
       spec:
         containers:
         - name: ceph-rbd-demo
           image: bjrdc206.reg/bjrdc-dev/hello-node:v1.0.1
           ports:
           - containerPort: 80
           volumeMounts:
             - mountPath: "/data"
               name: k8s-v1
         volumes:
           - name: k8s-v1
             persistentVolumeClaim:
               claimName: ceph-storageclass-pvc
   EOF            
   ```

5. 此时应该可以正常的访问到pod，并且pod上挂载了ceph的目录。同时在ceph上可以看到一个自动创建的image

   ```sh
   sudo rbd list -p k8s_pool_01
   
   k8s-mysql-cluster-v1
   k8s-mysql-v1
   k8s-statefulset-v1
   k8s-v1
   kubernetes-dynamic-pvc-7be164a4-d315-11ea-9a3c-4e8cdd04a447
   ```
   
6. 如果出问题了，可以查看provider的日志

   ```
    kubectl logs rbd-provisioner-76f6bc6669-qgz7n -n kube-system
   ```

7. 

#### 问题处理

##### unexpected error getting claim reference: selfLink was empty, can't make reference

vi /etc/kubernetes/manifests/kube-apiserver.yaml

```
spec:
  containers:
  - command:
    - kube-apiserver
```

add 

```
- --feature-gates=RemoveSelfLink=false
```

```
sudo systemctl restart kubelet
```



##### cannot open /etc/ceph/ceph.conf

到 rbd-provider中挂载/etc/ceph/ceph.conf文件即可。



##### monclient: get_monmap_and_config failed to get config

这个问题暂时无法处理，不知道原因。



#### 如何防止数据卷被删除呢？

TODO



### ceph-csi

ceph-csi 是ceph官方提供的按照kubernetes官方的csi（container storage interface）开发的一套支持ceph的插件。

#### 安装

按照官方文档进行安装即可 [官方文档](https://docs.ceph.com/en/latest/rbd/rbd-kubernetes/#create-a-pool)。但是官方文件中有如下几个坑

1. 镜像官方提供的是k8s.gcr.io下的，需要替换为quey.io的。
2. ceph-csi不需要对ceph的key进行加密。
3. 官方默认的yaml的namespace是default的，有的是空的，需要替换成有效的。
4. 不需要安装`ceph-common`

相关的配置文件如下

1. 创建configmap，clusterID为ceph的clusterid，从/etc/ceph/ceph.conf文件查看

   ```yaml
   cat >0-csi-config-map.yaml <<EOF
   ---
   apiVersion: v1
   kind: ConfigMap
   data:
     config.json: |-
       [
         {
           "clusterID": "5021e379-a950-49fa-bf5e-378354eba003",
           "monitors": [
             "172.16.15.251:6789"
           ]
         }
       ]
   metadata:
     name: ceph-csi-config
     namespace: bjrdc-dev
   EOF  
   ```

2. 创建kms-configmap，如果删除下面的yaml中对于ceph-csi-encryption-kms-config的引用，可以不创建该configma

   ```yaml
   cat >1-csi-kms-config-map.yaml <<EOF
   ---
   apiVersion: v1
   kind: ConfigMap
   data:
     config.json: |-
       {}
   metadata:
     name: ceph-csi-encryption-kms-config
     namespace: bjrdc-dev
   EOF  
   ```

3. 创建csi rbd-csi-provisioner 的RBAC

   ```yaml
   cat >3-csi-provisioner-rbac.yaml <<EOF
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: rbd-csi-provisioner
   
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: rbd-external-provisioner-runner
   rules:
     - apiGroups: [""]
       resources: ["nodes"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["secrets"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["events"]
       verbs: ["list", "watch", "create", "update", "patch"]
     - apiGroups: [""]
       resources: ["persistentvolumes"]
       verbs: ["get", "list", "watch", "create", "update", "delete", "patch"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims"]
       verbs: ["get", "list", "watch", "update"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims/status"]
       verbs: ["update", "patch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["storageclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshots"]
       verbs: ["get", "list"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotcontents"]
       verbs: ["create", "get", "list", "watch", "update", "delete"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["volumeattachments"]
       verbs: ["get", "list", "watch", "update", "patch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["volumeattachments/status"]
       verbs: ["patch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["csinodes"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotcontents/status"]
       verbs: ["update"]
     - apiGroups: [""]
       resources: ["configmaps"]
       verbs: ["get"]
     - apiGroups: [""]
       resources: ["serviceaccounts"]
       verbs: ["get"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: rbd-csi-provisioner-role
   subjects:
     - kind: ServiceAccount
       name: rbd-csi-provisioner
       namespace: bjrdc-dev
   roleRef:
     kind: ClusterRole
     name: rbd-external-provisioner-runner
     apiGroup: rbac.authorization.k8s.io
   
   ---
   kind: Role
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     # replace with non-default namespace name
     namespace: bjrdc-dev
     name: rbd-external-provisioner-cfg
   rules:
     - apiGroups: [""]
       resources: ["configmaps"]
       verbs: ["get", "list", "watch", "create", "update", "delete"]
     - apiGroups: ["coordination.k8s.io"]
       resources: ["leases"]
       verbs: ["get", "watch", "list", "delete", "update", "create"]
   
   ---
   kind: RoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: rbd-csi-provisioner-role-cfg
     # replace with non-default namespace name
     namespace: bjrdc-dev
   subjects:
     - kind: ServiceAccount
       name: rbd-csi-provisioner
       # replace with non-default namespace name
       namespace: bjrdc-dev
   roleRef:
     kind: Role
     name: rbd-external-provisioner-cfg
     apiGroup: rbac.authorization.k8s.io
   EOF  
   ```

5. 创建nodeplugin的rbac

   ```yaml
   cat >4-csi-nodeplugin-rbac.yaml <<EOF
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: rbd-csi-nodeplugin
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: rbd-csi-nodeplugin
   rules:
     - apiGroups: [""]
       resources: ["nodes"]
       verbs: ["get"]
     # allow to read Vault Token and connection options from the Tenants namespace
     - apiGroups: [""]
       resources: ["secrets"]
       verbs: ["get"]
     - apiGroups: [""]
       resources: ["configmaps"]
       verbs: ["get"]
     - apiGroups: [""]
       resources: ["serviceaccounts"]
       verbs: ["get"]
     - apiGroups: [""]
       resources: ["persistentvolumes"]
       verbs: ["get"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["volumeattachments"]
       verbs: ["list", "get"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: rbd-csi-nodeplugin
   subjects:
     - kind: ServiceAccount
       name: rbd-csi-nodeplugin
       namespace: bjrdc-dev
   roleRef:
     kind: ClusterRole
     name: rbd-csi-nodeplugin
     apiGroup: rbac.authorization.k8s.io
   EOF  
   ```

6. 创建provisiner

   ```yaml
   cat >5-csi-rbdplugin-provisioner.yaml <<EOF
   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: csi-rbdplugin-provisioner
     labels:
       app: csi-metrics
   spec:
     selector:
       app: csi-rbdplugin-provisioner
     ports:
       - name: http-metrics
         port: 8080
         protocol: TCP
         targetPort: 8680
   
   ---
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     name: csi-rbdplugin-provisioner
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: csi-rbdplugin-provisioner
     template:
       metadata:
         labels:
           app: csi-rbdplugin-provisioner
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               - labelSelector:
                   matchExpressions:
                     - key: app
                       operator: In
                       values:
                         - csi-rbdplugin-provisioner
                 topologyKey: "kubernetes.io/hostname"
         serviceAccountName: rbd-csi-provisioner
         priorityClassName: system-cluster-critical
         containers:
           - name: csi-provisioner
             image: quay.io/k8scsi/csi-provisioner:v2.1.2
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--retry-interval-start=500ms"
               - "--leader-election=true"
               #  set it to true to use topology based provisioning
               - "--feature-gates=Topology=false"
               # if fstype is not specified in storageclass, ext4 is default
               - "--default-fstype=ext4"
               - "--extra-create-metadata=true"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-snapshotter
             image: quay.io/k8scsi/csi-snapshotter:v4.0.0
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--leader-election=true"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             securityContext:
               privileged: true
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-attacher
             image: quay.io/k8scsi/csi-attacher:v3.1.0
             args:
               - "--v=5"
               - "--csi-address=$(ADDRESS)"
               - "--leader-election=true"
               - "--retry-interval-start=500ms"
             env:
               - name: ADDRESS
                 value: /csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-resizer
             image: quay.io/k8scsi/csi-resizer:v1.1.0
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--leader-election"
               - "--retry-interval-start=500ms"
               - "--handle-volume-inuse-error=false"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-rbdplugin
             securityContext:
               privileged: true
               capabilities:
                 add: ["SYS_ADMIN"]
             # for stable functionality replace canary with latest release version
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--nodeid=$(NODE_ID)"
               - "--type=rbd"
               - "--controllerserver=true"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--v=5"
               - "--drivername=rbd.csi.ceph.com"
               - "--pidlimit=-1"
               - "--rbdhardmaxclonedepth=8"
               - "--rbdsoftmaxclonedepth=4"
               - "--enableprofiling=false"
             env:
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
               - name: NODE_ID
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
               # - name: POD_NAMESPACE
               #   valueFrom:
               #     fieldRef:
               #       fieldPath: spec.namespace
               # - name: KMS_CONFIGMAP_NAME
               #   value: encryptionConfig
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - mountPath: /dev
                 name: host-dev
               - mountPath: /sys
                 name: host-sys
               - mountPath: /lib/modules
                 name: lib-modules
                 readOnly: true
               - name: ceph-csi-config
                 mountPath: /etc/ceph-csi-config/
               - name: ceph-csi-encryption-kms-config
                 mountPath: /etc/ceph-csi-encryption-kms-config/
               - name: keys-tmp-dir
                 mountPath: /tmp/csi/keys
           - name: csi-rbdplugin-controller
             securityContext:
               privileged: true
               capabilities:
                 add: ["SYS_ADMIN"]
             # for stable functionality replace canary with latest release version
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--type=controller"
               - "--v=5"
               - "--drivername=rbd.csi.ceph.com"
               - "--drivernamespace=$(DRIVER_NAMESPACE)"
             env:
               - name: DRIVER_NAMESPACE
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.namespace
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: ceph-csi-config
                 mountPath: /etc/ceph-csi-config/
               - name: keys-tmp-dir
                 mountPath: /tmp/csi/keys
           - name: liveness-prometheus
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--type=liveness"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--metricsport=8680"
               - "--metricspath=/metrics"
               - "--polltime=60s"
               - "--timeout=3s"
             env:
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi-provisioner.sock
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
             imagePullPolicy: "IfNotPresent"
         volumes:
           - name: host-dev
             hostPath:
               path: /dev
           - name: host-sys
             hostPath:
               path: /sys
           - name: lib-modules
             hostPath:
               path: /lib/modules
           - name: socket-dir
             emptyDir: {
               medium: "Memory"
             }
           - name: ceph-csi-config
             configMap:
               name: ceph-csi-config
           - name: ceph-csi-encryption-kms-config
             configMap:
               name: ceph-csi-encryption-kms-config
           - name: keys-tmp-dir
             emptyDir: {
               medium: "Memory"
             }
   EOF          
   ```

   

7. 创建rbdplugin

   ```yaml
   cat >6-csi-rbdplugin.yaml <<EOF
   ---
   kind: DaemonSet
   apiVersion: apps/v1
   metadata:
     name: csi-rbdplugin
   spec:
     selector:
       matchLabels:
         app: csi-rbdplugin
     template:
       metadata:
         labels:
           app: csi-rbdplugin
       spec:
         serviceAccountName: rbd-csi-nodeplugin
         hostNetwork: true
         hostPID: true
         priorityClassName: system-node-critical
         # to use e.g. Rook orchestrated cluster, and mons' FQDN is
         # resolved through k8s service, set dns policy to cluster first
         dnsPolicy: ClusterFirstWithHostNet
         containers:
           - name: driver-registrar
             # This is necessary only for systems with SELinux, where
             # non-privileged sidecar containers cannot access unix domain socket
             # created by privileged CSI driver container.
             securityContext:
               privileged: true
             image: quay.io/k8scsi/csi-node-driver-registrar:v2.1.0
             args:
               - "--v=5"
               - "--csi-address=/csi/csi.sock"
               - "--kubelet-registration-path=/var/lib/kubelet/plugins/rbd.csi.ceph.com/csi.sock"
             env:
               - name: KUBE_NODE_NAME
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - name: registration-dir
                 mountPath: /registration
           - name: csi-rbdplugin
             securityContext:
               privileged: true
               capabilities:
                 add: ["SYS_ADMIN"]
               allowPrivilegeEscalation: true
             # for stable functionality replace canary with latest release version
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--nodeid=$(NODE_ID)"
               - "--pluginpath=/var/lib/kubelet/plugins"
               - "--stagingpath=/var/lib/kubelet/plugins/kubernetes.io/csi/pv/"
               - "--type=rbd"
               - "--nodeserver=true"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--v=5"
               - "--drivername=rbd.csi.ceph.com"
               - "--enableprofiling=false"
               # If topology based provisioning is desired, configure required
               # node labels representing the nodes topology domain
               # and pass the label names below, for CSI to consume and advertise
               # its equivalent topology domain
               # - "--domainlabels=failure-domain/region,failure-domain/zone"
             env:
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
               - name: NODE_ID
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
               # - name: POD_NAMESPACE
               #   valueFrom:
               #     fieldRef:
               #       fieldPath: spec.namespace
               # - name: KMS_CONFIGMAP_NAME
               #   value: encryptionConfig
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - mountPath: /dev
                 name: host-dev
               - mountPath: /sys
                 name: host-sys
               - mountPath: /run/mount
                 name: host-mount
               - mountPath: /lib/modules
                 name: lib-modules
                 readOnly: true
               - name: ceph-csi-config
                 mountPath: /etc/ceph-csi-config/
               - name: ceph-csi-encryption-kms-config
                 mountPath: /etc/ceph-csi-encryption-kms-config/
               - name: plugin-dir
                 mountPath: /var/lib/kubelet/plugins
                 mountPropagation: "Bidirectional"
               - name: mountpoint-dir
                 mountPath: /var/lib/kubelet/pods
                 mountPropagation: "Bidirectional"
               - name: keys-tmp-dir
                 mountPath: /tmp/csi/keys
           - name: liveness-prometheus
             securityContext:
               privileged: true
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--type=liveness"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--metricsport=8680"
               - "--metricspath=/metrics"
               - "--polltime=60s"
               - "--timeout=3s"
             env:
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi.sock
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
             imagePullPolicy: "IfNotPresent"
         volumes:
           - name: socket-dir
             hostPath:
               path: /var/lib/kubelet/plugins/rbd.csi.ceph.com
               type: DirectoryOrCreate
           - name: plugin-dir
             hostPath:
               path: /var/lib/kubelet/plugins
               type: Directory
           - name: mountpoint-dir
             hostPath:
               path: /var/lib/kubelet/pods
               type: DirectoryOrCreate
           - name: registration-dir
             hostPath:
               path: /var/lib/kubelet/plugins_registry/
               type: Directory
           - name: host-dev
             hostPath:
               path: /dev
           - name: host-sys
             hostPath:
               path: /sys
           - name: host-mount
             hostPath:
               path: /run/mount
           - name: lib-modules
             hostPath:
               path: /lib/modules
           - name: ceph-csi-config
             configMap:
               name: ceph-csi-config
           - name: ceph-csi-encryption-kms-config
             configMap:
               name: ceph-csi-encryption-kms-config
           - name: keys-tmp-dir
             emptyDir: {
               medium: "Memory"
             }
   ---
   # This is a service to expose the liveness metrics
   apiVersion: v1
   kind: Service
   metadata:
     name: csi-metrics-rbdplugin
     labels:
       app: csi-metrics
   spec:
     ports:
       - name: http-metrics
         port: 8080
         protocol: TCP
         targetPort: 8680
     selector:
       app: csi-rbdplugin
   EOF    
   ```

   

#### storageclass

1. 创建ceph的secret，注意userkey是直接从ceph中获取，不需要在此hbase64

   ```sh
   sudo ceph auth get-key client.admin
   [sudo] password for bjrdc: 
   AQBGsuVglVG5DRAABh/ktIH+CZkCMMPR3VvuOw==
   ```

   ```yaml
   cat >2-csi-rbd-secret.yaml <<EOF
   ---
   apiVersion: v1
   kind: Secret
   metadata:
     name: csi-rbd-secret
     namespace: bjrdc-dev
   stringData:
     userID: admin
     userKey: AQBGsuVglVG5DRAABh/ktIH+CZkCMMPR3VvuOw==
   EOF  
   ```

2. 创建storageclass

   ```yaml
   cat > 0-csi-rbd-sc.yaml <<EOF
   ---
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
      name: csi-rbd-sc
      namespace: bjrdc-dev
   provisioner: rbd.csi.ceph.com
   parameters:
      clusterID: 5021e379-a950-49fa-bf5e-378354eba003
      pool: k8s_pool_01
      imageFeatures: layering
      csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
      csi.storage.k8s.io/provisioner-secret-namespace: bjrdc-dev
      csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
      csi.storage.k8s.io/controller-expand-secret-namespace: bjrdc-dev
      csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
      csi.storage.k8s.io/node-stage-secret-namespace: bjrdc-dev
   reclaimPolicy: Delete
   allowVolumeExpansion: true
   mountOptions:
      - discard
   EOF 
   ```

   

#### 使用

创建pvc

```yaml
cat >1-raw-block-pvc.yaml <<EOF
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: raw-block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
EOF  
```

创建pod

```yaml
cat >2-raw-block-pod.yaml <<EOF
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-raw-block-volume
spec:
  containers:
    - name: fc-container
      image: busybox:1.28.3
      command: ["/bin/sh", "-c"]
      args: ["tail -f /dev/null"]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: raw-block-pvc
        
EOF
```

### cephfs

cephf支持`ReadWriteMany`。cephfs的安装文件在 [ceph-csi](https://github.com/ceph/ceph-csi/blob/devel/docs/deploy-cephfs.md)

#### install

1. rabc

   ```yaml
   cat 0-csi-cephfs-nodeplugin-rbac.yaml 
   
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: cephfs-csi-nodeplugin
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: cephfs-csi-nodeplugin
   rules:
     - apiGroups: [""]
       resources: ["nodes"]
       verbs: ["get"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: cephfs-csi-nodeplugin
   subjects:
     - kind: ServiceAccount
       name: cephfs-csi-nodeplugin
       namespace: bjrdc-dev
   roleRef:
     kind: ClusterRole
     name: cephfs-csi-nodeplugin
     apiGroup: rbac.authorization.k8s.io
    
   ```

2. provisioner rabc

   ```yaml
   cat 1-csi-cephfs-provisioner-rbac.yaml 
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: cephfs-csi-provisioner
   
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: cephfs-external-provisioner-runner
   rules:
     - apiGroups: [""]
       resources: ["nodes"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["secrets"]
       verbs: ["get", "list"]
     - apiGroups: [""]
       resources: ["events"]
       verbs: ["list", "watch", "create", "update", "patch"]
     - apiGroups: [""]
       resources: ["persistentvolumes"]
       verbs: ["get", "list", "watch", "create", "delete", "patch"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims"]
       verbs: ["get", "list", "watch", "update"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["storageclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshots"]
       verbs: ["get", "list"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotcontents"]
       verbs: ["create", "get", "list", "watch", "update", "delete"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["volumeattachments"]
       verbs: ["get", "list", "watch", "update", "patch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["volumeattachments/status"]
       verbs: ["patch"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims/status"]
       verbs: ["update", "patch"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["csinodes"]
       verbs: ["get", "list", "watch"]
     - apiGroups: ["snapshot.storage.k8s.io"]
       resources: ["volumesnapshotcontents/status"]
       verbs: ["update"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: cephfs-csi-provisioner-role
   subjects:
     - kind: ServiceAccount
       name: cephfs-csi-provisioner
       namespace: bjrdc-dev
   roleRef:
     kind: ClusterRole
     name: cephfs-external-provisioner-runner
     apiGroup: rbac.authorization.k8s.io
   
   ---
   kind: Role
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     # replace with non-default namespace name
     namespace: bjrdc-dev
     name: cephfs-external-provisioner-cfg
   rules:
     # remove this once we stop supporting v1.0.0
     - apiGroups: [""]
       resources: ["configmaps"]
       verbs: ["get", "list", "create", "delete"]
     - apiGroups: ["coordination.k8s.io"]
       resources: ["leases"]
       verbs: ["get", "watch", "list", "delete", "update", "create"]
   
   ---
   kind: RoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: cephfs-csi-provisioner-role-cfg
     # replace with non-default namespace name
     namespace: bjrdc-dev
   subjects:
     - kind: ServiceAccount
       name: cephfs-csi-provisioner
       # replace with non-default namespace name
       namespace: bjrdc-dev
   roleRef:
     kind: Role
     name: cephfs-external-provisioner-cfg
     apiGroup: rbac.authorization.k8s.io
   ```

3. provisioner

   ```yaml
   cat 2-csi-cephfsplugin-provisioner.yaml 
   ---
   kind: Service
   apiVersion: v1
   metadata:
     name: csi-cephfsplugin-provisioner
     namespace: bjrdc-dev
     labels:
       app: csi-metrics
   spec:
     selector:
       app: csi-cephfsplugin-provisioner
     ports:
       - name: http-metrics
         port: 8080
         protocol: TCP
         targetPort: 8681
   
   ---
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     name: csi-cephfsplugin-provisioner
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: csi-cephfsplugin-provisioner
     replicas: 3
     template:
       metadata:
         labels:
           app: csi-cephfsplugin-provisioner
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               - labelSelector:
                   matchExpressions:
                     - key: app
                       operator: In
                       values:
                         - csi-cephfsplugin-provisioner
                 topologyKey: "kubernetes.io/hostname"
         serviceAccountName: cephfs-csi-provisioner
         priorityClassName: system-cluster-critical
         containers:
           - name: csi-provisioner
             image: quay.io/k8scsi/csi-provisioner:v2.1.2
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--leader-election=true"
               - "--retry-interval-start=500ms"
               - "--feature-gates=Topology=false"
               - "--extra-create-metadata=true"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-resizer
             image: quay.io/k8scsi/csi-resizer:v1.1.0
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--leader-election"
               - "--retry-interval-start=500ms"
               - "--handle-volume-inuse-error=false"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-snapshotter
             image: quay.io/k8scsi/csi-snapshotter:v4.0.0
             args:
               - "--csi-address=$(ADDRESS)"
               - "--v=5"
               - "--timeout=150s"
               - "--leader-election=true"
             env:
               - name: ADDRESS
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             securityContext:
               privileged: true
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-cephfsplugin-attacher
             image: quay.io/k8scsi/csi-attacher:v3.1.0
             args:
               - "--v=5"
               - "--csi-address=$(ADDRESS)"
               - "--leader-election=true"
               - "--retry-interval-start=500ms"
             env:
               - name: ADDRESS
                 value: /csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
           - name: csi-cephfsplugin
             securityContext:
               privileged: true
               capabilities:
                 add: ["SYS_ADMIN"]
             # for stable functionality replace canary with latest release version
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--nodeid=$(NODE_ID)"
               - "--type=cephfs"
               - "--controllerserver=true"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--v=5"
               - "--drivername=cephfs.csi.ceph.com"
               - "--pidlimit=-1"
               - "--enableprofiling=false"
             env:
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
               - name: NODE_ID
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi-provisioner.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - name: host-sys
                 mountPath: /sys
               - name: lib-modules
                 mountPath: /lib/modules
                 readOnly: true
               - name: host-dev
                 mountPath: /dev
               - name: ceph-csi-config
                 mountPath: /etc/ceph-csi-config/
               - name: keys-tmp-dir
                 mountPath: /tmp/csi/keys
           - name: liveness-prometheus
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--type=liveness"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--metricsport=8681"
               - "--metricspath=/metrics"
               - "--polltime=60s"
               - "--timeout=3s"
             env:
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi-provisioner.sock
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
             imagePullPolicy: "IfNotPresent"
         volumes:
           - name: socket-dir
             emptyDir: {
               medium: "Memory"
             }
           - name: host-sys
             hostPath:
               path: /sys
           - name: lib-modules
             hostPath:
               path: /lib/modules
           - name: host-dev
             hostPath:
               path: /dev
           - name: ceph-csi-config
             configMap:
               name: ceph-csi-config
           - name: keys-tmp-dir
             emptyDir: {
               medium: "Memory"
             }
   
   ```

4. cephplugin

   ```yaml
   cat 3-csi-cephfsplugin.yaml 
   ---
   kind: DaemonSet
   apiVersion: apps/v1
   metadata:
     name: csi-cephfsplugin
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: csi-cephfsplugin
     template:
       metadata:
         labels:
           app: csi-cephfsplugin
       spec:
         serviceAccountName: cephfs-csi-nodeplugin
         priorityClassName: system-node-critical
         hostNetwork: true
         # to use e.g. Rook orchestrated cluster, and mons' FQDN is
         # resolved through k8s service, set dns policy to cluster first
         dnsPolicy: ClusterFirstWithHostNet
         containers:
           - name: driver-registrar
             # This is necessary only for systems with SELinux, where
             # non-privileged sidecar containers cannot access unix domain socket
             # created by privileged CSI driver container.
             securityContext:
               privileged: true
             image: quay.io/k8scsi/csi-node-driver-registrar:v2.1.0
             args:
               - "--v=5"
               - "--csi-address=/csi/csi.sock"
               - "--kubelet-registration-path=/var/lib/kubelet/plugins/cephfs.csi.ceph.com/csi.sock"
             env:
               - name: KUBE_NODE_NAME
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - name: registration-dir
                 mountPath: /registration
           - name: csi-cephfsplugin
             securityContext:
               privileged: true
               capabilities:
                 add: ["SYS_ADMIN"]
               allowPrivilegeEscalation: true
             # for stable functionality replace canary with latest release version
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--nodeid=$(NODE_ID)"
               - "--type=cephfs"
               - "--nodeserver=true"
               - "--endpoint=$(CSI_ENDPOINT)"
               - "--v=5"
               - "--drivername=cephfs.csi.ceph.com"
               - "--enableprofiling=false"
               # If topology based provisioning is desired, configure required
               # node labels representing the nodes topology domain
               # and pass the label names below, for CSI to consume and advertise
               # its equivalent topology domain
               # - "--domainlabels=failure-domain/region,failure-domain/zone"
             env:
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
               - name: NODE_ID
                 valueFrom:
                   fieldRef:
                     fieldPath: spec.nodeName
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi.sock
             imagePullPolicy: "IfNotPresent"
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
               - name: mountpoint-dir
                 mountPath: /var/lib/kubelet/pods
                 mountPropagation: Bidirectional
               - name: plugin-dir
                 mountPath: /var/lib/kubelet/plugins
                 mountPropagation: "Bidirectional"
               - name: host-sys
                 mountPath: /sys
               - name: lib-modules
                 mountPath: /lib/modules
                 readOnly: true
               - name: host-dev
                 mountPath: /dev
               - name: host-mount
                 mountPath: /run/mount
               - name: ceph-csi-config
                 mountPath: /etc/ceph-csi-config/
               - name: keys-tmp-dir
                 mountPath: /tmp/csi/keys
           - name: liveness-prometheus
             securityContext:
               privileged: true
             image: quay.io/cephcsi/cephcsi:v3.4.0
             args:
               - "--type=liveness"
               - "--endpoint="
               - "--metricsport=8681"
               - "--metricspath=/metrics"
               - "--polltime=60s"
               - "--timeout=3s"
             env:
               - name: CSI_ENDPOINT
                 value: unix:///csi/csi.sock
               - name: POD_IP
                 valueFrom:
                   fieldRef:
                     fieldPath: status.podIP
             volumeMounts:
               - name: socket-dir
                 mountPath: /csi
             imagePullPolicy: "IfNotPresent"
         volumes:
           - name: socket-dir
             hostPath:
               path: /var/lib/kubelet/plugins/cephfs.csi.ceph.com/
               type: DirectoryOrCreate
           - name: registration-dir
             hostPath:
               path: /var/lib/kubelet/plugins_registry/
               type: Directory
           - name: mountpoint-dir
             hostPath:
               path: /var/lib/kubelet/pods
               type: DirectoryOrCreate
           - name: plugin-dir
             hostPath:
               path: /var/lib/kubelet/plugins
               type: Directory
           - name: host-sys
             hostPath:
               path: /sys
           - name: lib-modules
             hostPath:
               path: /lib/modules
           - name: host-dev
             hostPath:
               path: /dev
           - name: host-mount
             hostPath:
               path: /run/mount
           - name: ceph-csi-config
             configMap:
               name: ceph-csi-config
           - name: keys-tmp-dir
             emptyDir: {
               medium: "Memory"
             }
   ---
   # This is a service to expose the liveness metrics
   apiVersion: v1
   kind: Service
   metadata:
     name: csi-metrics-cephfsplugin
     namespace: bjrdc-dev  
     labels:
       app: csi-metrics
   spec:
     ports:
       - name: http-metrics
         port: 8080
         protocol: TCP
         targetPort: 8681
     selector:
       app: csi-cephfsplugin
   ```

5. 几个坑

   1. csi-cephfs模式下，需要内核在4.17版本以上，ubuntu18.04不行，故最好用ubuntu20.04

   2. 官方默认的`3-csi-cephfsplugin.yaml`中几个重要的args(NODE_ID\CSI_ENDPONIT)没有，需要自行添加。

      ```yaml
                args:
                  - "--nodeid=$(NODE_ID)"
                  - "--type=cephfs"
                  - "--nodeserver=true"
                  - "--endpoint=$(CSI_ENDPOINT)"
                  - "--v=5"
                  - "--drivername=cephfs.csi.ceph.com"
                  - "--enableprofiling=false"
      ```


#### storageclass

1. secret

   ```yaml
   cat 0-csi-cephfs-secret.yaml 
   ---
   apiVersion: v1
   kind: Secret
   metadata:
     name: csi-cephfs-secret
     namespace: bjrdc-dev
   stringData:
     # Required for statically provisioned volumes
     userID: admin
     userKey: AQBGsuVglVG5DRAABh/ktIH+CZkCMMPR3VvuOw==
   
     # Required for dynamically provisioned volumes
     adminID: admin
     adminKey: AQBGsuVglVG5DRAABh/ktIH+CZkCMMPR3VvuOw==
   ```

2. storageclass

   ```yaml
   cat 1-csi-cephfs-sc.yaml 
   ---
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
      name: csi-cephfs-sc
      namespace: bjrdc-dev
   provisioner: cephfs.csi.ceph.com
   parameters:
      clusterID: 5021e379-a950-49fa-bf5e-378354eba003
      pool: bjrdc-cephfs
      imageFeatures: layering
      csi.storage.k8s.io/provisioner-secret-name: csi-cephfs-secret
      csi.storage.k8s.io/provisioner-secret-namespace: bjrdc-dev
      csi.storage.k8s.io/controller-expand-secret-name: csi-cephfs-secret
      csi.storage.k8s.io/controller-expand-secret-namespace: bjrdc-dev
      csi.storage.k8s.io/node-stage-secret-name: csi-cephfs-secret
      csi.storage.k8s.io/node-stage-secret-namespace: bjrdc-dev
      fsName: cephfs
   reclaimPolicy: Retain
   allowVolumeExpansion: true
   mountOptions:
      - debug
   ```

#### excample

1. pvc

   ```yaml
   cat 1-cephfs-pvc.yaml 
   
   ---
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: csi-cephfs-pvc
     namespace: bjrdc-dev
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 100M
     storageClassName: csi-cephfs-sc
   ```

2. pod

   ```yaml
   cat 2-nginx-cephfs-deployment.yaml 
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-cephfs
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: nginx 
     replicas: 3
     template:
       metadata:
         labels:
           app: nginx 
       spec:
         containers:
         - name: nginx
           image: nginx:1.19.0
           ports:
           - containerPort: 80
           volumeMounts:
             - name: mypvc
               mountPath: /var/lib/www/html
         volumes:
           - name: mypvc
             persistentVolumeClaim:
               claimName: csi-cephfs-pvc
               readOnly: false
   ```

#### 验证

1. 

## Prometheus（监控）

> IPrometheuse.md
>
> 安装prometheus的基本步骤

| 步骤         | 说明                          |
| ------------ | ----------------------------- |
| namespace    | 设置prometheus的权限-非常重要 |
| secret       | storageclass密码              |
| storageclass |                               |
| pvc          |                               |
| configmap    | 设置prometheus.yaml           |
| deployment   |                               |
| service      |                               |
| daemonset    | 每个node上起一个node_export   |
|              |                               |

1. namespace

   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: pro-mon
   apiVersion: rbac.authorization.k8s.io/v1beta1
   ---
   kind: ClusterRole
   metadata:
     name: prometheus
     namespace: pro-mon
   rules:
   - apiGroups: [""]
     resources:
     - nodes
     - nodes/metrics  #让prometheus能够访问到nodes上的这个地址
     - nodes/proxy    #同上
     - configmaps
     - services
     - endpoints
     - pods
     verbs: ["get", "list", "watch"]
   - apiGroups:
     - extensions
     resources:
     - ingresses
     verbs: ["get", "list", "watch"]
   - nonResourceURLs: ["/metrics"]
     verbs: ["get"]
   ---  
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: ClusterRoleBinding
   metadata:
     name: prometheus
     namespace: pro-mon
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: prometheus
   subjects:
   - kind: ServiceAccount
     name: default
     namespace: pro-mon
   ```

2. 存储相关

   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-prometheus
     namespace: pro-mon
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   ---  
   apiVersion: v1
   kind: Secret
   metadata:
     name: ceph-rbd-secret
     namespace: pro-mon
   data:
     key: QVFCYUZCUmZPVndHQkJBQWJKQ2ZENTZrbGpMaHJ1aURmSW1odlE9PQ==
   ---  
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: prometheus-pvc
     namespace: pro-mon
   spec:
     storageClassName: ceph-storageclass-prometheus 
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 3Gi
   ```

3. **configmap**

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: prometheus-config
     namespace: pro-mon
   data:
     prometheus.yml: |
       global:
         scrape_interval: 15s
         scrape_timeout: 15s
       scrape_configs:
       - job_name: 'kubernetes-kubelet'
         scheme: https
         tls_config:
           ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
           insecure_skip_verify: true
         bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
         kubernetes_sd_configs:
         - role: node
         relabel_configs:
         - action: labelmap
           regex: __meta_kubernetes_node_label_(.+)
   
       - job_name: "kubernetes-nodes"
         kubernetes_sd_configs:
         - role: node
         relabel_configs:
         - source_labels: [__address__]
           regex: '(.*):10250'
           replacement: '${1}:9100'
           target_label: __address__
           action: replace
         - action: labelmap
           regex: __meta_kubernetes_node_label_(.+)
       
       - job_name: 'kubernetes-cadvisor'
         scheme: https
         tls_config:
           ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
         bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
         kubernetes_sd_configs:
         - role: node
         relabel_configs:
         - target_label: __address__
           replacement: kubernetes.default.svc:443
         - source_labels: [__meta_kubernetes_node_name]
           regex: (.+)
           target_label: __metrics_path__
           replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
         - action: labelmap
           regex: __meta_kubernetes_node_label_(.+)
   ```

   `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`和`/var/run/secrets/kubernetes.io/serviceaccount/token`这两个文件是 Pod 启动后自动注入进来的，通过这两个文件我们可以在 Pod 中访问 apiserver。

   **kubernetes-kubelet** kubenet自带的一些监控信息

   **kubernetes-nodes** 通过指定`kubernetes_sd_configs`的模式为`node`，Prometheus 就会自动从 Kubernetes 中发现所有的 node 节点并作为当前 job 监控的目标实例，发现的节点`/metrics`（由node_exporter提供）接口是默认的 kubelet 的 HTTP 接口。

   **kubernetes-cadvisor** cAdvisor能够获取当前节点上运行的所有容器的资源使用情况，通过访问kubelet的/metrics/cadvisor地址可以获取到cadvisor的监控指标，因此和获取kubelet监控指标类似，这里同样通过node模式自动发现所有的kubelet信息，并通过适当的relabel过程，修改监控采集任务的配置。 与采集kubelet自身监控指标相似，这里也有两种方式采集cadvisor中的监控指标

4. deployment

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: prometheus
     namespace: pro-mon 
     labels:
       app: prometheus
   spec:
     selector:
       matchLabels:
         app: prometheus
     template:
       metadata:
         labels:
           app: prometheus
       spec:
         containers:
         - image: bjrdc206.reg/library/prometheus:v2.20.1
           name: prometheus
           imagePullPolicy: IfNotPresent
           args:
           - "--config.file=/etc/prometheus/prometheus.yml"
           - "--storage.tsdb.path=/prometheus"
           - "--storage.tsdb.retention=7d"
           - "--web.enable-admin-api"
           - "--web.enable-lifecycle"
           ports:
           - containerPort: 9090
             name: http
           volumeMounts:
           - mountPath: "/prometheus"
             subPath: prometheus
             name: data
           - mountPath: "/etc/prometheus"
             name: config
           resources:
             requests:
               cpu: 1000m
               memory: 2Gi
             limits:
               cpu: 1000m
               memory: 2Gi
         securityContext:
           runAsUser: 0
         volumes:
         - name: config
           configMap:
             name: prometheus-config
         - name: data
           persistentVolumeClaim:
             claimName: prometheus-pvc
   ```

5. service

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: prometheus
     namespace: pro-mon
     labels:
       app: prometheus
   spec:
     selector:
         app: prometheus
     ports:
     - protocol : TCP
       port: 9090
   ```

6. daemonset

   ```yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: node-exporter
     namespace: pro-mon
     labels:
       name: node-exporter
   spec:
     selector:
       matchLabels:
         name: node-exporter
     template:
       metadata:
         labels:
           name: node-exporter
       spec:
         hostPID: true
         hostIPC: true
         hostNetwork: true
         containers:
         - name: node-exporter
           image: bjrdc206.reg/library/node-exporter:v1.0.1
           ports:
           - containerPort: 9100
           resources:
             requests:
               cpu: 0.15
           securityContext:
             privileged: true
           args:
           - --path.procfs
           - /host/proc
           - --path.sysfs
           - /host/sys
           - --collector.filesystem.ignored-mount-points
           - '"^/(sys|proc|dev|host|etc)($|/)"'
           volumeMounts:
           - name: dev
             mountPath: /host/dev
           - name: proc
             mountPath: /host/proc
           - name: sys
             mountPath: /host/sys
           - name: rootfs
             mountPath: /rootfs
         tolerations:
         - key: "node-role.kubernetes.io/master"
           operator: "Exists"
           effect: "NoSchedule"
         volumes:
           - name: proc
             hostPath:
               path: /proc
           - name: dev
             hostPath:
               path: /dev
           - name: sys
             hostPath:
               path: /sys
           - name: rootfs
             hostPath:
               path: /
   ```

## grafana

> 先安装prometheus，再安装grafana  

1. 存储声明

   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-grafana
     namespace: pro-mon
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   ---  
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: grafana-pvc
     namespace: pro-mon
   spec:
     storageClassName: ceph-storageclass-grafana
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 4Gi
   ```

2. deployment

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: grafana
     namespace: pro-mon
     labels:
       app: grafana
   spec:
     selector:
       matchLabels:
         app: grafana
     template:
       metadata:
         labels:
           app: grafana
       spec:
         containers:
         - name: grafana
           image: bjrdc206.reg/library/grafana/grafana:7.1.5
           imagePullPolicy: IfNotPresent
           ports:
           - containerPort: 3000
             name: grafana
           env:
           - name: GF_SECURITY_ADMIN_USER
             value: admin
           - name: GF_SECURITY_ADMIN_PASSWORD
             value: admin321
           readinessProbe:
             failureThreshold: 10
             httpGet:
               path: /api/health
               port: 3000
               scheme: HTTP
             initialDelaySeconds: 60
             periodSeconds: 10
             successThreshold: 1
             timeoutSeconds: 30
           livenessProbe:
             failureThreshold: 3
             httpGet:
               path: /api/health
               port: 3000
               scheme: HTTP
             periodSeconds: 10
             successThreshold: 1
             timeoutSeconds: 1
           resources:
             limits:
               cpu: 100m
               memory: 256Mi
             requests:
               cpu: 100m
               memory: 256Mi
           volumeMounts:
           - mountPath: /var/lib/grafana
             subPath: grafana
             name: storage
         securityContext:
           fsGroup: 472
           runAsUser: 472
         volumes:
         - name: storage
           persistentVolumeClaim:
             claimName: grafana-pvc
   ```

3. service

   ```
   apiVersion: v1
   kind: Service
   metadata:
     name: grafana
     namespace: pro-mon
     labels:
       app: grafana
   spec:
     ports:
       - port: 3000
     selector:
       app: grafana
   ```

4. 配置

   从grafana[官网](https://grafana.com/dashboards)

   中导入相关的dashboard，如747和741的这两个 dashboard。

   再修改一些参数——可能存在dashboard中的参数和prometheus中的不匹配的问题。

   

## 常用命令

### kubeadm

```sh
kubeadm token list

kubeadm join \
  <control-plane-host>:<control-plane-port> \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
  
kubeadm join \
  192.168.122.195:6443 \
  --token nx1jjq.u42y27ip3bhmj8vj \
  --discovery-token-ca-cert-hash sha256:c6de85f6c862c0d58cc3d10fd199064ff25c4021b6e88475822d6163a25b4a6c
```

```sh
kubeadm token create
kubeadm token list
kubeadm token create --print-join-command
```
### kubectl

#### 命令模式

kubectl   [get|describe|delete]  [node(s)|pod(s)|service(s)|role(s)|namespace(s)|cs|storageclass|pv|pvc|ep]



```sh
kubectl cluster-info
kubectl config view
```

#### 创建资源

```sh
kubectl create -f xx.yaml
kubectl create -f .
kubectl create configmap redis-cluster-config  -n bjrdc-dev --from-file=redis.conf=./redis.conf --from-file=redis-cluster-boot.py=./redis-cluster-boot.py
```

#### node

>Cluster Management Commands:
>certificate   Modify certificate resources.
>cluster-info  Display cluster info
>top           Display Resource (CPU/Memory/Storage) usage.
>cordon        Mark node as unschedulable
>uncordon      Mark node as schedulable
>drain         Drain node in preparation for maintenance
>taint         Update the taints on one or more nodes

```sh
kubectl get nodes 
kubectl cordon bjrdc207
kubectl uncordon bjrdc207
```



#### 获取资源

```sh
kubectl get ep kube-dns --namespace=kube-system
kubectl get pod -n bjrdc-dev --watch
kubectl get nodes
kubectl get namespace
kubectl get pod
kubectl get cs
kubectl get svc
kubectl get all --all-namespaces
kubectl get roles --all-namespaces
kubectl get service --all-namespaces
kubectl get --raw "/"
kubectl get events
kubectl get secret regcred --output=yaml
kubectl get deployment metrics-server -n kube-system --output=yaml
kubectl get serviceaccounts -n bjrdc-dev
kubectl get serviceaccounts -n bjrdc-dev -o yaml
```

#### 描述资源详情

```sh
kubectl describe namespaces kube-system
kubectl describe pods monitoring-influxdb-7f474cc79-cslt5 -n kube-system
kubectl describe clusterrole system:heapster
kubectl describe node bjrdc81
kubectl describe pod/mysql-statefulset-0 -n bjrdc-dev
```

#### logs

获取pod的log，该方法只能获取到控制台的log

```sh
kubectl logs pod xxxx -n kube-system -f
kubectl logs pod/mysql-statefulset-1 -c clone-mysql -n bjrdc-dev
#查看pod/mysql-statefulset-1下的容器 cone-mysql的日志
kubectl logs --tail=20 spring-cloud-k8s-consumer-7989fcf49c-h46rw -c spring-cloud-k8s-consumer-server -n bjrdc-dev -f
```

```
kubectl logs --namespace=kube-system -l k8s-app=kube-dns
```



#### lable

```sh
kubectl label node/10.47.136.60 role=entry
```

#### 删除资源

```sh
kubectl delete -f recommended.yaml
kubectl delete clusterrole system:heapster
kubectl delete pod hello-node
kubectl delete service spring-cloud-eureka -n bjrdc-dev
# cannot delete pod
kubectl delete deploment spring-cloud-eureka -n bjrdc-dev
# delete service with pod
kubectl delete event --all -n bjrdc-dev
```

#### 在线编辑资源信息

```sh
kubectl edit deployment kubernetes-hello-world
```

#### 控制台启动一个pod

```sh
kubectl run hello-node --image=hello-node:v1 --port=3000
```

#### exec

执行到pod内的任务，如果一个pod有多个container，需要增加-c参数

```sh
kubectl exec -it spring-cloud-config-68768fb466-mkjxz -n bjrdc-dev -- /bin/bash
kubectl exec -it spring-cloud-config-68768fb466-mkjxz -n bjrdc-dev -- /bin/sh
kubectl exec -it pod-shard-5c7b7f6bd6-2dhdj -c busybox -n bjrdc-dev -- hostname
```

#### 对比参数

```sh
kubectl diff -f ./my-manifest.yaml
```

#### 伸缩

```sh
kubectl scale --replicas=3 rs/foo                                 
# Scale a replicaset named 'foo' to 3
kubectl scale --replicas=2 deployment spring-cloud-k8s-consumer -n bjrdc-dev
# scale a depolyment spring-cloud-k8s-consumer to 2
# 伸缩后的deployment下的pod是可以起到负载均衡的效果的。通过service访问
kubectl scale --replicas=3 -f foo.yaml                            
# Scale a resource specified in "foo.yaml" to 3

kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  
# If the deployment named mysql's current size is 2, scale mysql to 3

kubectl scale --replicas=3 statefulset/gitlab-ci-runner -n gitlab-runner
```

#### 创建

```sh
kubectl create configmap redis-cluster-config  -n bjrdc-dev --from-file=redis.conf=./redis.conf --from-file=redis-cluster-boot.py=./redis-cluster-boot.py
```

#### 重启pod

```sh
kubectl get pod ubuntu -n bjrdc-dev -o yaml|kubectl replace --force -f -
kubectl get statefulset redis-stateful -n bjrdc-dev -o yaml|kubectl replace --force -f -
```

#### 修改kubectl默认namespace

```sh
kubectl config set-context --current --namespace=bjrdc-dev
```







## mysql

### mysql singal

> mysql单点部署，采用local和ceph两种卷额方式

#### on local

> local模式下，虽然可以启动mysql的服务，但是如果pod飘逸了后，由于在另外的node上没有数据，故会导致数据找不多，随意不推荐这种方式。本例只是作示例，生产环境不要这样来作。

1. 创建pv

   ```yaml
   cat >0-mysql-local-pv.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: mysql-local-pv
     namespace: bjrdc-dev
     labels:
       pv: mysql-local-pv
   spec:
     storageClassName: manual
     capacity:
       storage: 2Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/data/mysql"
   EOF
   ```

2. 创建pvc

   ```yaml
   cat >1-mysql-local-pvc.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: mysql-local-pv
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         pv: mysql-local-pv
     storageClassName: manual
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 2Gi
   EOF      
   ```

3. 创建deployment

   ```yaml
   cat >2-mysql-deployment.yaml <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mysql-local
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: mysql-local
     strategy:
       type: Recreate
     template:
       metadata:
         labels:
           app: mysql-local
       spec:
         containers:
         - image: mysql:5.7
           name: mysql-local
           env:
             # Use secret in real usage
           - name: MYSQL_ROOT_PASSWORD
             value: xxxx
           ports:
           - containerPort: 3306
             name: mysql-local
           volumeMounts:
           - name: mysql-persistent-storage
             mountPath: /var/lib/mysql
         volumes:
         - name: mysql-persistent-storage
           persistentVolumeClaim:
             claimName: mysql-local-pv
   EOF          
   ```

   

4. 创建service

   ```yaml
   cat >3-mysql-service.yaml <<EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql-local
     namespace: bjrdc-dev
     labels:
       app: mysql-local
   spec:
     selector:
         app: mysql-local
     ports:
     - protocol : TCP
       port: 3306
       targetPort: 3306
   EOF    
   ```

5. 测试

   ```sh
   mysql -u root -h mysql-local.bjrdc-dev.svc.cluster.local -p
   ```

   

#### on ceph

> 单机模式下，如果采用ceph作为存储，可以解决pod漂移后，不丢失存储的问题。
>
> 前提条件是需要一个部署好的ceph集群。

1. 创建pv

   ```yaml
   cat >0-mysql-pv.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: ceph-mysql-pv-01
     namespace: bjrdc-dev
     labels:
       pv: ceph-mysql-pv-01
   spec:
     capacity:
       storage: 5Gi
     accessModes:
       - ReadWriteOnce 
     rbd:
       monitors:
         - bjrdc208:6789 #ceph monitor
       pool: k8s_pool_01
       image: k8s-mysql-v1
       user: admin
       secretRef:
         name: ceph-normal-secret
       fsType: ext4
       readOnly: false
     persistentVolumeReclaimPolicy: Recycle
     EOF
   ```

2. 创建pvc

   ```yaml
    cat >1-mysql-pvc.yaml <EOF
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: ceph-mysql-pvc-01
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         pv: ceph-mysql-pv-01
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 3Gi
   EOF
   ```

3. deployment

   ```yaml
    cat >2-mysql-depoyment.yaml <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mysql-on-ceph-01
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: mysql-on-ceph-01
     strategy:
       type: Recreate
     template:
       metadata:
         labels:
           app: mysql-on-ceph-01
       spec:
         containers:
         - image: mysql:5.6
           name: mysql
           env:
             # Use secret in real usage
           - name: MYSQL_ROOT_PASSWORD
             value: xxxxx
           ports:
           - containerPort: 3306
             name: mysql
           volumeMounts:
           - name: mysql-persistent-storage
             mountPath: /var/lib/mysql
         volumes:
         - name: mysql-persistent-storage
           persistentVolumeClaim: 
             claimName: ceph-mysql-pvc-01
   EOF
   ```

4. 创建service

   ```yaml
   cat >3-mysql-service.yaml <<EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql-ceph-service-01
     namespace: bjrdc-dev
     labels:
       app: mysql-ceph-service-01
   spec:
     selector:
         app: mysql-on-ceph-01
     ports:
     - protocol : TCP
       port: 3306
       targetPort: 3306
   EOF
   ```

   

### mysql cluster

> 集群模式下，只能采用共享存储来进行数据存储，故必须一个ceph集群（采用local模式也不是完全不可以作cluster，就是做数据同步的过程较为复杂）

1. 创建ceph镜像，创建一个10G的Pool

   ```shell
   sudo ceph osd pool create k8s_pool_mysql_cluster_01 128 128
   sudo ceph osd pool set-quota k8s_pool_mysql_cluster_01 max_bytes $((10 * 1024 * 1024 * 1024))
   ```

2. 设置storageclass，在cluster模式下使用的是statefulset，此方式只能使用storageclass作为存储。

   ```yaml
   cat > 0-mysql-cluster-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-stateful-mysql
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_mysql_cluster_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

3. configmap for mysql

   ```yaml
   cat >1-mysql-cluster-configmap.yaml <<EOF
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: mysql-statefulset
     namespace: bjrdc-dev
     labels:
       app: mysql
   data:
     master.cnf: |
       # Apply this config only on the master.
       [mysqld]
       log-bin
     slave.cnf: |
       # Apply this config only on slaves.
       [mysqld]
       super-read-only
   EOF    
   ```

4. service for statefulset 

   ```yaml
   cat >2-mysql-cluster-service.yaml <<EOF
   # Headless service for stable DNS entries of StatefulSet members.
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: mysql-stateful
   spec:
     ports:
     - name: mysql-stateful-port
       port: 3306
     clusterIP: None
     selector:
       app: mysql-stateful
   ---
   # Client service for connecting to any MySQL instance for reads.
   # For writes, you must instead connect to the master: mysql-stateful-0.mysql-stateful-read.
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql-stateful-read
     namespace: bjrdc-dev
     labels:
       app: mysql-stateful
   spec:
     ports:
     - name: mysql-stateful-port
       port: 3306
     selector:
       app: mysql-stateful
   EOF    
   ```

5. 最关键的一步，配置statefulset

   ```yaml
   cat >3-mysql-cluster-stateful.yaml <<EOF
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: mysql-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: mysql-stateful
     serviceName: mysql-stateful-headless
     # must is mysql-stateful-headless,you cannot dig mysql-stateful-x.mysql-stateful-headless,if you not set serviceName.
     replicas: 3
     template:
       metadata:
         labels:
           app: mysql-stateful
       spec:
         initContainers:
         - name: init-mysql
           image: mysql:5.7
           command:
           - bash
           - "-c"
           - |
             set -ex
             # Generate mysql server-id from pod ordinal index.
             [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
             ordinal=${BASH_REMATCH[1]}
             echo [mysqld] > /mnt/conf.d/server-id.cnf
             # Add an offset to avoid reserved server-id=0 value.
             echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
             # Copy appropriate conf.d files from config-map to emptyDir.
             if [[ $ordinal -eq 0 ]]; then
               cp /mnt/config-map/master.cnf /mnt/conf.d/
             else
               cp /mnt/config-map/slave.cnf /mnt/conf.d/
             fi
           volumeMounts:
           - name: conf
             mountPath: /mnt/conf.d
           - name: config-map
             mountPath: /mnt/config-map
         - name: clone-mysql
           image: ist0ne/xtrabackup
           command:
           - bash
           - "-c"
           - |
             set -ex
             # Skip the clone if data already exists.
             [[ -d /var/lib/mysql/mysql ]] && exit 0
             # Skip the clone on master (ordinal index 0).
             [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
             ordinal=${BASH_REMATCH[1]}
             [[ $ordinal -eq 0 ]] && exit 0
             # Clone data from previous peer.
             ncat --recv-only mysql-stateful-$(($ordinal-1)).mysql-stateful-headless 3307 | xbstream -x -C /var/lib/mysql
             # Prepare the backup.
             xtrabackup --prepare --target-dir=/var/lib/mysql
           volumeMounts:
           - name: mysql-ceph-data
             mountPath: /var/lib/mysql
             subPath: mysql
           - name: conf
             mountPath: /etc/mysql/conf.d
         containers:
         - name: mysql
           image: mysql:5.7
           env:
           - name: MYSQL_ALLOW_EMPTY_PASSWORD
             value: "1"
           ports:
           - name: mysql
             containerPort: 3306
           volumeMounts:
           - name: mysql-ceph-data
             mountPath: /var/lib/mysql
             subPath: mysql
           - name: conf
             mountPath: /etc/mysql/conf.d
           resources:
             requests:
               cpu: 500m
               memory: 1Gi
           livenessProbe:
             exec:
               command: ["mysqladmin", "ping"]
             initialDelaySeconds: 30
             periodSeconds: 10
             timeoutSeconds: 5
           readinessProbe:
             exec:
               # Check we can execute queries over TCP (skip-networking is off).
               command: ["mysql", "-h", "127.0.0.1", "-e", "SELECT 1"]
             initialDelaySeconds: 5
             periodSeconds: 2
             timeoutSeconds: 1
         - name: xtrabackup
           image: ist0ne/xtrabackup
           ports:
           - name: xtrabackup
             containerPort: 3307
           command:
           - bash
           - "-c"
           - |
             set -ex
             cd /var/lib/mysql
   
             # Determine binlog position of cloned data, if any.
             if [[ -f xtrabackup_slave_info && "x$(<xtrabackup_slave_info)" != "x" ]]; then
               # XtraBackup already generated a partial "CHANGE MASTER TO" query
               # because we're cloning from an existing slave. (Need to remove the tailing semicolon!)
               cat xtrabackup_slave_info | sed -E 's/;$//g' > change_master_to.sql.in
               # Ignore xtrabackup_binlog_info in this case (it's useless).
               rm -f xtrabackup_slave_info xtrabackup_binlog_info
             elif [[ -f xtrabackup_binlog_info ]]; then
               # We're cloning directly from master. Parse binlog position.
               [[ `cat xtrabackup_binlog_info` =~ ^(.*?)[[:space:]]+(.*?)$ ]] || exit 1
               rm -f xtrabackup_binlog_info xtrabackup_slave_info
               echo "CHANGE MASTER TO MASTER_LOG_FILE='${BASH_REMATCH[1]}',\
                     MASTER_LOG_POS=${BASH_REMATCH[2]}" > change_master_to.sql.in
             fi
   
             # Check if we need to complete a clone by starting replication.
             if [[ -f change_master_to.sql.in ]]; then
               echo "Waiting for mysqld to be ready (accepting connections)"
               until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done
   
               echo "Initializing replication from clone position"
               mysql -h 127.0.0.1 \
                     -e "$(<change_master_to.sql.in), \
                             MASTER_HOST='mysql-stateful-0.mysql-stateful-headless', \
                             MASTER_USER='root', \
                             MASTER_PASSWORD='', \
                             MASTER_CONNECT_RETRY=10; \
                           START SLAVE;" || exit 1
               # In case of container restart, attempt this at-most-once.
               mv change_master_to.sql.in change_master_to.sql.orig
             fi
   
             # Start a server to send backups when requested by peers.
             exec ncat --listen --keep-open --send-only --max-conns=1 3307 -c \
               "xtrabackup --backup --slave-info --stream=xbstream --host=127.0.0.1 --user=root"
           volumeMounts:
           - name: mysql-ceph-data
             mountPath: /var/lib/mysql
             subPath: mysql
           - name: conf
             mountPath: /etc/mysql/conf.d
           resources:
             requests:
               cpu: 100m
               memory: 100Mi
         volumes:
         - name: conf
           emptyDir: {}
         - name: config-map
           configMap:
             name: mysql-statefulset
     volumeClaimTemplates:
     - metadata:
         name: mysql-ceph-data
       spec:
         accessModes: ["ReadWriteOnce"]
         storageClassName: ceph-storageclass-stateful-mysql
         resources:
           requests:
             storage: 451Mi
   EOF
   ```

6. 读写分离

   写入通过pod进行

   ```sh
   mysql -h  mysql-stateful-0.mysql-stateful-headless.bjrdc-dev.svc.cluster.local -u root -e "create database test_xxx"
   ```

   查询通过headless service进行

   ```sql
   mysql -h  mysql-stateful-headless.bjrdc-dev.svc.cluster.local -u root -e "show variables where variable_name='hostname'; show databases;"
   +---------------+------------------+
   | Variable_name | Value            |
   +---------------+------------------+
   | hostname      | mysql-stateful-2 |
   +---------------+------------------+
   +------------------------+
   | Database               |
   +------------------------+
   | information_schema     |
   | mysql                  |
   | performance_schema     |
   | sys                    |
   | test_xxx               |
   | xtrabackup_backupfiles |
   +------------------------+
   ```

7. 查看ceph与pvc的对应关系，需要通过pv查看

   1. 查找pvc

      ```bash
      kubectl get pvc -n bjrdc-dev|grep mysql-ceph
      mysql-ceph-data-mysql-stateful-0   Bound    pvc-4736b344-572b-4470-9d14-5384f949caa6   451Mi      RWO            ceph-storageclass-stateful-mysql   33h
      mysql-ceph-data-mysql-stateful-1   Bound    pvc-f5587f05-4137-46bf-a038-b305d0f0ff9e   451Mi      RWO            ceph-storageclass-stateful-mysql   38m
      mysql-ceph-data-mysql-stateful-2   Bound    pvc-f67d380f-fcc8-4999-925c-af58c6dcd0a4   451Mi      RWO            ceph-storageclass-stateful-mysql   133m
      ```

   2. 查看pvc信息

      ```yaml
      kubectl describe pv pvc-4736b344-572b-4470-9d14-5384f949caa6 -n bjrdc-dev
      Name:            pvc-4736b344-572b-4470-9d14-5384f949caa6
      Labels:          <none>
      Annotations:     pv.kubernetes.io/provisioned-by: ceph.com/rbd
                       rbdProvisionerIdentity: ceph.com/rbd
      Finalizers:      [kubernetes.io/pv-protection]
      StorageClass:    ceph-storageclass-stateful-mysql
      Status:          Bound
      Claim:           bjrdc-dev/mysql-ceph-data-mysql-stateful-0
      Reclaim Policy:  Delete
      Access Modes:    RWO
      VolumeMode:      Filesystem
      Capacity:        451Mi
      Node Affinity:   <none>
      Message:         
      Source:
          Type:          RBD (a Rados Block Device mount on the host that shares a pod's lifetime)
          CephMonitors:  [172.16.15.208:6789]
          RBDImage:      kubernetes-dynamic-pvc-f4d037f7-d458-11ea-9a3c-4e8cdd04a447
          FSType:        ext4
          RBDPool:       k8s_pool_mysql_cluster_01
          RadosUser:     admin
          Keyring:       /etc/ceph/keyring
          SecretRef:     &SecretReference{Name:ceph-rbd-secret,Namespace:,}
          ReadOnly:      false
      Events:            <none>
      ```

8. 下一步部署es

## Elasticsearch

> ES 与mysql一样也是有状态的，部署方式类似，在进行es的安装的时候，需要执行一定的`commad`详细参考如下yaml
>
> 安装需要重点关注的地方：
>
> 1. ES的安装首先需要理解es的配置文件，主要需要配置的几个参数
>
>    ```yaml
>    cluster.name: es-cluster-stateful-01
>        node.name: "${POD_NAME}"
>        network.host: 0.0.0.0
>        discovery.zen.ping.unicast.hosts: ["es-stateful-0.es-stateful-headless", "es-stateful-1.es-stateful-headless","es-stateful-2.es-stateful-headless"] 
>    ```
>
>    这些参数最终需要通过configmap挂载到pod中
>
> 2. 另外需要提前了解到es的image的默认的数据目录`/usr/share/elasticsearch/data `，以便将pvc挂在到对应的位置
>
> 3. 通过脚本设置一些启动参数如`ulimit -n 1024000`



### elasticsearch install

1. config storageclass

   ```yaml
   cat >0-es-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-es
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: bjrdc208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_es_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

2. configmap for elasticsearch.yaml

   > POD_NAME为一个环境变量，在statefulset中进行配置

   ```yaml
   cat >1-es-configmap.yaml <<EOF
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: es-config
     namespace: bjrdc-dev
   data:
     elasticsearch.yml: |
       cluster.name: es-cluster-stateful-01
       node.name: "${POD_NAME}"
       network.host: 0.0.0.0
       discovery.zen.ping.unicast.hosts: ["es-stateful-0.es-stateful-headless", "es-stateful-1.es-stateful-headless","es-stateful-2.es-stateful-headless"] 
   EOF
   ```

3. statefulset 配置

   需要执行`exec su elasticsearch /usr/share/elasticsearch/bin/elasticsearch`，因为默认的command似乎是被这个command声明的命令覆盖了，那么就要手动的启动

   `vm.max_map_count=262144`如果在容器执行不成功，就需要在host上修改systctl.conf，因为容器中没有这个权限。

   ```yaml
   cat 2-es-statefulset.yaml 
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: es-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: es-stateful 
     serviceName: es-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: es-stateful
       spec:
         initContainers:
         - name: busybox-init
           image: busybox
           command:
           - sh
           - "-c"
           - |
             set -ex
             chown -R 1000:1000 /usr/share/elasticsearch/data
           volumeMounts:
           - name: es-data
             mountPath: /usr/share/elasticsearch/data 
         containers:
         - name: es-stateful
           image: elasticsearch:6.8.11
           command:
           - bash
           - "-c"
           - |
             set -ex
             ulimit -n 1024000
             exec su elasticsearch /usr/share/elasticsearch/bin/elasticsearch
           env:
             - name: POD_NAME
               valueFrom:
                 fieldRef:
                   fieldPath: metadata.name
           ports:
           - containerPort: 9200
             protocol: TCP
           - containerPort: 9300
             protocol: TCP
           volumeMounts:
           - name: es-data
             mountPath: /usr/share/elasticsearch/data
           - name: es-config
             mountPath: /usr/share/elasticsearch/config/elasticsearch.yml
             subPath: elasticsearch.yml
         volumes:
           - name: es-config
             configMap:
               name: es-config
     volumeClaimTemplates:
     - metadata:
         name: es-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-es
         resources:
           requests:
             storage: 5Gi
   ```

4. headless service

   ```yaml
   cat >3-es-cluster-service.yaml <<EOF
   # Headless service for stable DNS entries of StatefulSet members.
   apiVersion: v1
   kind: Service
   metadata:
     name: es-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: es-stateful
   spec:
     ports:
     - name: es-stateful-port
       port: 9200
     clusterIP: None
     selector:
       app: es-stateful
   EOF    
   ```

   

5. 查看pod

   正常情况下，es的所有pod都起来了，通过如下命令可以查看

   ```sh
   kubectl get pod -n bjrdc-dev|grep es
   es-stateful-0                       1/1     Running   0          31m
   es-stateful-1                       1/1     Running   0          30m
   es-stateful-2                       1/1     Running   0          30m
   ```

6. 测试

   ```sh
   curl es-stateful-headless.bjrdc-dev.svc.cluster.local:9200/_cat/nodes
   10.244.3.40  25 13 2 0.04 0.23 0.26 mdi - es-stateful-2
   10.244.2.171 19 13 2 0.26 0.20 0.18 mdi - es-stateful-1
   10.244.1.138 23 13 3 0.62 0.45 0.29 mdi * es-stateful-0
   ```

   master

   ```sh
   curl es-stateful-headless.bjrdc-dev.svc.cluster.local:9200/_cat/master
   SBEGJoxxTmWpVFlDMVHnCg 10.244.1.138 10.244.1.138 es-stateful-0
   ```

   

### plugin config

TODO 安装自定义插件

### kibana install

> kibana 安装之前需要将es先安装好。
>
> kibana安装需要注意的地方
>
> 1. 需要通过configmap设置kibana.yml
>
>    ```yaml
>        server.host: "${POD_NAME}"
>        elasticsearch.url: "http://es-stateful-headless.bjrdc-dev.svc.cluster.local:9200"
>        elasticsearch.username: "kibana"
>        elasticsearch.password: "elastic"
>    ```
>
>    POD_NAME为一个环境变量通过如下配置设置
>
>    ```yaml
>            - name: POD_NAME
>              valueFrom:
>                fieldRef:
>                  fieldPath: metadata.name
>    ```
>
> 2. 安装kibana不需要使用statefulset

1. configmap

   ```yaml
   cat >0-es-configmap.yaml <<EOF
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: kibana-config
     namespace: bjrdc-dev
   data:
     kibana.yml: |
       server.host: "${POD_NAME}"
       elasticsearch.url: "http://es-stateful-headless.bjrdc-dev.svc.cluster.local:9200"
       elasticsearch.username: "kibana"
       elasticsearch.password: "elastic"
   EOF    
   ```

2. deployment

   ```yaml
   cat >1-es-kibana-deployment.yaml <<EOF
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     labels:
       app: kibana
     name: kibana
     namespace: bjrdc-dev
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kibana
     template:
       metadata:
         labels:
           app: kibana
       spec:
         containers:
         - name: kibana
           image: kibana:6.8.11
           ports:
           - containerPort: 5601
             protocol: TCP
           env:
           - name: POD_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
           volumeMounts:
           - name: kibana-config
             mountPath: /usr/share/kibana/config/kibana.yml
             subPath: kibana.yml
         volumes:
         - name: kibana-config
           configMap:
             name: kibana-config
   EOF          
   ```

3. service

   ```yaml
   cat >2-es-kibana-service.yaml <<EOF
   kind: Service
   apiVersion: v1
   metadata:
     labels:
       app: kibana
     name: kibana
     namespace: bjrdc-dev
   spec:
     ports:
     - port: 5601
     selector:
       app: kibana
   EOF    
   ```

4. 完成安装后，通过如下地址访问kibana

   ```sh
   curl http://kibana.bjrdc-dev.svc.cluster.local:5601/app/kibana#/home
   ```

   

## 日志收集

### logstash



### filebeat

>filebeat 是logstash的升级版本，使用golang开发更节省资源。
>
>filebeat与kubernetes的集成可以采用三种方式
>
>| 集成方式                    | 优缺点                                                       |
>| --------------------------- | ------------------------------------------------------------ |
>| 集成到container中           | 需要定制container，日后会带来升级的问题，如果因其他业务有container定制的需求，可以考虑将filebeat一并集成到里面去 |
>| multi-container for one pod | filebeat独立的container与其他的container在一个pod中集成，通过共享目录的方式实现文件的beat |
>| daemtset方式                | 尚未研究                                                     |
>
>filebeat并不能自动的清理文件，如果需要对文件进行清理，需要使用脚本或者log引擎自带功能进行
>
>安装方式如下

1. configmap

   网上的很多方式均不能正常的进行安装，在了解了filebeat的原理后，决定采用自定义filebeat.yml文件的方式进行安装。

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: filebeat-config
     namespace: bjrdc-dev
   data:
     filebeat.yml: |
       filebeat.config:
         modules:
           path: ${path.config}/modules.d/*.yml
           reload.enabled: false
       filebeat.inputs:
       - type: log
         enabled: true
         paths:
           - /logs/*
       output.elasticsearch:
         hosts: ["es-stateful-0.es-stateful-headless:9200","es-stateful-1.es-stateful-headless:9200","es-stateful-2.es-stateful-headless:9200"]
         username: ""
         password: ""
       setup.kibana:
         host: "kibana:5601"
   ```

2. 使用pod方式部署一个nginx 通过filebeat来获取日志

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: filebeat
     namespace: bjrdc-dev
     labels:
       app: filebeat
   spec:
         containers:
         - image: bjrdc206.reg/library/elastic/filebeat:6.8.11
           name: filebeat
           command:
           - sh
           - "-c"
           - |
             filebeat -e &
             sleep 5
             filebeat setup --dashboards
             filebeat modules enable system nginx mysql
             while true;do sleep 10;echo sleep 5;done
           volumeMounts:
           - name: app-logs
             mountPath: /logs
           - name: filebeat-config
             subPath: filebeat.yml
             mountPath: /usr/share/filebeat/filebeat.yml
         - name: nginx
           image: nginx:1.19.0
           ports:
           - containerPort: 80
           volumeMounts:
           - mountPath: /var/log/nginx
             name: app-logs
         volumes:
         - name: app-logs
           emptyDir: {}
         - name: filebeat-config
           configMap:
             name: filebeat-config
   ```

3. 使用service暴露nginx

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: filebeat-nginx
     namespace: bjrdc-dev
     labels:
       app: filebeat-nginx
   spec:
     ports:
     - port: 80
       name: web
     selector:
       app: filebeat
   ```

4. 验证

   通过如下命令访问nginx的服务，正常情况下，可以访问成功

   ```
   curl filebeat-nginx.bjrdc-dev.svc.cluster.local
   ```

5. 在kinbana上查看

   打开kinbana`http://kibana.bjrdc-dev.svc.cluster.local:5601/`在`Dev Tools`中查看是否有数据进来

   ```json
   GET filebeat-6.8.11-2020.09.03/_search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   同时查看是否indics已经创建`filebeat-6.8.11-2020.09.03`

6. 关于日志丢失

   如果pod重启，在重启之前未提交的日志可能丢失，因为该pod采用了local存储，该存储早pod重启后会被k8s回收。

   可以将这个local存储修改到ceph中

   TODO

## zookeeper

> zookeeper 如果要自己从0安装的话，会比较麻烦，需要设置myid，和修改zoo.cfg。如果要自己做，可以参考本文中的redis的配置。
>
> google官方提供了一个自动配置的容器`mirrorgooglecontainers/kubernetes-zookeeper:1.0-3.4.10`，该容器自带 start-zookeeper的脚本能够自动设置相关参数。
>
> 安装zookeeper有两种方式，一种是使用kubernet提供的kubernetes-zookeeper镜像，另外一种是使用zookeeper官方的镜像，相信如下：

#### kubernetes-zookeeper

kubernetes提供了一个zookeeper镜像，用于方便的启动zookeeper，其中内置了一些脚本。详细安装过程如下。

1. 仍然是配置storageclass（一旦有了storageclass，走遍天下都不怕）

   ```yaml
   cat >0-zookeeper-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-zookeeper
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_zookeeper_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

2. 配置statefulset,基本按照kubenetes官方的样例修改的

   ```yaml
   cat >1-zookeeper-cluster-statefulset.yaml <<EOF
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: zookeeper-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: zookeeper-stateful 
     serviceName: zookeeper-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: zookeeper-stateful
       spec:
         containers:
         - name: zookeeper-stateful
           image: bjrdc206.reg/library/kubernetes-zookeeper:1.0-3.4.10
           command:
           - bash
           - "-c" 
           - |
             set -ex
             start-zookeeper \
             --servers=3 \
             --data_dir=/var/lib/zookeeper/data \
             --data_log_dir=/var/lib/zookeeper/data/log \
             --conf_dir=/opt/zookeeper/conf \
             --client_port=2181 \
             --election_port=3888 \
             --server_port=2888 \
             --tick_time=2000 \
             --init_limit=10 \
             --sync_limit=5 \
             --heap=512M \
             --max_client_cnxns=60 \
             --snap_retain_count=3 \
             --purge_interval=12 \
             --max_session_timeout=40000 \
             --min_session_timeout=4000 \
             --log_level=INFO
           ports:
           - containerPort: 2181
             protocol: TCP
           - containerPort: 2888
             protocol: TCP
           - containerPort: 3888
             protocol: TCP
           livenessProbe:
             exec: 
               command: 
               - sh
               - -c
               - zookeeper-ready.sh 2181  
             initialDelaySeconds: 80
             periodSeconds: 30
   
           volumeMounts:
           - name: zookeeper-data
             mountPath: /var/lib/zookeeper
   
     volumeClaimTemplates:
     - metadata:
         name: zookeeper-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-zookeeper
         resources:
           requests:
             storage: 2Gi
   EOF          
   ```

3. headless 配置

   ```yaml
   cat >2-zookeeper-cluster-service.yaml <<EOF
   # Headless service for stable DNS entries of StatefulSet members.
   apiVersion: v1
   kind: Service
   metadata:
     name: zookeeper-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: zookeeper-stateful
   spec:
     ports:
     - name: zookeeper-stateful-port
       port: 2181
     - name: zookeeper-stateful-port2
       port: 2888
     - name: zookeeper-stateful-port3
       port: 3888
   
   
     clusterIP: None
     selector:
       app: zookeeper-stateful
   EOF    
   ```

4. 启动完成后，查看集群状态

   ```sh
   kubectl exec -it zookeeper-stateful-0 -n bjrdc-dev -- /opt/zookeeper/bin/zkServer.sh status
   ZooKeeper JMX enabled by default
   Using config: /opt/zookeeper/bin/../conf/zoo.cfg
   Mode: leader
   ```


#### zookeeper

kubernetes-zookeeper的镜像的版本比较落后，如果需要使用最新版本的则需要使用zookeeper官方的镜像。使用官方的镜像安装的思路是使用脚本设置环境变量，然后启动zookeeper。

1. storageclass（可选）

   使用前文创建的storageclass `csi-rbd-sc`
   
2. configmap

   ```yaml
   cat 1-zookeeper-configmap.yaml 
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: zookeeper-config
     namespace: bjrdc-dev
   data:
     zookeeper-start.sh:
       #!/bin/bash
       sleep 5;
       HOST=`hostname`;
       ZOO_MY_ID_=${HOST##*-};
       export ZOO_MY_ID=$((ZOO_MY_ID_+1));
       export ZOO_SERVERS="server.1=zookeeper-stateful-0.zookeeper-stateful-headless:2888:3888;2181 server.2=zookeeper-stateful-1.zookeeper-stateful-headless:2888:3888;2181 server.3=zookeeper-stateful-2.zookeeper-stateful-headless:2888:3888;2181";
       bash /docker-entrypoint.sh;
       zkServer.sh start-foreground;
   ```
   
   
   
3. statefulset

   ```yaml
   cat 2-zookeeper-cluster-statefulset.yaml 
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: zookeeper-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: zookeeper-stateful 
     serviceName: zookeeper-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: zookeeper-stateful
       spec: 
         containers:
         - name: zookeeper-stateful
           image: bjrdc206.reg/library/zookeeper:3.6.1
           command:
           - sh
           - "-c"
           - |
             set -ex
             bash /zookeeper-start.sh
           ports:
           - containerPort: 2181
             protocol: TCP
           - containerPort: 2888
             protocol: TCP
           - containerPort: 3888
             protocol: TCP
           livenessProbe:
             tcpSocket:
               port: 2181
             initialDelaySeconds: 80
             periodSeconds: 30
           volumeMounts:
           - name: zookeeper-data
             mountPath: /data
           - name: zookeeper-config
             mountPath: /zookeeper-start.sh
             subPath: zookeeper-start.sh
         volumes:
         - name: zookeeper-config
           configMap:
             name: zookeeper-config
   
     volumeClaimTemplates:
     - metadata:
         name: zookeeper-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: csi-rbd-sc
         resources:
           requests:
             storage: 2Gi
   ```

4. headless

   ```yaml
   cat 2-zookeeper-cluster-service.yaml 
   # Headless service for stable DNS entries of StatefulSet members.
   apiVersion: v1
   kind: Service
   metadata:
     name: zookeeper-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: zookeeper-stateful
   spec:
     ports:
     - name: zookeeper-stateful-port
       port: 2181
     - name: zookeeper-stateful-port2
       port: 2888
     - name: zookeeper-stateful-port3
       port: 3888
   
   
     clusterIP: None
     selector:
       app: zookeeper-stateful
   ```

5. 验证

   ```
    kubectl exec -it zookeeper-stateful-2 -n bjrdc-dev -- zkServer.sh status
   ```

   

## Kafka

> 在部署kafka之前需要先部署好zookeeper。

1. 又是 storageclass

   ```yaml
   cat 0-kafka-storageclass.yaml 
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-kafka
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_kafka_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   ```

2. statefulset

   ```yaml
   cat >1-kafka-cluster-statefulset.yaml <<EOF
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: kafka-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: kafka-stateful 
     serviceName: kafka-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: kafka-stateful
       spec:
         containers:
         - name: kafka-stateful
           image: bjrdc206.reg/library/kafka:2.12-2.5.0
           #command: 
           #- bash
           #- "-c"
           #- |
           #  set -ex
           #  while true;do sleep 5; echo doing; done
           ports:
           - containerPort: 9092
             protocol: TCP
           env:
           - name: KAFKA_ZOOKEEPER_CONNECT
             value: kzookeeper-stateful-0.kzookeeper-stateful-headless:2181
           - name: KAFKA_ADVERTISED_LISTENERS
             value: PLAINTEXT://:9092
           - name: KAFKA_LISTENERS
             value: PLAINTEXT://:9092
           volumeMounts:
           - name: kafka-data
             mountPath: /kafka
   
     volumeClaimTemplates:
     - metadata:
         name: kafka-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-kafka
         resources:
           requests:
             storage: 2Gi
   EOF          
   ```

3. headless

   ```yaml
   cat >2-kafka-cluster-service.yaml <<EOF
   # Headless service for stable DNS entries of StatefulSet members.
   apiVersion: v1
   kind: Service
   metadata:
     name: kafka-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: kafka-stateful
   spec:
     ports:
     - name: kafka-stateful-port
       port: 9092
     clusterIP: None
     selector:
       app: kafka-stateful
   EOF    
   ```

4. 测试

   ```
   kubectl exec -it kafka-stateful-0 -n bjrdc-dev -- kafka-topics.sh --describe --zookeeper kzookeeper-stateful-0.kzookeeper-stateful-headless:2181
   ```

5. 安装kafdrop

   deployment

   ```yaml
   cat 0-kafdrop-deployment.yaml 
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kafdrop
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: kafdrop
     template:
       metadata:
         labels:
           app: kafdrop
       spec:
         containers:
         - image: bjrdc206.reg/library/kafdrop:3.27.0
           name: kafdrop
           env: 
           - name: KAFKA_BROKERCONNECT
             value: kafka-stateful-0.kafka-stateful-headless:9092,kafka-stateful-1.kafka-stateful-headless:9092,kafka-stateful-2.kafka-stateful-headless:9092
           ports:
           - containerPort: 9000
           livenessProbe:
             httpGet:
               path: /actuator
               port: 9000
             initialDelaySeconds: 10
             periodSeconds: 10
   ```

   service

   ```yaml
   cat 1-kafdrop-service.yaml 
   apiVersion: v1
   kind: Service
   metadata:
     name: kafdrop
     namespace: bjrdc-dev
     labels:
       app: kafdrop
   spec:
     selector:
         app: kafdrop
     ports:
     - protocol : TCP
       port: 9000
   ```

6. 测试kafdrop

   ```sh
   curl http://kafdrop.bjrdc-dev.svc.cluster.local:9000/
   ```

   

## RabbitMQ

> rabbitmq 的安装在kubernetes上并没有太多的特殊的地方，主要需要解决如下几个问题
>
> 1. .erlang.cookie文件的同步问题：保证各个节点上的该文件一样
>
>    **该问题在刚开始的时候花费了我较多的时间，因为没有太多关注，后来查询官方文档后，才知道，如果没有设置该文件，则rabbit会自动创建一个随机的，这样就导致各个节点上的不一样了，无法链接到集群；同时通过测试发现设置的`RABBITMQ_ERLANG_COOKIE`并不能生效，也耽误了一定的时间**
>
>    ```
>    If the file does not exist, Erlang VM will try to create one with a randomly generated value when the RabbitMQ server starts up. Using such generated cookie files are appropriate in development environments only. Since each node will generate its own value independently, this strategy is not really viable in a clustered environment.
>    ```
>
>    [https://www.rabbitmq.com/clustering.html](https://www.rabbitmq.com/clustering.html)
>
> 2. 域名的问题：rabbit有一个短域名和长域名的概念，如果短域名下，是无法通过pod中的容器访问到其他的pod的，故无法创建集群。

1. 创建ceph pool

   ```sh
   sudo ceph osd pool create k8s_pool_rabbit_01
   sudo ceph osd pool application enable k8s_pool_rabbit_01 rbd
   ```

   rabbitmq以集群方案安装的思路是，采用storageclass作为存储，使用命令创建集群。

   rabbitMQ默认的存储路径为`/var/lib/rabbitmq/mnesia/`

2. 创建erlang.cookie

   ```sh
   echo $(openssl rand -base64 32) > erlang.cookie
   ```

3. 创建rabbit-start.sh

   > 在rabbit启动之前需要等待一段时间`30s`等headlessservice启动

   ```sh
   cat >rabbit-start.sh <<EOF
   set -ex
   echo  "waiting headless online"
   sleep 30
   hostname=`hostname`
   rabbitmq-server -detached
   echo "start rabbitmq-server"
   
   until rabbitmqctl node_health_check; do sleep 3; done
   echo "$hostname is started."
   if [[ "$hostname" -eq "$CLUSTER" ]]; then
     rabbitmqctl stop_app
     echo "stop_app on $hostname"
     echo  "reset on $hostname"
     rabbitmqctl start_app
     echo "start_app on $hostname"
     rabbitmq-plugins enable rabbitmq_management
     echo  "rabbitmq_management enabled on $hostname"
     bjrdc=`rabbitmqctl list_users|grep bjrdc||true`
     if [ -z "$bjrdc" ]; then
       echo "create user bjrdc.."
       rabbitmqctl add_user bjrdc xxxxx
       rabbitmqctl set_user_tags bjrdc administrator
       rabbitmqctl set_permissions -p / bjrdc ".*" ".*" ".*"
       echo "create user finished."
     else
       echo "bjrdc is in cluster so do nothing."     
     fi      
   else
     cluster=`rabbitmqctl cluster_status|grep $CLUSTER.$HEADLESS||true`
     echo "cluster is $cluster ."
     if [ -z "$cluster" ]; then
       echo "$hostname is not join to cluster $CLUSTER.$HEADLESS"
       rabbitmqctl stop_app
       rabbitmqctl join_cluster rabbit@$CLUSTER.$HEADLESS
       echo  "$hostname join cluster to rabbit@$CLUSTER.$HEADLESS"
       rabbitmqctl start_app
     else
       echo "$hostname is in cluster $CLUSTER.$HEADLESS"
     fi
   fi
   EOF
   ```

   

4. 创建的configmap 

   采用命令行方式创建，可以引入外部文件

   ```sh
   kubectl create configmap rabbit-cluster-config  -n bjrdc-dev --from-file=rabbit-start.sh=./rabbit-start.sh
   ```

5. 创建 .erlang.cookie的configmap

   ```sh
   kubectl create configmap rabbit-cluster-cookie  -n bjrdc-dev --from-file=erlang.cookie=./erlang.cookie
   ```

6. 创建 storageclass

   ```yaml
   cat >0-rabbit-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-rabbit
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_rabbit_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering
     
   EOF  
   ```

7. 创建statefulset

   ```yaml
   cat > 1-rabbit-cluster-statefulset.yaml <<EOF
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: rabbit-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: rabbit-stateful 
     serviceName: rabbit-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: rabbit-stateful
       spec:
         containers:
         - name: rabbit-stateful
           image: bjrdc206.reg/library/rabbitmq:3.7-management
           command:
           - bash
           - "-c" 
           - |
             set -ex
             exec /bjrdc/rabbit-start.sh &
             while true; do echo do...; sleep 10;done
           env: 
           - name: CLUSTER
             value: rabbit-stateful-0
           - name: HEADLESS
             value: rabbit-stateful-headless
           - name: MY_POD_NAME
             valueFrom:
               fieldRef:
                 fieldPath: metadata.name
           - name: RABBITMQ_NODENAME
             value: "rabbit@$(MY_POD_NAME).$(HEADLESS)"
           - name: RABBITMQ_USE_LONGNAME
             value: "true"
           ports:
           - containerPort: 15672
             protocol: TCP
           - containerPort: 5672
             protocol: TCP
           - containerPort: 25672
             protocol: TCP
           - containerPort: 4369
             protocol: TCP
   
           livenessProbe:
             httpGet:
               path: /api/overview
               port: 15672 
               httpHeaders:
                 - name: Authorization
                   value: Basic xxxxxxx
             initialDelaySeconds: 80
             periodSeconds: 30
   
           volumeMounts:
           - name: rabbit-data
             mountPath: /var/lib/rabbitmq/mnesia 
           - name: rabbit-config
             mountPath: /bjrdc/rabbit-start.sh
             subPath: rabbit-start.sh
           - name: rabbit-config-cookie
             mountPath: /var/lib/rabbitmq/.erlang.cookie
             subPath: erlang.cookie
         volumes:
           - name: rabbit-config
             configMap:
               name: rabbit-cluster-config
               defaultMode: 0777
           - name: rabbit-config-cookie
             configMap:
               name: rabbit-cluster-cookie
               defaultMode: 0400
   
     volumeClaimTemplates:
     - metadata:
         name: rabbit-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-rabbit
         resources:
           requests:
             storage: 2Gi
   EOF          
   ```

   xxxxxxx is base of user:password

   ```
   echo xxx:yyy|base64
   eHh4Onl5eQo=
   ```

   

8. 创建用户权限

   > 使用如下命令创建用户，具体已经包含在`rabbit-start.sh`中

   ```sh
   rabbitmqctl add_user bjrdc xxxxx
   rabbitmqctl set_user_tags bjrdc administrator
   rabbitmqctl set_permissions -p / bjrdc ".*" ".*" ".*"
   ```

9. 测试

   使用浏览器打开manager地址`http://rabbit-stateful-0.rabbit-stateful-headless.bjrdc-dev.svc.cluster.local:15672/#/`

   或者使用命令访问集群状态

   ```sh
   kubectl exec -it rabbit-stateful-0 -n bjrdc-dev -- rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit-stateful-0.rabbit-stateful-headless ...
   [{nodes,[{disc,['rabbit@rabbit-stateful-0.rabbit-stateful-headless',
                   'rabbit@rabbit-stateful-1.rabbit-stateful-headless',
                   'rabbit@rabbit-stateful-2.rabbit-stateful-headless']}]},
    {running_nodes,['rabbit@rabbit-stateful-1.rabbit-stateful-headless',
                    'rabbit@rabbit-stateful-2.rabbit-stateful-headless',
                    'rabbit@rabbit-stateful-0.rabbit-stateful-headless']},
    {cluster_name,<<"rabbit@rabbit-stateful-0.rabbit-stateful-headless.bjrdc-dev.svc.cluster.local">>},
    {partitions,[]},
    {alarms,[{'rabbit@rabbit-stateful-1.rabbit-stateful-headless',[]},
             {'rabbit@rabbit-stateful-2.rabbit-stateful-headless',[]},
             {'rabbit@rabbit-stateful-0.rabbit-stateful-headless',[]}]}]
   ```

   

## Redis

### singal

> 使用ceph作为存储的单节点配置方式

1. 万年不变的storageclass

   ```yaml
   cat >0-redis-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-redis
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_redis_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

2. 千年不变的configmap

   ```yaml
   cat> 1-redis-configmap.yaml <<EOF
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: redis-config
     namespace: bjrdc-dev
   data:
     redis.conf: |
       dir /data/redis
       save 900 1
       save 300 10
       save 60 10000
       bind 0.0.0.0
   EOF    
   ```
   
   
   
3. storageclass

   ```yaml
   cat >0-redis-storageclass.yaml <<EOF
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ceph-storageclass-redis
     namespace: bjrdc-dev
   provisioner: ceph.com/rbd
   parameters:
     monitors: 172.16.15.208:6789
     adminId: admin
     adminSecretName: ceph-rbd-secret
     adminSecretNamespace: bjrdc-dev
     pool: k8s_pool_redis_01
     userId: admin
     userSecretName: ceph-rbd-secret
     fsType: ext4
     imageFormat: "2"
     imageFeatures: "layering"
   EOF  
   ```

   

4. 必要的pvc

   ```yaml
   cat >2-redis-pvc.yaml <<EOF
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: ceph-redis-pvc
     namespace: bjrdc-dev
   spec:
     accessModes:
       - ReadWriteOnce
     storageClassName: ceph-storageclass-redis 
     resources:
       requests:
         storage: 3Gi
   EOF      
   ```

5. 必须要的deployment

   ```yaml
   cat >3-redis-deployment.yaml <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: redis-on-ceph-singal
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: redis-on-ceph-singal
     strategy:
       type: Recreate
     template:
       metadata:
         labels:
           app: redis-on-ceph-singal
       spec:
         containers:
         - image: redis:6.0.6
           name: redis
           ports:
           - containerPort: 6379
           volumeMounts:
           - name: redis-data
             mountPath: /data/redis
           - name: redis-config
             mountPath: /data/redis.conf
             subPath: redis.conf
           command:
           - bash
           - "-c"
           - |
             exec /usr/local/bin/redis-server /data/redis.conf
         volumes:
         - name: redis-data
           persistentVolumeClaim: 
             claimName: ceph-redis-pvc
         - name: redis-config
           configMap:
             name: redis-config
   EOF          
   ```

6. 可能需要service

   ```yaml
   cat >4-redis-service.yaml <<EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: redis-service-singal
     namespace: bjrdc-dev
     labels:
       app: redis-service-singal
   spec:
     selector:
         app: redis-on-ceph-singal
     ports:
     - protocol : TCP
       port: 6379
       targetPort: 6379
   EOF    
   ```

   

7. 验证

   ```sh
   kubectl exec -it redis-on-ceph-singal-c744b4fdb-pntns -n bjrdc-dev -- redis-cli 
   127.0.0.1:6379> get keys
   (nil)
   127.0.0.1:6379> set 001 chengtao
   OK
   127.0.0.1:6379> get 001
   "chengtao"
   ```

   

### cluster

> 集群方式部署和mysql的类似，需要用到`statefulset`
>
> 基本思路是使用statefulset创建节点后，通过redis的命令创建集群

#### 使用statefulset

1. 使用singal模式的storageclass，不需要再另外声明

2. 创建configmap

   ```yaml
   cat 0-redis-cluster-configmap.yaml 
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: redis-cluster-config
     namespace: bjrdc-dev
   data:
     redis.conf: |
       dir /data/redis
       appendonly yes
       cluster-enabled yes
       cluster-config-file /data/nodes.conf 
       cluster-node-timeout 5000 
       port 6379
       bind 0.0.0.0
   ```

   

3. 创建statefulset

   ```yaml
   cat 1-redis-cluster-statefulset.yaml 
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: redis-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: redis-stateful 
     serviceName: redis-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: redis-stateful
       spec:
         containers:
         - name: redis-stateful
           image: redis:6.0.6
           command:
           - bash
           - "-c"
           - |
             set -ex
             nodeconf=/data/nodes.conf
             dir=/data/redis
             if [ ! -d "$dir" ]; then
               mkdir "$dir"
             fi
             if [ -f "$nodeconf" ]; then
               sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" "$nodeconf"
             fi
             exec /usr/local/bin/redis-server /data/redis.conf 
           env:
           - name: POD_IP
             valueFrom:
               fieldRef:
                 fieldPath: status.podIP
           ports:
           - containerPort: 6379
             protocol: TCP
           volumeMounts:
           - name: redis-data
             mountPath: /data
           - name: redis-config
             mountPath: /data/redis.conf
             subPath: redis.conf
         volumes:
           - name: redis-config
             configMap:
               name: redis-cluster-config
     volumeClaimTemplates:
     - metadata:
         name: redis-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-redis
         resources:
           requests:
             storage: 2Gi
   ```

   

4. 创建headless service

   ```yaml
   cat 2-redis-cluster-service.yaml 
   apiVersion: v1
   kind: Service
   metadata:
     name: redis-stateful-headless
     namespace: bjrdc-dev
     labels:
       app: redis-stateful
   spec:
     ports:
     - name: redis-stateful-port
       port: 6379
     clusterIP: None
     selector:
       app: redis-stateful
   ```

   

5. 测试statefulset

   ```sh
   dig redis-stateful-headless.bjrdc-dev.svc.cluster.local
   nslookup redis-stateful-headless.bjrdc-dev.svc.cluster.local 10.96.0.10
   ```

   

6. 创建cluster

   ```sh
   redis-cli --cluster create redis-stateful-0.redis-stateful-headless redis-stateful-1.redis-stateful-headless
   #可惜这个方法执行不下去,原因是redis只能通过ip创建集群。。。
   #Node redis-stateful-1.redis-stateful-headless:6379 replied with error:
   #ERR Invalid node address specified: redis-stateful-0.redis-stateful-headless:6379
   ```

   通过如下方法可以替换当前的cluster的node的ip，集群重启后，ip全部变化的情况下，无法确保所有的node可用。

   ```sh
   kubectl exec -it redis-stateful-0 -n bjrdc-dev -- redis-cli --cluster create $(kubectl get pods -l app=redis-stateful -n bjrdc-dev -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')
   ```

   再通过其他办法来
   
   **通过脚本重建nodes.conf**
   
   
   
7. 查看cluster状态

   ```sh
   kubectl exec -it redis-stateful-0 -n bjrdc-dev -- redis-cli role
   kubectl exec -it redis-stateful-1 -n bjrdc-dev -- redis-cli role
   kubectl exec -it redis-stateful-0 -n bjrdc-dev -- redis-cli cluster info
   ```

8. 创建数据

   创建1000个数据，重启后，重建集群，看看数据是否安在

   ```sh
   for i in {0..1000}; do kubectl exec -it redis-stateful-0 -n bjrdc-dev -- redis-cli -c set $i $i; done
   ```

   ```sh
   kubectl exec -it redis-stateful-1 -n bjrdc-dev -- redis-cli -c get 875
   "875"
   ```

   

9. 重启后数据没了。。。

#### 通过python 修改nodes.conf

1. 转换思路，采用人工修改nodes.conf文件的方式，将nodes.conf文件修改对应的ip后，集群能够正常起来。下一步将使用python或者sh修改nodes.conf文件。

2. 编写python

   ```python
   cat >redis-cluster-boot.py <<EOF
   # coding:utf-8
   # Copyright (C)
   # Author: I
   # Contact: 12157724@qq.com
   import os
   from time import sleep
   
   replicas = 3
   node_file = "/data/nodes.conf"
   data_redis = "/data/redis"
   node_ok = "/data/node_ok"
   
   
   def code(host):
       ip = ""
       while True:
           p = os.popen("nslookup " + host + " 10.96.0.10 |grep Address|grep -v '#53'|awk '{print$2}'")
           result = p.readlines()
           if len(result) == 0:
               print("cannot detect IP for host %s" % host)
               sleep(5)
               continue
           else:
               ip = result[0].replace("\n", "")
               print("detect IP %s,for host %s" % (ip, host))
               break
       return ip
   
   
   def hosts():
       host_temp = "redis-stateful-{}.redis-stateful-headless"
       __list = range(0, replicas)
       return list(map(lambda i: host_temp.format(i), __list))
   
   
   def node_hash(host):
       __hash = ""
       while True:
           result = list(os.popen("redis-cli -h " + host + " cluster nodes|grep myself|awk '{print$1}'"))
           if len(result) == 0:
               print("cannot get hash for host", host)
               sleep(5)
               continue
           else:
               __hash = result[0].replace("\n","")
               break
       print("node_hash for host %s is %s" % (host, __hash))
       return __hash
   
   
   def mk_data_redis():
       if os.path.exists(data_redis) is not True:
           os.mkdir(data_redis)
           print("make dir ", data_redis)
   
   
   def doit():
       mk_data_redis()
       for host in hosts():
           hash = node_hash(host)
           ip = code(host)
           i = os.system(
               "sed -i -e \"/" + hash + "/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/" + ip + "/\" " + node_file)
           print("change nodes.conf result is ", i)
       if os.path.exists(node_ok) is not True:
           open(node_ok,"w")
           print("create node_ok file")
       else:
           print("node_ok is exists")
   
   
   
   if __name__ == '__main__':
       doit()
   EOF    
   ```

3. 编写redis.conf

   ```sh
   cat >redis.conf <<EOF
   dir /data/redis
   appendonly no
   cluster-enabled yes
   cluster-config-file /data/nodes.conf 
   cluster-node-timeout 5000 
   port 6379
   bind 0.0.0.0
   EOF
   ```

   

4. 创建configmap,使用命令行导入文件到configmap中

   ```
   kubectl create configmap redis-cluster-config  -n bjrdc-dev --from-file=redis.conf=./redis.conf --from-file=redis-cluster-boot.py=./redis-cluster-boot.py
   ```
   
5. 创建statefulset

   ```yaml
   cat >1-redis-cluster-statefulset.yaml <<EOF
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: redis-stateful
     namespace: bjrdc-dev
   spec:
     selector:
       matchLabels:
         app: redis-stateful 
     serviceName: redis-stateful-headless
     replicas: 3
     template:
       metadata:
         labels:
           app: redis-stateful
       spec:
         containers:
         - name: ubuntu
           image: bjrdc206.reg/library/ubuntu:18.04.bjrdc-dev
           imagePullPolicy: Always
           command:
           - bash
           - "-c"
           - |
             set ex
             python3  /data/cluster-boot.py
             while true; do echo do...; sleep 10;done
           volumeMounts:
           - name: redis-data
             mountPath: /data
           - name: redis-config
             mountPath: /data/cluster-boot.py
             subPath: redis-cluster-boot.py
         - name: redis-stateful
           image: redis:6.0.6
           command:
           - bash
           - "-c"
           - |
             set -ex 
             /usr/local/bin/redis-server /data/redis.conf --protected-mode no --cluster-announce-ip ${POD_IP} &
             while :
             do
             if [ -f "/data/node_ok" ]; then            
                 rm -rf /data/node_ok
                 kill -9 $(ps -ef|grep redis-server|awk '{print $2}')
                 exec /usr/local/bin/redis-server /data/redis.conf --protected-mode no --cluster-announce-ip ${POD_IP}
                 break
             else
                 echo "node_ok not prepare"
                 sleep 5
             fi
             done 
           env:
           - name: POD_IP
             valueFrom:
               fieldRef:
                 fieldPath: status.podIP
           ports:
           - containerPort: 6379
             protocol: TCP
           volumeMounts:
           - name: redis-data
             mountPath: /data
           - name: redis-config
             mountPath: /data/redis.conf
             subPath: redis.conf
         volumes:
           - name: redis-config
             configMap:
               name: redis-cluster-config
     volumeClaimTemplates:
     - metadata:
         name: redis-data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ceph-storageclass-redis
         resources:
           requests:
             storage: 2Gi
   EOF          
   ```

6. 验证集群状态

   重启redis-statefulset

   ```sh
   kubectl get statefulset redis-stateful -n bjrdc-dev -o yaml|kubectl replace --force -f -
   ```

   ```sh
   kubectl exec -it redis-stateful-0 -n bjrdc-dev -c redis-stateful -- redis-cli cluster info
   cluster_state:ok
   cluster_slots_assigned:16384
   cluster_slots_ok:16384
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:3
   cluster_size:3
   cluster_current_epoch:3
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:148
   cluster_stats_messages_pong_sent:148
   cluster_stats_messages_sent:296
   cluster_stats_messages_ping_received:148
   cluster_stats_messages_pong_received:148
   cluster_stats_messages_received:296
   ```

7. 实验数据是否丢失

   创建数据

   ```sh
   for i in {0..1000}; do kubectl exec -it redis-stateful-0 -n bjrdc-dev -c redis-stateful -- redis-cli -c set $i $i; done
   ```

   获取数据

   ```sh
   redis-cli -h redis-stateful-0.redis-stateful-headless.bjrdc-dev.svc.cluster.local -c  get 100
   "100"
   ```

   重启statefulset

   ```sh
   kubectl get statefulset redis-stateful -n bjrdc-dev -o yaml|kubectl replace --force -f -
   ```

   再次获取数据，一切正常

   ```sh
   redis-cli -h redis-stateful-0.redis-stateful-headless.bjrdc-dev.svc.cluster.local -c  get 100
   ```

## zipkin

> zipkin 可以使用 in kubernetes的模式，直接部署，而不是使用spring的application打包成docker后再启动。

### 简单部署

不设置存储

```yaml
cat 0-zipkin.yaml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zipkin
  namespace: bjrdc-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zipkin
  template:
    metadata:
      labels:
        app: zipkin
    spec:
      containers:
      - name: zipkin
        image: bjrdc206.reg/bjrdc-dev/openzipkin/zipkin
        imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: zipkin
  namespace: bjrdc-dev
spec:
  ports:
  - port: 9411
  selector:
    app: zipkin
```

### 复杂部署

需要配置后端存储

TODO

### 使用

在spring-cloud使用zipkin的时候，只需要在application.yaml中配置zipkin的参数（service或者地址），以及在pom.xml中关联依赖即可。

pom.xml

```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zipkin</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-sleuth</artifactId>
		</dependency>
```

application.yaml

```yaml
spring:
  application:
    name: consumer
  zipkin:
    service:
      name: ${spring.application.name}
    base-url: http://zipkin.bjrdc-dev.svc.cluster.local:9411
```

`但是实地的测试发现问题还是有的，主要是`

1. 无法通过service名称实现zipkin的发现，官方给的配置如下：

   If you want to find Zipkin through service discovery, you can pass the Zipkin’s service ID inside the URL, as shown in the following example for zipkinserver service ID:

   ```
   spring.zipkin.baseUrl: https://zipkinserver/
   ```

2. baseUrl 和base-url，在2.1.2.RELEASE版本上发现，baseUrl竟然不能用，只能用base-url


## CI

> kubernetes 可以为ci提供运行环境，相关配置参考[IGitlab.md](./IGitlab.md)



## The FieldRef Parameter

| **Name**                           | **Description**                                              |
| ---------------------------------- | ------------------------------------------------------------ |
| spec.nodename                      | The name of the node where the Pod is running.               |
| status.hostIP                      | The IP address of the node where the Pod us running.         |
| metadata.name                      | The Pod name (notice that this is different than the container’s name. A Pod may have more than one container) |
| metadata.namespace                 | The namespace of the Pod                                     |
| status.podIP                       | The IP address of the Pod                                    |
| spec.serviceAccountName            | The service account that was used with the Pod.              |
| metadata.uid                       | The UID of the running Pod                                   |
| metadata.labels[*‘label*’]         | The value of the label put on the Pod. For example, if a Pod is labeled env=prod, then metadata.labels[‘env’] returns ‘prod’. |
| metadata.annotations[‘annotation’] | Similar to labels, it gets the value of the specified annotation. |



| **Name**        | **Description**                                              |
| --------------- | ------------------------------------------------------------ |
| requests.cpu    | The amount of CPU specified in the requests field of the Pod definition |
| requests.memory | The amount of memory specified in the requests field of the Pod definition |
| limits.cpu      | The CPU limit of the Pod                                     |
| limits.memory   | The memory limit of the Pod                                  |

使用如下方式获取参数

```yaml
spec:
  containers:
  - image: bash
    name: mycontainer
    command: ['bash','-c','sleep 1000000']
    env:
    - name: MY_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
```



## 启动顺序

> 使用`initcontainer`确保服务的启动顺序。
>
> TODO

## 问题处理

查看日志

   ```sh
   journalctl -f -u kubelet
   ```

### You must be logged in to the server (Unauthorized)

> `颁发证书时/etc/kubernetes/admin.conf文件重新生成，但是`$HOME/.kube/config并没有得到替换。

```sh
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown bjrdc:bjrdc ~/.kube/config
```



### kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"

   现象

   ```sh
   jouralctl -xe --no-page 
   ...
   failed to run Kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
   ```

   问题定位，k8s的cgrou-driver与docker的不同（daemon.json）

   ```sh
   cat /var/lib/kubelet/kubeadm-flags.env
   KUBELET_KUBEADM_ARGS="--cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf"
   ```

   修改为

   ```sh
   KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf"
   ```

### /run/flannel/subnet.env: no such file or directory

   重新apply flannel，或者手动创建该文件（不推荐）

   ```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   ```

### coredns pod crash

   ```sh
kubectl -n kube-system get deployment coredns -o yaml |sed s/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | kubectl apply -f -
   ```

### Back-off restarting failed container

先检查健康检查地址是否正确

### Failed to update endpoint bjrdc-dev/redis-stateful-headless: Operation cannot be fulfilled on endpoints "redis-stateful-headless": the object has been modified; please apply your changes to the latest version and try again



### 集群重启后出现pod无法启动的问题

```
MountVolume.SetUp failed for volume "default-token" : secret "default-token" not found
```

不知道原因，可能是pod启动的时候default-token尚未准备好

简单处理版本是重启pod就可以了

### Spring-cloud

#### io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: GET at: https://10.96.0.1/api/v1/namespaces/bjrdc-dev/endpoints

[参考此处](#service发现)



### DNS 无法找到的问题

现象是在集群的一个node中的所有的pod中都无法ping到和查找到其他的任何service的ip。经实验发现，重启coredns的pod后问题得以解决。*出现问题的那台pod上有coredns的container，怀疑可能和路由有关*.

重启后的cornds的pod在master上运行的。

**暂时不能确定真正的原因，待日后在实验**


## spring-cloud

### kubernetes-maven-plugin

The **kubernetes-maven-plugin** brings your Java applications on to [Kubernetes](http://kubernetes.io/). It provides a tight integration into [Maven](http://maven.apache.org/) and benefits from the build configuration already provided. This plugin focus on two tasks: *Building Docker images* and *creating Kubernetes resource descriptors*. It can be configured very flexibly and supports multiple configuration models for creating: A *Zero-Config* setup allows for a quick ramp-up with some opinionated defaults. For more advanced requirements, an *XML configuration* provides additional configuration options which can be added to the `pom.xml`. For the full power, in order to tune all facets of the creation, external *resource fragments* and *Dockerfiles* can be used.



## 本地开发

> kubernets虽然提供了强大的平台，但是本地开发调试却比较麻烦，就像开发大数据系统一样，需要当前开发主机与所有节点能够通信

### spring-cloud on k8s

 经测试，在使用spring-cloud on k8s的模式下，直接自爱eclipse中执行java程序，可以顺利注册到集群中。

 可以通过eureka的界面上看到本地注册的服务，并且通过本地服务可以调用远程clauster中的服务。

 从原理上说，使用网络打通的方式应该只适合spring-cloud on k8s



### spring-cloud in k8s

 在spring-cloud in k8s的模式下，尚未测试。

 经测试后，竟然可以，你说神奇不申请（*估计spring-cloud-kubernetes是通过域名获取到cluster的api，然后通过api获取的service，以后有机会抓包看看*）通过如下代码竟然可以发现service

 ```java
	@Autowired
 	private DiscoveryClient discoveryClient;
 
 	@GetMapping("/services")
 	public List<String> services() {
 		return this.discoveryClient.getServices();
 	}
 ```


#### pom.xml

需要在pom.xml中配置如下

 ```xml
<dependency>
 <groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-kubernetes-core</artifactId>
 </dependency>
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-kubernetes-discovery</artifactId>
</dependency>
 <dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-kubernetes-ribbon</artifactId>
 </dependency>
 ```

使用如下方法测试

```
curl spring-cloud-k8s-consumer.bjrdc-dev.svc.cluster.local:8096/sc-k8s-consumer/feign/list
```



#### service发现

 ```sh
curl localhost:8086/sc-k8s-consumer/services
 ["hello-node","mysql","mysql-service","spring-cloud-config","spring-cloud-consumer","spring-cloud-dashboard","spring-cloud-eureka","spring-cloud-k8s-provider","spring-cloud-provider","spring-cloud-zipkin","spring-cloud-zuul","kubernetes","ingress-nginx-controller","ingress-nginx-controller-admission","kube-dns","metrics-server","dashboard-metrics-scraper","kubernetes-dashboard"]
 ```

调用provider方法

 ```sh
 curl localhost:8086/sc-k8s-provider/index/list
 ```

 将consumer也部署到k8s后，报如下错误

 ```
 io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: GET at: https://10.96.0.1/api/v1/namespaces/bjrdc-dev/endpoints/spring-cloud-k8s-provider. Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. endpoints "spring-cloud-k8s-provider" is forbidden: User "system:serviceaccount:bjrdc-dev:default" cannot get resource "endpoints" in API group "" in the namespace "bjrdc-dev".
 ```

这是因为权限的问题，应该是调用fabric8(jkube插件用的也是这个客户端)的客户端调用cluster的api的时候，获取不到权限。需要将serviceaccount default 的权限给加上

 ```yaml
 apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRole
 metadata:
 name: bjrdc-cr
 namespace: bjrdc-dev
 rules:
 - apiGroups: [""]
   resources: ["services", "endpoints", "pods"]
   verbs: ["get", "list", "watch"]
 ---
 apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRoleBinding
metadata:
   name: bjrdc-rb
  namespace: bjrdc-dev
 roleRef:
  apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: bjrdc-cr
 subjects:
 - kind: ServiceAccount
  name: default
   namespace: bjrdc-dev
 ```

 

在pod启动的时候，kubernetes会将该pod对应的token和ca.crt namespace 挂载在*/run/secrets/kubernetes.io/serviceaccount*

修改rbac后需要重启pod才能生效

### 使用 telepresence

 telepresence为k8s提供的一个开发客户端-服务端程序，类是vpn，将服务器和客户端打通

 具体使用方法尚未验证

 TODO

## 关于jkube插件

参考 [IGitlab.md](IGitlab.md)

## 其他

### 自定义hosts

kubernetes支持设置hostname，通过如下配置

```yaml
...
spec:
  restartPolicy: Never
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
  containers:
 ...
```



## 官方资料

### 资源

 k8s中有大量的资源对象，比较重要的有service、deployment、pod、namespace、ingress、role、clusterrole、serviceaccount

 

| 类别     | 名称                                                         | 说明 |
| :------- | ------------------------------------------------------------ | ---- |
| 资源对象 | Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling、Node、Namespace、Service、Ingress、Label、CustomResourceDefinition |      |
| 存储对象 | Volume、PersistentVolume、Secret、ConfigMap                  |      |
| 策略对象 | SecurityContext、ResourceQuota、LimitRange                   |      |
| 身份对象 | ServiceAccount、Role、ClusterRole                            |      |

### pod

 Pod 是在 Kubernetes 集群中运行部署应用或服务的最小单元，它是可以支持多容器的。

 Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

 Pod 对多容器的支持是 K8 最基础的设计理念。比如你运行一个操作系统发行版的软件仓库，一个 Nginx 容器用来发布软件，另一个容器专门用来从源仓库做同步，这两个容器的镜像不太可能是一个团队开发的，但是他们一块儿工作才能提供一个微服务；这种情况下，不同的团队各自开发构建自己的容器镜像，在部署的时候组合成一个微服务对外提供服务

### *副本控制器（Replication Controller，RC）*

 RC 是 Kubernetes 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是 1 个；少于指定数目，RC 就会启动运行新的 Pod 副本；多于指定数目，RC 就会杀死多余的 Pod 副本。即使在指定数目为 1 的情况下，通过 RC 运行 Pod 也比直接运行 Pod 更明智，因为 RC 也可以发挥它高可用的能力，保证永远有 1 个 Pod 在运行。

 **RC 是 Kubernetes 较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的 Web 服务**

### 副本集（Replica Set，RS）

 RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。

 副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

### 部署（Deployment）

 部署表示用户对 Kubernetes 集群的一次更新操作。

 部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到 0 的复合操作；这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。

 以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。

### 服务（Service）

 **Service可以看作是一组提供相同服务的Pod对外的访问接口**。借助Service，应用可以方便地实现服务发现和负载均衡。

 RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量，但是没有解决如何访问这些服务的问题。

 **一个 Pod 只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。**

 在 K8 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。

 在 Kubernetes 集群中微服务的负载均衡是由 Kube-proxy 实现的。Kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 Kube-proxy 就越多，高可用节点也随之增多。与之相比，我们平时在服务器端做个反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。



 Service有三种类型：

 - ClusterIP：默认类型，自动分配一个仅cluster内部可以访问的虚拟IP
- NodePort：在ClusterIP基础上为Service在每台机器上绑定一个端口，这样就可以通过`<NodeIP:NodePort`来访问该服务
 - LoadBalancer：在NodePort的基础上，借助cloud provider创建一个外部的负载均衡器，并将请求转发到`<NodeIP:NodePort`

### 任务（Job）

 Job 是 Kubernetes 用来控制批处理型任务的 API 对象。

 批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的 spec.completions 策略而不同：单 Pod 型任务有一个 Pod 成功就标志完成；定数成功型任务保证有 N 个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。

### 后台支撑服务集（DaemonSet）

 长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行；

 而后台支撑型服务的核心关注点在 Kubernetes 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点。

 典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。

### 有状态服务集（StatefulSet）

 Kubernetes 在 1.3 版本里发布了 Alpha 版的 PetSet 功能，在 1.5 版本里将 PetSet 功能升级到了 Beta 版本，并重新命名为 StatefulSet，最终在 1.9 版本里成为正式 GA 版本。在云原生应用的体系里，有下面两组近义词；

 第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。

 RC 和 RS 主要是控制提供无状态服务的，其所控制的 Pod 的名字是随机设置的，一个 Pod 出故障了就被丢弃掉，在另一个地方重启一个新的 Pod，名字变了。名字和启动在哪儿都不重要，重要的只是 Pod 总数；而 StatefulSet 是用来控制有状态服务，StatefulSet 中的每个 Pod 的名字都是事先确定的，不能更改。StatefulSet 中 Pod 的名字的作用，并不是《千与千寻》的人性原因，而是关联与该 Pod 对应的状态。

 对于 RC 和 RS 中的 Pod，一般不挂载存储或者挂载共享存储，保存的是所有 Pod 共享的状态，Pod 像牲畜一样没有分别（这似乎也确实意味着失去了人性特征）；对于 StatefulSet 中的 Pod，每个 Pod 挂载自己独立的存储，如果一个 Pod 出现故障，从其他节点启动一个同样名字的 Pod，要挂载上原来 Pod 的存储继续以它的状态提供服务。

 适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。StatefulSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 StatefulSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

### 集群联邦（Federation）

 Kubernetes 在 1.3 版本里发布了 beta 版的 Federation 功能。在云计算环境中，服务的作用距离范围从近到远一般可以有：同主机（Host，Node）、跨主机同可用区（Available Zone）、跨可用区同地区（Region）、跨地区同服务商（Cloud Service Provider）、跨云平台。Kubernetes 的设计定位是单一集群在同一个地域内，因为同一个地区的网络性能才能满足 Kubernetes 的调度和计算存储连接要求。而联合集群服务就是为提供跨 Region 跨服务商 Kubernetes 集群服务而设计的。

 每个 Kubernetes Federation 有自己的分布式存储、API Server 和 Controller Manager。用户可以通过 Federation 的 API Server 注册该 Federation 的成员 Kubernetes Cluster。当用户通过 Federation 的 API Server 创建、更改 API 对象时，Federation API Server 会在自己所有注册的子 Kubernetes Cluster 都创建一份对应的 API 对象。在提供业务请求服务时，Kubernetes Federation 会先在自己的各个子 Cluster 之间做负载均衡，而对于发送到某个具体 Kubernetes Cluster 的业务请求，会依照这个 Kubernetes Cluster 独立提供服务时一样的调度模式去做 Kubernetes Cluster 内部的负载均衡。而 Cluster 之间的负载均衡是通过域名服务的负载均衡来实现的。

 Federation V1 的设计是尽量不影响 Kubernetes Cluster 现有的工作机制，这样对于每个子 Kubernetes 集群来说，并不需要更外层的有一个 Kubernetes Federation，也就是意味着所有现有的 Kubernetes 代码和机制不需要因为 Federation 功能有任何变化。

 目前正在开发的 Federation V2，在保留现有 Kubernetes API 的同时，会开发新的 Federation 专用的 API 接口，详细内容可以在 [这里](https://github.com/kubernetes/community/tree/master/sig-multicluster) 找到。

### 存储卷（Volume）

 Kubernetes 集群中的存储卷跟 Docker 的存储卷有些类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。Kubernetes 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS，Google 和 Azure 云；支持多种分布式存储包括 GlusterFS 和 Ceph；也支持较容易使用的主机本地目录 emptyDir, hostPath 和 NFS。Kubernetes 还支持使用 Persistent Volume Claim 即 PVC 这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术（例如 AWS，Google 或 GlusterFS 和 Ceph），而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置。

### 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）

 PV 和 PVC 使得 Kubernetes 集群具备了存储的逻辑抽象能力，使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给 PV 的配置者，即集群的管理者。存储的 PV 和 PVC 的这种关系，跟计算的 Node 和 Pod 的关系是非常类似的；PV 和 Node 是资源的提供者，根据集群的基础设施变化而变化，由 Kubernetes 集群管理员配置；而 PVC 和 Pod 是资源的使用者，根据业务服务的需求变化而变化，有 Kubernetes 集群的使用者即服务的管理员来配置。

### 节点（Node）

 Kubernetes 集群中的计算能力由 Node 提供，最初 Node 称为服务节点 Minion，后来改名为 Node。Kubernetes 集群中的 Node 也就等同于 Mesos 集群中的 Slave 节点，是所有 Pod 运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征是上面要运行 kubelet 管理节点上运行的容器。

### 密钥对象（Secret）

 Secret 是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里。在 Kubernetes 集群中配置和使用服务不可避免的要用到各种敏感信息实现登录、认证等功能，例如访问 AWS 存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个 Secret 对象，而在配置文件中通过 Secret 对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

### 用户帐户（User Account）和服务帐户（Service Account）

 顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和 Kubernetes 集群中运行的 Pod 提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的 namespace 无关，所以用户账户是跨 namespace 的

 而服务帐户对应的是一个运行中程序的身份，与特定 namespace 是相关的。

### 命名空间（Namespace）

 命名空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

### RBAC 访问授权

 Kubernetes 在 1.3 版本中发布了 alpha 版的基于角色的访问控制（Role-based Access Control，RBAC）的授权模式。相对于基于属性的访问控制（Attribute-based Access Control，ABAC），RBAC 主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念。在 ABAC 中，Kubernetes 集群中的访问策略只能跟用户直接关联；而在 RBAC 中，访问策略可以跟某个角色关联，具体的用户在跟一个或多个角色相关联。显然，RBAC 像其他新功能一样，每次引入新功能，都会引入新的 API 对象，从而引入新的概念抽象，而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用。



## kubernetes1.20

 1.20版本开始弃用dockershim，个中原因夹杂政治、利益、技术等等，详细可以围观

 [官方解释](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

 [来龙去脉](https://blog.kelu.org/tech/2020/10/09/the-diff-between-docker-containerd-runc-docker-shim.html)

![kubernetes 与 docker](/ICESX/ISunflower/nonecode/images/k8s-1.webp)

### without docker

1. OCI

   Linux基金会于2015年6月成立OCI（Open Container Initiative）组织，旨在围绕容器格式和运行时制定一个开放的工业化标准，目前主要有两个标准文档：容器运行时标准 （runtime spec）和 容器镜像标准（image spec）

2. CRI

   Container Runtime Interface，CRI 是对容器操作的一组抽象，只要每种容器运行时都实现这组接口，kubelet 就能通过这组接口来适配所有的运行时。

3. containerd

   从docker剥离出来的用于对容器进行管理的守护进程，在1.20.x之后将替换docker作为CRI

4. cri-o

   

### 容器基础

1. Cgroup

   控制组提供了一种机制，可以将任务集及其所有将来的子级集合/划分为具有特殊行为的分层组。

2. namespace

   以一种抽象方式包装全局系统资源，使它在命名空间中的进程中看起来像它们具有自己的隔离的全局资源实例。

   Linux Namespace是Linux提供的一种内核级别环境隔离的方法。不知道你是否还记得很早以前的Unix有一个叫chroot的系统调用（通过修改根目录把用户jail到一个特定目录下），chroot提供了一种简单的隔离模式：chroot内部的文件系统无法访问外部的内容。Linux Namespace在此基础上，提供了对UTS、IPC、mount、PID、network、User等的隔离机制。

1. chroot
2. lxc

### 容器运行时

1. containerd
2. CRI-O
3. Docker

### containerd-shim

containerd-shim 是一个真实运行容器的载体，每启动一个容器都会起一个新的 containerd-shim 的一个进程， 它直接通过指定的三个参数：容器 id，boundle 目录（containerd 对应某个容器生成的目录，一般位于：/var/run/docker/libcontainerd/containerID，其中包括了容器配置和标准输入、标准输出、标准错误三个管道文件），运行时二进制（默认为 runC）来调用 runc 的 api 创建一个容器，上面的 docker 进程图中可以直观的显示。其主要作用是：它允许容器运行时(即 runC)在启动容器之后退出，简单说就是不必为每个容器一直运行一个容器运行时(runC)
即使在 containerd 和 dockerd 都挂掉的情况下，容器的标准 IO 和其它的文件描述符也都是可用的
向 containerd 报告容器的退出状态

有了它就可以在不中断容器运行的情况下升级或重启 dockerd，对于生产环境来说意义重大。
运行是二进制（默认为 runc）来调用 runc 的 api 创建一个容器（比如创建容器：最后拼装的命令如下：runc create 。。。。。）

### crictl

> 属于kubernetes

[官方地址](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md）

 *crictl* 是CRI 兼容的容器运行时命令行接口

crictl by default connects on Unix to:

- `unix:///var/run/dockershim.sock` or
- `unix:///run/containerd/containerd.sock` or
- `unix:///run/crio/crio.sock`

#### 配置

```sh
sudo echo "runtime-endpoint: unix:///var/run/containerd/containerd.sock" |sudo tee /etc/crictl.yaml
```

#### 相关命令

```sh
crictl pods
sudo crictl --runtime-endpoint /var/run/containerd/containerd.sock images

```

#### 与ctr

ctr是containerd的命令，crictl是kubernetes的命令，两者镜像是互通的，如果看不到，可能是namespace不一样导致的。

ctictl image = ctr -n=k8s.io image list

```
sudo ctr -n=k8s.io images import coredns1.8.0.tar
```





### 替换docker 为containerd

#### 净安装

v1.20.x之后的版本，不安装docker，按照上文描述的安装节点即可。[node_install](#node_install)

#### 替换

##### master

按照node的方式配置，之后进行init。

注**在init之前先备份文件/etc/kubernetes/**

```sh
sudo kubeadm init --apiserver-advertise-address=172.16.15.17 \
--image-repository=registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16 \
--kubernetes-version=v1.20.2 \
--cri-socket=/run/containerd/containerd.sock \
--ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,DirAvailable--var-lib-etcd,Port-10250
```

如果出现

when I run the command `kubeadm init`, it failed with `[kubelet-check] Initial timeout of 40s passed.`

检查containerd的配置/etc/containerd/config.toml中配置是否正确。

##### node

正常升级kubernetes到v1.20.x版本后，containerd会自动安装，需要通过如下配置来启用containerd

1. 修改containerd配置

   参考上文 containerd 配置

2. 配置kubelet使用containerd.sock

   ```sh
   cat <<EOF > /var/lib/kubelet/kubeadm-flags.env
   KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=/run/containerd/containerd.sock "
   EOF
   ```

3. 关闭docker

   ```
   sudo systemctl stop docker
   ```

   关闭docker后，当前节点上的容器会关闭

4. 重启kubelet

   ```
   sudo systemctl restart kubelet
   ```

   

5. 验证containerd

   ```
   sudo crictl --runtime-endpoint /var/run/containerd/containerd.sock images
   ```

6. 可能出现无法download pause镜像的问题

   `failed to pull image "bjrdc206.reg/gcr/pause:3.1": failed to pull and unpack image "bjrdc206.reg/gcr/pause:3.1":`

   需要安装证书

   ```sh
   sudo cp /home/bjrdc/bjrdc206.reg.crt /usr/local/share/ca-certificates/
   sudo cp  /home/bjrdc/ca.crt /etc/ca-certificates/update.d/
   sudo update-ca-certificates
   ```

   

## 集群管理

### 集群管理

[官方地址](https://kubernetes.io/docs/tasks/administer-cluster/)



### 集群升级

[参考地址](https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

使用操作系统自带的`apt`升级只能升级kubectl kubeadm不能升级组件，所以升级需要进行手动升级。

**如果已经通过操作系统的apt升级后，可以进行降级再进行升级**

#### 升级原理

首先通过apt升级或者降级kubeadm，再通过kubeadm升级其他组件即可。

需要注意

1. 不能跨版本升级，所有，如果跨版本后，需要先降级kubeadm
2. 使用kubeadm升级后，再进行apt升级kubelet和kubectl

#### 升级操作

1. 腾空控制节点

   ```
   kubectl drain bjrdc17 --ignore-daemonsets
   ```

   

2. 升级kubeadm

   ```sh
   sudo apt-cache policy kubeadm|more
   #查看可升级版本
   sudo apt-get install --allow-change-held-packages kubeadm=1.20.2-00
   #升级kubeadm到1.20.2
   ```

3. 查看升级plan

   ```
   sudo kubeadm upgrade plan                                          
   [upgrade/config] Making sure the configuration is correct:
   [upgrade/config] Reading configuration from the cluster...
   [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
   [preflight] Running pre-flight checks.
   [upgrade] Running cluster health checks
   [upgrade] Fetching available versions to upgrade to
   [upgrade/versions] Cluster version: v1.20.0
   [upgrade/versions] kubeadm version: v1.20.2
   [upgrade/versions] Latest stable version: v1.20.2
   [upgrade/versions] Latest stable version: v1.20.2
   [upgrade/versions] Latest version in the v1.20 series: v1.20.2
   [upgrade/versions] Latest version in the v1.20 series: v1.20.2
   
   Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
   COMPONENT   CURRENT       AVAILABLE
   kubelet     3 x v1.20.0   v1.20.2
               1 x v1.20.2   v1.20.2
   
   Upgrade to the latest version in the v1.20 series:
   
   COMPONENT                 CURRENT    AVAILABLE
   kube-apiserver            v1.20.0    v1.20.2
   kube-controller-manager   v1.20.0    v1.20.2
   kube-scheduler            v1.20.0    v1.20.2
   kube-proxy                v1.20.0    v1.20.2
   CoreDNS                   1.7.0      1.7.0
   etcd                      3.4.13-0   3.4.13-0
   
   You can now apply the upgrade by executing the following command:
   
           kubeadm upgrade apply v1.20.2
   
   _____________________________________________________________________
   
   
   The table below shows the current state of component configs as understood by this version of kubeadm.
   Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
   resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
   upgrade to is denoted in the "PREFERRED VERSION" column.
   
   API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
   kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
   kubelet.config.k8s.io     v1beta1           v1beta1             no
   _____________________________________________________________________
   ```

4. 升级到1.20.2

   ```
   sudo kubeadm upgrade apply v1.20.2
   ```

5. 升级kubelet、kubectl

   ```sh
   sudo apt-get install --allow-change-held-packages kubelet=1.20.2-00
   sudo apt-get install --allow-change-held-packages kubectl=1.20.2-00
   ```

6. 去掉控制节点保护

   ```
   kubectl uncordon bjrdc17
   ```

7. 问题处理
   this version of kubeadm only supports deploying clusters with the control plane version >= 1.19.0. Current version: v1.18.15

   先降级到1.18.0，然后在逐步升级。

   Specified version to upgrade to "v1.19.7" is higher than the kubeadm version "v1.19.0". Upgrade kubeadm first using the tool you used to install kubeadm

   需要将kubeadm先使用apt升级再进行集群升级。

   

   [preflight] Some fatal errors occurred:
   [ERROR CoreDNSUnsupportedPlugins]: CoreDNS cannot migrate the following plugins:
   [Option "max_concurrent" in plugin "forward" is unsupported by this migration tool in 1.6.7.]

   ```
   sudo kubeadm upgrade apply v1.19.0 --ignore-preflight-errors=CoreDNSUnsupportedPlugins
   ```

   

## 错误状态

### CrashLoopBackOff

的含义是，Kubernetes试图启动该Pod，但是过程中出现错误，导致容器启动失败或者正在被删除。



## 附件



### kubectl

- [kubectl run（创建容器镜像）](http://docs.kubernetes.org.cn/468.html)
- [kubectl expose（将资源暴露为新的 Service）](http://docs.kubernetes.org.cn/475.html)
- [kubectl annotate（更新资源的Annotations信息）](http://docs.kubernetes.org.cn/477.html)
- [kubectl autoscale（Pod水平自动伸缩）](http://docs.kubernetes.org.cn/486.html)
- [kubectl convert（转换配置文件为不同的API版本）](http://docs.kubernetes.org.cn/488.html)

- [kubectl create（创建一个集群资源对象](http://docs.kubernetes.org.cn/490.html)
- [kubectl create clusterrole（创建ClusterRole）](http://docs.kubernetes.org.cn/492.html)
- [kubectl create clusterrolebinding（为特定的ClusterRole创建ClusterRoleBinding）](http://docs.kubernetes.org.cn/494.html)
- [kubectl create configmap（创建configmap）](http://docs.kubernetes.org.cn/533.html)
- [kubectl create deployment（创建deployment）](http://docs.kubernetes.org.cn/535.html)
- [kubectl create namespace（创建namespace）](http://docs.kubernetes.org.cn/537.html)
- [kubectl create poddisruptionbudget（创建poddisruptionbudget）](http://docs.kubernetes.org.cn/539.html)
- [kubectl create quota（创建resourcequota）](http://docs.kubernetes.org.cn/541.html)
- [kubectl create role（创建role）](http://docs.kubernetes.org.cn/543.html)
- [kubectl create rolebinding（为特定Role或ClusterRole创建RoleBinding）](http://docs.kubernetes.org.cn/545.html)

- [kubectl create service（使用指定的子命令创建 Service服务）](http://docs.kubernetes.org.cn/564.html)
- [kubectl create service clusterip](http://docs.kubernetes.org.cn/566.html)
- [kubectl create service externalname](http://docs.kubernetes.org.cn/568.html)
- [kubectl create service loadbalancer](http://docs.kubernetes.org.cn/570.html)
- [kubectl create service nodeport](http://docs.kubernetes.org.cn/572.html)
- [kubectl create serviceaccount](http://docs.kubernetes.org.cn/574.html)

- [kubectl create secret（使用指定的子命令创建 secret）](http://docs.kubernetes.org.cn/548.html)
- [kubectl create secret tls](http://docs.kubernetes.org.cn/558.html)
- [kubectl create secret generic](http://docs.kubernetes.org.cn/556.html)
- [kubectl create secret docker-registry](http://docs.kubernetes.org.cn/554.html)

- [kubectl delete（删除资源对象）](http://docs.kubernetes.org.cn/618.html)
- [kubectl edit（编辑服务器上定义的资源对象）](http://docs.kubernetes.org.cn/623.html)
- [kubectl get（获取资源信息）](http://docs.kubernetes.org.cn/626.html)
- [kubectl label（更新资源对象的label）](http://docs.kubernetes.org.cn/628.html)
- [kubectl patch（使用patch更新资源对象字段）](http://docs.kubernetes.org.cn/632.html)
- [kubectl replace（替换资源对象）](http://docs.kubernetes.org.cn/635.html)
- [kubectl rolling-update（使用RC进行滚动更新）](http://docs.kubernetes.org.cn/638.html)
- [kubectl scale（扩缩Pod数量）](http://docs.kubernetes.org.cn/664.html)

- [kubectl rollout（对资源对象进行管理）](http://docs.kubernetes.org.cn/643.html)
- [kubectl rollout history（查看历史版本）](http://docs.kubernetes.org.cn/645.html)
- [kubectl rollout pause（标记资源对象为暂停状态）](http://docs.kubernetes.org.cn/647.html)
- [kubectl rollout resume（恢复已暂停资源）](http://docs.kubernetes.org.cn/650.html)
- [kubectl rollout status（查看资源状态）](http://docs.kubernetes.org.cn/652.html)
- [kubectl rollout undo（回滚版本）](http://docs.kubernetes.org.cn/654.html)

- [kubectl set（配置应用资源）](http://docs.kubernetes.org.cn/669.html)
- [kubectl set resources（指定Pod的计算资源需求）](http://docs.kubernetes.org.cn/671.html)
- [kubectl set selector（设置资源对象selector）](http://docs.kubernetes.org.cn/672.html)
- [kubectl set image（更新已有资源对象中的容器镜像）](http://docs.kubernetes.org.cn/670.html)
- [kubectl set subject（更新RoleBinding / ClusterRoleBinding中User、Group 或 ServiceAccount）](http://docs.kubernetes.org.cn/681.html)

