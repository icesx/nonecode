mysql
=========

## 安装
### 简单安装
```
$tar –zxvf mysql-enterprise-5.0.30-linux-i686-glibc23.tar.gz –C /usr/local/
$cd /usr/local/
$ln –s /cloud/mysql-enterprise-5.0.30-linux-i686-glibc23 mysql
$cd mysql
$groupadd mysql
$useradd -g mysql mysql
$chown -R mysql：mysql .
$sudo usermod  -a -G mysql docker [当前docker用户加入mysql组]
$sudo scripts/mysql_install_db --user=mysql
$cp /usr/local/mysql/support-files/my-medium.cnf /etc/my.cnf
$sudo ./mysqld_safe --user=mysql& 
```
### 配置
```
$sudo cp my-default.cnf /etc/my.cnf
$vi /etc/my.cnf
```
```
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci


[client]
default-character-set=utf8


```

### systemd

```
cat >mysql.service <<EOF
[Unit]
Description=mysql
After=network.target
[Service]
ExecStart=TODO....
KillMode=process
Restart=on-failure
[Install]
WantedBy=multi-user.target
Alias=mysql.service
EOF
```



### 编译安装

1. 安装 boost_1_59_0

   -- MySQL currently requires boost_1_59_0


*不需要安装，下载对应的版本放到指定路径即可*

1. 安装curses

   ```
   sudo apt install libncurses5-dev
   ```

3. ssl

   ```
   sudo apt install libssl-dev
   ```
   
   
   
4. 编译安装

```
cmake . -DCMAKE_INSTALL_PREFIX=/moa/mysql -DMYSQL_DATADIR=/moa/mysql/data/ -DWITH_BOOST=/home/bjrdc/software/boost_1_59_0 -DSYSCONFDIR=/moa/mysql/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DENABLE_DTRACE=0 -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EMBEDDED_SERVER=1 -DMYSQL_TCP_PORT=3308
```



