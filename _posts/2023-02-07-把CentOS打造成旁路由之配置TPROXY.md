---
layout: post
data: "2023-01-25"
tags: [config]
categories: [CentOS7]
---


## is tproxy really needed
> 
CentOS7的虽然有tproxy模块，但是默认不一定加载，虽然配置xray透明网关也可以用redir，不一定需要tproxy，但是tproxy是一个更先进的方式

## check Kernel for TPROXY support

```
grep TPROXY /boot/config-$(uname -r)
```
expect result:
```
CONFIG_NETFILTER_XT_TARGET_TPROXY=m
```

## check if Kernel TPROXY module is available

```
ls /lib/modules/$(uname -r)/kernel/net/netfilter | grep TPROXY
```
expect result:
```
xt_TPROXY.ko.xz
```

## check if module could be loaded(verbose and dry-run)

```
sudo modprobe -v -n xt_TPROXY
```
expect result:
```
# no error or
insmod /lib/modules/4.15.0-23-generic/kernel/net/netfilter/xt_TPROXY.ko
```

## load module

```
sudo modprobe -v xt_TPROXY
```
expect result:
```
insmod /lib/modules/4.15.0-23-generic/kernel/net/netfilter/xt_TPROXY.ko
```

## check if module is loaded

```
lsmod | grep -i tproxy
```
expect result:
```
xt_TPROXY              17327  0
nf_defrag_ipv6         35104  2 xt_TPROXY,nf_conntrack_ipv6
nf_defrag_ipv4         12729  2 xt_TPROXY,nf_conntrack_ipv4
```

## 持久化

```
cd /etc/modules-load.d/
echo "xt_TPROXY" > xt_TPROXY.conf
```

## check for iptables extension 'tproxy' 

```
iptables -j TPROXY --help 
```
expect result:
```
... 
TPROXY target options: 
--on-port port Redirect connection to port, or the original port if 0 
--on-ip ip Optionally redirect to the given IP 
--tproxy-mark value[/mask] Mark packets with the given value/mask

参考
https://github.com/tinyproxy/tinyproxy/issues/181