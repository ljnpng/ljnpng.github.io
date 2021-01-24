---
title: VirtualBox + CentOS8
date: 2021-01-15 00:24:13
top: false
---

# VirtualBox + CentOS8

## 下载

下载地址：[网易镜像](http://mirrors.163.com/centos/8/isos/x86_64/)

版本说明

CentOS-Stream是8后续一个发行版本。Intel的CPU选择x86-64架构的包就可以了。boot版本的iso是用作启动修复使用的，dvd是完整的版本，还有一个mini版只包含了基本的组件，用作练习linux命令来说的话其实选择mini版就够了。

## 安装

跟着安装向导填写名称分配内存硬盘就可以了。

## 网络设置

用虚拟机安装CentOS默认是没有打开网络的，需要在VirtualBox里配置网卡，以及在CentOS里开启网络。

1. 网络选择桥接网卡(桥接网卡可以主机和虚拟机可以互相访问)

   ![image-20210124235954868](https://raw.githubusercontent.com/ljnpng/ImageHostingService/main/image-20210124235954868.png)

2. `nmcli`命令查看网络设备，原本enp0s3是disconnect状态，所以需要去修改配置文件启动该网络设备

   ```bash
   [neo@bogon ~]$ nmcli
   enp0s3: connected to enp0s3
           "Intel 82540EM"
           ethernet (e1000), 08:00:27:AB:BD:07, hw, mtu 1500
           ip4 default
           inet4 192.168.1.103/24
           route4 0.0.0.0/0
           route4 192.168.1.0/24
           inet6 fe80::cc46:2496:5ae7:a786/64
           route6 fe80::/64
           route6 ff00::/8
   
   virbr0: connected (externally) to virbr0
           "virbr0"
           bridge, 52:54:00:ED:2C:F0, sw, mtu 1500
           inet4 192.168.122.1/24
           route4 192.168.122.0/24
   
   lo: unmanaged
           "lo"
           loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536
   
   virbr0-nic: unmanaged
           "virbr0-nic"
           tun, 52:54:00:ED:2C:F0, sw, mtu 1500
   
   DNS configuration:
           servers: 218.104.128.106 58.22.96.66
           interface: enp0s3
   
   Use "nmcli device show" to get complete information about known devices and
   "nmcli connection show" to get an overview on active connection profiles.
   
   Consult nmcli(1) and nmcli-examples(7) manual pages for complete usage details.
   
   ```

3. 修改ifcfg-enp0s3配置

   ```bash
   [neo@bogon ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=dhcp
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=enp0s3
   UUID=a8a910be-884b-4cc6-9733-df6845fcc1dd
   DEVICE=enp0s3
   ONBOOT=yes #这个里默认是no，需要改成yes
   ```

   

4. 重启网络

   ```bash
   [neo@bogon ~]$ sudo systemctl restart NetworkManager #重启网络
   ```

   

5. 虚拟机测试

   ```bash
   [neo@bogon ~]$ ping baidu.com #测试网络连接
   PING baidu.com (220.181.38.148) 56(84) bytes of data.
   64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=1 ttl=49 time=49.8 ms
   64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=2 ttl=49 time=50.4 ms
   ```

   

6. 主机ssh连接虚拟机

   ```bash
   liao@DESKTOP-NIFNKS6 MINGW64 ~
   $ ssh neo@192.168.1.103
   neo@192.168.1.103's password:
   Activate the web console with: systemctl enable --now cockpit.socket
   
   Last login: Sun Jan 24 11:02:08 2021 from 192.168.1.108
   ```

## 无图形界面启动

VirtualBox支持无图形界面启动，然后通过ssh控制虚拟机即可。