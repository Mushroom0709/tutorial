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
# 使用本地jar包文件
1.  将jar包复制到指定项目的lib文件夹中
2.  在项目的`pom.xml`文件中，增加本地包的配置信息  
    在 `/project/dependencyManagement/dependencies` 节点下添加
    ```xml
    <dependency>
        <groupId>groupId</groupId>
        <artifactId>DartifactId</artifactId>
        <version>Dversion</version>
        <scope>system</scope>
        <systemPath>${project.basedir}/libs/package.jar</systemPath>
    </dependency>
    ```
    例:
    ```shell
    [x@centos7 hadoop-2.7.6-src]# ls
    BUILDING.txt        hadoop-client                 hadoop-dist               hadoop-minicluster   hadoop-yarn-project  README.txt
    dev-support         hadoop-client-modules         hadoop-hdfs-project       hadoop-project       LICENSE.txt
    hadoop-assemblies   hadoop-cloud-storage-project  hadoop-mapreduce-project  hadoop-project-dist  NOTICE.txt
    hadoop-build-tools  hadoop-common-project         hadoop-maven-plugins      hadoop-tools         pom.xml
    [x@centos7 hadoop-2.7.6-src]# mkdir libs
    [x@centos7 hadoop-2.7.6-src]# ls
    BUILDING.txt        hadoop-client                 hadoop-dist               hadoop-minicluster   hadoop-yarn-project  pom.xml
    dev-support         hadoop-client-modules         hadoop-hdfs-project       hadoop-project       libs                 README.txt
    hadoop-assemblies   hadoop-cloud-storage-project  hadoop-mapreduce-project  hadoop-project-dist  LICENSE.txt
    hadoop-build-tools  hadoop-common-project         hadoop-maven-plugins      hadoop-tools         NOTICE.txt
    [x@centos7 hadoop-2.7.6-src]# cd libs
    [x@centos7 libs]# cp ~/document/snappy-java-snappy-java-1.0.4.1/target/snappy-java-1.0.4.1.jar ./
    [x@centos7 libs]# ls
    snappy-java-1.0.4.1.jar
    [x@centos7 libs]# ls
    snappy-java-1.0.4.1.jar
    [x@centos7 libs]# jar tf snappy-java-1.0.4.1.jar 
    META-INF/MANIFEST.MF
    META-INF/
    META-INF/maven/
    META-INF/maven/org.xerial.snappy/
    META-INF/maven/org.xerial.snappy/snappy-java/
    META-INF/maven/org.xerial.snappy/snappy-java/LICENSE
    META-INF/maven/org.xerial.snappy/snappy-java/pom.properties
    META-INF/maven/org.xerial.snappy/snappy-java/pom.xml
    ...
    [x@centos7 libs]# cd ..
    [x@centos7 hadoop-2.7.6-src]# vim pom.xml
    ```
    在 `/project/dependencyManagement/dependencies` 节点下添加
    ```xml
    <dependency>
        <groupId>org.xerial.snappy</groupId>
        <artifactId>snappy-java</artifactId>
        <version>1.0.4.1</version>
        <scope>system</scope>
        <systemPath>/root/document/hadoop-2.7.6-src/libs/snappy-java-1.0.4.1.jar</systemPath>
    </dependency>
    ```

Mushroom @20201019