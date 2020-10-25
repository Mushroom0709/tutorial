# 安装
### 一、配置JDK
1.  找到OpneJDK所在位置  
    ```shell
    [x@centos7 ~]# which java
    /usr/bin/java
    [x@centos7 ~]# ls -lrt /usr/bin/java
    lrwxrwxrwx. 1 root root 22 Oct 15 20:13 /usr/bin/java -> /etc/alternatives/java
    [x@centos7 ~]# ls -lrt /etc/alternatives/java
    lrwxrwxrwx. 1 root root 74 Oct 15 20:13 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.aarch64/jre/bin/java
    ```
2. 配置环境变量JAVA_HOME
    1.  在`/etc/profile`中追加  
        ```shell
        export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.aarch64
        export JRE_HOME=$JAVA_HOME/jre
        export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
        export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
        ```
    2.  生效
        ```shell
        [x@centos7 ~]# source /etc/profile
        ```
    3.  验证
        ```shell
        [x@centos7 ~]# echo $JAVA_HOME
        /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.aarch64
        ```
### 二、配置maven
1.  安装
    ```shell
    [x@centos7 download]# wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.6.1-bin.tar.gz
    [x@centos7 download]# tar -xvf  apache-maven-3.6.1-bin.tar.gz
    [x@centos7 download]# sudo mv -f apache-maven-3.6.1 /usr/local/
    ```
2.  配置环境变量
    1.  在`/etc/profile`中追加  
        ```shell
        export MAVEN_HOME=/usr/local/apache-maven-3.6.1
        export PATH=${PATH}:${MAVEN_HOME}/bin
        ```
    2.  生效
        ```shell
        [x@centos7 ~]# source /etc/profile
        ```
    3.  验证
        ```shell
        [x@centos7 ~]# # mvn -v
        ```
3.  配置仓库源
    1.  在`/usr/local/apache-maven-3.6.1/conf/settings.xml` 中 `mirrors` 节点下增加
        ```xml
        <mirror>
            <id>huaweicloud</id>
            <mirrorOf>*</mirrorOf>
            <url>https://repo.huaweicloud.com/repository/maven/</url>
        </mirror>
        ```
    2.  本地仓库地址为 `${user.home}/.m2/repository` ,若要更改或者添加，请在 `settings` 中修改或添加 
        ```xml
        <localRepository>the repository path you want</localRepository>
        ```

Mushroom @20201019