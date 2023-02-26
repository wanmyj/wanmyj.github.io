---
layout: post
tags: [cmd, shell]
categories: [CentOS7]
---
# CentOS7的常用shell操作 

## 创建RSA密钥对，把公钥给其他服务使用
Shell命令 `ssh-keygen -t rsa -b 4096` ，一路回车   
查看生成的公钥 `cat ~/.ssh/id_rsa.pub`

## 使用其他设备的公钥，配置ssh登录后台

Shell命令 `vi /etc/ssh/sshd_config`   
取消注释 `PubkeyAuthentication yes`   
找到下面一行 `AuthorizedKeysFile xxxxx`   
> 一般是 `AuthorizedKeysFile .ssh/authorized_keys`

然后`cd ~/.ssh/ && vi authorized_keys`   
另起一行，把其他设备的公钥粘贴在里面，esc :wq 退出   
重新加载一下sshd 就可以了 `service sshd reload`