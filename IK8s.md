IK8S
=====

### 架构

![架构图](https://jimmysong.io/kubernetes-handbook/images/architecture.png)

- etcd保存了整个集群的状态；
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）的管理；
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
- kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；

![架构图](https://jimmysong.io/kubernetes-handbook/images/kubernetes-high-level-component-archtecture.jpg)
## install

1. 准备工作

   1. 确保mac和uuid唯一

   您可以使用命令 `ip link` 或 `ifconfig -a` 来获取网络接口的 MAC 地址

   ```
   sudo cat /sys/class/dmi/id/product_uuid
   ```

   2. 在 Linux 中，nftables 当前可以作为内核 iptables 子系统的替代品。 `iptables` 工具可以充当兼容性层，其行为类似于 iptables 但实际上是在配置 nftables。 nftables 后端与当前的 kubeadm 软件包不兼容：它会导致重复防火墙规则并破坏 `kube-proxy`。

      如果您系统的 `iptables` 工具使用 nftables 后端，则需要把 `iptables` 工具切换到“旧版”模式来避免这些问题。 默认情况下，至少在 Debian 10 (Buster)、Ubuntu 19.04、Fedora 29 和较新的发行版本中会出现这种问题。RHEL 8 不支持切换到旧版本模式，因此与当前的 kubeadm 软件包不兼容。

   ```bash
   update-alternatives --set iptables /usr/sbin/iptables-legacy
   update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
   update-alternatives --set arptables /usr/sbin/arptables-legacy
   update-alternatives --set ebtables /usr/sbin/ebtables-legacy
   ```

2. 安装官方教程

```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

2. 手动

   官方教程中无法访问google，需要手动安装
	1. 本地下载gpg
   ```
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add 
   ```
	2. 增加source 
   cat <<EOF | sudo  /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
	3. 安装on ipv6
   ```
   sudo apt install miredo
   sudo service miredo start
   sudo apt update
   sudo apt-get install -y kubelet kubeadm kubectl
   ```

3. 或者切换aliyun

   ```
   sudo su root
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
   # 添加 k8s 镜像源
   sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   ```
   
### 镜像搜索

[https://cr.console.aliyun.com/cn-hangzhou/instances/images](https://links.jianshu.com/go?to=https%3A%2F%2Fcr.console.aliyun.com%2Fcn-hangzhou%2Finstances%2Fimages)

## 配置

### master

1. disable swap
   
```
sudo vi /etc/fstab
#/dev/mapper/fw--vg-swap_1 none            swap    sw              0       0
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
   ```
   
   3. enable docker

      sudo apt install docker.io
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


​      

   4. 初始化

      ```
      kubeadm init \
      --apiserver-advertise-address=172.16.15.17 \
      --image-repository registry.aliyuncs.com/google_containers \
      --pod-network-cidr=10.244.0.0/16 \
      --kubernetes-version=v1.18.0
      #如果无法下载，需要设置--kubernetes-version为当然registry服务器上有的版本
      ```

      ```
      kubectl get nodes
      ```

      

      执行完成后，回有内容，在slaver节点执行。

      kubeadm join 172.16.15.17:6443 --token 1cx9wb.3bkuu2eq5qh8vn9k  --discovery-token-ca-cert-hash sha256:f680fe2f1575db37e653e2879ded96efc40e5104acd69aa66947b350ec3d35ce

      

   5. .kube

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



   2. 部署 flannel 网络

      > Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务；简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址，并使Docker容器可以互连。

      ```
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
      ```

      *但是这个网络是做什么的呢？*

   3. 查看pod

```
kubectl get pod --all-namespaces
```

9. 安装dashborad

```
wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
kubectl create -f recommended.yaml 
```



#### Node

1. 环境准备

> disable swap
>
> ```
> sudo vi /etc/fstab
> #/dev/mapper/fw--vg-swap_1 none            swap    sw              0       0
> ```
>
> sysctl
>
> ```
> cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
> sudo sysctl --system
> ```
>
> docker
>
> ```
> sudo apt install -y docker.io
> systemctl enable docker.service
> cat > /etc/docker/daemon.json <<EOF
> {
>   "graph": "/docker",
>   "exec-opts": ["native.cgroupdriver=systemd"],
>   "log-driver": "json-file",
>   "log-opts": {
>     "max-size": "100m"
>   },
>   "storage-driver": "overlay2"
> }
> EOF
> ```
>
> 

1. 安装kubctl

   ```
   sudo su root
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
   # 添加 k8s 镜像源
   sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   sudo apt update
   sudo apt-get install -y  kubectl kubeadm
   ```

2. join

   1. on master

   ```
   kubeadm token list
   kubeadm token create --print-join-command
   kubeadm join 172.16.15.17:6443 --token h81gdw.duityezgzrxsl4g7     --discovery-token-ca-cert-hash sha256:18f9acf00a214334c0a8d284e5808a9eec346bfe99bee6b9ebb5b016c9d6ca1f
   ```

   2. on node

   ```
   kubeadm join 172.16.15.17:6443 --token h81gdw.duityezgzrxsl4g7     --discovery-token-ca-cert-hash sha256:18f9acf00a214334c0a8d284e5808a9eec346bfe99bee6b9ebb5b016c9d6ca1f
   ```


### dashborader

1. yml

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

2. install

````
kubectl apply -f recommended.yaml
````

3. 访问

   forbidden

```
# 生成client-certificate-data
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt

# 生成client-key-data
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key

# 生成p12
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"

```

> 下载生成的kubecfg.p12文件，并导入浏览器
>
> 使用浏览器打开
>
> https://bjrdc17:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

4. 绑定角色

> 可以使用token登录，但是此时还有权限的问题，无法访问资源，还需要绑定角色
>
> 创建角色admin-user.rbac.yaml
>
> ```
> apiVersion: v1
> kind: ServiceAccount
> metadata:
>   name: admin-user
>   namespace: kube-system
> ---
> # Create ClusterRoleBinding
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: admin-user
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: cluster-admin
> subjects:
> - kind: ServiceAccount
>   name: admin-user
>   namespace: kube-system
> ```
>
> ```
> kubectl  create -f admin-user.rbac.yaml 
> ```
>
> 查看token
>
> ```
> kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
> 
> ')
> Name:         admin-user-token-w4knf
> Namespace:    kube-system
> Labels:       <none>
> Annotations:  kubernetes.io/service-account.name: admin-user
>               kubernetes.io/service-account.uid: 65323ead-467f-448d-b7ee-1c52a002f3c2
> 
> Type:  kubernetes.io/service-account-token
> 
> Data
> ====
> token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InhZbkI0S001RXlYbXV5UHgwZVBKYzBYMUFUQnF2NFhGUW1iLTlRNW45ZFkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXc0a25mIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2NTMyM2VhZC00NjdmLTQ0OGQtYjdlZS0xYzUyYTAwMmYzYzIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.g4zufIj7ZUUrv9BgtPCd6djE5z7APV6bhE_OchKzczULdbuSkMBrLWwwbHm-0Jg5cUN37fTS-lFsMPxrt2Uw2_m0omx7N47qU-3LBdYxAwiBS-OBUDq6qfyWZoYsQizqdAf1y9kaxUZNbQ1iRMFqyH9-xgp-gk2rbixlOr0ToCOiDC0_FNjJ9bRnhjzQVCXoKQ0XefLuEv21AqeOpaN0U0lP8txziRIOI83grhtbF4RqDHxF0ZoiIakJ5KhKozff29am9lUYScNJpNc6ooqU2wvoNgXHeyODWohXOi9Q1cFPETpA_6kjKYxwpcqsMfJ85lTVPMOCadLV4YJq_h4Kfg
> ca.crt:     1025 bytes
> namespace:  11 bytes
> ```
>
> 使用此token登录dashboard

### heapster

> Heapster was initially [deprecated](https://github.com/kubernetes-retired/heapster/blob/master/docs/deprecation.md) in 1.11; users were encouraged to move to the `metrics-server` for similar functionality. With 1.18, the `cluster-monitoring` addons (Heapster, InfluxDB, and Grafana) have been removed from the Kubernetes source tree and therefore removed from the `cdk-addons` snap as well. Customers relying on these addons should migrate to a `metrics-server` solution prior to upgrading. Note: these removals do not affect the Kubernetes Dashboard nor the methods described in

### metrics-server

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

### Ingress

> An API object that manages external access to the services in a cluster, typically HTTP.
>
> Ingress may provide load balancing, SSL termination and name-based virtual hosting.

> 一般在生产环境中使用。推荐使用ingress。
>
> Load balancer的问题是每一个服务都要有一个Load balancer，服务多了之后会很麻烦，这时就会用Ingress，它的缺点是配置起来比较复杂。

> 默认安装中并没有ingress的controller，需要另行安装。

#### 安装Ingress

1. 下载yaml，不需要进行修改可以直接使用

   ```
   wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
   
   ```

2. 准备image

   由于默认使用的镜像是`quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0`这个镜像在k8s中无法下载，但是不知道为何在docker中可以。

   ```
   docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0
   docker tag quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0 bjrdc206:443/bjrdc-dev/nginx-ingress-controller:0.33.0
   docker login bjrdc206:443
   sudo docker push bjrdc206:443/bjrdc-dev/nginx-ingress-controller:0.33.0
   ```

   修改deploy.yaml 将其中的`quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0`替换为`bjrdc206:443/bjrdc-dev/nginx-ingress-controller:0.33.0`

3. 安装ingress-nginx

   ```
   kubectl create -f deploy.yaml
   ```

4. 配置ingress

   安装了ingrss后，需要为服务配置ingress，假设已将安装了hello-node的服务，使用如下yaml，为该服务增加ingress

   ```
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
         - path: /
           backend:
             serviceName: hello-node
             servicePort: 3000
   EOF
   ```

5. 验证

   ```
   sudo sh -c "echo 172.16.10.17 ingress.bjrdc17 >> /etc/hosts"
   ```

   查看ingress端口

   ```
   kubectl get service -n ingress-nginx        
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
   ingress-nginx-controller             LoadBalancer   10.103.204.188   <pending>     80:32628/TCP,443:31687/TCP   3h50m
   ingress-nginx-controller-admission   ClusterIP      10.101.7.151     <none>        443/TCP 
   ```

   访问

   ```
   curl ingress.bjrdc17:32628
   ```

6. 配置固定端口

   将ingress的service的type设置为NodePort即可

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

7. **Ingress**的理解

   > ingress其实也是一个service，当然可以配置为loadbanlace或者nodeport两种方式。
   >
   > ingress的意义在于被ingress的service可以设置为clusterip，这样就省去了大连的nodeport，而只需要一个配置给ingress的nodeport。
   >
   > 理论上讲，可以在集群外部搭建一个nginx作为loadbanlce为所有的ingress的nodeip作负载均衡。


## IP与网络 

service地址和pod地址在不同网段，service地址为虚拟地址，不配在pod上或主机上，外部访问时，先到Node节点网络，再转到service网络，最后代理给pod网络。

### 服务暴露（expose）

有三种方式暴露服务，NodePort,Loadbanlace,ingress

> ClusterIP 服务是 Kubernetes 的默认服务。它给你一个集群内的服务，集群内的其它应用都可以访问该服务。集群外部无法访问它.
>
> 开启clusterIP后必须使用loadbanlace或者ingress来实现服务的透传
>
> ```
> apiVersion: v1
> kind: Service
> metadata:  
>   name: my-internal-service
> selector:    
>   app: my-app
> spec:
>   type: ClusterIP
> ```



> NodePort 服务是引导外部流量到你的服务的最原始方式。NodePort，正如这个名字所示，在所有节点（虚拟机）上开放一个特定端口，任何发送到该端口的流量都被转发到对应服务。
>
> **开启NodePort后，可以通过任何一个NodeIP和nodeport来访问服务**
>
> ```
> apiVersion: v1
> kind: Service
> metadata:  
>   name: my-nodeport-service
> selector:    
>   app: my-app
> spec:
>   type: NodePort
>   ports:  
>   - name: http
>     port: 80
>     targetPort: 80
>     nodePort: 30036
>     protocol: TCP
> ```



> Ingress
>
> 参见上文



> Loadbanlace
>
> 

### Node IP

可以是物理机的IP（也可能是虚拟机IP）。

每个Service都会在Node节点上开通一个端口，外部可以通过NodeIP:NodePort即可访问Service里的Pod,和我们访问服务器部署的项目一样，IP:端口/项目名

```
kubectl describe node nodeName
```



### Pod IP
Pod IP是每个Pod的IP地址，他是Docker Engine根据docker网桥的IP地址段进行分配的，通常是一个虚拟的二层网络

同Service下的pod可以直接根据PodIP相互通信，不同Service下的pod在集群间pod通信要借助于 cluster ip
pod和集群外通信，要借助于node ip

```
kubectl get pods
kubectl describe pod podName
```



### Cluster IP

Service的IP地址，此为虚拟IP地址。外部网络无法ping通，只有kubernetes集群内部访问使用。

在kubernetes查询Cluster IP

```
kubectl -n 命名空间 get Service即可看到ClusterIP
```

### 三种IP网络间的通信

service地址和pod地址在不同网段，service地址为虚拟地址，不配在pod上或主机上，外部访问时，先到Node节点网络，再转到service网络，最后代理给pod网络。



## DNS

> This tells dnsmasq that queries for anything in the `cluster.local` domain should be forwarded to the DNS server at 10.96.0.10. This happens to be the default IP address of the `kube-dns` service in the `kube-system` namespace. If your cluster’s DNS service has a different IP address, you’ll need to specify it instead

>kubernetes安装的时候，会自动的安装一个kube-dns的服务，该服务用于对service设置域名（因为service的clusterIp是会变化的，在service重启，或者故障的时候），建议使用域名进行service的访问，域名的格式如下

> \${servicename}.\${namespace}.svc.cluster.local

```
ping hello-node.bjrdc-dev.svc.cluster.local
ping mysql.bjrdc-dev.svc.cluster.local
```



## 常用命令

### kubeadm

```
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

```
kubeadm token create
kubeadm token list
kubeadm token create --print-join-command
```
### kubectl

kubectl   [get|describe|delete]  [node(s)|pod(s)|service(s)|role(s)|namespace(s)|cs]



```
kubectl cluster-info
```

```
kubectl create -f xx.yaml
```



```
kubectl get nodes
kubectl get namespace
kubectl get pod
kubectl get pods --all-namespaces
kubectl get cs
kubectl get svc
kubectl get all --all-namespaces
kubectl get roles --all-namespaces
kubectl get service --all-namespaces
kubectl get serivce xxx --url
kubectl get --raw "/"
kubectl get events
kubectl get secret regcred --output=yaml
kubectl get deployment metrics-server -n kube-system --output=yaml
```

```
kubectl describe namespaces kube-system
kubectl describe pods monitoring-influxdb-7f474cc79-cslt5 -n kube-system
kubectl describe clusterrole system:heapster
kubectl describe node bjrdc81
```

```
kubectl logs pod xxxx -n kube-system -f
```

```
kubectl label node/10.47.136.60 role=entry
```

```
kubectl delete -f recommended.yaml
kubectl delete clusterrole system:heapster
kubectl delete pod hello-node
```

```
kubectl config view
```

```
kubectl edit deployment kubernetes-hello-world
```



```
kubectl run hello-node --image=hello-node:v1 --port=3000
```



## hardor

>Harbor 是 Vmwar 公司开源的 企业级的 Docker Registry 管理项目
>
>它主要 提供 Dcoker Registry 管理UI，可基于角色访问控制, AD/LDAP 集成，日志审核等功能，完全的支持中文。

### 安装

Generate a CA certificate private key

```
openssl genrsa  -out ca.key 4096
```
Generate the CA certificate.
```
openssl req -x509 -new -nodes -sha512 -days 3650  -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=bjrdc206.reg"  -key ca.key -out ca.crt
```
Generate a Server Certificate
```
penssl genrsa -out bjrdc206.key 4096

```
Generate a certificate signing request (CSR).
```
openssl req -sha512 -new     -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=bjrdc206.reg"     -key bjrdc206.reg.key -out bjrdc206.reg.csr
```

> I had the same issue as you on Ubuntu 18.04.x. Removing (or commenting out) `RANDFILE = $ENV::HOME/.rnd` from `/etc/ssl/openssl.cnf` worked for me.

Generate an x509 v3 extension file

```
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

Use the `v3.ext` file to generate a certificate for your Harbor host.

```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in bjrdc206.reg.csr \
    -out bjrdc206.reg.crt
```

Copy the server certificate and key into the certficates folder on your Harbor host

```
cp bjrdc206.reg.crt /docker/cert/
cp bjrdc206.reg.key /docker/cert/
```

Convert `yourdomain.com.crt` to `yourdomain.com.cert`, for use by Docker.

```
openssl x509 -inform PEM -in bjrdc206.reg.crt -out bjrdc206.reg.cert
```

Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host. You must create the appropriate folders first

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



```
systemctl enable docker.service
systemctl restart docker
```

install docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```



vi harbor.yml

```
hostname: bjrdc206

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



```
./install.sh
```

### push

修改docker配置，让https生效

```
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

```
 docker login bjrdc206.reg
 docker tag hello-world bjrdc206:443/bjrdc-dev/hello-world:v1.0.0
 sudo docker push bjrdc206.reg/bjrdc-dev/hello-world:v1.0.0
```

### register

为harbo增加远程源

在harbor的管理界面中，的“registries”中增加`https://registry-1.docker.io`

### 重启

```
sudo docker-compose down
sudo docker-compose up -d
```



## YAML

### namespace

```
apiVersion: v1
kind: Namespace
metadata:
   name: bjrdc-dev
   labels:
     name: bjrdc-dev
```



### image

```
cat >Dockerfile <<EOF
FROM node:8.10.0
EXPOSE 8080
COPY server.js .
CMD [ "node", "server.js" ]
EOF
```



```
sudo docker build -t hello-node:v1 .
```



```
sudo docker tag hello-node:v1 bjrdc206:443/bjrdc-dev/hello-node:v1.0.0
sudo docker push bjrdc206:443/bjrdc-dev/hello-node:v1.0.0
```



### deployment



```
cat >deloyment<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-node
  namespace: bjrdc-dev 
spec:
  selector:
    matchLabels:
      app: hello-node
  replicas: 2
  template:
    metadata:
      labels:
        app: hello-node
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



```
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
    app: hello-node
EOF
```

```
kubectl create -f service.yaml
```

```
kubectl get services -n bjrdc-dev
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
hello-node   ClusterIP   10.102.118.239   <none>        3000/TCP   39m

curl 10.102.118.239:3000
```



#### targetPort port 

**注意：** 需要注意的是， `Service` 能够将一个接收 `port` 映射到任意的 `targetPort`。 默认情况下，`targetPort` 将被设置为与 `port` 字段相同的值。

targetPort:pod 的服务端口

port：service将pod的端口映射为集群的端口。

```
kubectl get services --all-namespaces
NAMESPACE              NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
bjrdc-dev              hello-node                  ClusterIP   10.102.118.239   <none>        3000/TCP                 12m
curl 10.102.118.239:3000
```



#### apiversion

> Deployment
> 1.6版本之前 apiVsersion：extensions/v1beta1
>
> 1.6版本到1.9版本之间：apps/v1beta1
>
> 1.9版本之后:apps/v1

> 1. v1 
>
> Kubernetes API的稳定版本，包含很多核心对象：pod、service等



> 2. app/v1 
>
> ​	在kubernetes1.9版本中，引入apps/v1，deployment等资源从extensions/v1beta1, apps/v1beta1 和 apps/v1beta2迁入apps/v1，原来的v1beta1等被废弃。



> apps/v1代表：包含一些通用的应用层的api组合，如：Deployments, RollingUpdates, and ReplicaSets

#### Label



## 应用

### spring-cloud



### node.js



## 问题处理

```
netstat -lntup|grep kube-sche
kubectl  get pods -n kube-system
```

```
journalctl -f -u kubelet
```



## 基本概念

### pod

> Pod 是在 Kubernetes 集群中运行部署应用或服务的最小单元，它是可以支持多容器的。
>
> Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。
>
> Pod 对多容器的支持是 K8 最基础的设计理念。比如你运行一个操作系统发行版的软件仓库，一个 Nginx 容器用来发布软件，另一个容器专门用来从源仓库做同步，这两个容器的镜像不太可能是一个团队开发的，但是他们一块儿工作才能提供一个微服务；这种情况下，不同的团队各自开发构建自己的容器镜像，在部署的时候组合成一个微服务对外提供服务

### *副本控制器（Replication Controller，RC）*

> RC 是 Kubernetes 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是 1 个；少于指定数目，RC 就会启动运行新的 Pod 副本；多于指定数目，RC 就会杀死多余的 Pod 副本。即使在指定数目为 1 的情况下，通过 RC 运行 Pod 也比直接运行 Pod 更明智，因为 RC 也可以发挥它高可用的能力，保证永远有 1 个 Pod 在运行。
>
> **RC 是 Kubernetes 较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的 Web 服务**

### 副本集（Replica Set，RS）

> RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。
>
> 副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

### 部署（Deployment）

> 部署表示用户对 Kubernetes 集群的一次更新操作。
>
> 部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到 0 的复合操作；这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。
>
> 以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。

### 服务（Service）

> **Service可以看作是一组提供相同服务的Pod对外的访问接口**。借助Service，应用可以方便地实现服务发现和负载均衡。
>
> RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量，但是没有解决如何访问这些服务的问题。
>
> **一个 Pod 只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。**
>
> 在 K8 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。
>
> 在 Kubernetes 集群中微服务的负载均衡是由 Kube-proxy 实现的。Kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 Kube-proxy 就越多，高可用节点也随之增多。与之相比，我们平时在服务器端做个反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。



> Service有三种类型：
>
> - ClusterIP：默认类型，自动分配一个仅cluster内部可以访问的虚拟IP
>- NodePort：在ClusterIP基础上为Service在每台机器上绑定一个端口，这样就可以通过`<NodeIP>:NodePort`来访问该服务
> - LoadBalancer：在NodePort的基础上，借助cloud provider创建一个外部的负载均衡器，并将请求转发到`<NodeIP>:NodePort`

### 任务（Job）

> Job 是 Kubernetes 用来控制批处理型任务的 API 对象。
>
> 批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的 spec.completions 策略而不同：单 Pod 型任务有一个 Pod 成功就标志完成；定数成功型任务保证有 N 个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。

### 后台支撑服务集（DaemonSet）

> 长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行；
>
> 而后台支撑型服务的核心关注点在 Kubernetes 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点。
>
> 典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。

### 有状态服务集（StatefulSet）

> Kubernetes 在 1.3 版本里发布了 Alpha 版的 PetSet 功能，在 1.5 版本里将 PetSet 功能升级到了 Beta 版本，并重新命名为 StatefulSet，最终在 1.9 版本里成为正式 GA 版本。在云原生应用的体系里，有下面两组近义词；第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。RC 和 RS 主要是控制提供无状态服务的，其所控制的 Pod 的名字是随机设置的，一个 Pod 出故障了就被丢弃掉，在另一个地方重启一个新的 Pod，名字变了。名字和启动在哪儿都不重要，重要的只是 Pod 总数；而 StatefulSet 是用来控制有状态服务，StatefulSet 中的每个 Pod 的名字都是事先确定的，不能更改。StatefulSet 中 Pod 的名字的作用，并不是《千与千寻》的人性原因，而是关联与该 Pod 对应的状态。

> 对于 RC 和 RS 中的 Pod，一般不挂载存储或者挂载共享存储，保存的是所有 Pod 共享的状态，Pod 像牲畜一样没有分别（这似乎也确实意味着失去了人性特征）；对于 StatefulSet 中的 Pod，每个 Pod 挂载自己独立的存储，如果一个 Pod 出现故障，从其他节点启动一个同样名字的 Pod，要挂载上原来 Pod 的存储继续以它的状态提供服务。

> 适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。StatefulSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 StatefulSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

### 集群联邦（Federation）

> Kubernetes 在 1.3 版本里发布了 beta 版的 Federation 功能。在云计算环境中，服务的作用距离范围从近到远一般可以有：同主机（Host，Node）、跨主机同可用区（Available Zone）、跨可用区同地区（Region）、跨地区同服务商（Cloud Service Provider）、跨云平台。Kubernetes 的设计定位是单一集群在同一个地域内，因为同一个地区的网络性能才能满足 Kubernetes 的调度和计算存储连接要求。而联合集群服务就是为提供跨 Region 跨服务商 Kubernetes 集群服务而设计的。

> 每个 Kubernetes Federation 有自己的分布式存储、API Server 和 Controller Manager。用户可以通过 Federation 的 API Server 注册该 Federation 的成员 Kubernetes Cluster。当用户通过 Federation 的 API Server 创建、更改 API 对象时，Federation API Server 会在自己所有注册的子 Kubernetes Cluster 都创建一份对应的 API 对象。在提供业务请求服务时，Kubernetes Federation 会先在自己的各个子 Cluster 之间做负载均衡，而对于发送到某个具体 Kubernetes Cluster 的业务请求，会依照这个 Kubernetes Cluster 独立提供服务时一样的调度模式去做 Kubernetes Cluster 内部的负载均衡。而 Cluster 之间的负载均衡是通过域名服务的负载均衡来实现的。

> Federation V1 的设计是尽量不影响 Kubernetes Cluster 现有的工作机制，这样对于每个子 Kubernetes 集群来说，并不需要更外层的有一个 Kubernetes Federation，也就是意味着所有现有的 Kubernetes 代码和机制不需要因为 Federation 功能有任何变化。

> 目前正在开发的 Federation V2，在保留现有 Kubernetes API 的同时，会开发新的 Federation 专用的 API 接口，详细内容可以在 [这里](https://github.com/kubernetes/community/tree/master/sig-multicluster) 找到。

### 存储卷（Volume）

> Kubernetes 集群中的存储卷跟 Docker 的存储卷有些类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。Kubernetes 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS，Google 和 Azure 云；支持多种分布式存储包括 GlusterFS 和 Ceph；也支持较容易使用的主机本地目录 emptyDir, hostPath 和 NFS。Kubernetes 还支持使用 Persistent Volume Claim 即 PVC 这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术（例如 AWS，Google 或 GlusterFS 和 Ceph），而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置。

### 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）

> PV 和 PVC 使得 Kubernetes 集群具备了存储的逻辑抽象能力，使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给 PV 的配置者，即集群的管理者。存储的 PV 和 PVC 的这种关系，跟计算的 Node 和 Pod 的关系是非常类似的；PV 和 Node 是资源的提供者，根据集群的基础设施变化而变化，由 Kubernetes 集群管理员配置；而 PVC 和 Pod 是资源的使用者，根据业务服务的需求变化而变化，有 Kubernetes 集群的使用者即服务的管理员来配置。

### 节点（Node）

> Kubernetes 集群中的计算能力由 Node 提供，最初 Node 称为服务节点 Minion，后来改名为 Node。Kubernetes 集群中的 Node 也就等同于 Mesos 集群中的 Slave 节点，是所有 Pod 运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征是上面要运行 kubelet 管理节点上运行的容器。

### 密钥对象（Secret）

> Secret 是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里。在 Kubernetes 集群中配置和使用服务不可避免的要用到各种敏感信息实现登录、认证等功能，例如访问 AWS 存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个 Secret 对象，而在配置文件中通过 Secret 对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

### 用户帐户（User Account）和服务帐户（Service Account）

> 顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和 Kubernetes 集群中运行的 Pod 提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的 namespace 无关，所以用户账户是跨 namespace 的；而服务帐户对应的是一个运行中程序的身份，与特定 namespace 是相关的。

### 命名空间（Namespace）

> 命名空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

### RBAC 访问授权

> Kubernetes 在 1.3 版本中发布了 alpha 版的基于角色的访问控制（Role-based Access Control，RBAC）的授权模式。相对于基于属性的访问控制（Attribute-based Access Control，ABAC），RBAC 主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念。在 ABAC 中，Kubernetes 集群中的访问策略只能跟用户直接关联；而在 RBAC 中，访问策略可以跟某个角色关联，具体的用户在跟一个或多个角色相关联。显然，RBAC 像其他新功能一样，每次引入新功能，都会引入新的 API 对象，从而引入新的概念抽象，而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用。

## 云原生

这里我们抛出一个我们自己的理解：云原生代表着原生为云设计。详细的解释是：应用原生被设计为在云上以最佳方式运行，充分发挥云的优势。



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

