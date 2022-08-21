openstack
============

## 基本概念

### 结点

#### vm10

The vm10 node runs the Identity service, Image service, Placement service, management portions
of Compute, management portion of Networking, various Networking agents, and the Dashboard. It also
includes supporting services such as an SQL database, message queue, and NTP.
Optionally, the vm10 node runs portions of the Block Storage, Object Storage, Orchestration, and
Telemetry services.
The vm10 node requires a minimum of two network interfaces.

#### Compute

The compute node runs the hypervisor portion of Compute that operates instances. By default, Compute
uses the KVM hypervisor. The compute node also runs a Networking service agent that connects instances
to virtual networks and provides firewalling services to instances via security groups.
You can deploy more than one compute node. Each node requires a minimum of two network interfaces.

#### Block Storage

The optional Block Storage node contains the disks that the Block Storage and Shared File System ser-
vices provision for instances.
For simplicity, service traﬃc between compute nodes and this node uses the management network. Pro-
duction environments should implement a separate storage network to increase performance and security.
You can deploy more than one block storage node. Each node requires a minimum of one network
interface.

#### Object Storage

The optional Object Storage node contain the disks that the Object Storage service uses for storing ac-
counts, containers, and objects.
For simplicity, service traﬃc between compute nodes and this node uses the management network. Pro-
duction environments should implement a separate storage network to increase performance and security.
This service requires two nodes. Each node requires a minimum of one network interface. You can
deploy more than two object storage nodes.



## centos

1. 在所有的节点上
	
	```
	sudo systemctl disable NetworkManager 
	sudo systemctl enable network 
	sudo systemctl stop NetworkManager.service 
	sudo systemctl start network.service
	```
	
	
	
2. 在主节点上执行如下：
	yum search centos-release-openstack
	[root@hadoop100 ~]# yum search centos-release-openstack
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	
	 * base: mirrors.sina.cn
	 * extras: mirrors.yun-idc.com
	 * updates: mirrors.sina.cn
	================================================== N/S matched: centos-release-openstack ===================================================
	centos-release-openstack.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-kilo.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-liberty.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-mitaka.noarch : OpenStack from the CentOS Cloud SIG repo configs
	【centos-release-openstack-mitaka为最新版本】
	yum install -y centos-release-openstack-mitaka
	yum update -y
	yum install -y openstack-packstack
	
3. 无密码登陆
	。。。。。。
	
4. vm10
	packstack --install-hosts=hadoop100,hadoop100,hadoop50,hadoop1,hadoop150
	packstack --install-hosts=hadoop100,hadoop100,hadoop50,hadoop1,hadoop150 --cinder-volumes-create=n --cinder-host=hadoop100,hadoop50,hadoop1,hadoop150
	
5. 问题处理
	A、mysql与mariadb的冲突——centos7之后使用的默认数据库是mariadb【虽然命令仍然是mysql】，有可能资源里一斤有mysql了，此时需要
		#rm  -rf /etc/yum.repos.d/mysql-*
		#yum clean dbcache
	B、root无法登陆的问题
		修改mariadb的默认密码为空密码
	