3. 编译选项

   | Formats                         | Description                                                  | Default                    | Introduced | Removed |
   | ------------------------------- | ------------------------------------------------------------ | -------------------------- | ---------- | ------- |
   | `ADD_GDB_INDEX`                 | Whether to enable generation of .gdb_index section in binaries |                            | 8.0.18     |         |
   | `BUILD_CONFIG`                  | Use same build options as official releases                  |                            |            |         |
   | `BUNDLE_RUNTIME_LIBRARIES`      | Bundle runtime libraries with server MSI and Zip packages for Windows | `OFF`                      | 8.0.11     |         |
   | `CMAKE_BUILD_TYPE`              | Type of build to produce                                     | `RelWithDebInfo`           |            |         |
   | `CMAKE_CXX_FLAGS`               | Flags for C++ Compiler                                       |                            |            |         |
   | `CMAKE_C_FLAGS`                 | Flags for C Compiler                                         |                            |            |         |
   | `CMAKE_INSTALL_PREFIX`          | Installation base directory                                  | `/usr/local/mysql`         |            |         |
   | `CMAKE_INSTALL_PRIV_LIBDIR`     | Installation private library directory                       |                            | 8.0.18     |         |
   | `COMPILATION_COMMENT`           | Comment about compilation environment                        |                            |            |         |
   | `COMPILATION_COMMENT_SERVER`    | Comment about compilation environment for use by mysqld      |                            | 8.0.14     |         |
   | `CPACK_MONOLITHIC_INSTALL`      | Whether package build produces single file                   | `OFF`                      |            |         |
   | `DEFAULT_CHARSET`               | The default server character set                             | `utf8mb4`                  |            |         |
   | `DEFAULT_COLLATION`             | The default server collation                                 | `utf8mb4_0900_ai_ci`       |            |         |
   | `DISABLE_DATA_LOCK`             | Exclude the performance schema data lock instrumentation     | `OFF`                      |            |         |
   | `DISABLE_PSI_COND`              | Exclude Performance Schema condition instrumentation         | `OFF`                      |            |         |
   | `DISABLE_PSI_ERROR`             | Exclude the performance schema server error instrumentation  | `OFF`                      |            |         |
   | `DISABLE_PSI_FILE`              | Exclude Performance Schema file instrumentation              | `OFF`                      |            |         |
   | `DISABLE_PSI_IDLE`              | Exclude Performance Schema idle instrumentation              | `OFF`                      |            |         |
   | `DISABLE_PSI_MEMORY`            | Exclude Performance Schema memory instrumentation            | `OFF`                      |            |         |
   | `DISABLE_PSI_METADATA`          | Exclude Performance Schema metadata instrumentation          | `OFF`                      |            |         |
   | `DISABLE_PSI_MUTEX`             | Exclude Performance Schema mutex instrumentation             | `OFF`                      |            |         |
   | `DISABLE_PSI_PS`                | Exclude the performance schema prepared statements           | `OFF`                      |            |         |
   | `DISABLE_PSI_RWLOCK`            | Exclude Performance Schema rwlock instrumentation            | `OFF`                      |            |         |
   | `DISABLE_PSI_SOCKET`            | Exclude Performance Schema socket instrumentation            | `OFF`                      |            |         |
   | `DISABLE_PSI_SP`                | Exclude Performance Schema stored program instrumentation    | `OFF`                      |            |         |
   | `DISABLE_PSI_STAGE`             | Exclude Performance Schema stage instrumentation             | `OFF`                      |            |         |
   | `DISABLE_PSI_STATEMENT`         | Exclude Performance Schema statement instrumentation         | `OFF`                      |            |         |
   | `DISABLE_PSI_STATEMENT_DIGEST`  | Exclude Performance Schema statements_digest instrumentation | `OFF`                      |            |         |
   | `DISABLE_PSI_TABLE`             | Exclude Performance Schema table instrumentation             | `OFF`                      |            |         |
   | `DISABLE_PSI_THREAD`            | Exclude the performance schema thread instrumentation        | `OFF`                      |            |         |
   | `DISABLE_PSI_TRANSACTION`       | Exclude the performance schema transaction instrumentation   | `OFF`                      |            |         |
   | `DISABLE_SHARED`                | Do not build shared libraries, compile position-dependent code | `OFF`                      |            | 8.0.18  |
   | `DOWNLOAD_BOOST`                | Whether to download the Boost library                        | `OFF`                      |            |         |
   | `DOWNLOAD_BOOST_TIMEOUT`        | Timeout in seconds for downloading the Boost library         | `600`                      |            |         |
   | `ENABLED_LOCAL_INFILE`          | Whether to enable LOCAL for LOAD DATA                        | `OFF`                      |            |         |
   | `ENABLED_PROFILING`             | Whether to enable query profiling code                       | `ON`                       |            |         |
   | `ENABLE_DEBUG_SYNC`             | Whether to enable Debug Sync support                         | `ON`                       |            | 8.0.1   |
   | `ENABLE_DOWNLOADS`              | Whether to download optional files                           | `OFF`                      |            |         |
   | `ENABLE_DTRACE`                 | Whether to include DTrace support                            |                            |            | 8.0.1   |
   | `ENABLE_EXPERIMENTAL_SYSVARS`   | Whether to enabled experimental InnoDB system variables      | `OFF`                      | 8.0.11     |         |
   | `ENABLE_GCOV`                   | Whether to include gcov support                              |                            |            |         |
   | `ENABLE_GPROF`                  | Enable gprof (optimized Linux builds only)                   | `OFF`                      |            |         |
   | `FORCE_INSOURCE_BUILD`          | Whether to force an in-source build                          | `OFF`                      | 8.0.14     |         |
   | `FORCE_UNSUPPORTED_COMPILER`    | Whether to permit unsupported compiler                       | `OFF`                      |            |         |
   | `FPROFILE_GENERATE`             | Whether to generate profile guided optimization data         | `OFF`                      | 8.0.19     |         |
   | `FPROFILE_USE`                  | Whether to use profile guided optimization data              | `OFF`                      | 8.0.19     |         |
   | `IGNORE_AIO_CHECK`              | With -DBUILD_CONFIG=mysql_release, ignore libaio check       | `OFF`                      |            |         |
   | `INSTALL_BINDIR`                | User executables directory                                   | `PREFIX/bin`               |            |         |
   | `INSTALL_DOCDIR`                | Documentation directory                                      | `PREFIX/docs`              |            |         |
   | `INSTALL_DOCREADMEDIR`          | README file directory                                        | `PREFIX`                   |            |         |
   | `INSTALL_INCLUDEDIR`            | Header file directory                                        | `PREFIX/include`           |            |         |
   | `INSTALL_INFODIR`               | Info file directory                                          | `PREFIX/docs`              |            |         |
   | `INSTALL_LAYOUT`                | Select predefined installation layout                        | `STANDALONE`               |            |         |
   | `INSTALL_LIBDIR`                | Library file directory                                       | `PREFIX/lib`               |            |         |
   | `INSTALL_MANDIR`                | Manual page directory                                        | `PREFIX/man`               |            |         |
   | `INSTALL_MYSQLKEYRINGDIR`       | Directory for keyring_file plugin data file                  | `platform specific`        |            |         |
   | `INSTALL_MYSQLSHAREDIR`         | Shared data directory                                        | `PREFIX/share`             |            |         |
   | `INSTALL_MYSQLTESTDIR`          | mysql-test directory                                         | `PREFIX/mysql-test`        |            |         |
   | `INSTALL_PKGCONFIGDIR`          | Directory for mysqlclient.pc pkg-config file                 | `INSTALL_LIBDIR/pkgconfig` |            |         |
   | `INSTALL_PLUGINDIR`             | Plugin directory                                             | `PREFIX/lib/plugin`        |            |         |
   | `INSTALL_SBINDIR`               | Server executable directory                                  | `PREFIX/bin`               |            |         |
   | `INSTALL_SECURE_FILE_PRIVDIR`   | secure_file_priv default value                               | `platform specific`        |            |         |
   | `INSTALL_SHAREDIR`              | aclocal/mysql.m4 installation directory                      | `PREFIX/share`             |            |         |
   | `INSTALL_STATIC_LIBRARIES`      | Whether to install static libraries                          | `ON`                       |            |         |
   | `INSTALL_SUPPORTFILESDIR`       | Extra support files directory                                | `PREFIX/support-files`     |            |         |
   | `LINK_RANDOMIZE`                | Whether to randomize order of symbols in mysqld binary       | `OFF`                      | 8.0.1      |         |
   | `LINK_RANDOMIZE_SEED`           | Seed value for LINK_RANDOMIZE option                         | `mysql`                    | 8.0.1      |         |
   | `MAX_INDEXES`                   | Maximum indexes per table                                    | `64`                       |            |         |
   | `MUTEX_TYPE`                    | InnoDB mutex type                                            | `event`                    |            |         |
   | `MYSQLX_TCP_PORT`               | TCP/IP port number used by X Plugin                          | `33060`                    |            |         |
   | `MYSQLX_UNIX_ADDR`              | Unix socket file used by X Plugin                            | `/tmp/mysqlx.sock`         |            |         |
   | `MYSQL_DATADIR`                 | Data directory                                               |                            |            |         |
   | `MYSQL_MAINTAINER_MODE`         | Whether to enable MySQL maintainer-specific development environment | `OFF`                      |            |         |
   | `MYSQL_PROJECT_NAME`            | Windows/OS X project name                                    | `MySQL`                    |            |         |
   | `MYSQL_TCP_PORT`                | TCP/IP port number                                           | `3306`                     |            |         |
   | `MYSQL_UNIX_ADDR`               | Unix socket file                                             | `/tmp/mysql.sock`          |            |         |
   | `ODBC_INCLUDES`                 | ODBC includes directory                                      |                            |            |         |
   | `ODBC_LIB_DIR`                  | ODBC library directory                                       |                            |            |         |
   | `OPTIMIZER_TRACE`               | Whether to support optimizer tracing                         |                            |            |         |
   | `REPRODUCIBLE_BUILD`            | Take extra care to create a build result independent of build location and time |                            | 8.0.11     |         |
   | `SYSCONFDIR`                    | Option file directory                                        |                            |            |         |
   | `SYSTEMD_PID_DIR`               | Directory for PID file under systemd                         | `/var/run/mysqld`          |            |         |
   | `SYSTEMD_SERVICE_NAME`          | Name of MySQL service under systemd                          | `mysqld`                   |            |         |
   | `TMPDIR`                        | tmpdir default value                                         |                            |            |         |
   | `USE_LD_GOLD`                   | Whether to use GNU gold linker                               | `ON`                       |            |         |
   | `USE_LD_LLD`                    | Whether to use llvm lld linker                               | `ON`                       | 8.0.16     |         |
   | `WIN_DEBUG_NO_INLINE`           | Whether to disable function inlining                         | `OFF`                      |            |         |
   | `WITHOUT_xxx_STORAGE_ENGINE`    | xuanxiangExclude storage engine xxx from build               |                            |            |         |
   | `WITH_ANT`                      | Path to Ant for building GCS Java wrapper                    |                            | 8.0.11     |         |
   | `WITH_ASAN`                     | Enable AddressSanitizer                                      | `OFF`                      |            |         |
   | `WITH_ASAN_SCOPE`               | Enable AddressSanitizer -fsanitize-address-use-after-scope Clang flag | `OFF`                      | 8.0.4      |         |
   | `WITH_AUTHENTICATION_LDAP`      | Whether to report error if LDAP authentication plugins cannot be built | `OFF`                      | 8.0.2      |         |
   | `WITH_AUTHENTICATION_PAM`       | Build PAM authentication plugin                              | `OFF`                      |            |         |
   | `WITH_AWS_SDK`                  | Location of Amazon Web Services software development kit     |                            | 8.0.2      |         |
   | `WITH_BOOST`                    | The location of the Boost library sources                    |                            |            |         |
   | `WITH_CLIENT_PROTOCOL_TRACING`  | Build client-side protocol tracing framework                 | `ON`                       |            |         |
   | `WITH_CURL`                     | Location of curl library                                     |                            | 8.0.2      |         |
   | `WITH_DEBUG`                    | Whether to include debugging support                         | `OFF`                      |            |         |
   | `WITH_DEFAULT_COMPILER_OPTIONS` | Whether to use default compiler options                      | `ON`                       |            |         |
   | `WITH_DEFAULT_FEATURE_SET`      | Whether to use default feature set                           | `ON`                       |            |         |
   | `WITH_EDITLINE`                 | Which libedit/editline library to use                        | `bundled`                  |            |         |
   | `WITH_GMOCK`                    | Path to googlemock distribution                              |                            |            |         |
   | `WITH_ICU`                      | Type of ICU support                                          | `bundled`                  | 8.0.4      |         |
   | `WITH_INNODB_EXTRA_DEBUG`       | Whether to include extra debugging support for InnoDB.       | `OFF`                      |            |         |
   | `WITH_INNODB_MEMCACHED`         | Whether to generate memcached shared libraries.              | `OFF`                      |            |         |
   | `WITH_JEMALLOC`                 | Whether to link with -ljemalloc                              | `OFF`                      | 8.0.16     |         |
   | `WITH_KEYRING_TEST`             | Build the keyring test program                               | `OFF`                      |            |         |
   | `WITH_LIBEVENT`                 | Which libevent library to use                                | `bundled`                  |            |         |
   | `WITH_LIBWRAP`                  | Whether to include libwrap (TCP wrappers) support            | `OFF`                      |            |         |
   | `WITH_LOCK_ORDER`               | Whether to enable LOCK_ORDER tooling                         | `OFF`                      | 8.0.17     |         |
   | `WITH_LSAN`                     | Whether to run LeakSanitizer, without AddressSanitizer       | `OFF`                      | 8.0.16     |         |
   | `WITH_LTO`                      | Enable link-time optimizer                                   | `OFF`                      | 8.0.13     |         |
   | `WITH_LZ4`                      | Type of LZ4 library support                                  | `bundled`                  |            |         |
   | `WITH_LZMA`                     | Type of LZMA library support                                 | `bundled`                  | 8.0.4      | 8.0.16  |
   | `WITH_MECAB`                    | Compiles MeCab                                               |                            |            |         |
   | `WITH_MSAN`                     | Enable MemorySanitizer                                       | `OFF`                      |            |         |
   | `WITH_MSCRT_DEBUG`              | Enable Visual Studio CRT memory leak tracing                 | `OFF`                      |            |         |
   | `WITH_MYSQLX`                   | Whether to disable X Protocol                                | `ON`                       | 8.0.11     |         |
   | `WITH_NUMA`                     | Set NUMA memory allocation policy                            |                            |            |         |
   | `WITH_PROTOBUF`                 | Which Protocol Buffers package to use                        | `bundled`                  |            |         |
   | `WITH_RAPID`                    | Whether to build rapid development cycle plugins             | `ON`                       |            |         |
   | `WITH_RAPIDJSON`                | Type of RapidJSON support                                    | `bundled`                  | 8.0.13     |         |
   | `WITH_RE2`                      | Type of RE2 library support                                  | `bundled`                  | 8.0.4      | 8.0.18  |
   | `WITH_ROUTER`                   | Whether to build MySQL Router                                | `ON`                       | 8.0.16     |         |
   | `WITH_SSL`                      | Type of SSL support                                          | `system`                   |            |         |
   | `WITH_SYSTEMD`                  | Enable installation of systemd support files                 | `OFF`                      |            |         |
   | `WITH_SYSTEM_LIBS`              | Set system value of library options not set explicitly       | `OFF`                      | 8.0.11     |         |
   | `WITH_TEST_TRACE_PLUGIN`        | Build test protocol trace plugin                             | `OFF`                      |            |         |
   | `WITH_TSAN`                     | Enable ThreadSanitizer                                       | `OFF`                      |            |         |
   | `WITH_UBSAN`                    | Enable Undefined Behavior Sanitizer                          | `OFF`                      |            |         |
   | `WITH_UNIT_TESTS`               | Compile MySQL with unit tests                                | `ON`                       |            |         |
   | `WITH_UNIXODBC`                 | Enable unixODBC support                                      | `OFF`                      |            |         |
   | `WITH_VALGRIND`                 | Whether to compile in Valgrind header files                  | `OFF`                      |            |         |
   | `WITH_ZLIB`                     | Type of zlib support                                         | `bundled`                  |            |         |
   | `WITH_ZSTD`                     | Type of zstd support                                         | `bundled`                  | 8.0.18     |         |
   | `WITH_xxx_STORAGE_ENGINE`       | Compile storage engine xxx statically into server            |                            |            |         |

   | Formats | Description | Default | Introduced | Removed |
   | ------- | ----------- | ------- | ---------- | ------- |
   |         |             |         |            |         |

