---
layout: post
tags: [config, git]
categories: [GitOp]
---

# `git show`的时候，非ascii字符的文件名显示成`\nnn`形式应该怎么解决
设置 Git 配置文件：在 Git 中运行 `git config --global core.quotepath off` ，这将关闭引用路径，并允许在 Git 输出中显示非 ASCII 字符。

# `git push` ，不希望用git默认的用户名，需要用特别用户名
* 现在git必须要证书认证才能push，为了安全，也为了减少麻烦，还是在本机为每个git账户单独配置一套证书对
```

ssh-keygen -f '~/.ssh/personal-key' -b 4096

# First create a directory and initialize it
mkdir repo-dir
cd repo-dir
git init

# Configure the SSH command
git config --local core.sshcommand 'ssh -i ~/.ssh/personal_key -F /dev/null'
git config --local user.name 'YourName'
git config --local user.email 'your.email@company.com'
git config --local push.autosetupremotes true

# Add the origin URL and Pull the code
git remote add origin 'git@.....'
git pull origin main
```
* 或者，`push`时直接指定用户名(已经过时了 - August 13, 2021)
```
git push https://'B_user_name':'B_password'@github.com/B_user_name/project.git
```
至于后面的一串repo目录，直接用`git remote -v`命令查看

# 清空一切，包括submodule
```
git clean -xfd
git submodule foreach --recursive git clean -xfd
git reset --hard
git submodule foreach --recursive git reset --hard
git submodule update --init --recursive
```

# 常用的git alias
```
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.fo 'fetch origin'
git config --global alias.poh 'push origin HEAD'
```
