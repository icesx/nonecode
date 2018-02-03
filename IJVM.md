### 开启GC相关的配置
	JVM的GC日志可以打开，并保存到指定文件中
	产生的文件可以通过第三方的分析工具进行分析【easygc.io】，或者自己分析
		
### -XX:ParallelGCThreads
	在使用CMS收集器的时候配置，并行收集的线程数量	
	he determining factor is the value N returned by the Java method Runtime.availableProcessors(). For N <= 8 parallel GC will use just as many, i.e., N GC threads. For N > 8 available processors, the number of GC threads will be computed as 3+5N/8.
### -XX:CMSInitiatingOccupancyFraction
	默认值为68
	-XX:CMSInitiatingOccupancyFraction=<value> where value denotes the utilization of old generation heap space in percent. For example, value=75 means that the first CMS cycle starts when 75% of the old generation is occupied
	上面介绍了promontion faild产生的原因是EDEN空间不足的情况下将EDEN与From survivor中的存活对象存入To survivor区时,To survivor区的空间不足，再次晋升到old gen区，而old gen区内存也不够的情况下产生了promontion faild从而导致full gc.那可以推断出：eden+from survivor < old gen区剩余内存时，不会出现promontion faild的情况，即：
	(Xmx-Xmn)*(1-CMSInitiatingOccupancyFraction/100)>=(Xmn-Xmn/(SurvivorRatior+2))  进而推断出：

CMSInitiatingOccupancyFraction <=((Xmx-Xmn)-(Xmn-Xmn/(SurvivorRatior+2)))/(Xmx-Xmn)*100

### gc文件的手工分析
	In normal (all most all) GC events, real time will be less than user + sys time. It’s because of multiple GC threads work concurrently to share the work load, thus real time will be less than user + sys time. Say user + sys time is 2 seconds. If 5 GC threads are concurrently working then real time should be some where in the neighbourhood of 400 milliseconds ( 2 seconds / 5 GC threads).

	But in certain circumstances you might see real time to be greater than user + sys time.

####Example:
	[Times: user=0.20 sys=0.01, real=18.45 secs]
	If you notice multiple occurrences of this scenario in your GC log then it might be indicative of one of the following problems:

	1. Heavy I/O activity
	2. Lack of CPU
	【意思是说，如果有大量的real>user+sys的情况的时候，说明系统的CPU资源不够，或者CPU在等待IO】
	【在实地的场景中，会引起HBase的regionserver挂掉】