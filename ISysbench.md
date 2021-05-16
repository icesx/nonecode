sysbench
====

### 安装

1. download

```
wget https://github.com/akopytov/sysbench/archive/1.0.19.tar.gz
```

2. install

```
sudo apt install automake pkg-config libmysqlclient-dev

```

### 测试

#### mysql

```sh
./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=bjrdc100 --mysql-port=3308 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 prepare
./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=bjrdc100 --mysql-port=3308 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
```

结果

```
SQL statistics:
    queries performed:
        read:                            5662552
        write:                           0
        other:                           808936
        total:                           6471488
    transactions:                        404468 (3369.13 per sec.)
    queries:                             6471488 (53906.09 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          120.0493s
    total number of events:              404468

Latency (ms):
         min:                                    1.29
         avg:                                   37.98
         max:                                  520.82
         95th percentile:                      106.75
         sum:                             15360963.93

Threads fairness:
    events (avg/stddev):           3159.9062/24.32
    execution time (avg/stddev):   120.0075/0.02

```



#### IO

1. 基本命令

   ```
   sysbench fileio help
   ```

2. 案例

   测试操作系统IO性能

   ```
   cd /mnt/vdb  #一定要到你测试的磁盘目录下执行，否则可能测试系统盘了
   sysbench  fileio --threads=16 --file-total-size=2G --file-test-mode=rndrw prepare
   sysbench  fileio --threads=20 --file-total-size=2G --file-test-mode=rndrw run
   sysbench  fileio --threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup
   ```

   ```
   bjrdc@bjrdc100:/moa-ceph$ sysbench  fileio --threads=20 --file-total-size=10G --file-test-mode=rndrw run
   sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)
   
   Running the test with following options:
   Number of threads: 20
   Initializing random number generator from current time
   
   
   Extra file open flags: 0
   128 files, 80MiB each
   10GiB total file size
   Block size 16KiB
   Number of IO requests: 0
   Read/Write ratio for combined random IO test: 1.50
   Periodic FSYNC enabled, calling fsync() each 100 requests.
   Calling fsync() at the end of test, Enabled.
   Using synchronous I/O mode
   Doing random r/w test
   Initializing worker threads...
   
   Threads started!
   
   
   File operations:
       reads/s:                      1154.51
       writes/s:                     769.02
       fsyncs/s:                     2450.02
   
   Throughput:
       read, MiB/s:                  18.04
       written, MiB/s:               12.02
   
   General statistics:
       total time:                          10.2396s
       total number of events:              44792
   
   Latency (ms):
            min:                                  0.00
            avg:                                  4.50
            max:                                325.01
            95th percentile:                     23.10
            sum:                             201370.01
   
   Threads fairness:
       events (avg/stddev):           2239.6000/126.07
       execution time (avg/stddev):   10.0685/0.09
   
   ```

   iops 计算

   ```
   （18.04+12.02）×1024/16（Block size）=1923.84
   ```

   