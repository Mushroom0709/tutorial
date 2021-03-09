## 配置基础环境
1.  配置源 
    ```shell
    [x@centos ~]# yum -y install ncurses ncurses-devel libaio-devel zlib-devel bzip2 net-tools cmake openssl openssl-devel
    [x@centos ~]# yum -y install gcc* gcc-g++* libstdc++* glibc* java libaio
    ```
2.  升级Cmake版本为3.5.2 (为编译mysql准备，yum安装则不需要此步骤)
    ```shell
    [x@centos Document]# tar -zvxf cmake-3.5.2.tar.gz
    [x@centos Document]# cd cmake-3.5.2
    [x@centos cmake-3.5.2]# ./bootstrap
    [x@centos cmake-3.5.2]# make -j 96
    [x@centos cmake-3.5.2]# make install
    [x@centos cmake-3.5.2]# cmake -version
    cmake version 3.5.2
    
    CMake suite maintained and supported by Kitware (kitware.com/cmake).
    [x@centos cmake-3.5.2]# cd ../
    ```
3.  升级GCC为9.3.0 (为编译mysql准备，yum安装则不需要此步骤)
    1.  相关依赖下载([仓库地址](https://ftp.gnu.org/gnu/))
        +   [gmp-6.1.0](https://ftp.gnu.org/gnu/gmp/gmp-6.1.0.tar.bz2)
        +   [mpfr-3.1.4](https://ftp.gnu.org/gnu/mpfr/mpfr-3.1.4.tar.bz2)
        +   [mpc-1.0.3](https://ftp.gnu.org/gnu/mpc/mpc-1.0.3.tar.gz)
        +   [isl-0.18](http://isl.gforge.inria.fr/isl-0.18.tar.bz2)
    2.  解压相关依赖
        ```shell
        [x@centos Document]# tar -vxf gmp-6.1.0.tar.bz2
        [x@centos Document]# tar -vxf mpfr-3.1.4.tar.bz2
        [x@centos Document]# tar -zvxf mpc-1.0.3.tar.gz
        [x@centos Document]# tar -vxf isl-0.18.tar.bz2
        [x@centos Document]# tar -zvxf gcc-9.3.0.tar.gz
        [x@centos Document]# mv gmp-6.1.0  gcc-9.3.0/gmp
        [x@centos Document]# mv mpfr-3.1.4  gcc-9.3.0/mpfr
        [x@centos Document]# mv mpc-1.0.3  gcc-9.3.0/mpc
        [x@centos Document]# mv isl-0.18  gcc-9.3.0/isl
        ```
    2.  编译gcc
        ```shell
        [x@centos Document]# cd gcc-9.3.0
        [x@centos gcc-9.3.0]# ./configure \
                              --prefix=/usr \
                              --mandir=/usr/share/man  \
                              --infodir=/usr/share/info \
                              --enable-languages=c,c++,fortran \
                              --enable-shared \
                              --enable-linker-build-id \
                              --without-included-gettext \
                              --enable-threads=posix \
                              --disable-multilib \
                              --disable-nls \
                              --disable-libsanitizer \
                              --disable-browser-plugin \
                              --enable-checking=release \
                              --build=aarch64-linux
        [x@centos gcc-9.3.0]# make -j 96
        [x@centos gcc-9.3.0]# make install
        [x@centos gcc-9.3.0]# ldconfig
        [x@centos gcc-9.3.0]# gcc -v
        Using built-in specs.
        COLLECT_GCC=gcc
        COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/aarch64-linux/9.3.0/lto-wrapper
        Target: aarch64-linux
        Configured with: ./configure --enable-languages=c,c++,fortran --enable-shared --enable-linker-build-id --without-included-gettext --enable-threads=posix --disable-multilib --disable-nls --disable-libsanitizer --disable-browser-plugin --enable-checking=release --build=aarch64-linux
        Thread model: posix
        gcc version 9.3.0 (GCC)
        ```

## 编译安装 MySQL-5.7.27
1.  下载源码包
    ```shell
    [x@centos Document]# wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.27.tar.gz
    [x@centos Document]# wget https://udomain.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
    ```
2.  预备环境
    1.  解压并配置boost
    ```shell
    [x@centos Document]# yum -y install bison* ncurses*
    [x@centos Document]# tar -zvxf mysql-5.7.27.tar.gz
    [x@centos Document]# tar -zvxf boost_1_59_0.tar.gz
    [x@centos Document]# mkdir -p mysql-5.7.27/boost/
    [x@centos Document]# mv boost_1_59_0 mysql-5.7.27/boost/  
    ```
    2.  修改文件 `mysql-5.7.27/sql/mysqld.cc`
    ```shell
    16 #include "mysqld.h"
    17 #include "mysqld_daemon.h"
    18 
    19 #include <vector>
    20 #include <algorithm>
    21 #include <functional>
    22 #include <list>
    23 #include <set>
    24 #include <string>
    ```
    为
    ```shell
    16 #include "mysqld.h"
    17 #include "mysqld_daemon.h"
    18 
    19 #include <sys/prctl.h>
    20 
    21 #include <vector>
    22 #include <algorithm>
    23 #include <functional>
    24 #include <list>
    25 #include <set>
    26 #include <string>
    ```
3.  安装
    ```shell
    [x@centos Document]# cd mysql-5.7.27
    [x@centos mysql-5.7.27]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
                             -DMYSQL_DATADIR=/data/data \
                             -DSYSCONFDIR=/etc \
                             -DWITH_INNOBASE_STORAGE_ENGINE=1 \
                             -DWITH_PARTITION_STORAGE_ENGINE=1 \
                             -DWITH_FEDERATED_STORAGE_ENGINE=1 \
                             -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
                             -DWITH_MYISAM_STORAGE_ENGINE=1 \
                             -DENABLED_LOCAL_INFILE=1 \
                             -DENABLE_DTRACE=0 \
                             -DDEFAULT_CHARSET=utf8mb4 \
                             -DDEFAULT_COLLATION=utf8mb4_general_ci \
                             -DWITH_EMBEDDED_SERVER=1 \
                             -DDOWNLOAD_BOOST=1 \
                             -DWITH_BOOST=boost/boost_1_59_0
    [x@centos mysql-5.7.27]# make -j96
    [x@centos mysql-5.7.27]# make install
    ```
## yum安装 MySQL-5.7.27
1.  更换MySQL鲲鹏源地址
    ```shell
    [x@centos ~]# mv /etc/yum.repos.d/ /etc/yum.repos.d-bak
    [x@centos ~]# mkdir /etc/yum.repos.d
    [x@centos ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-AltArch-7.repo
    ```
    建立 `/etc/yum.repos.d/CentOS-Base-kunpeng.repo` 并写入
    ```shell
    [kunpeng]
    name=CentOS-kunpeng - Base - mirrors.huaweicloud.com
    baseurl=https://mirrors.huaweicloud.com/kunpeng/yum/el/7/aarch64/
    gpgcheck=0
    enabled=1
    ```
    ```shell
    [x@centos ~]# yum clean all
    [x@centos ~]# yum makecache
    ```
2.  安装MySQL
    ```shell
    [x@centos ~]# yum install -y mysql-5.7.27-1.el7.aarch64 --enablerepo=[kunpeng]
    ```

## 配置MySQL
1.  初始化
    ```shell
    [x@centos ~]# mkdir -p /home/mysql
    [x@centos ~]# chown -R mysql:mysql /home/mysql/
    [x@centos ~]# passwd mysql (可选)
    [x@centos ~]# chown -R mysql:mysql /usr/local/mysql
    [x@centos ~]# mkdir -p /data/mysql/log /data/mysql/data /data/mysql/run /data/mysql/tmp
    [x@centos ~]# chown -R mysql:mysql /data/mysql
    ```

2.  删除并新建 `/etc/my.cnf` 填如以下内容
    ```shell
    [mysqld_safe] 
    log-error=/data/mysql/log/mysql.log
    pid-file=/data/mysql/run/mysql.pid
    
    [client] 
    socket=/data/mysql/run/mysql.sock 
    default-character-set=utf8 
    
    [mysqld] 
    basedir=/usr/local/mysql
    tmpdir=/data/mysql/tmp 
    datadir=/data/mysql/data 
    socket=/data/mysql/run/mysql.sock 
    port=3306 
    user=mysql
    innodb_page_size=4096 #设置页大小 
    ssl=0 #关闭ssl 
    transaction-isolation=READ-COMMITTED #修改默认隔离级别
    
    #buffers 
    innodb_buffer_pool_size=250G #设置buffer pool size,一般为服务器内存60% 
    innodb_buffer_pool_instances=64 #设置buffer pool instance个数，提高并发能力 
    innodb_log_buffer_size=256M #设置log buffer size大小 
    
    #tune 
    sync_binlog=1 #设置每次sync_binlog事务提交刷盘 
    innodb_flush_log_at_trx_commit=1 #每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush(刷到磁盘)中去 
    innodb_use_native_aio=1 #开启异步IO 
    innodb_spin_wait_delay=180 #设置spin_wait_delay 参数，防止进入系统自旋 
    innodb_sync_spin_loops=25  #设置spin_loops 循环次数，防止进入系统自旋 
    innodb_flush_method=O_DIRECT #设置innodb数据文件及redo log的打开、刷写模式
    innodb_page_cleaners=32  #设置将脏数据写入到磁盘的线程数 
    
    #perf special 
    innodb_flush_neighbors=0 #检测该页所在区(extent)的所有页，如果是脏页，那么一起进行刷新，SSD关闭该功能
    ```
    在 `/etc/profile` 末尾添加
    ```shell
    export PATH=/usr/local/mysql/bin:$PATH
    ```
    ```shell
    [x@centos mysql]# source /etc/profile
    [x@centos mysql]# chown -R mysql:mysql /etc/my.cnf
    ```
4.  启动
    ```shell
    [x@centos ~]# mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql --datadir=/data/mysql/data --user=mysql
    2020-11-11T03:53:38.539569Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
    2020-11-11T03:53:38.674718Z 0 [Warning] InnoDB: New log files created, LSN=45790
    2020-11-11T03:53:38.704152Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
    2020-11-11T03:53:38.767756Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 7b3d3336-23d1-11eb-86b3-fa163e2d686f.
    2020-11-11T03:53:38.770057Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    2020-11-11T03:53:38.770472Z 1 [Note] A temporary password is generated for root@localhost: u(Jjw*vzs8jf
    [x@centos ~]# chkconfig mysql on
    [x@centos ~]# mysqld --defaults-file=/etc/my.cnf --user=mysql &
    [x@centos ~]# mysql -uroot -p
    mysql > ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_passwd'; (可选)
    Query OK, 0 row affected (0.00 sec)
    mysql > flush privileges; (上一条指令执行后执行)
    Query OK, 1 row affected (0.00 sec)
    ```

## 准备测试工具 BenchMarkSQL
1.  下载 [BenchMarkSQL](https://mirrors.huaweicloud.com/kunpeng/archive/kunpeng_solution/database/patch/BenchMarkSQL.zip)
2.  解压到~/software/
3.  增加mysql测试配置文件
    ```shell
    [x@centos BenchMarkSQL]# mysql -uroot -p
    mysql > create database tpcc;
    Query OK, 1 row affected (0.00 sec)
    mysql > exit
    [x@centos BenchMarkSQL]# cd /home/BenchMarkSQL/run
    [x@centos run]# chmod 777 *.sh
    [x@centos run]# cp props.mysql x_mysql.properties
    [x@centos run]# vi x_mysql.properties
    ```
    |参数       |值                     |说明        |
    |-----------|----------------------:|:---------:|
    |conn       |dbc:mysql://ip:port/db |db链接     |
    |user       |                       |用户名     |
    |password   |                       |密码       |
    |warehouses |                       |初始化加载数据时，需要创建多少仓库的数据。例如输入100，则创建100仓库数据，每一个数据仓库的数据量大概是 76823.04KB，可有少量的上下浮动，因为测试过程中将会插入或删除现有记录。|
    |loadworker |                       |表示加载数据时，每次提交进程数。|
4.  测试
    ```shell
    [x@centos run]# ./runDatabaseBuild.sh x_mysql.properties (建立测试db)
    [x@centos run]# ./runBenchmark.sh x_mysql.properties (测试)
    [x@centos run]# ./runDatabaseDestroy.sh x_mysql.properties (删除测试db)
    ```   