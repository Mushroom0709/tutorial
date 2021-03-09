#   KunPeng平台Hadoop-3.1.1-编译

##  环境准备
1.  安装基本环境
    ```shell
    [x@centos software]# yum install gcc* libstdc++* libtool autoconf automake ant protobuf protobuf-devel openssl-devel  -y
    [x@centos software]# wget https://cmake.org/files/v3.15/cmake-3.15.7.tar.gz
    [x@centos software]# tar -zvxf cmake-3.15.7.tar.gz
    [x@centos software]# cd cmake-3.15.7
    [x@centos cmake-3.15.7]# ./configure
    [x@centos cmake-3.15.7]# make -j8
    [x@centos cmake-3.15.7]# make install
    [x@centos cmake-3.15.7]# ldconfig
    [x@centos cmake-3.15.7]# cd ..
    ```
2.  参见 [Maven及JDK配置](../maven/安装及配置华为KunPeng仓库.md)
3.  参见 [解决 `arm` 平台 `char` 问题](./解决arm平台char问题.md)
3.  下载 Hadoop-3.1.1 及 后期依赖
    ```shell
    [x@centos software]# wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1-src.tar.gz
    [x@centos software]# wget https://repository.mulesoft.org/nexus/content/repositories/public/com/amazonaws/DynamoDBLocal/1.11.86/DynamoDBLocal-1.11.86.jar
##  编译
1.  安装 DynamoDBLocal-1.11.86 (华为鲲鹏仓库中并没有DynamoDBLocal-1.11.86，需要手动安装)
    ```
    [x@centos software]# jar -tf DynamoDBLocal-1.11.86.jar | grep "META-INF/maven/"
    META-INF/maven/
    META-INF/maven/com/
    META-INF/maven/com/amazonaws/
    META-INF/maven/com/amazonaws/DynamoDBLocal/
    META-INF/maven/com/amazonaws/DynamoDBLocal/pom.properties
    META-INF/maven/com/amazonaws/DynamoDBLocal/pom.xml
    [x@centos software]# mvn install:install-file -Dfile=DynamoDBLocal-1.11.86.jar -DgroupId=com.amazonaws -DartifactId=DynamoDBLocal -Dversion=1.11.86 -Dpackaging=jar
    [INFO] Scanning for projects...
    [INFO] 
    [INFO] ------------------< org.apache.maven:standalone-pom >-------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] --------------------------------[ pom ]---------------------------------
    [INFO] 
    [INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
    [INFO] Installing /root/software/DynamoDBLocal-1.11.86.jar to /root/.m2/repository/com/amazonaws/DynamoDBLocal/1.11.86/DynamoDBLocal-1.11.86.jar
    [INFO] Installing /tmp/mvninstall1265591713641685906.pom to /root/.m2/repository/com/amazonaws/DynamoDBLocal/1.11.86/DynamoDBLocal-1.11.86.pom
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  0.338 s
    [INFO] Finished at: 2020-10-26T10:14:38+08:00
    [INFO] ------------------------------------------------------------------------
    ```
2.  释放文件
    ```shell
    [x@centos software]# tar -zvxf hadoop-3.1.1-src.tar.gz
    [x@centos software]# cd hadoop-3.1.1-src
    ```
3.  设置加速仓库,在 `project/repositories/` 节点下(头部)添加:
    ```xml
    <repository>
        <id>kunpengmaven</id>
        <name>kunpeng maven</name>
        <url>https://mirrors.huaweicloud.com/kunpeng/maven</url>
    </repository>
    <repository>
        <id>hortonworkmaven</id>
        <name>hortonwork maven</name>
        <url>https://repo.hortonworks.com/content/repositories/releases</url>
    </repository>
    <repository>
        <id>mulesoftmaven</id>
        <name>mulesoft maven</name>
        <url>https://repository.mulesoft.org/nexus/content/repositories/public</url>
    </repository>
    ```
4.  编译打包
    ```shell
    [x@centos hadoop-3.1.1-src]# mvn package -Pdist,native -Dtar -DskipTests -Dmaven.javadoc.skip=true
    ```
5.  遇到报错，提示找不到 java symbol:
    ```
    [INFO] BUILD FAILURE
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  01:07 min
    [INFO] Finished at: 2020-10-26T10:17:29+08:00
    [INFO] ------------------------------------------------------------------------
    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile (default-testCompile) on project hadoop-aws: Compilation failure: Compilation failure: 
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[542,60] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<com.amazonaws.services.s3.model.InitiateMultipartUploadRequest>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[564,47] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<com.amazonaws.services.s3.model.UploadPartRequest>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[591,60] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<com.amazonaws.services.s3.model.CompleteMultipartUploadRequest>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[611,53] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<com.amazonaws.services.s3.model.AbortMultipartUploadRequest>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[633,39] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<com.amazonaws.services.s3.model.DeleteObjectRequest>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[646,23] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<java.lang.String>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    [ERROR] /root/software/hadoop-3.1.1-src/hadoop-tools/hadoop-aws/src/test/java/org/apache/hadoop/fs/s3a/commit/staging/StagingTestBase.java:[647,23] cannot find symbol
    [ERROR]   symbol:   method getArgumentAt(int,java.lang.Class<java.lang.String>)
    [ERROR]   location: variable invocation of type org.mockito.invocation.InvocationOnMock
    ```
    解决办法 (org.mockito:mockito-all 版本的兼容性问题):
    修改 `hadoop-project/pom.xml` 中 `project/dependencyManagement/dependencys/` 节点下
    ```xml
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-all</artifactId>
      <version>1.8.5</version>
    </dependency>
    ```
    为
    ```xml
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-all</artifactId>
      <version>1.10.19</version>
    </dependency>
    ```
6.  继续编译打包
    ```shell
    [x@centos hadoop-3.1.1-src]# mvn package -Pdist,native -Dtar -DskipTests -Dmaven.javadoc.skip=true
    ```
    编译成功则结果如下:
    ```shell
    main:
        [mkdir] Created dir: /root/software/hadoop-3.1.1-src/hadoop-cloud-storage-project/target/test-dir
    [INFO] Executed tasks
    [INFO] 
    [INFO] --- maven-remote-resources-plugin:1.5:process (default) @ hadoop-cloud-storage-project ---
    [INFO] 
    [INFO] --- maven-site-plugin:3.6:attach-descriptor (attach-descriptor) @ hadoop-cloud-storage-project ---
    [INFO] No site descriptor found: nothing to attach.
    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Summary for Apache Hadoop Main 3.1.1:
    [INFO] 
    [INFO] Apache Hadoop Main ................................. SUCCESS [  0.906 s]
    [INFO] Apache Hadoop Build Tools .......................... SUCCESS [  1.110 s]
    [INFO] Apache Hadoop Project POM .......................... SUCCESS [  0.325 s]
    [INFO] Apache Hadoop Annotations .......................... SUCCESS [  0.341 s]
    [INFO] Apache Hadoop Project Dist POM ..................... SUCCESS [  0.073 s]
    [INFO] Apache Hadoop Assemblies ........................... SUCCESS [  0.071 s]
    [INFO] Apache Hadoop Maven Plugins ........................ SUCCESS [  1.730 s]
    [INFO] Apache Hadoop MiniKDC .............................. SUCCESS [  0.412 s]
    [INFO] Apache Hadoop Auth ................................. SUCCESS [  8.535 s]
    [INFO] Apache Hadoop Auth Examples ........................ SUCCESS [  0.690 s]
    [INFO] Apache Hadoop Common ............................... SUCCESS [  7.285 s]
    [INFO] Apache Hadoop NFS .................................. SUCCESS [  0.957 s]
    [INFO] Apache Hadoop KMS .................................. SUCCESS [  1.058 s]
    [INFO] Apache Hadoop Common Project ....................... SUCCESS [  0.029 s]
    [INFO] Apache Hadoop HDFS Client .......................... SUCCESS [  1.748 s]
    [INFO] Apache Hadoop HDFS ................................. SUCCESS [  2.203 s]
    [INFO] Apache Hadoop HDFS Native Client ................... SUCCESS [  0.257 s]
    [INFO] Apache Hadoop HttpFS ............................... SUCCESS [  1.191 s]
    [INFO] Apache Hadoop HDFS-NFS ............................. SUCCESS [  0.332 s]
    [INFO] Apache Hadoop HDFS-RBF ............................. SUCCESS [  0.533 s]
    [INFO] Apache Hadoop HDFS Project ......................... SUCCESS [  0.027 s]
    [INFO] Apache Hadoop YARN ................................. SUCCESS [  0.026 s]
    [INFO] Apache Hadoop YARN API ............................. SUCCESS [  0.618 s]
    [INFO] Apache Hadoop YARN Common .......................... SUCCESS [  1.205 s]
    [INFO] Apache Hadoop YARN Registry ........................ SUCCESS [  0.807 s]
    [INFO] Apache Hadoop YARN Server .......................... SUCCESS [  0.025 s]
    [INFO] Apache Hadoop YARN Server Common ................... SUCCESS [  0.989 s]
    [INFO] Apache Hadoop YARN NodeManager ..................... SUCCESS [  0.907 s]
    [INFO] Apache Hadoop YARN Web Proxy ....................... SUCCESS [  0.626 s]
    [INFO] Apache Hadoop YARN ApplicationHistoryService ....... SUCCESS [  0.661 s]
    [INFO] Apache Hadoop YARN Timeline Service ................ SUCCESS [  0.671 s]
    [INFO] Apache Hadoop YARN ResourceManager ................. SUCCESS [  2.162 s]
    [INFO] Apache Hadoop YARN Server Tests .................... SUCCESS [  0.942 s]
    [INFO] Apache Hadoop YARN Client .......................... SUCCESS [  0.593 s]
    [INFO] Apache Hadoop YARN SharedCacheManager .............. SUCCESS [  0.785 s]
    [INFO] Apache Hadoop YARN Timeline Plugin Storage ......... SUCCESS [  1.197 s]
    [INFO] Apache Hadoop YARN TimelineService HBase Backend ... SUCCESS [  0.025 s]
    [INFO] Apache Hadoop YARN TimelineService HBase Common .... SUCCESS [  1.671 s]
    [INFO] Apache Hadoop YARN TimelineService HBase Client .... SUCCESS [  1.376 s]
    [INFO] Apache Hadoop YARN TimelineService HBase Servers ... SUCCESS [  0.025 s]
    [INFO] Apache Hadoop YARN TimelineService HBase Server 1.2  SUCCESS [  1.811 s]
    [INFO] Apache Hadoop YARN TimelineService HBase tests ..... SUCCESS [  2.466 s]
    [INFO] Apache Hadoop YARN Router .......................... SUCCESS [  0.897 s]
    [INFO] Apache Hadoop YARN Applications .................... SUCCESS [  0.025 s]
    [INFO] Apache Hadoop YARN DistributedShell ................ SUCCESS [  0.742 s]
    [INFO] Apache Hadoop YARN Unmanaged Am Launcher ........... SUCCESS [  0.456 s]
    [INFO] Apache Hadoop MapReduce Client ..................... SUCCESS [  0.115 s]
    [INFO] Apache Hadoop MapReduce Core ....................... SUCCESS [  1.037 s]
    [INFO] Apache Hadoop MapReduce Common ..................... SUCCESS [  0.637 s]
    [INFO] Apache Hadoop MapReduce Shuffle .................... SUCCESS [  1.360 s]
    [INFO] Apache Hadoop MapReduce App ........................ SUCCESS [  0.802 s]
    [INFO] Apache Hadoop MapReduce HistoryServer .............. SUCCESS [  0.973 s]
    [INFO] Apache Hadoop MapReduce JobClient .................. SUCCESS [  0.832 s]
    [INFO] Apache Hadoop Mini-Cluster ......................... SUCCESS [  1.376 s]
    [INFO] Apache Hadoop YARN Services ........................ SUCCESS [  0.023 s]
    [INFO] Apache Hadoop YARN Services Core ................... SUCCESS [  1.025 s]
    [INFO] Apache Hadoop YARN Services API .................... SUCCESS [  1.002 s]
    [INFO] Apache Hadoop YARN Site ............................ SUCCESS [  0.023 s]
    [INFO] Apache Hadoop YARN UI .............................. SUCCESS [  0.023 s]
    [INFO] Apache Hadoop YARN Project ......................... SUCCESS [  1.019 s]
    [INFO] Apache Hadoop MapReduce HistoryServer Plugins ...... SUCCESS [  0.487 s]
    [INFO] Apache Hadoop MapReduce NativeTask ................. SUCCESS [  0.644 s]
    [INFO] Apache Hadoop MapReduce Uploader ................... SUCCESS [  0.538 s]
    [INFO] Apache Hadoop MapReduce Examples ................... SUCCESS [  0.477 s]
    [INFO] Apache Hadoop MapReduce ............................ SUCCESS [  0.151 s]
    [INFO] Apache Hadoop MapReduce Streaming .................. SUCCESS [  0.171 s]
    [INFO] Apache Hadoop Distributed Copy ..................... SUCCESS [  0.272 s]
    [INFO] Apache Hadoop Archives ............................. SUCCESS [  0.106 s]
    [INFO] Apache Hadoop Archive Logs ......................... SUCCESS [  0.233 s]
    [INFO] Apache Hadoop Rumen ................................ SUCCESS [  0.247 s]
    [INFO] Apache Hadoop Gridmix .............................. SUCCESS [  0.161 s]
    [INFO] Apache Hadoop Data Join ............................ SUCCESS [  0.118 s]
    [INFO] Apache Hadoop Extras ............................... SUCCESS [  0.154 s]
    [INFO] Apache Hadoop Pipes ................................ SUCCESS [  0.022 s]
    [INFO] Apache Hadoop OpenStack support .................... SUCCESS [  0.586 s]
    [INFO] Apache Hadoop Amazon Web Services support .......... SUCCESS [  2.025 s]
    [INFO] Apache Hadoop Kafka Library support ................ SUCCESS [  5.033 s]
    [INFO] Apache Hadoop Azure support ........................ SUCCESS [  5.279 s]
    [INFO] Apache Hadoop Aliyun OSS support ................... SUCCESS [  1.327 s]
    [INFO] Apache Hadoop Client Aggregator .................... SUCCESS [  1.576 s]
    [INFO] Apache Hadoop Scheduler Load Simulator ............. SUCCESS [  1.836 s]
    [INFO] Apache Hadoop Resource Estimator Service ........... SUCCESS [  2.506 s]
    [INFO] Apache Hadoop Azure Data Lake support .............. SUCCESS [  2.571 s]
    [INFO] Apache Hadoop Image Generation Tool ................ SUCCESS [  0.405 s]
    [INFO] Apache Hadoop Tools Dist ........................... SUCCESS [  1.413 s]
    [INFO] Apache Hadoop Tools ................................ SUCCESS [  0.025 s]
    [INFO] Apache Hadoop Client API ........................... SUCCESS [01:27 min]
    [INFO] Apache Hadoop Client Runtime ....................... SUCCESS [01:03 min]
    [INFO] Apache Hadoop Client Packaging Invariants .......... SUCCESS [  0.201 s]
    [INFO] Apache Hadoop Client Test Minicluster .............. SUCCESS [02:02 min]
    [INFO] Apache Hadoop Client Packaging Invariants for Test . SUCCESS [  0.168 s]
    [INFO] Apache Hadoop Client Packaging Integration Tests ... SUCCESS [  0.095 s]
    [INFO] Apache Hadoop Distribution ......................... SUCCESS [  1.605 s]
    [INFO] Apache Hadoop Client Modules ....................... SUCCESS [  0.022 s]
    [INFO] Apache Hadoop Cloud Storage ........................ SUCCESS [  0.815 s]
    [INFO] Apache Hadoop Cloud Storage Project ................ SUCCESS [  0.021 s]
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  06:09 min
    [INFO] Finished at: 2020-10-26T10:32:36+08:00
    [INFO] ------------------------------------------------------------------------
    ```

# 验证