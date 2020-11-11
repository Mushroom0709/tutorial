#  CentOS配置静态IP
## CentOS 7
1. 确定网卡名称  
   `[x@centos7 ~]# ip addr`
2. 修改配置文件  
   +  `[x@centos7 ~]# vi /etc/sysconfig/network-scripts/网卡名`
   +  修改 `BOOTPROTO=dhcp` 为 `BOOTPROTO=static`
   +  修改 `ONBOOT=no` 为 `ONBOOT=yes`
   +  增加
      ```shell
      IPADDR=IP地址  
      NETMASK=子网掩码  
      GATEWAY=网关  
      DNS1=主DNS  
      DNS2=备DNS
      ```
3. 生效配置  
   `[x@centos7 ~]# systemctl restart network`

-----------------------------------------------------------------------------------