---
layout: post
data: "2023-01-25"
tags: [config]
categories: [CentOS7]
---


## CentOS7的防火墙管理使用的是firewalld，这在设置iptables规则时不方便，所以要先要改造成用iptables来管理防火墙

[Reference](https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7)

Linux的防火墙管理，底层都是netfilter，用户层本来是iptables，CentOS7之后在这上面又加了一层firewalld，firewalld也是操作iptables实现的

第一步：保存当前的firewalld工作所用的iptables rules
```
iptables -S         | tee ~/firewalld_iptables_rules_filter 
ip6tables -S        | tee ~/firewalld_ip6tables_rules_filter 
iptables -S -t nat  | tee ~/firewalld_iptables_rules_nat 
ip6tables -S -t nat | tee ~/firewalld_ip6tables_rules_nat 
iptables -S -t mangle   | tee ~/firewalld_iptables_rules_mangle 
ip6tables -S -t mangle  | tee ~/firewalld_ip6tables_rules_mangle

cat firewalld_iptables* > firewalld_iptables_combined 
cat firewalld_ip6tables* > firewalld_ip6tables_combined
grep 'ACCEPT\|DROP\|QUEUE\|RETURN\|REJECT\|LOG\|MASQUERADE' ~/firewalld_iptables_combined | tee ~/firewalld_iptables_rules_essence 
grep 'ACCEPT\|DROP\|QUEUE\|RETURN\|REJECT\|LOG\|MASQUERADE' ~/firewalld_ip6tables_combined | tee ~/firewalld_ip6tables_rules_essence
```
直接dump的话，文件内容可能非常多，因为firewalld会生成管理框架，我们不需要再使用这些管理框架，也就不用这些rules，只需要获取关键的rule就好，即`ACCEPT, DROP, QUEUE, RETURN, REJECT, LOG, MASQUERADE`   

重要的rule用此命令产生
```
# Rules that only jump to user-created chains will not be shown.   
grep 'ACCEPT\|DROP\|QUEUE\|RETURN\|REJECT\|LOG\|MASQUERADE' ~/firewalld_iptables_rules_filter
```
如果没做什么修改，mangle表里没任何东西，仅开启IP转发，filter和nat的v4表里多一行，v6表里没什么变化

第二步：安装iptables service
```
yum install iptables-services
```
This will download and install the systemd scripts used to manage the iptables service. It will also write some default iptables and ip6tables configuration files to the /etc/sysconfig directory.

第三步：建立iptables rules
by modifying the /etc/sysconfig/iptables and /etc/sysconfig/ip6tables files. These files hold the rules that will be read and applied when we start the iptables service.
先看一下信息
```
sudo head -2 /etc/sysconfig/iptables
```
如果是以下，这俩文件随便编辑
```
# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
```
编辑脚本见下面，编辑完做个测试
```
sudo sh -c 'iptables-restore -t < /etc/sysconfig/iptables'
sudo sh -c 'ip6tables-restore -t < /etc/sysconfig/ip6tables'
```
如果是以下，This means that the system-config-firewall management tool is installed and being used to manage this file. Any manual changes will be overwritten by the tool.
```
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
```
要用这个工具来编辑
```
sudo system-config-firewall-tui
```

第四步，关闭firewalld service，启动iptables service
```
sudo systemctl stop firewalld && sudo systemctl start iptables; sudo systemctl start ip6tables
sudo firewall-cmd --state
sudo iptables -S 
sudo ip6tables -S
```
这时候the iptables and ip6tables services are active，现在要测试一下网络，万一有什么异常，重启以后还可以恢复

第五步，关闭firewalld service，启动iptables service
```
sudo systemctl disable firewalld
sudo systemctl mask firewalld
sudo systemctl enable iptables 
sudo systemctl enable ip6tables
```

番外篇 Debug
```
iptables -t raw -A PREROUTING --source 192.168.1.2 -j TRACE 
iptables -t raw -A OUTPUT -p tcp --destination 192.168.0.0/24 --dport 80 -j TRACE
# to find the trace message
dmesg -w
```

规则文件(带透明网关)记得把 /proc/sys/net/ipv4/ip_forward 设置成1
/etc/sysconfig/iptables
```
*filter 
:INPUT ACCEPT [0:0] 
:FORWARD ACCEPT [0:0] 
:OUTPUT ACCEPT [0:0] 
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -p icmp -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT 
-A INPUT -m conntrack --ctstate INVALID -j DROP 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT 
-A FORWARD -i lo -j ACCEPT 
-A FORWARD -p icmp -j ACCEPT 
-A FORWARD -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT 
-A FORWARD -m conntrack --ctstate INVALID -j DROP 
-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
-A OUTPUT -o lo -j ACCEPT 
COMMIT 
*nat 
:PREROUTING ACCEPT [0:0] 
:INPUT ACCEPT [0:0] 
:OUTPUT ACCEPT [0:0] 
:POSTROUTING ACCEPT [0:0] 
-A POSTROUTING ! -o lo -j MASQUERADE 
COMMIT
```
/etc/sysconfig/ip6tables
```
*filter 
:INPUT ACCEPT [0:0] 
:FORWARD ACCEPT [0:0] 
:OUTPUT ACCEPT [0:0] 
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT 
-A INPUT -d fe80::/64 -p udp -m udp --dport 546 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT 
-A INPUT -p ipv6-icmp -j ACCEPT 
-A INPUT -m conntrack --ctstate INVALID -j DROP 
-A INPUT -j REJECT --reject-with icmp6-adm-prohibited 
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT 
-A FORWARD -i lo -j ACCEPT 
-A FORWARD -m conntrack --ctstate INVALID -j DROP 
-A FORWARD -p ipv6-icmp -j ACCEPT 
-A FORWARD -j REJECT --reject-with icmp6-adm-prohibited 
-A OUTPUT -o lo -j ACCEPT 
COMMIT
```