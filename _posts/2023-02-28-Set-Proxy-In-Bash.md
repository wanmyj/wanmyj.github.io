---
layout: post
tags: [iptables, tproxy, config]
categories: [PROXY]
---

```
# 本机代理
export http_proxy=socks5://127.0.0.1:12346 
export https_proxy=socks5://127.0.0.1:12346
# 同局域网代理
export http_proxy=socks5://192.168.1.113:12346  
export https_proxy=socks5://192.168.1.113:12346
# http代理
export http_proxy=10.1.1.2:8080
export https_proxy=10.1.1.2:8080
# 取消代理
unset http_proxy
unset https_proxy
# 检查代理是否工作 预期返回301
curl -so /dev/null -w "%{http_code}" google.com -x socks5://127.0.0.1:12346
# git push with proxy
git -c http.proxy=http://xx:12345 push origin HEAD
```