6. 关于LVM
	默认情况下使用packstack安装openstack的时候，它会生成一个镜像文件在/var/libcinder/cinder-volumes，默认大小为20GB
	[在安装的时候需要让packstack不要生成这个卷，而使用自己构建的卷- --cinder-volumes-create=n】
	这个cinder-volumes会映射到/dev/loop[0-9]下，如/dev/loop2
		[root@hadoop100 cinder]# losetup 
		NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
		/dev/loop0         0      0         1  0 /var/lib/docker/devicemapper/devicemapper/data
		/dev/loop1         0      0         1  0 /var/lib/docker/devicemapper/devicemapper/metadata
		/dev/loop2         0      0         0  0 /var/lib/cinder/cinder-volumes
		/dev/loop3         0      0         1  0 /srv/loopback-device/swiftloopback
	

			  --- Volume group ---
		  VG Name               cinder-volumes
		  System ID             
		  Format                lvm2
		.
		 .  
		  .
		  --- Physical volumes ---
		  PV Name               /dev/loop2     
		  PV UUID               uB8KGN-9zH0-kfeq-swva-KAOU-fAkU-2Dvpih
		  PV Status             allocatable
		  Total PE / Free PE    5273 / 5273
		之后会再将这个/dev/loop2做成一个lvm：cinder-volumes
	按照这套原理，需要增加cinder-volumes的大小需要：
	A、首先使用lvm，调整根分区“/”大小
	B、使用如下命令调整相关的文件大小和虚拟卷大小
		#truncate -s +900G cinder-volumes
		#losetup -fv cinder-volumes
		#pvresize /dev/loop2
		#losetup -d /dev/loop2 可以删除loop设备，但是一定记住不要使用rm 删除/dev/loop2

7. 问题
	openstack只支持lvm以iscsc的方式，哎

	1. 本地存储

　　		对于本地存储，cinder-volume可以使用lvm驱动，该驱动当前的实现需要在主机上事先用lvm命令创建一个cinder-volumes的vg, 当该主机接受到创建卷请求的时候，cinder-volume在该vg上创建一个LV, 并且用openiscsi将这个卷当作一个iscsi tgt给export.
		当然还可以将若干主机的本地存储用sheepdog虚拟成一个共享存储，然后使用sheepdog驱动。
7、卸载openstack

yum -y remove openstack-* openvswitch memcached httpd python-neutron rabbitmq-server

## ubuntu

[参考](https://docs.openstack.org/xena/install/)

### 环境

#### security

#### network

##### Configure network interfaces[¶](https://docs.openstack.org/neutron/xena/install/environment-networking-vm10-ubuntu.html#configure-network-interfaces)

1. Configure the first interface as the management interface:

   IP address: 192.168.56.10

   Network mask: 255.255.255.0 (or /24)

   Default gateway: 192.168.56.1

2. The provider interface uses a special configuration without an IP address assigned to it. Configure the second interface as the provider interface:

   Replace `INTERFACE_NAME` with the actual interface name. For example, *eth1* or *ens224*.

   - Edit the `/etc/network/interfaces` file to contain the following:

     ```ini
     # The provider network interface
     auto enp0s9
     iface enp0s9 inet manual
     up ip link set dev $IFACE up
     down ip link set dev $IFACE down
     ```

3. Reboot the system to activate the changes.

##### Configure name resolution[¶](https://docs.openstack.org/neutron/xena/install/environment-networking-vm10-ubuntu.html#configure-name-resolution)

1. Set the hostname of the node to `vm10`.

2. Edit the `/etc/hosts` file to contain the following:

   ```
   # vm10
   192.168.56.10       vm10
   
   # compute1
   xxxx       compute1
   ```

   

#### ntp

1. install

   ```
   sudo apt install chrony
   ```

2. For Ubuntu, edit the `/etc/chrony/chrony.conf` file:

   ```
   server NTP_SERVER iburst
   ```

   Replace `NTP_SERVER` with the hostname or IP address of a suitable more accurate (lower stratum) NTP server. The configuration supports multiple `server` keys.

3. To enable other nodes to connect to the chrony daemon on the vm10 node, add this key to the same `chrony.conf` file mentioned above:

   ```
   allow 192.168.56.0/24
   ```

4. 

#### apt

```
sudo add-apt-repository cloud-archive:xena
```

#### [rabbitmq](https://docs.openstack.org/install-guide/environment-messaging-ubuntu.html)

1. Install the package:

   ```
   sudo apt install rabbitmq-server -y
   sudo systemctl restart rabbitmq-server.service
   ```

2. Add the `openstack` user:

   ```
   sudo rabbitmqctl add_user openstack openstack-me 
   
   Creating user "openstack" ...
   ```

   Replace `openstack-me` with a suitable password.

3. Permit configuration, write, and read access for the `openstack` user:

   ```
   sudo rabbitmqctl set_permissions openstack ".*" ".*" ".*"
   
   Setting permissions for user "openstack" in vhost "/" ...
   ```



#### [memecached](https://docs.openstack.org/install-guide/environment-memcached-ubuntu.html)

1. Install the packages:

   For Ubuntu 18.04 and newer versions use:

   ```sh
   sudo apt install memcached python3-memcache -y
   ```

2. Edit the `/etc/memcached.conf` file and configure the service to use the management IP address of the vm10 node. This is to enable access by other nodes via the management network:

   ```
   -l 192.168.56.10
   ```

3. restart

   ```sh
   sudo systemctl restart memcached.service
   ```

#### [etcd](https://docs.openstack.org/install-guide/environment-etcd-ubuntu.html)

1. Install the `etcd` package:

   ```
   sudo apt install etcd -y
   ```

2. Edit the `/etc/default/etcd` file and set the `ETCD_INITIAL_CLUSTER`, `ETCD_INITIAL_ADVERTISE_PEER_URLS`, `ETCD_ADVERTISE_CLIENT_URLS`, `ETCD_LISTEN_CLIENT_URLS` to the management IP address of the vm10 node to enable access by other nodes via the management network:

   ```ini
   ETCD_NAME="vm10"
   ETCD_DATA_DIR="/var/lib/etcd"
   ETCD_INITIAL_CLUSTER_STATE="new"
   ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
   ETCD_INITIAL_CLUSTER="vm10=http://192.168.56.10:2380"
   ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.10:2380"
   ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.10:2379"
   ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
   ETCD_LISTEN_CLIENT_URLS="http://192.168.56.10:2379"
   ```

3. restart

   ```
   sudo systemctl restart etcd
   ```

   

#### mysql

安装略

```mysql
CREATE DATABASE keystone;
CREATE USER 'keystone'@'%' IDENTIFIED BY 'openstack-me';
GRANT ALL PRIVILEGES ON keystone.* TO  'keystone'@'%'; 
```

### admin-openrc

```
export OS_USERNAME=admin
export OS_PASSWORD=openstack-me
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://vm10:5000/v3
export OS_IDENTITY_API_VERSION=3
```



### keystone

[参考这里](https://docs.openstack.org/keystone/xena/install/keystone-install-ubuntu.html)

#### 安装

Run the following command to install the packages:

```
sudo apt install keystone -y
```

edit /etc/keystone/keystone.conf

```ini
[database]
# ...
connection = mysql+pymysql://keystone:openstack-me@vm10/keystone
```

```ini
[token]
# ...
provider = fernet
```

Populate the Identity service database:

```
#su -s /bin/sh -c "keystone-manage db_sync" keystone
```

Initialize Fernet key repositories:

```
sudo keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
sudo keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

Bootstrap the Identity service:

```
sudo keystone-manage bootstrap --bootstrap-password openstack-me \
  --bootstrap-admin-url http://vm10:5000/v3/ \
  --bootstrap-internal-url http://vm10:5000/v3/ \
  --bootstrap-public-url http://vm10:5000/v3/ \
  --bootstrap-region-id RegionOne
```

Configure the Apache HTTP server

Edit the `/etc/apache2/apache2.conf` file and configure the `ServerName` option to reference the vm10 node:

```
ServerName vm10
```

```
sudo service apache2 restart
```

Configure the administrative account by setting the proper environmental variables:

```
export OS_USERNAME=admin
export OS_PASSWORD=openstack-me
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://vm10:5000/v3
export OS_IDENTITY_API_VERSION=3
```

#### Create a domain, projects, users, and roles

1. Although the “default” domain already exists from the keystone-manage bootstrap step in this guide, a formal way to create a new domain would be:

   ```
   openstack domain create --description "An Example Domain" example 
   ```

2. This guide uses a service project that contains a unique user for each service that you add to your environment. Create the `service` project:

   ```
   openstack project create --domain default --description "Service Project" service
   ```

3. Regular (non-admin) tasks should use an unprivileged project and user. As an example, this guide creates the `myproject` project and `myuser` user.

   1. Create the `myproject` project:

      ```
      openstack project create --domain default --description "Demo Project" myproject
      ```

   2. Create the `myuser` user:

      ```
      openstack user create --domain default   --password-prompt myuser
      User Password:
      Repeat User Password:
      ```

   3. Create the `myrole` role:

      ```
      openstack role create myrole
      ```

   4. Add the `myrole` role to the `myproject` project and `myuser` user:

      ```
      openstack role add --project myproject --user myuser myrole
      ```

   

#### Verify operation

1. Unset the temporary `OS_AUTH_URL` and `OS_PASSWORD` environment variable:

   ```
   unset OS_AUTH_URL OS_PASSWORD
   ```

2. As the `admin` user, request an authentication token:

   ```
   openstack --os-auth-url http://vm10:5000/v3 \
     --os-project-domain-name Default --os-user-domain-name Default \
     --os-project-name admin --os-username admin token issue
   
   Password:
   +------------+-----------------------------------------------------------------+
   | Field      | Value                                                           |
   +------------+-----------------------------------------------------------------+
   | expires    | 2016-02-12T20:14:07.056119Z                                     |
   | id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
   |            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
   |            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
   | project_id | 343d245e850143a096806dfaefa9afdc                                |
   | user_id    | ac3377633149401296f6c0d92d79dc16                                |
   +------------+-----------------------------------------------------------------+
   ```

   This command uses the password for the `admin` user.

3. As the `myuser` user created in the previous, request an authentication token:

   ```
   openstack --os-auth-url http://vm10:5000/v3 \
     --os-project-domain-name Default --os-user-domain-name Default \
     --os-project-name myproject --os-username myuser token issue
   
   Password:
   +------------+-----------------------------------------------------------------+
   | Field      | Value                                                           |
   +------------+-----------------------------------------------------------------+
   | expires    | 2016-02-12T20:15:39.014479Z                                     |
   | id         | gAAAAABWvi9bsh7vkiby5BpCCnc-JkbGhm9wH3fabS_cY7uabOubesi-Me6IGWW |
   |            | yQqNegDDZ5jw7grI26vvgy1J5nCVwZ_zFRqPiz_qhbq29mgbQLglbkq6FQvzBRQ |
   |            | JcOzq3uwhzNxszJWmzGC7rJE_H0A_a3UFhqv8M4zMRYSbS2YF0MyFmp_U       |
   | project_id | ed0b60bf607743088218b0a533d5943f                                |
   | user_id    | 58126687cbcc4888bfa9ab73a2256f27                                |
   +------------+-----------------------------------------------------------------+
   ```

### Nova

#### 控制结点

##### 准备

1. To create the databases, complete these steps:

   ```
   CREATE DATABASE nova_api;
   CREATE DATABASE nova;
   CREATE DATABASE nova_cell0;
   ```

2. 赋权限

   ```
   CREATE USER 'nova'@'%' IDENTIFIED BY 'openstack-me';
   GRANT ALL PRIVILEGES ON nova_api.* TO  'nova'@'%';
   GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%';
   GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%';
   ```

3. source admin-openrc

   ```
   . admin-openrc
   ```

4. Create the Compute service credentials:

   1. Create the `nova` user:

      ```
      openstack user create --domain default --password-prompt nova
      ```

   2. Add the `admin` role to the `nova` user:

      ```
      openstack role add --project service --user nova admin
      ```

      This command provides no output.

   3. Create the `nova` service entity:

      ```
      openstack service create --name nova  --description "OpenStack Compute" compute
      ```

5. Create the Compute API service endpoints:

   ```shell
   openstack endpoint create --region RegionOne compute public http://vm10:8774/v2.1
   openstack endpoint create --region RegionOne compute internal http://vm10:8774/v2.1
   openstack endpoint create --region RegionOne compute admin http://vm10:8774/v2.1
   ```

6. Install Placement service and configure user and endpoints:

   - Refer to the [Placement service install guide](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#configure-user-and-endpoints) for more information.

##### 安装

1. Install the packages:

   ```
   sudo apt install nova-api nova-conductor nova-novncproxy nova-scheduler
   ```

2. Edit the `/etc/nova/nova.conf` file and complete the following actions:

   - In the `[api_database]` and `[database]` sections, configure database access:

     ```ini
     [api_database]
     # ...
     connection = mysql+pymysql://nova:openstack-me@vm10/nova_api
     
     [database]
     # ...
     connection = mysql+pymysql://nova:openstack-me@vm10/nova
     ```

     Replace `NOVA_DBPASS` with the password you chose for the Compute databases.

   - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

     ```ini
     [DEFAULT]
     # ...
     transport_url = rabbit://openstack:openstack-me@vm10:5672/
     ```

     Replace `openstack-me` with the password you chose for the `openstack` account in `RabbitMQ`.

   - In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access:

     ```ini
     [api]
     # ...
     auth_strategy = keystone
     
     [keystone_authtoken]
     # ...
     www_authenticate_uri = http://vm10:5000/
     auth_url = http://vm10:5000/
     memcached_servers = vm10:11211
     auth_type = password
     project_domain_name = Default
     user_domain_name = Default
     project_name = service
     username = nova
     password = openstack-me
     ```

     Replace `NOVA_PASS` with the password you chose for the `nova` user in the Identity service.

     

     Comment out or remove any other options in the `[keystone_authtoken]` section.

   - In the `[DEFAULT]` section, configure the `my_ip` option to use the management interface IP address of the vm10 node:

     ```ini
     [DEFAULT]
     # ...
     my_ip = 192.168.56.10
     ```

   - Configure the `[neutron]` section of **/etc/nova/nova.conf**. Refer to the [Networking service install guide](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#configure-the-compute-service-to-use-the-networking-service) for more information.

   - In the `[vnc]` section, configure the VNC proxy to use the management interface IP address of the vm10 node:

     ```ini
     [vnc]
     enabled = true
     server_listen = $my_ip
     server_proxyclient_address = $my_ip
     ```
     
   - In the `[glance]` section, configure the location of the Image service API:
   
     ```ini
     [glance]
     # ...
     api_servers = http://vm10:9292
     ```
   
   - In the `[oslo_concurrency]` section, configure the lock path:
   
     ```ini
     [oslo_concurrency]
     # ...
     lock_path = /var/lib/nova/tmp
     ```
   
   - Due to a packaging bug, remove the `log_dir` option from the `[DEFAULT]` section.
   
   - In the `[placement]` section, configure access to the Placement service:
   
     ```ini
     [placement]
     # ...
     region_name = RegionOne
     project_domain_name = Default
     project_name = service
     auth_type = password
     user_domain_name = Default
     auth_url = http://vm10:5000/v3
     username = placement
     password = openstack-me
     ```
   
     Replace `PLACEMENT_PASS` with the password you choose for the `placement` service user created when installing [Placement](https://docs.openstack.org/placement/xena/install/). Comment out or remove any other options in the `[placement]` section.
   
3. Populate the `nova-api` database:

   ```
   # su -s /bin/sh -c "nova-manage api_db sync" nova
   ```

   Ignore any deprecation messages in this output.

4. Register the `cell0` database:

   ```shell
   # su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
   ```

5. Create the `cell1` cell:

   ```shell
   # su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
   ```

6. Populate the nova database:

   ```shell
   # su -s /bin/sh -c "nova-manage db sync" nova
   ```

7. Verify nova cell0 and cell1 are registered correctly:

   ```
   # su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
   +-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
   |  Name |                 UUID                 |                   Transport URL                    |                     Database Connection                      | Disabled |
   +-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
   | cell0 | 00000000-0000-0000-0000-000000000000 |                       none:/                       | mysql+pymysql://nova:****@vm10/nova_cell0?charset=utf8 |  False   |
   | cell1 | f690f4fd-2bc5-4f15-8145-db561a7b9d3d | rabbit://openstack:****@vm10:5672/nova_cell1 | mysql+pymysql://nova:****@vm10/nova_cell1?charset=utf8 |  False   |
   +-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
   ```



8. Restart the Compute services:

   ```
   sudo service nova-api restart
   sudo service nova-scheduler restart
   sudo service nova-conductor restart
   sudo service nova-novncproxy restart
   ```

#### 计算结点

##### Install and configure components[¶](https://docs.openstack.org/nova/xena/install/compute-install-ubuntu.html#install-and-configure-components)

1. Install the packages:

   ```
   sudo apt install nova-compute
   ```

2. Edit the `/etc/nova/nova.conf` file and complete the following actions:

   - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

     ```ini
     [DEFAULT]
     # ...
     transport_url = rabbit://openstack:openstack-me@vm10
     ```

     Replace `openstack-me` with the password you chose for the `openstack` account in `RabbitMQ`.

   - In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access:

     ```ini
     [api]
     # ...
     auth_strategy = keystone
     
     [keystone_authtoken]
     # ...
     www_authenticate_uri = http://vm10:5000/
     auth_url = http://vm10:5000/
     memcached_servers = vm10:11211
     auth_type = password
     project_domain_name = Default
     user_domain_name = Default
     project_name = service
     username = nova
     password = openstack-me
     ```

     Replace `openstack-me` with the password you chose for the `nova` user in the Identity service.

     Note

     Comment out or remove any other options in the `[keystone_authtoken]` section.

- In the `[DEFAULT]` section, configure the `my_ip` option:

  ```ini
  [DEFAULT]
  # ...
  #my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
  my_ip = 192.168.56.10
  ```

  Replace `MANAGEMENT_INTERFACE_IP_ADDRESS` with the IP address of the management network interface on your compute node, typically 10.0.0.31 for the first node in the [example architecture](https://docs.openstack.org/nova/xena/install/overview.html#overview-example-architectures).

- Configure the `[neutron]` section of **/etc/nova/nova.conf**. Refer to the [Networking service install guide](https://docs.openstack.org/neutron/xena/install/compute-install-ubuntu.html#configure-the-compute-service-to-use-the-networking-service) for more details.

- In the `[vnc]` section, enable and configure remote console access:

  ```ini
  [vnc]
  # ...
  enabled = true
  server_listen = 0.0.0.0
  server_proxyclient_address = $my_ip
  novncproxy_base_url = http://vm10:6080/vnc_auto.html
  ```

  The server component listens on all IP addresses and the proxy component only listens on the management interface IP address of the compute node. The base URL indicates the location where you can use a web browser to access remote consoles of instances on this compute node.

  Note

  If the web browser to access remote consoles resides on a host that cannot resolve the `vm10` hostname, you must replace `vm10` with the management interface IP address of the vm10 node.

- In the `[glance]` section, configure the location of the Image service API:

  ```ini
  [glance]
  # ...
  api_servers = http://vm10:9292
  ```

- In the `[oslo_concurrency]` section, configure the lock path:

  ```ini
  [oslo_concurrency]
  # ...
  lock_path = /var/lib/nova/tmp
  ```

- In the `[placement]` section, configure the Placement API:

  ```ini
  [placement]
  # ...
  region_name = RegionOne
  project_domain_name = Default
  project_name = service
  auth_type = password
  user_domain_name = Default
  auth_url = http://vm10:5000/v3
  username = placement
  password = openstack-me
  ```

  Replace `PLACEMENT_PASS` with the password you choose for the `placement` user in the Identity service. Comment out any other options in the `[placement]` section.

##### Finalize installation[¶](https://docs.openstack.org/nova/xena/install/compute-install-ubuntu.html#finalize-installation)

1. Determine whether your compute node supports hardware acceleration for virtual machines:

   ```
   egrep -c '(vmx|svm)' /proc/cpuinfo
   ```

   If this command returns a value of `one or greater`, your compute node supports hardware acceleration which typically requires no additional configuration.

   If this command returns a value of `zero`, your compute node does not support hardware acceleration and you must configure `libvirt` to use QEMU instead of KVM.

   - Edit the `[libvirt]` section in the `/etc/nova/nova-compute.conf` file as follows:

     ```ini
     [libvirt]
     # ...
     virt_type = qemu
     ```

2. Restart the Compute service:

   ```
   sudo service nova-compute restart
   ```

### Cinder

#### 安装

##### Prerequisites[¶](https://docs.openstack.org/cinder/xena/install/cinder-vm10-install-ubuntu.html#prerequisites)

Before you install and configure the Block Storage service, you must create a database, service credentials, and API endpoints.

1. To create the database, complete these steps:

   1. Use the database access client to connect to the database server as the `root` user:

      ```
      # mysql
      ```

   2. Create the `cinder` database:

      ```mysql
      CREATE DATABASE cinder;
      ```

   3. Grant proper access to the `cinder` database:

      ```mysql
      CREATE USER 'cinder'@'%' IDENTIFIED BY 'openstack-me';
      GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%';
      ```

   4. Exit the database access client.

2. Source the `admin` credentials to gain access to admin-only CLI commands:

   ```
   . admin-openrc
   ```

3. To create the service credentials, complete these steps:

   1. Create a `cinder` user:

      ```
      openstack user create --domain default --password-prompt cinder
      ```

   2. Add the `admin` role to the `cinder` user:

      ```
      openstack role add --project service --user cinder admin
      ```

   3. Create the `cinderv3` service entity:

      ```
      openstack service create --name cinderv3  --description "OpenStack Block Storage" volumev3
      ```

      Note

      Beginning with the Xena release, the Block Storage services require only one service entity. For prior releases, please consult the documentation for that specific release.

4. Create the Block Storage service API endpoints:

   ```
   openstack endpoint create --region RegionOne  volumev3 public http://vm10:8776/v3/%\(project_id\)s
   openstack endpoint create --region RegionOne  volumev3 internal http://vm10:8776/v3/%\(project_id\)s
   openstack endpoint create --region RegionOne  volumev3 admin http://vm10:8776/v3/%\(project_id\)s
   ```

##### Install and configure components[¶](https://docs.openstack.org/cinder/xena/install/cinder-vm10-install-ubuntu.html#install-and-configure-components)

1. Install the packages:

   ```
   sudo apt install cinder-api cinder-scheduler
   ```

2. Edit the `/etc/cinder/cinder.conf` file and complete the following actions:

   1. In the `[database]` section, configure database access:

      ```
      [database]
      # ...
      connection = mysql+pymysql://cinder:openstack-me@vm10/cinder
      ```

      Replace `openstack-me` with the password you chose for the Block Storage database.

   2. In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

      ```ini
      [DEFAULT]
      # ...
      transport_url = rabbit://openstack:openstack-me@vm10
      ```

      Replace `openstack-me` with the password you chose for the `openstack` account in `RabbitMQ`.

   3. In the `[DEFAULT]` and `[keystone_authtoken]` sections, configure Identity service access:

      ```ini
      [DEFAULT]
      # ...
      auth_strategy = keystone
      
      [keystone_authtoken]
      www_authenticate_uri = http://vm10:5000
      auth_url = http://vm10:5000
      memcached_servers = vm10:11211
      auth_type = password
      project_domain_name = default
      user_domain_name = default
      project_name = service
      username = cinder
      password = openstack-me
      ```
      
      Replace `CINDER_PASS` with the password you chose for the `cinder` user in the Identity service.
      
      Note
      
      Comment out or remove any other options in the `[keystone_authtoken]` section.
      
   4. In the `[DEFAULT]` section, configure the `my_ip` option to use the management interface IP address of the vm10 node:
   
      ```ini
      [DEFAULT]
      # ...
      my_ip = 192.168.56.10
      ```
   
3. In the `[oslo_concurrency]` section, configure the lock path:

   ```ini
   [oslo_concurrency]
   # ...
   lock_path = /var/lib/cinder/tmp
   ```

4. Populate the Block Storage database:

   ```
   # su -s /bin/sh -c "cinder-manage db sync" cinder
   ```

   Note

   Ignore any deprecation messages in this output.

##### Configure Compute to use Block Storage[¶](https://docs.openstack.org/cinder/xena/install/cinder-vm10-install-ubuntu.html#configure-compute-to-use-block-storage)

1. Edit the `/etc/nova/nova.conf` file and add the following to it:

   ```
   [cinder]
   os_region_name = RegionOne
   ```

##### Finalize installation[¶](https://docs.openstack.org/cinder/xena/install/cinder-vm10-install-ubuntu.html#finalize-installation)

1. Restart the Compute API service:

   ```
   sudo service nova-api restart
   ```

2. Restart the Block Storage services:

   ```
   sudo  service cinder-scheduler restart
   sudo  service apache2 restart
   ```

#### 配置卷

##### Prerequisites[¶](https://docs.openstack.org/cinder/xena/install/cinder-storage-install-ubuntu.html#prerequisites)

Before you install and configure the Block Storage service on the storage node, you must prepare the storage device.

Perform these steps on the storage node.

1. Install the supporting utility packages:

   ```
   sudo apt install lvm2 thin-provisioning-tools 
   ```

   Note

   Some distributions include LVM by default.

2. Create the LVM physical volume `/dev/sdc`:

   ```
   sudo pvcreate /dev/sdb
   
   Physical volume "/dev/sdb" successfully created
   ```

3. Create the LVM volume group `cinder-volumes`:

   ```
   sudo vgcreate cinder-volumes /dev/sdb
   
   Volume group "cinder-volumes" successfully created
   ```

   The Block Storage service creates logical volumes in this volume group.

4. Only instances can access Block Storage volumes. However, the underlying operating system manages the devices associated with the volumes. By default, the LVM volume scanning tool scans the `/dev` directory for block storage devices that contain volumes. If projects use LVM on their volumes, the scanning tool detects these volumes and attempts to cache them which can cause a variety of problems with both the underlying operating system and project volumes. You must reconfigure LVM to scan only the devices that contain the `cinder-volumes` volume group. Edit the `/etc/lvm/lvm.conf` file and complete the following actions:

   - In the `devices` section, add a filter that accepts the `/dev/sdc` device and rejects all other devices:

     ```
     devices {
     ...
     filter = [ "a/sdb/", "r/.*/"]
     ```

     Each item in the filter array begins with `a` for **accept** or `r` for **reject** and includes a regular expression for the device name. The array must end with `r/.*/` to reject any remaining devices. You can use the **vgs -vvvv** command to test filters.

     **Warning**

     If your storage nodes use LVM on the operating system disk, you must also add the associated device to the filter. For example, if the `/dev/sda` device contains the operating system:

     ```
     filter = [ "a/sda/", "a/sdc/", "r/.*/"]
     ```

     Similarly, if your compute nodes use LVM on the operating system disk, you must also modify the filter in the `/etc/lvm/lvm.conf` file on those nodes to include only the operating system disk. For example, if the `/dev/sda` device contains the operating system:

     ```
     filter = [ "a/sda/", "r/.*/"]
     ```

##### Install and configure components[¶](https://docs.openstack.org/cinder/xena/install/cinder-storage-install-ubuntu.html#install-and-configure-components)

1. Install the packages:

   ```
   sudo apt install cinder-volume
   ```

2. Edit the `/etc/cinder/cinder.conf` file and complete the following actions:

   - In the `[database]` section, configure database access:

     ```ini
     [database]
     # ...
     connection = mysql+pymysql://cinder:openstack-me@vm10/cinder
     ```

     Replace `CINDER_DBPASS` with the password you chose for the Block Storage database.

   - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

     ```ini
     [DEFAULT]
     # ...
     transport_url = rabbit://openstack:openstack-me@vm10
     ```

     Replace `openstack-me` with the password you chose for the `openstack` account in `RabbitMQ`.

   - In the `[DEFAULT]` and `[keystone_authtoken]` sections, configure Identity service access:

     ```ini
     [DEFAULT]
     # ...
     auth_strategy = keystone
     
     [keystone_authtoken]
     # ...
     www_authenticate_uri = http://vm10:5000
     auth_url = http://vm10:5000
     memcached_servers = vm10:11211
     auth_type = password
     project_domain_name = default
     user_domain_name = default
     project_name = service
     username = cinder
     password = openstack-me
     ```

     Replace `CINDER_PASS` with the password you chose for the `cinder` user in the Identity service.

     Note

     

     Comment out or remove any other options in the `[keystone_authtoken]` section.

   - In the `[DEFAULT]` section, configure the `my_ip` option:

     ```ini
     [DEFAULT]
     # ...
     my_ip = 192.168.56.10
     ```

     Replace `MANAGEMENT_INTERFACE_IP_ADDRESS` with the IP address of the management network interface on your storage node, typically 10.0.0.41 for the first node in the [example architecture](https://docs.openstack.org/install-guide/overview.html#example-architecture).

   - In the `[lvm]` section, configure the LVM back end with the LVM driver, `cinder-volumes` volume group, iSCSI protocol, and appropriate iSCSI service:

     ```ini
     [lvm]
     # ...
     volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
     volume_group = cinder-volumes
     target_protocol = iscsi
     target_helper = tgtadm
     ```

   - In the `[DEFAULT]` section, enable the LVM back end:

     ```
     [DEFAULT]
     # ...
     enabled_backends = lvm
     ```

      

     Note

     

     Back-end names are arbitrary. As an example, this guide uses the name of the driver as the name of the back end.

   - In the `[DEFAULT]` section, configure the location of the Image service API:

     ```
     [DEFAULT]
     # ...
     glance_api_servers = http://vm10:9292
     ```

   - In the `[oslo_concurrency]` section, configure the lock path:

     ```ini
     [oslo_concurrency]
     # ...
     lock_path = /var/lib/cinder/tmp
     ```

##### tgt

```
sudo apt install tgt
sudo sh -c 'echo "include /var/lib/cinder/volumes/*" >> /etc/tgt/conf.d/cinder.conf'
sudo service tgt restart
```



##### Finalize installation[¶](https://docs.openstack.org/cinder/xena/install/cinder-storage-install-ubuntu.html#finalize-installation)

1. Restart the Block Storage volume service including its dependencies:

   ```
   sudo service tgt restart
   sudo service cinder-volume restart
   ```

#### Install and configure the backup service(可选)

##### Install and configure components[¶](https://docs.openstack.org/cinder/xena/install/cinder-backup-install-ubuntu.html#install-and-configure-components) 

Note



Perform these steps on the Block Storage node.

1. Install the packages:

   ```
   sudo apt install cinder-backup
   ```

2. Edit the `/etc/cinder/cinder.conf` file and complete the following actions:

- In the `[DEFAULT]` section, configure backup options:

  ```
  [DEFAULT]
  # ...
  backup_driver = cinder.backup.drivers.swift.SwiftBackupDriver
  backup_swift_url = SWIFT_URL
  ```

  Replace `SWIFT_URL` with the URL of the Object Storage service. The URL can be found by showing the object-store API endpoints:

  ```
  openstack catalog show object-store
  ```

##### Finalize installation[¶](https://docs.openstack.org/cinder/xena/install/cinder-backup-install-ubuntu.html#finalize-installation)

Restart the Block Storage backup service:

```
# service cinder-backup restart
```

  

### placement

#### Prerequisites[¶](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#prerequisites)

Before you install and configure the placement service, you must create a database, service credentials, and API endpoints.

#### Create Database[¶](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#create-database)

1. To create the database, complete these steps:

   - Use the database access client to connect to the database server as the `root` user:

     ```
     # mysql
     ```

   - Create the `placement` database:

     ```sql
     CREATE DATABASE placement;
     ```

   - Grant proper access to the database:

     ```sql
     CREATE USER 'placement'@'%' IDENTIFIED BY 'openstack-me';
     GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%';
     ```

     Replace `openstack-me` with a suitable password.

   - Exit the database access client.

#### Configure User and Endpoints[¶](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#configure-user-and-endpoints)

1. Source the `admin` credentials to gain access to admin-only CLI commands:

   ```
   $ . admin-openrc
   ```

2. Create a Placement service user using your chosen `PLACEMENT_PASS`:

   ```
   openstack user create --domain default --password-prompt placement
   ```

3. Add the Placement user to the service project with the admin role:

   ```
   openstack role add --project service --user placement admin
   ```

   Note

   This command provides no output.

4. Create the Placement API entry in the service catalog:

   ```
   openstack service create --name placement --description "Placement API" placement
   ```

5. Create the Placement API service endpoints: 

   Note

   Depending on your environment, the URL for the endpoint will vary by port (possibly 8780 instead of 8778, or no port at all) and hostname. You are responsible for determining the correct URL.

   ```
   openstack endpoint create --region RegionOne placement public http://vm10:8778
   openstack endpoint create --region RegionOne placement internal http://vm10:8778
   openstack endpoint create --region RegionOne placement admin http://vm10:8778
   ```

#### Install and configure components[¶](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#install-and-configure-components) 

Note

Default configuration files vary by distribution. You might need to add these sections and options rather than modifying existing sections and options. Also, an ellipsis (`...`) in the configuration snippets indicates potential default configuration options that you should retain.

1. Install the packages:

   ```
   sudo apt install placement-api
   ```

2. Edit the `/etc/placement/placement.conf` file and complete the following actions:

   - In the `[placement_database]` section, configure database access:

     ```ini
     [placement_database]
     # ...
     connection = mysql+pymysql://placement:openstack-me@vm10/placement
     ```

     Replace `openstack-me` with the password you chose for the placement database.

   - In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access:

     ```
     [api]
     # ...
     auth_strategy = keystone
     
     [keystone_authtoken]
     # ...
     auth_url = http://vm10:5000/v3
     memcached_servers = vm10:11211
     auth_type = password
     project_domain_name = Default
     user_domain_name = Default
     project_name = service
     username = placement
     password = openstack-me
     ```

     Replace `PLACEMENT_PASS` with the password you chose for the `placement` user in the Identity service.

     

     Note

     Comment out or remove any other options in the `[keystone_authtoken]` section. 

     Note

     The value of `user_name`, `password`, `project_domain_name` and `user_domain_name` need to be in sync with your keystone config.

3. Populate the `placement` database:

   ```
   # su -s /bin/sh -c "placement-manage db sync" placement
   ```

   Note

   Ignore any deprecation messages in this output.

#### Finalize installation[¶](https://docs.openstack.org/placement/xena/install/install-ubuntu.html#finalize-installation)

- Reload the web server to adjust to get new configuration settings for placement.

  ```
  sudo service apache2 restart
  ```

### glance

#### Prerequisites[¶](https://docs.openstack.org/glance/xena/install/install-ubuntu.html#prerequisites)

Before you install and configure the Image service, you must create a database, service credentials, and API endpoints.

1. To create the database, complete these steps:

   - Use the database access client to connect to the database server as the `root` user:

     ```
     # mysql
     ```

   - Create the `glance` database:

     ```mysql
      CREATE DATABASE glance;
      CREATE USER 'glance'@'%' IDENTIFIED BY 'openstack-me';
      GRANT ALL PRIVILEGES ON glance.* TO  'glance'@'%'; 
     ```

2. Source the `admin` credentials to gain access to admin-only CLI commands:

   . admin-openrc

3. To create the service credentials, complete these steps:

   - Create the `glance` user:

     ```
     openstack user create --domain default --password-prompt glance
     ```

   - Add the `admin` role to the `glance` user and `service` project:

     ```
     openstack role add --project service --user glance admin
     ```

   - Create the `glance` service entity:

     ```
     openstack service create --name glance --description "OpenStack Image" image
     ```

4. Create the Image service API endpoints:

   ```
   openstack endpoint create --region RegionOne image public http://vm10:9292
   openstack endpoint create --region RegionOne image internal http://vm10:9292
   openstack endpoint create --region RegionOne image admin http://vm10:9292
   ```

1. Register quota limits (**optional**):

   If you decide to use per-tenant quotas in Glance, you must register the limits in Keystone first:

   ```
   openstack --os-cloud devstack-system-admin registered limit create --service glance --default-limit 1000 --region RegionOne image_size_total
   openstack --os-cloud devstack-system-admin registered limit create --service glance --default-limit 1000 --region RegionOne image_stage_total
   openstack --os-cloud devstack-system-admin registered limit create --service glance --default-limit 100 --region RegionOne image_count_total
   openstack --os-cloud devstack-system-admin registered limit create --service glance --default-limit 100 --region RegionOne image_count_uploading
   ```

   Be sure to also set `use_keystone_quotas=True` in your `glance-api.conf` file.

#### Install and configure components[¶](https://docs.openstack.org/glance/xena/install/install-ubuntu.html#install-and-configure-components)

Default configuration files vary by distribution. You might need to add these sections and options rather than modifying existing sections and options. Also, an ellipsis (`...`) in the configuration snippets indicates potential default configuration options that you should retain.

1. Install the packages:

   ```
   sudo apt install glance
   ```

2. Edit the `/etc/glance/glance-api.conf` file and complete the following actions:

   - In the `[database]` section, configure database access:

     ```ini
     [database]
     # ...
     connection = mysql+pymysql://glance:openstack-me@vm10/glance
     ```

     Replace `GLANCE_DBPASS` with the password you chose for the Image service database.

   - In the `[keystone_authtoken]` and `[paste_deploy]` sections, configure Identity service access:

     ```ini
     [keystone_authtoken]
     # ...
     www_authenticate_uri = http://vm10:5000
     auth_url = http://vm10:5000
     memcached_servers = vm10:11211
     auth_type = password
     project_domain_name = Default
     user_domain_name = Default
     project_name = service
     username = glance
     password = openstack-me
     
     [paste_deploy]
     # ...
     flavor = keystone
     ```

     Replace `GLANCE_PASS` with the password you chose for the `glance` user in the Identity service.

     Comment out or remove any other options in the `[keystone_authtoken]` section.

   - In the `[glance_store]` section, configure the local file system store and location of image files:

     ```ini
     [glance_store]
     # ...
     stores = file,http
     default_store = file
     filesystem_store_datadir = /var/lib/glance/images/
     ```

   - In the `[oslo_limit]` section, configure access to keystone(可选)

     ```ini
     [oslo_limit]
     auth_url = http://vm10:5000
     auth_type = password
     user_domain_id = default
     username = glance-me
     system_scope = all
     password = openstack-me
     endpoint_id = ENDPOINT_ID
     region_name = RegionOne
     ```

     Make sure that the MY_SERVICE account has reader access to system-scope resources (like limits):

     ```
     openstack role add --user glance-me --user-domain Default --system all reader
     ```

     See [the oslo_limit docs](https://docs.openstack.org/oslo.limit/latest/user/usage.html#configuration) for more information about configuring the unified limits client.

   - In the `[DEFAULT]` section, optionally enable per-tenant quotas:

     ```
     [DEFAULT]
     use_keystone_quotas = True
     ```

     Note that you must have created the registered limits as described above if this is enabled.

3. Populate the Image service database:

   ```
   # su -s /bin/sh -c "glance-manage db_sync" glance
   ```

Ignore any deprecation messages in this output.

#### Finalize installation[¶](https://docs.openstack.org/glance/xena/install/install-ubuntu.html#finalize-installation)

1. Restart the Image services:

   ```
   sudo service glance-api restart
   ```



### neutron

#### Prerequisites[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#prerequisites)

Before you configure the OpenStack Networking (neutron) service, you must create a database, service credentials, and API endpoints.

1. To create the database, complete these steps:

   - Use the database access client to connect to the database server as the `root` user:

     ```
     mysql -u root -p
     ```

   - Create the `neutron` database:

     ```mysql
     CREATE DATABASE neutron;
     ```

   - Grant proper access to the `neutron` database, replacing `openstack-me` with a suitable password:

     ```mysql
     CREATE USER 'neutron'@'%' IDENTIFIED BY 'openstack-me';
     GRANT ALL PRIVILEGES ON neutron.* TO  'neutron'@'%';
     ```

   - Exit the database access client.

2. Source the `admin` credentials to gain access to admin-only CLI commands:

   ```
   $ . admin-openrc
   ```

3. To create the service credentials, complete these steps:

   - Create the `neutron` user:

     ```
     openstack user create --domain default --password-prompt neutron
     ```

   - Add the `admin` role to the `neutron` user:

     ```
     openstack role add --project service --user neutron admin
     ```

     Note

     This command provides no output.

   - Create the `neutron` service entity:

     ```
     openstack service create --name neutron --description "OpenStack Networking" network
     ```

4. Create the Networking service API endpoints:

   ```
   openstack endpoint create --region RegionOne network public http://vm10:9696
   openstack endpoint create --region RegionOne network internal http://vm10:9696
   openstack endpoint create --region RegionOne network admin http://vm10:9696
   ```

#### Configure networking options[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#configure-networking-options)

You can deploy the Networking service using one of two architectures represented by options 1 and 2.

Option 1 deploys the simplest possible architecture that only supports attaching instances to provider (external) networks. No self-service (private) networks, routers, or floating IP addresses. Only the `admin` or other privileged user can manage provider networks.

Option 2 augments option 1 with layer-3 services that support attaching instances to self-service networks. The `demo` or other unprivileged user can manage self-service networks including routers that provide connectivity between self-service and provider networks. Additionally, floating IP addresses provide connectivity to instances using self-service networks from external networks such as the Internet.

Self-service networks typically use overlay networks. Overlay network protocols such as VXLAN include additional headers that increase overhead and decrease space available for the payload or user data. Without knowledge of the virtual network infrastructure, instances attempt to send packets using the default Ethernet maximum transmission unit (MTU) of 1500 bytes. The Networking service automatically provides the correct MTU value to instances via DHCP. However, some cloud images do not use DHCP or ignore the DHCP MTU option and require configuration using metadata or a script. 

Note

Option 2 also supports attaching instances to provider networks.

Choose one of the following networking options to configure services specific to it. Afterwards, return here and proceed to [Configure the metadata agent](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#neutron-vm10-metadata-agent-ubuntu).

- [Networking Option 1: Provider networks](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html)
- [Networking Option 2: Self-service networks](https://docs.openstack.org/neutron/xena/install/vm10-install-option2-ubuntu.html)

#### Networking Option 1: Provider networks

Install and configure the Networking components on the *vm10* node.

##### Install the components[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html#install-the-components)

```
sudo apt install neutron-server neutron-plugin-ml2 \
  neutron-linuxbridge-agent neutron-dhcp-agent \
  neutron-metadata-agent
```

##### Configure the server component[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html#configure-the-server-component)

The Networking server component configuration includes the database, authentication mechanism, message queue, topology change notifications, and plug-in. 

Note

Default configuration files vary by distribution. You might need to add these sections and options rather than modifying existing sections and options. Also, an ellipsis (`...`) in the configuration snippets indicates potential default configuration options that you should retain.

- Edit the `/etc/neutron/neutron.conf` file and complete the following actions:

  - In the `[database]` section, configure database access:

    ```ini
    [database]
    # ...
    connection = mysql+pymysql://neutron:openstack-me@vm10/neutron
    ```

    Replace `openstack-me` with the password you chose for the database.

    Note

    Comment out or remove any other `connection` options in the `[database]` section.

  - In the `[DEFAULT]` section, enable the Modular Layer 2 (ML2) plug-in and disable additional plug-ins:

    ```ini
    [DEFAULT]
    # ...
    core_plugin = ml2
    service_plugins =
    ```

  - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

    ```ini
    [DEFAULT]
    # ...
    transport_url = rabbit://openstack:openstack-me@vm10
    ```

    Replace `openstack-me` with the password you chose for the `openstack` account in RabbitMQ.

  - In the `[DEFAULT]` and `[keystone_authtoken]` sections, configure Identity service access:

    ```ini
    [DEFAULT]
    # ...
    auth_strategy = keystone
    
    [keystone_authtoken]
    # ...
    www_authenticate_uri = http://vm10:5000
    auth_url = http://vm10:5000
    memcached_servers = vm10:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = openstack-me
    ```

    Replace `openstack-me` with the password you chose for the `neutron` user in the Identity service.

    Note

    Comment out or remove any other options in the `[keystone_authtoken]` section.

  - In the `[DEFAULT]` and `[nova]` sections, configure Networking to notify Compute of network topology changes:

    ```ini
    [DEFAULT]
    # ...
    notify_nova_on_port_status_changes = true
    notify_nova_on_port_data_changes = true
    
    [nova]
    # ...
    auth_url = http://vm10:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = openstack-me
    ```

    Replace `NOVA_PASS` with the password you chose for the `nova` user in the Identity service.

- In the `[oslo_concurrency]` section, configure the lock path:

  ```ini
  [oslo_concurrency]
  # ...
  lock_path = /var/lib/neutron/tmp
  ```

##### Configure the Modular Layer 2 (ML2) plug-in[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html#configure-the-modular-layer-2-ml2-plug-in)

The ML2 plug-in uses the Linux bridge mechanism to build layer-2 (bridging and switching) virtual networking infrastructure for instances.

- Edit the `/etc/neutron/plugins/ml2/ml2_conf.ini` file and complete the following actions:

  - In the `[ml2]` section, enable flat and VLAN networks:

    ```
    [ml2]
    # ...
    type_drivers = flat,vlan
    ```

  - In the `[ml2]` section, disable self-service networks:

    ```
    [ml2]
    # ...
    tenant_network_types =
    ```

  - In the `[ml2]` section, enable the Linux bridge mechanism:

    ```
    [ml2]
    # ...
    mechanism_drivers = linuxbridge
    ```

    Warning

    After you configure the ML2 plug-in, removing values in the `type_drivers` option can lead to database inconsistency.

  - In the `[ml2]` section, enable the port security extension driver:

    ```
    [ml2]
    # ...
    extension_drivers = port_security
    ```

  - In the `[ml2_type_flat]` section, configure the provider virtual network as a flat network:

    ```
    [ml2_type_flat]
    # ...
    flat_networks = provider
    ```

  - In the `[securitygroup]` section, enable ipset to increase efficiency of security group rules:

    ```ini
    [securitygroup]
    # ...
    enable_ipset = true
    ```

##### Configure the Linux bridge agent[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html#configure-the-linux-bridge-agent)

The Linux bridge agent builds layer-2 (bridging and switching) virtual networking infrastructure for instances and handles security groups.

- Edit the `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` file and complete the following actions:

  - In the `[linux_bridge]` section, map the provider virtual network to the provider physical network interface:

    ```
    [linux_bridge]
    physical_interface_mappings = provider:enp0s9
    ```

    Replace `PROVIDER_INTERFACE_NAME` with the name of the underlying provider physical network interface. See [Host networking](https://docs.openstack.org/neutron/xena/install/environment-networking-ubuntu.html) for more information.

  - In the `[vxlan]` section, disable VXLAN overlay networks:

    ```
    [vxlan]
    enable_vxlan = false
    ```

  - In the `[securitygroup]` section, enable security groups and configure the Linux bridge iptables firewall driver:

    ```
    [securitygroup]
    # ...
    enable_security_group = true
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ```

  - Ensure your Linux operating system kernel supports network bridge filters by verifying all the following `sysctl` values are set to `1`:

    ```
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    ```

    To enable networking bridge support, typically the `br_netfilter` kernel module needs to be loaded. Check your operating system’s documentation for additional details on enabling this module.

##### Configure the DHCP agent[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-option1-ubuntu.html#configure-the-dhcp-agent)

The DHCP agent provides DHCP services for virtual networks.

- Edit the `/etc/neutron/dhcp_agent.ini` file and complete the following actions:

  - In the `[DEFAULT]` section, configure the Linux bridge interface driver, Dnsmasq DHCP driver, and enable isolated metadata so instances on provider networks can access metadata over the network:

    ```ini
    [DEFAULT]
    # ...
    interface_driver = linuxbridge
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    enable_isolated_metadata = true
    ```

##### Create the provider network（安装neutron完成后执行）[¶](https://docs.openstack.org/install-guide/launch-instance-networks-provider.html#create-the-provider-network)

1. On the vm10 node, source the `admin` credentials to gain access to admin-only CLI commands:

   ```
   $ . admin-openrc
   ```

2. Create the network:

   ```
   openstack network create  --share --external \
     --provider-physical-network provider \
     --provider-network-type flat provider
   ```

   The `--share` option allows all projects to use the virtual network.

   The `--external` option defines the virtual network to be external. If you wish to create an internal network, you can use `--internal` instead. Default value is `internal`.

   The `--provider-physical-network provider` and `--provider-network-type flat` options connect the flat virtual network to the flat (native/untagged) physical network on the `eth1` interface on the host using information from the following files:

   `ml2_conf.ini`:

   ```
   [ml2_type_flat]
   flat_networks = provider
   ```

   `linuxbridge_agent.ini`:

   ```
   [linux_bridge]
   physical_interface_mappings = provider:enp0s9
   ```

3. Create a subnet on the network:

   ```
   openstack subnet create --network provider   --allocation-pool start=10.11.0.2,end=10.11.0.250   --dns-nameserver 114.114.114.114 --gateway 10.11.0.1   --subnet-range 10.11.0.0/24 provider-sub
   ```

   Replace `PROVIDER_NETWORK_CIDR` with the subnet on the provider physical network in CIDR notation.

   Replace `START_IP_ADDRESS` and `END_IP_ADDRESS` with the first and last IP address of the range within the subnet that you want to allocate for instances. This range must not include any existing active IP addresses.

   Replace `DNS_RESOLVER` with the IP address of a DNS resolver. In most cases, you can use one from the `/etc/resolv.conf` file on the host.

   Replace `PROVIDER_NETWORK_GATEWAY` with the gateway IP address on the provider network, typically the “.1” IP address.

   **Example**

   The provider network uses 203.0.113.0/24 with a gateway on 203.0.113.1. A DHCP server assigns each instance an IP address from 203.0.113.101 to 203.0.113.250. All instances use 8.8.4.4 as a DNS resolver.

   ```
   openstack subnet create --network provider \
     --allocation-pool start=203.0.113.101,end=203.0.113.250 \
     --dns-nameserver 8.8.4.4 --gateway 203.0.113.1 \
     --subnet-range 203.0.113.0/24 provider
   
   Created a new subnet:
   ```

Return to [Launch an instance - Create virtual networks](https://docs.openstack.org/install-guide/launch-instance.html#launch-instance-networks).



#### Configure the metadata agent[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#configure-the-metadata-agent)

The metadata agent provides configuration information such as credentials to instances.

- Edit the `/etc/neutron/metadata_agent.ini` file and complete the following actions:

  - In the `[DEFAULT]` section, configure the metadata host and shared secret:

    ```ini
    [DEFAULT]
    # ...
    nova_metadata_host = vm10
    metadata_proxy_shared_secret = openstack-me
    ```

    Replace `openstack-me` with a suitable secret for the metadata proxy.

#### Configure the Compute service to use the Networking service[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#configure-the-compute-service-to-use-the-networking-service)

Note

The Nova compute service must be installed to complete this step. For more details see the compute install guide found under the Installation Guides section of the [docs website](https://docs.openstack.org/).

- Edit the `/etc/nova/nova.conf` file and perform the following actions:

  - In the `[neutron]` section, configure access parameters, enable the metadata proxy, and configure the secret:

    ```ini
    [neutron]
    # ...
    auth_url = http://vm10:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = openstack-me
    service_metadata_proxy = true
    metadata_proxy_shared_secret = openstack-me
    ```

    Replace `openstack-me` with the password you chose for the `neutron` user in the Identity service.

    Replace `openstack-me` with the secret you chose for the metadata proxy.

    See the [compute service configuration guide](https://docs.openstack.org/nova/xena/configuration/config.html#neutron) for the full set of options including overriding the service catalog endpoint URL if necessary.

#### Finalize installation[¶](https://docs.openstack.org/neutron/xena/install/vm10-install-ubuntu.html#finalize-installation)

1. Populate the database:

   ```
   su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
     --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
   ```

   Note

   

   Database population occurs later for Networking because the script requires complete server and plug-in configuration files.

2. Restart the Compute API service:

   ```
   service nova-api restart
   ```

3. Restart the Networking services.

   For both networking options:

   ```
   sudo service neutron-server restart
   sudo service neutron-linuxbridge-agent restart
   sudo service neutron-dhcp-agent restart
   sudo service neutron-metadata-agent restart
   ```

   For networking option 2, also restart the layer-3 service:

   ```
   sudo service neutron-l3-agent restart
   ```

### horizon

#### Install and configure components[¶](https://docs.openstack.org/horizon/xena/install/install-ubuntu.html#install-and-configure-components)

Note

Default configuration files vary by distribution. You might need to add these sections and options rather than modifying existing sections and options. Also, an ellipsis (`...`) in the configuration snippets indicates potential default configuration options that you should retain.

1. Install the packages:

   ```
   sudo apt install openstack-dashboard -y
   ```

2. Edit the `/etc/openstack-dashboard/local_settings.py` file and complete the following actions:

   - Configure the dashboard to use OpenStack services on the `vm10` node:

     ```
     OPENSTACK_HOST = "vm10"
     ```

   - In the Dashboard configuration section, allow your hosts to access Dashboard:

     ```
     ALLOWED_HOSTS = ['*']
     ```

     Note

     - Do not edit the `ALLOWED_HOSTS` parameter under the Ubuntu configuration section.
     - `ALLOWED_HOSTS` can also be `['*']` to accept all hosts. This may be useful for development work, but is potentially insecure and should not be used in production. See the [Django documentation](https://docs.djangoproject.com/en/dev/ref/settings/#allowed-hosts) for further information.

   - Configure the `memcached` session storage service:

     ```python
     SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
     
     CACHES = {
         'default': {
              'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
              'LOCATION': '192.168.56.10:11211',
         }
     }
     ```

      

     Note

     

     Comment out any other session storage configuration.

   - Enable the Identity API version 3:

     ```
     OPENSTACK_KEYSTONE_URL = "http://%s:5000/identity/v3" % OPENSTACK_HOST
     ```

   - Enable support for domains:

     ```
     OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
     ```

   - Configure API versions:

     ```
     OPENSTACK_API_VERSIONS = {
         "identity": 3,
         "image": 2,
         "volume": 3,
     }
     ```

   - Configure `Default` as the default domain for users that you create via the dashboard:

     ```
     OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
     ```

   - Configure `user` as the default role for users that you create via the dashboard:

     ```
     OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
     ```

   - If you chose networking option 1, disable support for layer-3 networking services:

     ```
     OPENSTACK_NEUTRON_NETWORK = {
         ...
         'enable_router': False,
         'enable_quotas': False,
         'enable_ipv6': False,
         'enable_distributed_router': False,
         'enable_ha_router': False,
         'enable_fip_topology_check': False,
     }
     ```

   - Optionally, configure the time zone:

     ```
     TIME_ZONE = "TIME_ZONE"
     ```

     Replace `TIME_ZONE` with an appropriate time zone identifier. For more information, see the [list of time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

3. Add the following line to `/etc/apache2/conf-available/openstack-dashboard.conf` if not included.

   ```
   WSGIApplicationGroup %{GLOBAL}
   ```

#### Finalize installation[¶](https://docs.openstack.org/horizon/xena/install/install-ubuntu.html#finalize-installation)

- Reload the web server configuration:

  ```
  sudo systemctl reload apache2.service
  ```

### zun(未实施)

#### Install and configure vm10 node[¶](https://docs.openstack.org/zun/xena/install/vm10-install.html#install-and-configure-vm10-node)

This section describes how to install and configure the Container service on the vm10 node for Ubuntu 16.04 (LTS) and CentOS 7.

##### Prerequisites[¶](https://docs.openstack.org/zun/xena/install/vm10-install.html#prerequisites)

Before you install and configure Zun, you must create a database, service credentials, and API endpoints.

1. To create the database, complete these steps:

   - Use the database access client to connect to the database server as the `root` user:

     ```
     # mysql
     ```

   - Create the `zun` database:

     ```
     CREATE DATABASE zun;
     ```

   - Grant proper access to the `zun` database:

     ```
      CREATE USER 'zun'@'%' IDENTIFIED BY 'openstack-me';
      GRANT ALL PRIVILEGES ON zun.* TO  'zun'@'%'; 
     ```

     Replace `ZUN_DBPASS` with a suitable password.

   - Exit the database access client.

2. Source the `admin` credentials to gain access to admin-only CLI commands:

   ```
   $ . admin-openrc
   ```

3. To create the service credentials, complete these steps:

   - Create the `zun` user:

     ```
     openstack user create --domain default --password-prompt zun
     ```

   - Add the `admin` role to the `zun` user:

     ```
     openstack role add --project service --user zun admin
     ```

     Note

     This command provides no output.

   - Create the `zun` service entities:

     ```
     openstack service create --name zun --description "Container Service" container
     ```

4. Create the Container service API endpoints:

   ```
   openstack endpoint create --region RegionOne     container public http://vm10:9517/v1
   openstack endpoint create --region RegionOne     container internal http://vm10:9517/v1
   openstack endpoint create --region RegionOne     container admin http://vm10:9517/v1
   ```

##### Install and configure components[¶](https://docs.openstack.org/zun/xena/install/vm10-install.html#install-and-configure-components)

1. Create zun user and necessary directories:

   - Create user:

     ```
     sudo groupadd --system zun
     sudo useradd --home-dir "/var/lib/zun" \
           --create-home \
           --system \
           --shell /bin/false \
           -g zun \
           zun
     ```

   - Create directories:

     ```
     sudo mkdir -p /etc/zun
     sudo chown zun:zun /etc/zun
     ```

2. Install the following dependencies:

   For Ubuntu, run:

   ```
   sudo apt-get install python3-pip git
   ```

   For CentOS, run:

   ```
   sudo yum install python3-pip git python3-devel libffi-devel gcc openssl-devel
   ```

3. Clone and install zun:

   ```
   # cd /var/lib/zun
   # git clone https://opendev.org/openstack/zun.git
   # chown -R zun:zun zun
   # cd zun
   # pip3 install -r requirements.txt
   # python3 setup.py install
   ```

4. Generate a sample configuration file:

   ```
   # su -s /bin/sh -c "oslo-config-generator \
       --config-file etc/zun/zun-config-generator.conf" zun
   # su -s /bin/sh -c "cp etc/zun/zun.conf.sample \
       /etc/zun/zun.conf" zun
   ```

5. Copy api-paste.ini:

   ```
   # su -s /bin/sh -c "cp etc/zun/api-paste.ini /etc/zun" zun
   ```

6. Edit the `/etc/zun/zun.conf`:

   - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

     ```
     [DEFAULT]
     ...
     transport_url = rabbit://openstack:RABBIT_PASS@vm10
     ```

     Replace `RABBIT_PASS` with the password you chose for the `openstack` account in `RabbitMQ`.

   - In the `[api]` section, configure the IP address that Zun API server is going to listen:

     ```
     [api]
     ...
     host_ip = 10.0.0.11
     port = 9517
     ```

     Replace `10.0.0.11` with the management interface IP address of the vm10 node if different.

   - In the `[database]` section, configure database access:

     ```
     [database]
     ...
     connection = mysql+pymysql://zun:ZUN_DBPASS@vm10/zun
     ```

     Replace `ZUN_DBPASS` with the password you chose for the zun database.

   - In the `[keystone_auth]` section, configure Identity service access:

     ```
     [keystone_auth]
     memcached_servers = vm10:11211
     www_authenticate_uri = http://vm10:5000
     project_domain_name = default
     project_name = service
     user_domain_name = default
     password = ZUN_PASS
     username = zun
     auth_url = http://vm10:5000
     auth_type = password
     auth_version = v3
     auth_protocol = http
     service_token_roles_required = True
     endpoint_type = internalURL
     ```

   - In the `[keystone_authtoken]` section, configure Identity service access:

     ```
     [keystone_authtoken]
     ...
     memcached_servers = vm10:11211
     www_authenticate_uri = http://vm10:5000
     project_domain_name = default
     project_name = service
     user_domain_name = default
     password = ZUN_PASS
     username = zun
     auth_url = http://vm10:5000
     auth_type = password
     auth_version = v3
     auth_protocol = http
     service_token_roles_required = True
     endpoint_type = internalURL
     ```

     Replace ZUN_PASS with the password you chose for the zun user in the Identity service.

   - In the `[oslo_concurrency]` section, configure the `lock_path`:

     ```
     [oslo_concurrency]
     ...
     lock_path = /var/lib/zun/tmp
     ```

   - In the `[oslo_messaging_notifications]` section, configure the `driver`:

     ```
     [oslo_messaging_notifications]
     ...
     driver = messaging
     ```

   - In the `[websocket_proxy]` section, configure the IP address that the websocket proxy is going to listen to:

     ```
     [websocket_proxy]
     ...
     wsproxy_host = 10.0.0.11
     wsproxy_port = 6784
     base_url = ws://vm10:6784/
     ```

     

      

     Note

     

     This `base_url` will be used by end users to access the console of their containers so make sure this URL is accessible from your intended users and the port `6784` is not blocked by firewall.

     Replace `10.0.0.11` with the management interface IP address of the vm10 node if different.

   

    

   Note

   

   Make sure that `/etc/zun/zun.conf` still have the correct permissions. You can set the permissions again with:

   \# chown zun:zun /etc/zun/zun.conf

7. Populate Zun database:

   ```
   # su -s /bin/sh -c "zun-db-manage upgrade" zun
   ```

##### Finalize installation[¶](https://docs.openstack.org/zun/xena/install/vm10-install.html#finalize-installation)

1. Create an upstart config, it could be named as `/etc/systemd/system/zun-api.service`:

   

    

   Note

   

   CentOS might install binary files into `/usr/bin/`. If it does, replace `/usr/local/bin/` directory with the correct in the following example files.

   ```
   [Unit]
   Description = OpenStack Container Service API
   
   [Service]
   ExecStart = /usr/local/bin/zun-api
   User = zun
   
   [Install]
   WantedBy = multi-user.target
   ```

2. Create an upstart config, it could be named as `/etc/systemd/system/zun-wsproxy.service`:

   ```
   [Unit]
   Description = OpenStack Container Service Websocket Proxy
   
   [Service]
   ExecStart = /usr/local/bin/zun-wsproxy
   User = zun
   
   [Install]
   WantedBy = multi-user.target
   ```

3. Enable and start zun-api and zun-wsproxy:

   ```
   # systemctl enable zun-api
   # systemctl enable zun-wsproxy
   ```

   ```
   # systemctl start zun-api
   # systemctl start zun-wsproxy
   ```

4. Verify that zun-api and zun-wsproxy services are running:

   ```
   # systemctl status zun-api
   # systemctl status zun-wsproxy
   ```

#### Install and configure a compute node[¶](https://docs.openstack.org/zun/xena/install/compute-install.html#install-and-configure-a-compute-node)

This section describes how to install and configure the Compute service on a compute node.

Note

This section assumes that you are following the instructions in this guide step-by-step to configure the first compute node. If you want to configure additional compute nodes, prepare them in a similar fashion. Each additional compute node requires a unique IP address.

##### Prerequisites[¶](https://docs.openstack.org/zun/xena/install/compute-install.html#prerequisites)

Before you install and configure Zun, you must have Docker and Kuryr-libnetwork installed properly in the compute node, and have Etcd installed properly in the controller node. Refer [Get Docker](https://docs.docker.com/engine/install#supported-platforms) for Docker installation and [Kuryr libnetwork installation guide](https://docs.openstack.org/kuryr-libnetwork/latest/install), [Etcd installation guide](https://docs.openstack.org/install-guide/environment-etcd.html)

##### Install and configure components[¶](https://docs.openstack.org/zun/xena/install/compute-install.html#install-and-configure-components)

1. Create zun user and necessary directories:

   - Create user:

     ```
     # groupadd --system zun
     # useradd --home-dir "/var/lib/zun" \
           --create-home \
           --system \
           --shell /bin/false \
           -g zun \
           zun
     ```

   - Create directories:

     ```
     # mkdir -p /etc/zun
     # chown zun:zun /etc/zun
     ```

   - Create CNI directories:

     ```
     # mkdir -p /etc/cni/net.d
     # chown zun:zun /etc/cni/net.d
     ```

2. Install the following dependencies:

   For Ubuntu, run:

   ```
   # apt-get install python3-pip git numactl
   ```

   For CentOS, run:

   ```
   # yum install python3-pip git python3-devel libffi-devel gcc openssl-devel numactl
   ```

3. Clone and install zun:

   ```
   # cd /var/lib/zun
   # git clone https://opendev.org/openstack/zun.git
   # chown -R zun:zun zun
   # cd zun
   # pip3 install -r requirements.txt
   # python3 setup.py install
   ```

4. Generate a sample configuration file:

   ```
   # su -s /bin/sh -c "oslo-config-generator \
       --config-file etc/zun/zun-config-generator.conf" zun
   # su -s /bin/sh -c "cp etc/zun/zun.conf.sample \
       /etc/zun/zun.conf" zun
   # su -s /bin/sh -c "cp etc/zun/rootwrap.conf \
       /etc/zun/rootwrap.conf" zun
   # su -s /bin/sh -c "mkdir -p /etc/zun/rootwrap.d" zun
   # su -s /bin/sh -c "cp etc/zun/rootwrap.d/* \
       /etc/zun/rootwrap.d/" zun
   # su -s /bin/sh -c "cp etc/cni/net.d/* /etc/cni/net.d/" zun
   ```

5. Configure sudoers for `zun` users:

   

    

   Note

   

   CentOS might install binary files into `/usr/bin/`. If it does, replace `/usr/local/bin/` directory with the correct in the following command.

   ```
   # echo "zun ALL=(root) NOPASSWD: /usr/local/bin/zun-rootwrap \
       /etc/zun/rootwrap.conf *" | sudo tee /etc/sudoers.d/zun-rootwrap
   ```

6. Edit the `/etc/zun/zun.conf`:

   - In the `[DEFAULT]` section, configure `RabbitMQ` message queue access:

     ```
     [DEFAULT]
     ...
     transport_url = rabbit://openstack:RABBIT_PASS@controller
     ```

     Replace `RABBIT_PASS` with the password you chose for the `openstack` account in `RabbitMQ`.

   - In the `[DEFAULT]` section, configure the path that is used by Zun to store the states:

     ```
     [DEFAULT]
     ...
     state_path = /var/lib/zun
     ```

   - In the `[database]` section, configure database access:

     ```
     [database]
     ...
     connection = mysql+pymysql://zun:ZUN_DBPASS@controller/zun
     ```

     Replace `ZUN_DBPASS` with the password you chose for the zun database.

   - In the `[keystone_auth]` section, configure Identity service access:

     ```
     [keystone_auth]
     memcached_servers = controller:11211
     www_authenticate_uri = http://controller:5000
     project_domain_name = default
     project_name = service
     user_domain_name = default
     password = ZUN_PASS
     username = zun
     auth_url = http://controller:5000
     auth_type = password
     auth_version = v3
     auth_protocol = http
     service_token_roles_required = True
     endpoint_type = internalURL
     ```

   - In the `[keystone_authtoken]` section, configure Identity service access:

     ```
     [keystone_authtoken]
     ...
     memcached_servers = controller:11211
     www_authenticate_uri= http://controller:5000
     project_domain_name = default
     project_name = service
     user_domain_name = default
     password = ZUN_PASS
     username = zun
     auth_url = http://controller:5000
     auth_type = password
     ```

     Replace ZUN_PASS with the password you chose for the zun user in the Identity service.

   - In the `[oslo_concurrency]` section, configure the `lock_path`:

     ```
     [oslo_concurrency]
     ...
     lock_path = /var/lib/zun/tmp
     ```

   - (Optional) If you want to run both containers and nova instances in this compute node, in the `[compute]` section, configure the `host_shared_with_nova`:

     ```
     [compute]
     ...
     host_shared_with_nova = true
     ```

   

    

   Note

   

   Make sure that `/etc/zun/zun.conf` still have the correct permissions. You can set the permissions again with:

   \# chown zun:zun /etc/zun/zun.conf

7. Configure Docker and Kuryr:

   - Create the directory `/etc/systemd/system/docker.service.d`

     ```
     # mkdir -p /etc/systemd/system/docker.service.d
     ```

   - Create the file `/etc/systemd/system/docker.service.d/docker.conf`. Configure docker to listen to port 2375 as well as the default unix socket. Also, configure docker to use etcd3 as storage backend:

     ```
     [Service]
     ExecStart=
     ExecStart=/usr/bin/dockerd --group zun -H tcp://compute1:2375 -H unix:///var/run/docker.sock --cluster-store etcd://controller:2379
     ```

   - Restart Docker:

     ```
     # systemctl daemon-reload
     # systemctl restart docker
     ```

   - Edit the Kuryr config file `/etc/kuryr/kuryr.conf`. Set `capability_scope` to `global` and `process_external_connectivity` to `False`:

     ```
     [DEFAULT]
     ...
     capability_scope = global
     process_external_connectivity = False
     ```

   - Restart Kuryr-libnetwork:

     ```
     # systemctl restart kuryr-libnetwork
     ```

8. Configure containerd:

   - Generate config file for containerd:

     ```
     # containerd config default > /etc/containerd/config.toml
     ```

   - Edit the `/etc/containerd/config.toml`. In the `[grpc]` section, configure the `gid` as the group ID of the `zun` user:

     ```
     [grpc]
       ...
       gid = ZUN_GROUP_ID
     ```

     Replace `ZUN_GROUP_ID` with the real group ID of `zun` user. You can retrieve the ID by (for example):

     ```
     # getent group zun | cut -d: -f3
     ```

     

      

     Note

     

     Make sure that `/etc/containerd/config.toml` still have the correct permissions. You can set the permissions again with:

     \# chown zun:zun /etc/containerd/config.toml

   - Restart containerd:

     ```
     # systemctl restart containerd
     ```

9. Configure CNI:

   - Download and install the standard loopback plugin:

     ```
     # mkdir -p /opt/cni/bin
     # curl -L https://github.com/containernetworking/plugins/releases/download/v0.7.1/cni-plugins-amd64-v0.7.1.tgz \
           | tar -C /opt/cni/bin -xzvf - ./loopback
     ```

   - Install the Zun CNI plugin:

     ```
     # install -o zun -m 0555 -D /usr/local/bin/zun-cni /opt/cni/bin/zun-cni
     ```

     

      

     Note

     

     CentOS might install binary files into `/usr/bin/`. If it does, replace `/usr/local/bin/zun-cni` with the correct path in the command above.

##### Finalize installation[¶](https://docs.openstack.org/zun/xena/install/compute-install.html#finalize-installation)

1. Create an upstart config for zun compute, it could be named as `/etc/systemd/system/zun-compute.service`:

   

    

   Note

   

   CentOS might install binary files into `/usr/bin/`. If it does, replace `/usr/local/bin/` directory with the correct in the following example file.

   ```
   [Unit]
   Description = OpenStack Container Service Compute Agent
   
   [Service]
   ExecStart = /usr/local/bin/zun-compute
   User = zun
   
   [Install]
   WantedBy = multi-user.target
   ```

2. Create an upstart config for zun cni daemon, it could be named as `/etc/systemd/system/zun-cni-daemon.service`:

   

    

   Note

   

   CentOS might install binary files into `/usr/bin/`, If it does, replace `/usr/local/bin/` directory with the correct in the following example file.

   ```
   [Unit]
   Description = OpenStack Container Service CNI daemon
   
   [Service]
   ExecStart = /usr/local/bin/zun-cni-daemon
   User = zun
   
   [Install]
   WantedBy = multi-user.target
   ```

3. Enable and start zun-compute:

   ```
   # systemctl enable zun-compute
   # systemctl start zun-compute
   ```

4. Enable and start zun-cni-daemon:

   ```
   # systemctl enable zun-cni-daemon
   # systemctl start zun-cni-daemon
   ```

5. Verify that zun-compute and zun-cni-daemon services are running:

   ```
   # systemctl status zun-compute
   # systemctl status zun-cni-daemon
   ```

##### Enable Kata Containers (Optional)[¶](https://docs.openstack.org/zun/xena/install/compute-install.html#enable-kata-containers-optional)

By default, `runc` is used as the container runtime. If you want to use Kata Containers instead, this section describes the additional configuration steps.

Note

Kata Containers requires nested virtualization or bare metal. See the [official document](https://github.com/kata-containers/documentation/tree/master/install#prerequisites) for details.

1. Enable the repository for Kata Containers:

   For Ubuntu, run:

   ```
   # curl -sL http://download.opensuse.org/repositories/home:/katacontainers:/releases:/$(arch):/master/xUbuntu_$(lsb_release -rs)/Release.key | apt-key add -
   # add-apt-repository "deb http://download.opensuse.org/repositories/home:/katacontainers:/releases:/$(arch):/master/xUbuntu_$(lsb_release -rs)/ /"
   ```

   For CentOS, run:

   ```
   # yum-config-manager --add-repo "http://download.opensuse.org/repositories/home:/katacontainers:/releases:/$(arch):/master/CentOS_7/home:katacontainers:releases:$(arch):master.repo"
   ```

2. Install Kata Containers:

   For Ubuntu, run:

   ```
   # apt-get update
   # apt install kata-runtime kata-proxy kata-shim
   ```

   For CentOS, run:

   ```
   # yum install kata-runtime kata-proxy kata-shim
   ```

3. Configure Docker to add Kata Container as runtime:

   - Edit the file `/etc/systemd/system/docker.service.d/docker.conf`. Append `--add-runtime` option to add kata-runtime to Docker:

     ```
     [Service]
     ExecStart=
     ExecStart=/usr/bin/dockerd --group zun -H tcp://compute1:2375 -H unix:///var/run/docker.sock --cluster-store etcd://controller:2379 --add-runtime kata=/usr/bin/kata-runtime
     ```

   - Restart Docker:

     ```
     # systemctl daemon-reload
     # systemctl restart docker
     ```

4. Configure containerd to add Kata Containers as runtime:

   - Edit the `/etc/containerd/config.toml`. In the `[plugins.cri.containerd]` section, add the kata runtime configuration:

     ```
     [plugins]
       ...
       [plugins.cri]
         ...
         [plugins.cri.containerd]
           ...
           [plugins.cri.containerd.runtimes.kata]
             runtime_type = "io.containerd.kata.v2"
     ```

   - Restart containerd:

     ```
     # systemctl restart containerd
     ```

5. Configure Zun to use Kata runtime:

   - Edit the `/etc/zun/zun.conf`. In the `[DEFAULT]` section, configure `container_runtime` as kata:

     ```
     [DEFAULT]
     ...
     container_runtime = kata
     ```

   - Restart zun-compute:

     ```
     # systemctl restart zun-compute
     ```



## 运行实例

### 问题处理

#### Host 'vm10' is not mapped to any cell

```
sudo nova-manage cell_v2 discover_hosts --verbose
```

### image upload

```
openstack image create cirros --file .cirros-0.5.1-x86_64-disk.img
```



### flavor

