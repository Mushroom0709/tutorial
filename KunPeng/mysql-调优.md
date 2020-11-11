## 配置基础环境
1.  配置源
    `[x@centos ~]# yum -y install ncurses ncurses-devel libaio-devel zlib-devel bzip2 net-tools cmake openssl openssl-devel`
    `[x@centos ~]# yum -y install gcc* gcc-g++* libstdc++* glibc* java`
2.  升级Cmake版本为3.5.2
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
3.  升级GCC为9.3.0
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

## 安装 MYSql-5.7.27
1.  下载源码包
    `[x@centos Document]# wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.27.tar.gz`
    `[x@centos Document]# wget https://udomain.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz`
2.  预备环境
    1.  `[x@centos Document]# yum -y install bison* ncurses*`
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
    [x@centos Document]# tar -zvxf mysql-5.7.27.tar.gz
    [x@centos Document]# tar -zvxf boost_1_59_0.tar.gz
    [x@centos Document]# mkdir -p mysql-5.7.27/boost/
    [x@centos Document]# mv boost_1_59_0 mysql-5.7.27/boost/
    [x@centos Document]# cp boost_1_59_0.tar.gz mysql-5.7.27/boot/
    [x@centos Document]# cp 0001-Bug-94699-Mysql-deadlock-and-bugcheck-on-aarch64.patch mysql-5.7.27/
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
4.  配置mysql
    ```shell
    [x@centos mysql-5.7.27]# cd /usr/local/mysql/
    [x@centos mysql]# groupadd mysql
    [x@centos mysql]# useradd -g mysql mysql
    [x@centos mysql]# chown -R mysql:mysql /usr/local/mysql
    [x@centos mysql]# mkdir -p /data/log /data/data /data/run
    [x@centos mysql]# cp support-files/mysql.server /etc/init.d/mysql
    [x@centos mysql]# bin/mysqld --initialize --basedir=/usr/local/mysql --datadir=/data/data --user=mysql
    2020-11-11T03:53:38.539569Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
    2020-11-11T03:53:38.674718Z 0 [Warning] InnoDB: New log files created, LSN=45790
    2020-11-11T03:53:38.704152Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
    2020-11-11T03:53:38.767756Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 7b3d3336-23d1-11eb-86b3-fa163e2d686f.
    2020-11-11T03:53:38.770057Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    2020-11-11T03:53:38.770472Z 1 [Note] A temporary password is generated for root@localhost: u(Jjw*vzs8jf
    [x@centos mysql]# vi /data/log/mysql.log (空文件保存)
    [x@centos mysql]# vi /data/run/mysql.pid (空文件保存)
    [x@centos mysql]# chown -R mysql:mysql /data
    [x@centos mysql]# ln -s /data/data/mysql.sock /tmp/mysql.sock
    ```
    修改配置文件 `/etc/my.cnf`
    ```
    1 [mysqld]
    2 datadir=/var/lib/mysql
    3 socket=/var/lib/mysql/mysql.sock
    4 # Disabling symbolic-links is recommended to prevent assorted security risks
    5 symbolic-links=0
    6 # Settings user and group are ignored when systemd is used.
    7 # If you need to run mysqld under a different user or group,
    8 # customize your systemd unit file for mariadb according to the
    9 # instructions in http://fedoraproject.org/wiki/Systemd
    10 
    11 [mysqld_safe]
    12 log-error=/var/log/mariadb/mariadb.log
    13 pid-file=/var/run/mariadb/mariadb.pid
    14 
    15 #
    16 # include all files from the config directory
    17 #
    18 !includedir /etc/my.cnf.d
    ```
    为
    ```
    1 [mysqld]
    2 datadir=/data/data
    3 socket=/data/data/mysql.sock
    4 # Disabling symbolic-links is recommended to prevent assorted security risks
    5 symbolic-links=0
    6 # Settings user and group are ignored when systemd is used.
    7 # If you need to run mysqld under a different user or group,
    8 # customize your systemd unit file for mariadb according to the
    9 # instructions in http://fedoraproject.org/wiki/Systemd
    10 
    11 [mysqld_safe]
    12 log-error=/data/log/mysql.log
    13 pid-file=/data/run/mysql.pid
    14 
    15 #
    16 # include all files from the config directory
    17 #
    18 !includedir /etc/my.cnf.d
    ```
    在 `/etc/profile` 末尾添加
    ```shell
    export PATH=/usr/local/mysql/bin:$PATH
    ```
    `[x@centos mysql]# source /etc/profile`
5.  启动
    ```shell
    [x@centos mysql]# chkconfig mysql on
    [x@centos mysql]# service mysql start
    [x@centos mysql]# mysql -uroot -p
    ```

## 准备测试工具 BenchMarkSQL
1.  下载 [BenchMarkSQL](https://mirrors.huaweicloud.com/kunpeng/archive/kunpeng_solution/database/patch/BenchMarkSQL.zip)
2.  解压到~/software/
3.  增加mysql测试配置文件
    ```shell
    [x@centos BenchMarkSQL]# mysql -uroot -p
    mysql > create database tpcc;
    OK
    mysql > exit
    [x@centos BenchMarkSQL]# cd /home/BenchMarkSQL/run
    [x@centos run]# chmod 777 *.sh
    [x@centos run]# cp props.mysql x_mysql.properties
    [x@centos run]# vi my_mysql.properties
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