Solr
======
### 研究了2天基本上讲solr的一些东西搞清楚了
####单机安装
1. 下载安装
`$ unzip -q solr-5.1.0.zip /:$ cd solr-5.1.0/`
2. 启动
`./bin/solr start`
3. 建立名称为”mycore”的core
`./bin/solr create -c mycore`
安装完成后，在solr/server/solr/mycore/下有此mycore的所有文件
4. solrcloud模式

### wiki
	https://wiki.apache.org/solr/
	https://cwiki.apache.org/confluence/display/solr/
### solr原理
#### 接口
#### 配置
####配置文件
1. mycore的配置文件在
`solr/server/solr/mycore/conf/slorconfig.xml`
	
2. schema

  在solrconfig.xml中有如下的行：

  ```
    <schemaFactory class="ManagedIndexSchemaFactory">
  			    <bool name="mutable">true</bool>
  			    <str name="managedSchemaResourceName">managed-schema</str>
  			  </schemaFactory>	
  ```

3. 请求地址相关
  
  在solr/server/solr/mycore/conf/slorconfig.xml中由如下的相关篇日志需要关注
  `<requestHandler name="/select" class="solr.SearchHandler">`
  
  ​	【使用solr的client进行调用的时候执行的http请求如：】
    			QueryResponse response = solr.query(
    			new SolrQuery().setRows(100).setSort("price", SolrQuery.ORDER.asc).set(CommonParams.Q, "price:*"));
    			SolrDocumentList list = response.getResults();
    			对应的url为：
    			GET /solr/mycore/select?rows=100&sort=price+asc&q=price%3A*&wt=javabin&version=2
    ` <requestHandler name="/update/extract" startup="lazy"  class="solr.extraction.ExtractingRequestHandler" >`
  
  【为另外一个url，可用于上传文件】
    			POST /solr/mycore/update/extract?fmap.content=attr_content&commit=true&softCommit=false&waitSearcher=true&wt=javabin&version=2
    C、几个关键字
    	fq (Filter Query)
    	fl (Field List)
    	Specifies a default field, overriding the definition of a default field in the schema.xml file.
### tomcat
	A、安装网上的教程讲solr-5.3.1移植到tomcat里，倒不是很复杂，如下几部即可
		zip -r /TOOLS/software/solr-5.3.1/server/solr-webapp/webapp solr.war
		cp solr.war tomcat/webapps
		catalina.sh run
		cp -r /TOOLS/software/solr-5.3.1/server/lib/ext /tomcat/webapps/solr/WEB-INF/lib/
		vi /tomcat/webapps/solr/WEB-INF/web.xml
			    <env-entry>
			       <env-entry-name>solr/home</env-entry-name>
			       <env-entry-value>/DOING/solr-data</env-entry-value>
			       <env-entry-type>java.lang.String</env-entry-type>
			    </env-entry>
	
	B、问题：启动tomcat后，solr可以访问，但是无法创建core
		未完待续。。。。。。	
### 中文分词
	A、选型
		经过查找后，找了一个mmseg的分词
		编写测试case后，感觉还行
	B、配置：配置也比较简答，
		I、cp cp /DOING/m2/repository/com/chenlb/mmseg4j/mmseg4j-core/1.10.0/mmseg4j-core-1.10.0.jar,/DOING/m2/repository/com/chenlb/mmseg4j/mmseg4j-solr/2.3.0/mmseg4j-solr-2.3.0.jar ${solr}/contrib/extraction/lib/
		II、vi /TOOLS/software/solr-5.3.1/server/solr/mycore/conf/managed-schema
			<fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
			    <analyzer>
				<tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/cloud/software/solr-5.3.1/dicpath"/>
			    </analyzer>
			</fieldtype>
			<fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
			    <analyzer>
				<tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/cloud/software/solr-5.3.1/dicpath/>
			    </analyzer>
			</fieldtype>
			<fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
			    <analyzer>
				<tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/cloud/software/solr-5.3.1/dicpath" />
			    </analyzer>
			</fieldtype>			
		III、需要在/cloud/software/solr-5.3.1/dicpath目录下创建词库文件words.dic,并且词库文件每行一个单词，换行符需要使用windows的

