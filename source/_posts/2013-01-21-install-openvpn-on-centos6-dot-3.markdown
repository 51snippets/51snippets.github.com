---
layout: post
title: "install openvpn on centos6.3"
date: 2013-01-21 10:32
comments: true
categories: 
---

#centos6.3 安装openvpn
##准备工作
OpenVPN需要TUN设备支持，还需要iptables的nat模块支持。
如果在OpenVZ虚拟化的VPS上，需要管理员在主机上打开TUN/TAP设备。
<!-- more -->
####1、查看tun/tap是否激活

######主机检查TUN
```
modinfo tun
```
不报错则表示支持

######OpenVZ虚拟化的VPS上检查TUN/TAP
```
cat /dev/net/tun
```

如果结果为active，输出为：

```
cat: /dev/net/tun: File descriptor in bad state
```


####2、添加RPM源
32bit Package:

CentOS 5:

```
wget http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el5.rf.i386.rpm
```

CentOS 6:

```
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-1.el6.rf.i686.rpm
```

64bit Package:

CentOS 5:

```
wget http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el5.rf.x86_64.rpm
```

CentOS 6:

```
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
```

下载安装LZO，LZO是一种压缩算法

```
wget http://openvpn.net/release/lzo-1.08-4.rf.src.rpm
```

安装rpm（需要编译环境的支持）

additional rpms

```
yum install gcc make rpm-build autoconf.noarch zlib-devel pam-devel openssl-devel -y
rpmbuild --rebuild lzo-1.08-4.rf.src.rpm
rpm -Uvh lzo-*.rpm
rpm -Uvh rpmforge-release*
```

LZO另外一种安装方法
```
wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.06.tar.gz
tar -zxvf lzo-2.06.tar.gz
cd lzo-2.06
./configure
make
make install
```


####3、安装openvpn
```
yum install openvpn -y
```

##配置


####1、将easy-rsa sample目录拷到```/etc/openvpn```

```
cp -R /usr/share/doc/openvpn-2.2.2/easy-rsa/ /etc/openvpn/
```

修改```/etc/openvpn/easy-rsa/2.0/vars```

```
export KEY_CONFIG=/etc/openvpn/easy-rsa/2.0/openssl-1.0.0.cnf
```

####2、创建证书、公钥私钥
```
cd /etc/openvpn/easy-rsa/2.0
chmod 755 *
source ./vars
./vars
./clean-all
```

创建CA

```
./build-ca //一路默认选项
```

创建Server key

```
./build-key-server server //server name
```

创建Client key

```
./build-key client1 //client name
```

创建diffie-hellman文件

```
./build-dh
```

创建tls-auth 的key

```
openvpn --genkey --secret ta.key
mv ta.key keys/
```

####3、配置server.conf
拷贝配置示例

```
cp /usr/share/doc/openvpn-2.2.2/sample-config-files/server.conf /etc/openvpn
```

```
dev tun
proto tcp
port 1194
ca ca.crt
cert /etc/openvpn/easy-rsa/2.0/keys/server.crt
key /etc/openvpn/easy-rsa/2.0/keys/server.key
dh /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem
tls-auth /etc/openvpn/easy-rsa/2.0/keys/ta.key 0
#user nobody
#group nogroup
server 10.8.0.0 255.255.255.0
persist-key
persist-tun
status openvpn-status.log
verb 3
#client-to-client
keepalive 5 30
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
comp-lzo
```

####4、其它设置

######设置IP包转发
修改/etc/sysctl.conf

```
net.ipv4.ip_forward = 1
```

重启

```
sysctl -p
```

######Iptables 设置

xen and KVM based VPS’s
```
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
OpenVZ based VPS’s
```
iptables -t nat -A POSTROUTING -o venet0 -j SNAT --to-source 主机IP地址
```
or
```
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source 主机IP地址
```

CSF防火墙设置

```
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 10.8.0.0/24 -j ACCEPT
iptables -A FORWARD -j REJECT
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -j SNAT --to-source 主机IP地址
```

保存设置
```
service iptables save
```

如果iptables save报错
```
iptables: Saving firewall rules to /etc/sysconfig/iptables: /etc/init.d/iptables: line 268: restorecon: command not found
```
要安装一个软件包：
```
yum install policycoreutils
```

##启动openvpn

```
service openvpn start
chkconfig openvpn on
```

##客户端配置

从服务器中下载所需文件至客户端config目录
```
ca.crt
client1.crt
client1.key
ta.key
```

创建客户端配置文件`client.vpn`
```
client
dev tun
proto udp
remote VPN主机IP地址
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client1.crt
key client1.key
tls-auth ta.key 1
comp-lzo
verb 3
```

##心得
客户端和服务器端配置需保持一致
服务器端口可以修改
可以改成tcp方式，貌似在某些网络环境下比较稳定


##参考链接
[install-openvpn-on-centos](https://safesrv.net/install-openvpn-on-centos/)
[how-to-install-vpn-server-with-openvpn-on-centos-6](http://vjetnamnet.com/how-to-install-vpn-server-with-openvpn-on-centos-6/)
[CentOS安装OpenVPN](http://www.live-in.org/archives/1112.html)