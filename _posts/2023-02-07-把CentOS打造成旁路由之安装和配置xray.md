---
layout: post
data: "2023-01-25"
tags: [config]
categories: [CentOS7]
---


## CentOS7的防火墙管理使用的是firewalld，这在设置iptables规则时不方便，所以要先要改造成用iptables来管理防火墙

```
[Link Text]({{ site.baseurl }}{% post_url path/to/post %})
[Link Text]({% post_url path/to/post %})
```

## 下载安装xray
```
wget https://github.com/XTLS/Xray-core/releases/download/v1.3.0/Xray-linux-64.zip 
unzip -d ./Xray Xray-linux-64.zip 
ls -alh ./Xray 
```

### 根据 Xray 的官方建议，Xray 程序所使用的文件及默认位置如下： 
* xray 程序文件：/usr/local/bin/xray 
* xray 配置文件：/usr/local/etc/xray/config.json 
* geoip 规则文件：/usr/local/share/xray/geoip.dat 
* geosite 规则文件：/usr/local/share/xray/geosite.dat 
* xray 连接日志文件：/var/log/xray/access.log 
* xray 错误日志文件：/var/log/xray/error.log 
### 于是，使用如下命令将已有文件复制到对应目录，并创建其他所需要文件： 
```
cp ./Xray/xray /usr/local/bin/ 
chmod +x /usr/local/bin/xray 
mkdir -p /usr/local/share/xray 
cp ./Xray/*.dat /usr/local/share/xray/ 
mkdir -p /usr/local/etc/xray/ 
touch /usr/local/etc/xray/config.json 
mkdir -p /var/log/xray 
touch /var/log/xray/access.log 
touch /var/log/xray/error.log 
```
### 为了更好的分流体验，请替换默认路由规则文件为 Loyalsoldier/v2ray-rules-dat，否则 Xray-core 将无法加载本配置。 
```
curl -oL /usr/local/share/xray/geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat 
curl -oL /usr/local/share/xray/geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat
```
### 创建config文件

之前已经创建了 Xray 的配置文件”/usr/local/etc/xray/config.json”，参考文件内容如下： 
> 
https://xtls.github.io/document/level-2/tproxy.html#xray-%E9%85%8D%E7%BD%AE

## 添加账户，目的是使用GID规避流量回环的问题
### 使用以下命令可以创建一个名为 xray_tproxy 为 0，gid 为 23333 的用户： 
```
grep -qw xray_tproxy /etc/passwd || echo "xray_tproxy:x:0:23333:::" >> /etc/passwd 
```
### 使用如下命令可以验证创建的用户是否正常： 
```
$ sudo -u xray_tproxy id 
```

如： 
```
iptables -t mangle -A OUTPUT -j XRAY_SELF 
```
改为 
```
iptables -t mangle -A OUTPUT -m owner ! --gid-owner 23333 -j XRAY_SELF 
```
### 设置最大文件打开数为一个比较大的数字
```
ulimit -SHn 1000000 
sudo -u xray_tproxy /usr/local/bin/xray -c /usr/local/etc/xray/config.json & 
```
### 检查最大文件打开数是否设置成功 
```
cat /proc/Xray的pid/limits 
# pid 的获取方法为运行ps或ps -aux或ps -a或pidof xray
```
## 添加xray service并enable
### 添加一个系统服务文件”/etc/systemd/system/xray.service”，内容如下：
参考： https://www.rultr.com/tutorials/vps/5587.html
```
[Unit]
Description=Xray Service
Documentation=https://github.com/xtls
After=network.target nss-lookup.target
[Service]
# 这里的user注意别填错了
User=xray_tproxy
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
```
## 服务文件添加完成后，即可使用如下命令启动 Xray 并查看状态了：
```
systemctl enable --now xray
systemctl status xray
```
### 查一下启动xray的用户
```
systemctl show -pUser,UID xray
```
接下来就开始设置iptables和路由表了