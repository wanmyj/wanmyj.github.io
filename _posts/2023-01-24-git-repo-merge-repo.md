---
layout: post
tags: [config]
---


# 需要把 [repo1](https://github.com/abhinavs/moonwalk.git) 里所有内容复制到一个已经存在的[repo2](xx)里面，这个仓库目前可能不是空的

可以使用 `git remote` 和 `git pull` 命令来实现此操作。

首先，本地系统上，进入现有仓库[repo2](xx)的目录。
然后，添加远程仓库地址，这里是[repo1](https://github.com/abhinavs/moonwalk.git)
```
cd your_existing_empty_repository
git remote add moonwalk https://github.com/abhinavs/moonwalk.git
```
接着，使用 `git pull` 命令将所有内容拉取到您的现有仓库中。

```
git pull moonwalk master
```

这样你就成功的把[repo1](https://github.com/abhinavs/moonwalk.git)的内容复制到了你的仓库下面了。

注意,如果你的本地现有空仓库中已经存在文件和文件夹，操作会报错。

另外,如果你想保留[repo1](https://github.com/abhinavs/moonwalk.git)的历史，可以使用 git clone 命令