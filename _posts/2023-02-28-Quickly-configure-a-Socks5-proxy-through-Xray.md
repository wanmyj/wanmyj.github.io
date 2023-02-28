---
layout: post
tags: [iptables, tproxy, config]
categories: [PROXY]
---
# 本文旨在快速配置一台境外的 CentOS7 作为Xray server，供在国内的 CentOS7 使用socks代理
根据 [此v2ex帖子](https://www.v2ex.com/t/715757) 的讨论，如果仅仅使用socks5作为代理方式，则是非常不安全的。
1. 知道你在用 Sock5
2. 可以破解你的 Sock5 密码
3. 可以获得你的 TCP 包
4. 可以知道你的 HTTPS 的 Host （因为 SNI ）
5. 不可以知道通信内容

因此 境内和境外的通信一定不能够只是用socks5，要完全加密。
# 境内和境外机器都需要的操作
## 下载安装xray
```
wget https://github.com/XTLS/Xray-core/releases/download/v1.3.0/Xray-linux-64.zip 
unzip -d ./Xray Xray-linux-64.zip 
ls -alh ./Xray 
```
## 将已有文件复制到对应目录，并创建其他所需要文件： 
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
# 境外机器的操作
## 创建config文件

之前已经创建了 Xray 的配置文件”/usr/local/etc/xray/config.json”，参考文件内容如下： 
> 
https://xtls.github.io/document/config.html#%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%85%8D%E7%BD%AE

```json
{
  "inbounds": [
    {
      "port": 12121, // 服务器监听端口
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-2324-4d53-ad4f-8cda48b30811"
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

# 境内机器的操作
## 创建config文件

之前已经创建了 Xray 的配置文件”/usr/local/etc/xray/config.json”，参考文件内容如下： 
> 
https://xtls.github.io/document/config.html#%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE
```json
{
  "inbounds": [
    {
      "port": 12346, // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "server", // 服务器地址，请修改为你自己的服务器 ip 或域名
            "port": 12121, // 服务器端口
            "users": [
              {
                "id": "b831381d-2324-4d53-ad4f-8cda48b30811"
              }
            ]
          }
        ]
      }
    },
    {
      "protocol": "freedom",
      "tag": "direct"
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "direct"
      }
    ]
  }
}
```

因为只需要socks代理，所以不用规避流量回环的问题

# 境内和境外机器都需要的操作
## 打开端口
这里分别用firewalld和iptables做示范
``` shell
# 境内机器打开12346端口
iptables -A INPUT -p tcp --dport 12346 -j ACCEPT
service iptables save 

# 境外机器打开12121端口
firewall-cmd --permanent --add-port=12121/udp 
```
## 添加xray service并enable
添加一个系统服务文件”/etc/systemd/system/xray.service”，内容如下：
参考： https://www.rultr.com/tutorials/vps/5587.html
``` config
[Unit]
Description=Xray Service
Documentation=https://github.com/xtls
After=network.target nss-lookup.target
[Service]
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
接下来参考以下链接就可以使用了 
[Set-Proxy-In-Bash]({% post_url 2023-02-28-Set-Proxy-In-Bash %})