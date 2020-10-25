#   Netty-4.0.52-编译安装

##  准备环境
1.  安装基本编译环境
    ```shell
    [x@centos software]# yum install gcc* libstdc++* libtool autoconf automake ant bzip2 protobuf protobuf-devel openssl-devel  -y
    [x@centos software]# wget https://cmake.org/files/v3.15/cmake-3.15.7.tar.gz
    [x@centos software]# tar -zvxf cmake-3.15.7.tar.gz
    [x@centos software]# cd cmake-3.15.7
    [x@centos cmake-3.15.7]# ./configure
    [x@centos cmake-3.15.7]# make -j8
    [x@centos cmake-3.15.7]# make install
    [x@centos cmake-3.15.7]# ldconfig
    [x@centos cmake-3.15.7]# cd ..
    ```
2.  参见 [配置Maven](../maven/安装及配置华为KunPeng仓库.md)
3.  下载 netty-4.0.52 及 后期依赖
    ```shell
    [x@centos software]# wget https://codeload.github.com/netty/netty/tar.gz/netty-4.0.52.Final
    [x@centos software]# wget https://codeload.github.com/netty/netty-tcnative/tar.gz/netty-tcnative-parent-2.0.6.Final (鲲鹏仓库中没有，需要手动编译)
    [x@centos software]# wget https://archive.apache.org/dist/apr/apr-1.6.2.tar.gz (netty-tcnative-parent-2.0.6的依赖，由于版本过于远久，仓库位置迁移，需要手动下载)
    [x@centos software]# wget https://ftp.openssl.org/source/old/1.0.2/openssl-1.0.2l.tar.gz (netty-tcnative-parent-2.0.6的依赖，XXX防火墙，需要手动下载)
    [x@centos software]# wget http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.5.5.tar.gz (netty-tcnative-parent-2.0.6的依赖，XXX防火墙，需要手动下载)
    ```
4.  解决 `arm` 平台 `-fsigned-char` 问题
    ```shell
    [x@centos software]# command -v gcc
    /usr/bin/gcc
    [x@centos software]# mv /usr/bin/gcc /usr/bin/gcc-impl
    [x@centos software]# vi /usr/bin/gcc
    #! /bin/sh 
    [x@centos software]# chmod +x /usr/bin/gcc
    /usr/bin/gcc-impl -fsigned-char "$@"
    [x@centos software]# command -v g++
    /usr/bin/g++
    [x@centos software]# mv /usr/bin/g++ /usr/bin/g++-impl
    [x@centos software]# vi /usr/bin/g++
    #! /bin/sh 
    /usr/bin/g++-impl -fsigned-char "$@"
    [x@centos software]# chmod +x /usr/bin/g++
    ```

