# 添加jar包到本地仓库
1.  安装jar包
    ```shell
    [x@centos7 ~]# mvn install:install-file -Dfile=jar包的路径 -DgroupId=gruopId中的内容 -DartifactId=actifactId的内容 -Dversion=version的内容 -Dpackaging=jar
    ```
    例
    ```shell
    [x@centos7 ~]# ls
    snappy-java-1.0.4.1.jar
    [root@localhost target]# jar tf snappy-java-1.0.4.1.jar 
    META-INF/MANIFEST.MF
    META-INF/
    META-INF/maven/
    META-INF/maven/org.xerial.snappy/
    META-INF/maven/org.xerial.snappy/snappy-java/
    META-INF/maven/org.xerial.snappy/snappy-java/LICENSE
    META-INF/maven/org.xerial.snappy/snappy-java/pom.properties
    META-INF/maven/org.xerial.snappy/snappy-java/pom.xml
    ...
    [x@centos7 ~]# mvn install:install-file -Dfile=snappy-java-1.0.4.1.jar -DgroupId=org.xerial.snappy -DartifactId=snappy-java -Dversion=1.0.4.1 -Dpackaging=jar
    ```
2.  强制项目使用本地仓库
    ```xml
    <dependency>
        <groupId>DgroupId</groupId>
        <artifactId>DartifactId</artifactId>
        <version>Dversion</version>
    </dependency>
    ```