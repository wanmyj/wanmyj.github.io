---
layout: post
tags: [config, network, tcpdump, wireshark]
categories: [CentOS7]
---

Windows安装putty，CentOS7上安装tcpdump。   
运行之前putty 先登录一次CentOS的ssh后端

```
# ideapad
"D:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1 -s 0 -w - not port 2002" | wireshark -k -i -  
// above version >= 7.0  
// below version < 7.0  
"C:\Users\current_user\Downloads\plink.exe" -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.100 -s 0 -w - host 192.168.31.212 or 192.168.31.1 and not port 2002" | wireshark -k -i -
```

```
# 第一次使用要confirm一下host key，到putty里连一次就好了
"C:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 22 root@192.168.1.113 "tcpdump -n -i enp0s25 -s 0 -c 200 ip host 192.168.1.2 -w -" | wireshark -k -i -
```

```
抓100个包，不包含2022端口
"D:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.100 -s 0 -c 100 -w - not port 2002" | wireshark -k -i -

只抓某个 mac 地址的包
"D:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.10 ether host d8:e2:df:1d:28:ef -s 0 -c 1000 -w -" | wireshark -k -i -

抓wan口 port 53 包 （不能用）可能是因为nginx或者dnsmasq 
"C:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.100 -s 0 -c 1000 -w - port 53" | wireshark -k -i - 
```
```
抓wan口(eth1.100)全部包 
"C:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.100 -s 0 -c 1000 -w -" | wireshark -k -i -

抓多个IP的包  -n 显示IP
tcpdump -n -i eth1 '(ip host 116.228.111.118 or 180.168.255.18 or 1.1.1.1 or 192.168.5.107) and not port 2002'
```
```
抓WAN口的包 要用pppoes
tcpdump -n -c 100 -i eth1.100 pppoes and port 443

抓WAN口100个的DNS包
"C:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth1.100 -s 0 -c 100 -w - pppoes and port 53" | wireshark -k -i -

抓LAN(eth0.10)口某一个IP(192.168.5.100)的3000个包
"C:\Program Files\PuTTY\plink.exe" -batch -ssh -pw password -P 2002 root@192.168.5.1 "tcpdump -n -i eth0.10 ip host 192.168.5.100 -s 0 -c 3000 -w -" | wireshark -k -i -
```