##  编译
1.  编译 netty-tcnative-netty-tcnative-parent-2.0.6 (netty 依赖)
    1.  配置依赖
        1.  安装 apr-1.5.2
             ```shell
            [x@centos software]# tar -zvxf apr-1.6.2.tar.gz
            [x@centos software]# cd apr-1.6.2
            [x@centos apr-1.6.2]# ./configure --prefix=/usr/local/apr
            [x@centos apr-1.6.2]# make -j8
            [x@centos apr-1.6.2]# make install
            [x@centos apr-1.6.2]# ldconfig
            [x@centos apr-1.6.2]# cd ..
            ```
    2.  释放文件
        ```shell
        [x@centos software]# tar -zvxf netty-tcnative-netty-tcnative-parent-2.0.6.Final.tar.gz
        [x@centos software]# mv netty-tcnative-netty-tcnative-parent-2.0.6.Final netty-tcnative-netty-tcnative-parent-2.0.6
        [x@centos software]# cd netty-tcnative-netty-tcnative-parent-2.0.6
        ```
    3.  跳过 `boringssl-static`,修改 pom.xml 中:`project/profiles` 节点下
        ```xml
        <profile>
        <id>all</id>
        <activation>
            <property>
            <name>!moduleSelector</name>
            </property>
        </activation>
        <modules>
            <module>openssl-dynamic</module>
            <module>openssl-static</module>
            <module>boringssl-static</module>
            <module>libressl-static</module>
        </modules>
        </profile>
        ```
        为
        ```xml
        <profile>
        <id>all</id>
        <activation>
            <property>
            <name>!moduleSelector</name>
            </property>
        </activation>
        <modules>
            <module>openssl-dynamic</module>
            <module>openssl-static</module>
            <!--<module>boringssl-static</module>-->
            <module>libressl-static</module>
        </modules>
        </profile>
        ```
    4.  编译安装到本地仓库
        ```shell
        [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# mvn install -DskipTests
        ```
    5.  编译报错(无法下载 apr-1.6.2.tar.gz) 如下
        ```shell
        [get] Getting: http://www.us.apache.org/dist/apr/apr-1.6.2.tar.gz
        [get] To: /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/apr-1.6.2.tar.gz
        [get] http://www.us.apache.org/dist/apr/apr-1.6.2.tar.gz moved to https://downloads.apache.org/apr/apr-1.6.2.tar.gz
        [get] Error opening connection java.io.FileNotFoundException: https://downloads.apache.org/apr/apr-1.6.2.tar.gz
        [get] Error opening connection java.io.FileNotFoundException: https://downloads.apache.org/apr/apr-1.6.2.tar.gz
        [get] Error opening connection java.io.FileNotFoundException: https://downloads.apache.org/apr/apr-1.6.2.tar.gz
        [get] Can't get http://www.us.apache.org/dist/apr/apr-1.6.2.tar.gz to /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/apr-1.6.2.tar.gz
        [INFO] ------------------------------------------------------------------------
        [INFO] Reactor Summary for Netty/TomcatNative [Parent] 2.0.6.Final:
        [INFO] 
        [INFO] Netty/TomcatNative [Parent] ........................ SUCCESS [  3.241 s]
        [INFO] Netty/TomcatNative [OpenSSL - Dynamic] ............. SUCCESS [ 16.655 s]
        [INFO] Netty/TomcatNative [OpenSSL - Static] .............. FAILURE [  9.490 s]
        [INFO] Netty/TomcatNative [BoringSSL - Static] ............ SKIPPED
        [INFO] Netty/TomcatNative [LibreSSL - Static] ............. SKIPPED
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD FAILURE
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time:  29.785 s
        [INFO] Finished at: 2020-10-25T19:50:53+08:00
        [INFO] ------------------------------------------------------------------------
        [ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.8:run (build-apr-linux-mac) on project netty-tcnative-openssl-static: An Ant BuildException has occured: Can't get http://www.us.apache.org/dist/apr/apr-1.6.2.tar.gz to /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/apr-1.6.2.tar.gz
        [ERROR] around Ant part ...<get src="http://www.us.apache.org/dist/apr/${aprTarGzFile}" dest="/root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/${aprTarGzFile}" verbose="on"/>... @ 6:181 in /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/antrun/build-main.xml
        ```
        解决办法:  
        1.  修改 `pom.xml` 中 `project/profiles/profile(<id>build-apr-linux-mac</id>)/build/plugins/plugin/executions/execution/configuration/target/` 节点下 
            ```xml
            <get src="http://www.us.apache.org/dist/apr/${aprTarGzFile}" dest="${project.build.directory}/${aprTarGzFile}" verbose="on" />
            ```
            为
            ```xml
            <!--<get src="http://www.us.apache.org/dist/apr/${aprTarGzFile}" dest="${project.build.directory}/${aprTarGzFile}" verbose="on" />-->
            ```
        2.  复制 `arp-1.6.2.tar.gz` 到 `openssl-static/target/`
            ```shell
            [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# cp -p ../apr-1.6.2.tar.gz openssl-static/target/
            ```
    6.  继续编译安装到本地仓库
        ```shell
        [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# mvn install -DskipTests
        ```
    7.  编译卡死(ftp 无法下载openssl-1.0.2l.tar.gz) 如下
        ```shell
        main:
            [ftp] getting files
            [ftp] transferring old/1.0.2/openssl-1.0.2l.tar.gz to /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/openssl-static/target/old/1.0.2/openssl-1.0.2l.tar.gz
        ```
        解决办法:  
        1.  修改 `openssl-static/pom.xml` 中:`project/profiles/profile(<id>build-openssl-linux</id>)/build/plugins/plugin/executions/execution/configuration/target/` 节点下 
            ```xml
            <ftp action="get" server="ftp.openssl.org" remotedir="source" userid="anonymous" password="anonymous" passive="yes" verbose="yes">
                <fileset dir="${project.build.directory}">
                    <include name="**/openssl-${opensslVersion}.tar.gz" />
                </fileset>
            </ftp>
            ```
            为
            ```xml
            <!--
            <ftp action="get" server="ftp.openssl.org" remotedir="source" userid="anonymous" password="anonymous" passive="yes" verbose="yes">
            <fileset dir="${project.build.directory}">
                <include name="**/openssl-${opensslVersion}.tar.gz" />
            </fileset>
            </ftp>
            -->
            ```
        2.  复制 `arp-1.6.2.tar.gz` 到 `openssl-static/target/`
            ```shell
            [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# cp -p ../openssl-1.0.2l.tar.gz openssl-static/target/
            ```
        **@Mushroom:特别提醒，有三个build-openssl的profile，分别是linux,nuix(mac),win 自己盯着看仔细了，别改错节点不生效。**
    8.  继续编译安装到本地仓库
        ```shell
        [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# mvn install -DskipTests
        ```
    9.  遇到报错
        ```shell
        main:
        [checksum] Could not find file /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/apr-1.6.2.tar.gz to generate checksum for.
        [INFO] ------------------------------------------------------------------------
        [INFO] Reactor Summary for Netty/TomcatNative [Parent] 2.0.6.Final:
        [INFO] 
        [INFO] Netty/TomcatNative [Parent] ........................ SUCCESS [  3.255 s]
        [INFO] Netty/TomcatNative [OpenSSL - Dynamic] ............. SUCCESS [  4.082 s]
        [INFO] Netty/TomcatNative [OpenSSL - Static] .............. SUCCESS [02:37 min]
        [INFO] Netty/TomcatNative [LibreSSL - Static] ............. FAILURE [  1.729 s]
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD FAILURE
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time:  02:46 min
        [INFO] Finished at: 2020-10-25T22:26:05+08:00
        [INFO] ------------------------------------------------------------------------
        [ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.8:run (build-apr-linux-mac) on project netty-tcnative-libressl-static: An Ant BuildException has occured: Could not find file /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/apr-1.6.2.tar.gz to generate checksum for.
        [ERROR] around Ant part ...<checksum file="/root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/${aprTarGzFile}" verifyProperty="isEqual" property="98492e965963f852ab29f9e61b2ad700" algorithm="MD5"/>... @ 6:203 in /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/antrun/build-main.xml
        ```
        解决办法 (前面修改过了，不用改配置文件了):  
        1.  复制 `arp-1.6.2.tar.gz` 到 `libressl-static/target/`
            ```shell
            [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# cp -p ../apr-1.6.2.tar.gz libressl-static/target/
            ```
    10. 遇到如下情况
        ```shell
        main:
        [get] Getting: http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.5.5.tar.gz
        [get] To: /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/libressl-2.5.5.tar.gz
        ```
        卡住或下载很慢。
        解决办法:   
        1.  修改 `libressl-static/pom.xml` 中:`project/profiles/profile(<id>build-libressl-linux-mac</id>)/build/plugins/plugin/executions/execution/configuration/target/` 节点下 
            ```xml
            <get src="http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/${libresslArchive}" dest="${project.build.directory}/${libresslArchive}" verbose="on" />
            ```
            为
            ```xml
            <!--
            <get src="http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/${libresslArchive}" dest="${project.build.directory}/${libresslArchive}" verbose="on" />
            -->
            ```
        2.  复制 `libressl-2.5.5.tar.gz` 到 `libressl-static/target/`
            ```shell
            [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# rm -rf libressl-static/target/libressl-2.5.5.tar.gz && cp -p ../libressl-2.5.5.tar.gz libressl-static/target/
            ```
    11. 继续编译安装到本地仓库
        ```shell
        [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# mvn install -DskipTests
        ```
        输出如下则成功:
        ```shell
        [INFO] --- maven-install-plugin:2.4:install (default-install) @ netty-tcnative-libressl-static ---
        [INFO] Installing /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/netty-tcnative-libressl-static-2.0.6.Final.jar to /root/.m2/repository/io/netty/netty-tcnative-libressl-static/2.0.6.Final/netty-tcnative-libressl-static-2.0.6.Final.jar
        [INFO] Installing /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/pom.xml to /root/.m2/repository/io/netty/netty-tcnative-libressl-static/2.0.6.Final/netty-tcnative-libressl-static-2.0.6.Final.pom
        [INFO] Installing /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/netty-tcnative-libressl-static-2.0.6.Final-sources.jar to /root/.m2/repository/io/netty/netty-tcnative-libressl-static/2.0.6.Final/netty-tcnative-libressl-static-2.0.6.Final-sources.jar
        [INFO] Installing /root/software/netty-tcnative-netty-tcnative-parent-2.0.6/libressl-static/target/netty-tcnative-libressl-static-2.0.6.Final-linux-aarch_64.jar to /root/.m2/repository/io/netty/netty-tcnative-libressl-static/2.0.6.Final/netty-tcnative-libressl-static-2.0.6.Final-linux-aarch_64.jar
        [INFO] ------------------------------------------------------------------------
        [INFO] Reactor Summary for Netty/TomcatNative [Parent] 2.0.6.Final:
        [INFO] 
        [INFO] Netty/TomcatNative [Parent] ........................ SUCCESS [  3.232 s]
        [INFO] Netty/TomcatNative [OpenSSL - Dynamic] ............. SUCCESS [  4.087 s]
        [INFO] Netty/TomcatNative [OpenSSL - Static] .............. SUCCESS [02:37 min]
        [INFO] Netty/TomcatNative [LibreSSL - Static] ............. SUCCESS [ 35.334 s]
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time:  03:20 min
        [INFO] Finished at: 2020-10-26T00:03:50+08:00
        [INFO] ------------------------------------------------------------------------
        ```
    12. 退出当前目录
        ```shell
        [x@centos netty-tcnative-netty-tcnative-parent-2.0.6]# cd ..
        ```
