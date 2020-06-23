Raid
====



### 长兴测试结果

	1. 7.2K rpm的硬盘，raid5，H330【无缓存】
		连续写 50MB/S
		写入磁盘寻道延迟 100ms
	2. 7.2K rpm raid5 ,H730P[2G]
		连续写200-600MB/S
		写入磁盘寻道延迟 70ms
	3.  1W  rom raid5 ，H730P[2G]
		连续写1GB/S【虚拟机都起来后可以达到700MB/S】
		写入磁盘寻道延迟 5ms
	4.  1W和72K两个raid5的磁盘，通过vmware合并成一个后
		H330 10MB
		H730P 10MB/S，未开区wirte back
		H730P 300MB-400MB/S，开启wire back【发现开启回写缓存有一定用处，但是发现有的虚拟机只能达到10MB/S】
		写入磁盘寻道延迟 50ms
	5. 72Kraid10,
		H730P，开启write back 300MB/S

### 重庆-巴南
	1. 7.2k 12GB 2TSAS 12块 raid5 h730P-2G centos
		#dd if=/dev/zero of=test.iso bs=4k count=5000000
		756MB/S
	2. 7.2k 12GB 2TSAS 12块 raid5 h730P-2G ubuntu16.04
		#dd if=/dev/zero of=test.iso bs=4k count=5000000
		1.0GB/S
	3. centos与ubuntu两者性能差别估计在于内核版本吧。。。。。。


​	