5. 初始化

```
mysqld --initialize [with random root password]

mysqld --initialize-insecure [with blank root password]

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

如果没有设置root密码，登录后使用mysql交互命令，有如下错误：

```
You must reset your password using ALTER USER statement before executing this statement
```

解决办法：

```
set password=password('zgjx@321');
```

## 密码权限

```
create database cdc default charset uft8;
GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON cdc.* TO 'bjrdc'@'%' IDENTIFIED BY  'xxxx@123';
```



```
GRANT ALL PRIVILEGES ON dbt2.* TO 'bjrdc'@'%';
```



## 基准测试

### DBT2

1. download

https://downloads.mysql.com/source/dbt2-0.37.50.15.tar.gz

2. 安装

```
  sudo apt install autoconf
  ./configure --prefix=/home/bjrdc/software/dbt2 --with-mysql --with-mysql-includes=/moa/mysql/include --with-mysql-libs=/moa/mysql/lib
  make
  make install 
```

3. 使用

```
./datagen -w 3 -d ./dbt2-w3 --mysql
./mysql_load_db.sh --path ../dbt2-w3 --host localhost --mysql-path /moa/mysql/bin/mysql --password dbt2
```

使用的过程中发现使用无法跑过去

### Sysbench

[参考]: ./ISysbench.md

### 性能测试

#### 准备工作

```
create database dbt2
GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON dbt2.* TO 'bjrdc'@'%' IDENTIFIED BY  'zgjx@321';
flush privileges;
```



### mysql base

> 测试脚本
>
> ```
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3306 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 prepare
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3306 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
> ```
>
> 测试结果
>
> ```
> SQL statistics:
>     queries performed:
>         read:                            2093994
>         write:                           598284
>         other:                           299142
>         total:                           2991420
>     transactions:                        149571 (1245.12 per sec.)
>     queries:                             2991420 (24902.44 per sec.)
>     ignored errors:                      0      (0.00 per sec.)
>     reconnects:                          0      (0.00 per sec.)
> 
> General statistics:
>     total time:                          120.1238s
>     total number of events:              149571
> 
> Latency (ms):
>          min:                                    2.81
>          avg:                                  102.75
>          max:                                  712.80
>          95th percentile:                      231.53
>          sum:                             15368439.04
> 
> Threads fairness:
>     events (avg/stddev):           1168.5234/17.44
>     execution time (avg/stddev):   120.0659/0.04
> ```

#### mysql 8 编译安装

> 测试脚本
>
> ```
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3308 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 prepare
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3308 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
> ```
>
> 测试结果
>
> ```
> SQL statistics:
>        queries performed:
>            read:                            2025968
>            write:                           578845
>            other:                           289423
>            total:                           2894236
>        transactions:                        144711 (1204.96 per sec.)
>        queries:                             2894236 (24099.30 per sec.)
>        ignored errors:                      1      (0.01 per sec.)
>        reconnects:                          0      (0.00 per sec.)
> 
> General statistics:
>        total time:                          120.0944s
>        total number of events:              144711
> 
> Latency (ms):
>             min:                                    2.77
>             avg:                                  106.17
>             max:                                  808.00
>             95th percentile:                      248.83
>             sum:                             15363556.38
> 
> Threads fairness:
>        events (avg/stddev):           1130.5547/17.31
>        execution time (avg/stddev):   120.0278/0.03
> 
> ```

#### on ceph(cephfs)

> 测试脚本
>
> ```
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=bjrdc100 --mysql-port=3307 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 prepare
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=bjrdc100 --mysql-port=3307 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
> ```
>
> 测试结果
>
> ```
> SQL statistics:
>     queries performed:
>         read:                            420882
>         write:                           120252
>         other:                           60126
>         total:                           601260
>     transactions:                        30063  (250.04 per sec.)
>     queries:                             601260 (5000.82 per sec.)
>     ignored errors:                      0      (0.00 per sec.)
>     reconnects:                          0      (0.00 per sec.)
> 
> General statistics:
>     total time:                          120.2306s
>     total number of events:              30063
> 
> Latency (ms):
>          min:                                    8.96
>          avg:                                  511.43
>          max:                                 6588.60
>          95th percentile:                     1561.52
>          sum:                             15375224.85
> 
> Threads fairness:
>     events (avg/stddev):           234.8672/10.39
>     execution time (avg/stddev):   120.1189/0.04
> 
> ```

### on ceph(cephfs) localhost

> 测试脚本
>
> ```
> ./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3307 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
> ```
>
> 测试结果
>
> ```
> SQL statistics:
>     queries performed:
>         read:                            2069970
>         write:                           591420
>         other:                           295710
>         total:                           2957100
>     transactions:                        147855 (1231.25 per sec.)
>     queries:                             2957100 (24625.05 per sec.)
>     ignored errors:                      0      (0.00 per sec.)
>     reconnects:                          0      (0.00 per sec.)
> 
> General statistics:
>     total time:                          120.0831s
>     total number of events:              147855
> 
> Latency (ms):
>          min:                                    2.95
>          avg:                                  103.92
>          max:                                  676.48
>          95th percentile:                      235.74
>          sum:                             15364726.46
> 
> Threads fairness:
>     events (avg/stddev):           1155.1172/16.43
>     execution time (avg/stddev):   120.0369/0.02
> 
> ```
>
> 

#### on ceph (rbd)

>首先挂在ceph-rdb卷
>
>```
>sudo rbd map rdb_pool_01/volume01
>```
>
>
>
>测试脚本
>
>```
>./sysbench ../share/sysbench/oltp_read_write.lua --mysql-host=localhost --mysql-port=3309 --mysql-user=bjrdc --mysql-password='zgjx@321' --mysql-db=dbt2 --db-driver=mysql --tables=10 --table-size=1000000 --report-interval=10 --threads=128 --time=120 run
>```
>
>测试结果
>
>```
>SQL statistics:
>queries performed:
>   read:                            1998164
>   write:                           570902
>   other:                           285451
>   total:                           2854517
>transactions:                        142725 (1188.39 per sec.)
>queries:                             2854517 (23767.96 per sec.)
>ignored errors:                      1      (0.01 per sec.)
>reconnects:                          0      (0.00 per sec.)
>
>General statistics:
>total time:                          120.0975s
>total number of events:              142725
>
>Latency (ms):
>    min:                                    2.80
>    avg:                                  107.66
>    max:                                  712.90
>    95th percentile:                      235.74
>    sum:                             15365546.17
>
>Threads fairness:
>events (avg/stddev):           1115.0391/16.93
>execution time (avg/stddev):   120.0433/0.02
>```
>
>

### 测试结论

1. 自己编译的版本没有发行版本性能好，1/2（原因是发行版本测试使用的localhost）

2. ceph的性能最差，是发行版本的1/5（原因是发行版本测试使用的localhost）

3. 但是如果将测试脚本中的`--mysql-host=bjrdc100`都修改为`--mysql-host=localhost`，则性能基本相同都是1200左右的tps

4. ceph的cephfs和rbd良种模式差别不大

   ​	

## Master-Slaver

### master

1. my.cnf,add config as blow

   ```
   server-id = 1
   log_bin = /usr/local/mysql/data/mysql-bin.log
   log_bin_index =/usr/local/mysql/data/mysql-bin.log.index
   relay_log = /usr/local/mysql/data/mysql-relay-bin
   relay_log_index = /usr/local/mysql/data/mysql-relay-bin.index
   
   ```

2. 重启mysql

3. set privileges

   ```
   create user 'replication_user'@'%' identified by 'PASSWORD';
   GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
   FLUSH PRIVILEGES;
   ```

4. 查看master状态

   ```
   mysql> show master status;
   +------------------+-----------+--------------+------------------+-------------------+
   | File             | Position  | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +------------------+-----------+--------------+------------------+-------------------+
   | mysql-bin.000011 | 959803933 |              |                  |                   |
   +------------------+-----------+--------------+------------------+-------------------+
   1 row in set (0.00 sec)
   
   ```

   

### slaver

1. my.cnfg

   ```
   server-id = 2
   log_bin = /usr/local/mysql/data/mysql-bin.log
   log_bin_index =/usr/local/mysql/data/mysql-bin.log.index
   relay_log = /usr/local/mysql/data/mysql-relay-bin
   relay_log_index = /usr/local/mysql/data/mysql-relay-bin.index
   ```

2. 重启mysql

3. set privielges

   ```
   stop slave
   CHANGE MASTER TO MASTER_HOST = 'bjrdc100', MASTER_USER = 'replication_user', MASTER_LOG_FILE = 'mysql-bin.000010', MASTER_LOG_POS =82985581 , MASTER_PASSWORD = 'PASSWORD';
   start slave
   ```

4. 注意事项

   > 可以使用`reset slave`来恢复slave配置，在出现`Slave failed to initialize relay log info structure from the repositor`错误的时候

   > 如果slaver断开，重新配置，要在master上重新查询master的状态。

### 测试

​	此时从master上创建表，在slaver上可以看到表自动创建了。

## 使用

### 创建用户

1. 第一种

```
GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON cdc.* TO 'docker'@'%' IDENTIFIED BY  'xxxx@123';

GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON bdp.* TO 'bjrdc'@'%' IDENTIFIED BY  'xxxx@123';
```
2. 第二种

```
CREATE USER 'bjrdc'@'%' IDENTIFIED BY 'zgjx@321';
GRANT ALL PRIVILEGES ON * . * TO 'bjrdc'@'%';
```



### 权限处理

```
./mysqld_safe --skip-grant-table
```
### 异常处理

ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

to do this
```
$ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```
### 关闭
```
$mysqladmin -u root -p shutdown
```

### 修改密码
```
$mysql -u root -p
USE mysql
update user set host='%', Password=password('ipanelappsmgs123456') where User='root';
flush privileges;
```
### 显示链接
```
$./mysqladmin -uadmin -p -h10.140.1.1 processlist
$./mysqladmin  -uadmin -p -h10.140.1.1 status
```
### 异常处理
Permissions issue can't find file /mysql/host.frm error code errno 13£º
```
chown -R root . 
chown -R mysql data
chgrp -R mysql . 
```
### 参数调整
```
log-queries-not-using-indexes = nouseindex.log 
log-error=log-error.log
log=log-query.log
log-queries-not-using-indexes
log-warnings=2
log-slow-queries=log-slow-query.log
log-update=log-update.log
long_query_time=2

general_log=ON
general_log_file=/tmp/mysql.log
```
### 删除多余链接
```
for i in `mysql -u root -pgehua -e "show processlist;"|grep Sleep|awk '{print $1}'`; do mysql -uroot -pgehua -e "kill $i;";done; 
```
### 动态调整参数
```
mysql> show variables like '%sort_buffer_size%';
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)