2.  编译 netty-4.0.52
    1.  释放文件
        ```shell
        [x@centos software]# tar -zvxf netty-netty-4.0.52.Final.tar.gz
        [x@centos software]# mv netty-netty-4.0.52.Final netty-4.0.52
        [x@centos software]# cd netty-4.0.52
        ```
    2.  去除x86的限制,修改 pom.xml 中 `project/build/plugins/` 节点下
        ```xml
        <plugin>
            <artifactId>maven-enforcer-plugin</artifactId>
            <version>${enforcer.plugin.version}</version>
            <executions>
            <execution>
                <id>enforce-tools</id>
                <goals>
                <goal>enforce</goal>
                </goals>
                <configuration>
                <rules>
                    <requireJavaVersion>
                    <!-- Enforce JDK 1.8+ for compilation. -->
                    <!-- This is needed because of java.util.zip.Deflater and NIO UDP multicast. -->
                    <version>[1.8.0,)</version>
                    </requireJavaVersion>
                    <requireMavenVersion>
                    <version>[3.1.1,)</version>
                    </requireMavenVersion>
                    <requireProperty>
                    <regexMessage>
                        x86_64 JDK must be used.
                    </regexMessage>
                    <property>os.detected.arch</property>
                    <regex>^x86_64$</regex>
                    </requireProperty>
                </rules>
                </configuration>
            </execution>
            </executions>
        </plugin>
        ```
        为
        ```xml
        <plugin>
            <artifactId>maven-enforcer-plugin</artifactId>
            <version>${enforcer.plugin.version}</version>
            <executions>
            <execution>
                <id>enforce-tools</id>
                <goals>
                <goal>enforce</goal>
                </goals>
                <configuration>
                <rules>
                    <requireJavaVersion>
                    <!-- Enforce JDK 1.8+ for compilation. -->
                    <!-- This is needed because of java.util.zip.Deflater and NIO UDP multicast. -->
                    <version>[1.8.0,)</version>
                    </requireJavaVersion>
                    <requireMavenVersion>
                    <version>[3.1.1,)</version>
                    </requireMavenVersion>
                    <!--
                    <requireProperty>
                    <regexMessage>
                        x86_64 JDK must be used.
                    </regexMessage>
                    <property>os.detected.arch</property>
                    <regex>^x86_64$</regex>
                    </requireProperty>
                    -->
                </rules>
                </configuration>
            </execution>
            </executions>
        </plugin>
        ```
    3.  编译并安装到本地仓库
        ```shell
        [x@centos netty-4.0.52]# mvn install -DskipTests
        ```
        输出如下则成功:
        ```shell
        [INFO] Reactor Summary for Netty 4.0.52.Final:
        [INFO] 
        [INFO] Netty/Dev-Tools .................................... SUCCESS [  0.839 s]
        [INFO] Netty .............................................. SUCCESS [  4.026 s]
        [INFO] Netty/Common ....................................... SUCCESS [  4.511 s]
        [INFO] Netty/Buffer ....................................... SUCCESS [  2.800 s]
        [INFO] Netty/Transport .................................... SUCCESS [  3.349 s]
        [INFO] Netty/Codec ........................................ SUCCESS [  3.545 s]
        [INFO] Netty/Codec/HAProxy ................................ SUCCESS [  1.506 s]
        [INFO] Netty/Handler ...................................... SUCCESS [  3.907 s]
        [INFO] Netty/Codec/HTTP ................................... SUCCESS [  3.856 s]
        [INFO] Netty/Codec/Socks .................................. SUCCESS [  1.568 s]
        [INFO] Netty/Transport/RXTX ............................... SUCCESS [  1.391 s]
        [INFO] Netty/Transport/SCTP ............................... SUCCESS [  1.646 s]
        [INFO] Netty/Transport/UDT ................................ SUCCESS [  2.599 s]
        [INFO] Netty/Example ...................................... SUCCESS [  4.193 s]
        [INFO] Netty/Testsuite .................................... SUCCESS [  2.380 s]
        [INFO] Netty/Testsuite/Autobahn ........................... SUCCESS [  1.492 s]
        [INFO] Netty/Testsuite/OSGI ............................... SUCCESS [  2.271 s]
        [INFO] Netty/Microbench ................................... SUCCESS [  3.277 s]
        [INFO] Netty/Transport/Native/Epoll ....................... SUCCESS [  4.216 s]
        [INFO] Netty/All-in-One ................................... SUCCESS [  1.707 s]
        [INFO] Netty/BOM .......................................... SUCCESS [  0.005 s]
        [INFO] Netty/Tarball ...................................... SUCCESS [  0.480 s]
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time:  56.150 s
        [INFO] Finished at: 2020-10-26T00:30:30+08:00
        [INFO] ------------------------------------------------------------------------
        ```

