prometheus
===
## install

### master

#### docker 

 使用docker安装

 ```sh
 docker run --name prometheus -d -p 127.0.0.1:9090:9090 quay.io/prometheus/prometheus
 ```

 

#### 手动

下载

 ```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.19.2/prometheus-2.19.2.linux-amd64.tar.gz
 ```


### 配置

#### 配置service



 ```sh
cat prometheus.service <<EOF
 [Unit]
 Description=Prometheus
 After=network.target
[Service]
 ExecStart=/cloud/prometheus-2.19.2.linux-amd64/prometheus \
--config.file=/cloud/prometheus-2.19.2.linux-amd64/prometheus.yml
 KillMode=process
 Restart=on-failure
 [Install]
 WantedBy=multi-user.target
 Alias=prometheus.service
 EOF
 ```




#### 启动service



 ```sh
sudo cp prometheuse.service /lib/systemd/system/
 sudo systemctl enable prometheuse
 sudo systemctl start prometheuse.service
 ```




#### 添加监控节点



 vi $prometheuse_home/prometheus.yml,add content as blow

 ```yaml
 scrape_configs:
 # The job name is added as a label `job=<job_name` to any timeseries scraped from this config.
  - job_name: 'prometheus'
 
    # metrics_path defaults to '/metrics'
     # scheme defaults to 'http'.
 
     static_configs:
     - targets: ['bjrdc25:9090']
 
   - job_name: 'node'
     static_configs:
     - targets: ['bjrdc25:9100','bjrdc254:9100']
 ```

#### 验证

 访问如下地址

 ```
 http://bjrdc25:9090/targets
 ```

 查看节点状态

### Node

在被监控节点增加node_exporter 服务

#### 下载

 ```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
 ```




#### 配置systemd service



 ```sh
cat node_exporter.service <<EOF
 [Unit]
Description=node_exporter
 After=network.target
 
 [Service]
ExecStart=/home/bjrdc/software/node_exporter-1.0.1.linux-amd64/node_exporter
 KillMode=process
Restart=on-failure
 [Install]
 WantedBy=multi-user.target
 Alias=node_exporter.service
 EOF
 ```




#### 启动service



 ```sh
sudo cp node_exporter.service /lib/systemd/system/
 sudo systemctl enable node_exporter
sudo systemctl start node_exporter.service
 ```

 在prometheus后台应该可以看到新增加的节点

 ```sh
http://bjrdc25:9090/targets
 ```


## PQL

 ### 简单例子

```
(sum(node_memory_MemTotal_bytes) - sum(node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes) ) / sum(node_memory_MemTotal_bytes) * 100
```



## grafana

### apt

```sh
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Alternatively you can add the beta repository, see in the table above
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

sudo apt-get update
sudo apt-get install grafana
```

启动服务

```sh
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
```

访问

```
curl bjrdc25:3000
```

admin:admin

### 手动

```sh
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_7.0.6_amd64.deb
sudo dpkg -i grafana_7.0.6_amd64.deb
```

出现异常

```
Failed to start grafana-server.service: The name org.freedesktop.PolicyKit1 was not provided by any
```

```
sudo 
```



### dashboard

通过iimport的方式导入已有的dashborad，官方的dashboard在地址`https://grafana.com/grafana/dashboards`里



### alert