### 集群配置
	### 主机启动
		 ./solr -c -z hadoop7:2181
	### 辅机启动
		 ./solr -c -z hadoop7:2181
	### 所有的节点的服务都启动后，在其中一个节点上创建collection
		 ./solr create_collection -c cdcore -d ../server/solr/configsets/sample_techproducts_configs -shards 3 -replicationFactor 2【这个命令创建的collection可能无法通过api创建filetype】
		./solr create_collection -c cdcore -d ../server/solr/configsets/data_driven_schema_configs/ -shards 3 -replicationFactor 3
	### 重启
		./solr restart -c -z hadoop7 -m 1g
	### 异常处理
		org.apache.solr.common.SolrException: Error loading config name for collection
		问题：如果链接不上zookeeper,则使用zookeeper的zoCli.sh rmr /clusterstate.json,重启solr，然后再删除server/solr/下的分片目录，再执行
		./solr create_collection -c cdcore -d ../server/solr/configsets/data_driven_schema_configs -shards 3 -replicationFactor 2 -
	
		160107 14:22:14,607 ERROR [CloudSolrClient.java.requestWithRetryOnStaleState(902)][main] Request to collection cdcore failed due to (400) org.apache.solr.client.solrj.impl.HttpSolrClient$RemoteSolrException: Error from server at http://172.16.1.9:8983/solr/cdcore: Document is missing mandatory uniqueKey field: id, retry? 0
		./solr create_collection -c cdcore -d ../server/solr/configsets/data_driven_schema_configs -shards 4 -replicationFactor 2
	【在实地的使用中，由于share是挂在不同的机器上的，replication是复制的数量，多了影响创建索引的性能，所以可以将replicationFcator设置为2，这样就可以将数据分到4台机器上，每个数据有2个备份】
	【replicationFactor】：发现设置为2的时候，第三台机器加进去没有出来，看样子是有多少个服务器，就设置多少个分片
	### data_driven_schema_configs vs sample_techproducts_configs
		data_driven_schema_configs 可以修改schema
		sample_techproducts_configs 会抛异常 errorMessages=schema is not editable
	### 关于配置：
		solr在cloud模式下，创建core之后，相关的配置文件会提交到zookeeper，在zookeeper的"/configs/cdcore/"目录下
		zookeeper由于会持久化这些配置，所以重启zookeeper后，这些配置仍然在
		按照solr官方的推荐，是将这些配置文件下载下来作为版本控制
		使用solr的javaapi可以下载这些配置，并可以上传，相关代码我已经实现
	### 关于性能：
		由于在创建core的时候，设置了分片，所以在创建索引的时候，会产生重复的写
		如果一个core有三个，则会将数据分成三块，这三块会分发到三个不同的主机上
		如果一个share有三个分片，则这三个分片，相当于三个备份。
		如果需要提高写额性能，就需要增加share，减少分片。
		如果需要增加读的性能，？
		一个简单的增加写性能的方法：关闭三个share中的两个主机，让share写到一台机器上，待索引重建完场后，启动其他两个share所在的主机，让他们自动同步数据。
### 常用命令
	./solr delete -c cdcore
### 集群添加节点
	A、如果replicationFactor=1的时候，进行splitshared的时候，split出来的新shared只能和原来的shared在一个节点上，原理见B
	B、shared在split的时候，split的新的shared会在shared所在的leader节点上产生一个，然后在其他的节点上再进行产生一个。
	C、
### 问题处理
	ERROR: Error loading config name for collection
	关闭solr，登陆到zookepper，rmr /clusterstate.json
	#./zkCli.sh -server haoop-xx:3181

### 关于排序
	solr的排序基于Luncene，主要是评分算法，在solr中对与排序结果进行修改的可以通过如下的方法：
	第一、不同的field不同的boost，boost越大，则计算出来的评分越大。
	第二、对于一个给定boost的field，可以通过编写自定义的Similarty来进行评分的修改。其中solr默认的使用tf-idf的评分机制，但是在有些场景下并不适用，所以需要修改。如果一个field的是多值的，如使用逗号分割的，完全匹配的评分要高于部分匹配。如果要降低两者的评分差距，则需要自己实现Similarity，让结果均返回1.0f

### 记一次数据恢复过程
	A、在solr5.3的版本中有一个bug，在异常关闭服务器的情况下，导致solr启动不起来，报如下类似错误
		codec footer mismatch: actual footer=1852273495 vs expected footer=-1071082520"
	B、此问题初夏的时候，可以在日志冲查询到有一个文件名称，如_8xx.cfg.将相同shared的不同的服务器上的相同名称的文件cp到当前机器下，重启当前机器的solr

