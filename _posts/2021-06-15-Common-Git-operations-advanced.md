---
layout: post
tags: [config, git]
categories: [GitOp]
---

# `git show`的时候，非ascii字符的文件名显示成`\nnn`形式应该怎么解决
设置 Git 配置文件：在 Git 中运行 `git config --global core.quotepath off` ，这将关闭引用路径，并允许在 Git 输出中显示非 ASCII 字符。