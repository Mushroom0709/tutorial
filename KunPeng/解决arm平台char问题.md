# 解决arm平台char问题
```shell
[x@centos m]# command -v gcc
/usr/bin/gcc
[x@centos m]# mv /usr/bin/gcc /usr/bin/gcc-impl
[x@centos m]# vi /usr/bin/gcc
#! /bin/sh 
/usr/bin/gcc-impl -fsigned-char "$@"
[x@centos m]# chmod +x /usr/bin/gcc
[x@centos m]# command -v g++
/usr/bin/g++
[x@centos m]# mv /usr/bin/g++ /usr/bin/g++-impl
[x@centos m]# vi /usr/bin/g++
#! /bin/sh 
/usr/bin/g++-impl -fsigned-char "$@"
[x@centos m]# chmod +x /usr/bin/g++
```