mapreduce
=======

## mapreduce的过程包含：
+ map
		I、map
		II、comparator【用于分组，将k-v转换程k-【v、v、v】】
		III、combiner【本地的reduce】
		IV、compress
+ reduce
		I、shuffling
		II、sort
		III、reduce
+ mapreduce的默认的comprator 是org.apache.hadoop.io.Text$Comparator

+ MapReduce提供Partitioner接口，它的作用就是根据key或value及reduce的数量来决定当前的这对输出数据最终应该交由哪个reduce task处理。默认对key hash后再以reduce task数量取模。默认的取模方式只是为了平均reduce的处理能力，如果用户自己对Partitioner有需求，可以订制并设置到job上。 