mysql> set SESSION sort_buffer_size=7000000;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 7000000    |
+---------------------------+------------+
1 rows in set (0.00 sec)


mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)


mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)

ÓÃset GLOBAL ÃüÁîÉèÖÃÈ«ŸÖ±äÁ¿

mysql> set GLOBAL sort_buffer_size = 7000000;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%sort_buffer_size%';         
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)

mysql> show variables like '%sort_buffer_size%';
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 7000000    |
+---------------------------+------------+
1 rows in set (0.00 sec)
```


### 自动备份

#### imysql_backup[mine].sh
```
#!/bin/bash
#mql_backup.sh: backup mysql databases and keep newest 5 days backup.
#
# ----------------------------------------------------------------------
# This is a free shell script under GNU GPL version 2.0 or above
# Copyright (C) 2006 Sam Tang
# Feedback/comment/suggestions : http://www.real-blog.com/
# ----------------------------------------------------------------------
# your mysql login information
# db_user is mysql username
# db_passwd is mysql password
# db_host is mysql host
# -----------------------------
db_user="root"
db_passwd="i"
db_host="localhost"

# the directory for story your backup file.
backup_dir="/i/mysql_auto_backup"
# date format for backup file (dd-mm-yyyy)
time="$(date +"%Y_%m_%d")"
# mysql, mysqldump and some other bin's path
MYSQL="/usr/local/mysql/bin/mysql"
MYSQLDUMP="/usr/local/mysql/bin/mysqldump"
MKDIR="/bin/mkdir"
RM="/bin/rm"
MV="/bin/mv"
GZIP="/bin/gzip"
# check the directory for store backup is writeable
test ! -w $backup_dir && echo "Error: $backup_dir is un-writeable." && exit 0
# the directory for story the newest backup
test ! -d "$backup_dir/backup.0/" && $MKDIR "$backup_dir/backup.0/"
# get all databases
all_db="$($MYSQL -u $db_user -h $db_host -p$db_passwd -Bse 'show databases')"
for db in $all_db
do
	$MYSQLDUMP -u $db_user -h $db_host -p$db_passwd $db | $GZIP -9 > "$backup_dir/backup.0/$time.$db.gz"
done	
	# delete the oldest backup
	test -d "$backup_dir/backup.16/" && $RM -rf "$backup_dir/backup.16"
	# rotate backup directory
for int in 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
do
	if(test -d "$backup_dir"/backup."$int")
	then
		next_int=`expr $int + 1`
		$MV "$backup_dir"/backup."$int" "$backup_dir"/backup."$next_int"
	fi
done
exit 0;
```
#### xtrabackup

> TODO

### 查询链接

`./mysqladmin -uadmin -p -h10.140.1.1 processlist`

### 查看链接状态
`./mysqladmin  -uadmin -p -h10.140.1.1 status`



