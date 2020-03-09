IK8S
=====

## install

1. 安装官方教程

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
```
   

   
## 配置
   
### master
   
1. disable swap
   
```
      sudo vi /etc/fstab
      #/dev/mapper/fw--vg-swap_1 none            swap    sw              0       0
      ```

   


​      
​      
   2. enable docker
   
      ```
      systemctl enable docker.service
      ```
   
      
   
   3. docker 配置
   
      detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
   
      ```
      cat > /etc/docker/daemon.json <<EOF
      {
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
          "max-size": "100m"
        },
        "storage-driver": "overlay2"
      }
      EOF
      ```
   
      
   
   4. 初始化
   
      ```
      kubeadm init \
      --apiserver-advertise-address=172.16.15.17 \
      --image-repository registry.aliyuncs.com/google_containers \
      --pod-network-cidr=10.244.0.0/16
      ```
   
      执行完成后，回有内容，在slaver节点执行。
   
      kubeadm join 172.16.15.17:6443 --token 1cx9wb.3bkuu2eq5qh8vn9k  --discovery-token-ca-cert-hash sha256:f680fe2f1575db37e653e2879ded96efc40e5104acd69aa66947b350ec3d35ce
   
   5. 部署 flannel 网络

      ```
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
      ```

      *但是这个网络是做什么的呢？*

      

# Creating a single control-plane cluster with kubeadm



## 相关命令

1. sudo kubeadm token create --print-join-command
2. 