Elasticsearch
=======
### 安装

1. linux
	$vi /etc/sysctl.conf 
	添加下面配置：
	vm.max_map_count=655360
	set limit to 1024000
2. 解压之后直接运行
	$elasticsearch -d
3. config
	$vi $es/config/elasticsearch.yml
	network.host: ${HOSTNAME}
	path.data: xxx
	path.logs: yyy
	discovery.zen.pi	ng.unicast.hosts:["hadoop-cx168","hadoop-cx169","hadoop-cv170"]
2. curl localhost:9200
3. 在haoop-cx168 169 170 上分别启动es集群就起来了
	$elasticsearch -d
	
4. 创建索引
	$curl -H 'Content-Type:application/json' -X PUT hadoop-cx168:9200/blogs/i/1 -d '
{
"f_n",'jon',
"s_n",'x'
}'
###. xpack install
1. 插件
	$bin/elasticsearch-plugin  install file:///TOOLS/software/elasticsearch/x-pack/x-pack-6.2.2.zip 
	注：必须加file:///
	在线安装
	bin/elasticsearch-plugin install x-pack 
2. 设置密码
	${es_home}/bin/x-park/setup-passwords interactive
3. Install X-Pack into Kibana
	./kibana-plugin  install file:///TOOLS/software/elasticsearch/x-pack/x-pack-6.2.2.zip
4.  Add credentials to the kibana.yml file
	server.host: "solar25"
	elasticsearch.url: "http://solar25:9200"
	elasticsearch.username: "kibana"
	elasticsearch.password: "elastic"
5. Navigate to Kibana at http://localhost:5601/
Log in as the built-in elastic user with the auto-generated password from step 3
6. 安装插件后要重启才能生效
7. ik
	wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.2/elasticsearch-analysis-ik-6.4.2.zip
	/cloud/software/elasticsearch/elasticsearch-6.2.2/bin/elasticsearch-plugin  install file:///cloud/es-plugin/elasticsearch-analysis-ik-6.4.2.zip
### kibana


### plugin
1. 底层仍然使用lunce的
2. 

### type
#### keyword
1. keyword 类型
	no analyzer
2. 
#### string
1. 5.0之后去除,变为text
#### text
1. 具备分词能力

### analysis

### mapping

### query


