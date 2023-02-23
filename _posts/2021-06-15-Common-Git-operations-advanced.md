---
layout: post
tags: [config, git]
categories: [GitOp]
---

# git show 的时候，非ascii字符的文件名显示成 \nnn 形式应该怎么解决
设置 Git 配置文件：在 Git 中运行 `git config --global core.quotepath off` ，这将关闭引用路径，并允许在 Git 输出中显示非 ASCII 字符。

# git push ，不希望用git默认的用户名，需要用特别用户名
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

# 清空一切，包括 submodule 
```
git clean -xfd
git submodule foreach --recursive git clean -xfd
git reset --hard
git submodule foreach --recursive git reset --hard
git submodule update --init --recursive
```

# 常用的 git alias 
```
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.fo 'fetch origin'
git config --global alias.poh 'push origin HEAD'
```
# Git - how to find first commit of specific branch
加入我们知道`branch`是从 `master` 拉出来的
```
git log master..branch --oneline | tail -1

git cherry master -v | head -n 1
```
`cherry` - Shows the commits on your current branch that aren't present on a branch upstream (docs). An upstream (master in this case) is a point your current branch derives from.   
-v - Shows commit names (instead of just SHA's) - verbose.   
head - Unix command which prints n number of lines from a text

# Update branch without checkout
```
# 更新本地的master版本
Git fetch origin master:master
# 更新本地的某个branch-b，使其指向branch-a
Git branch -f branch-b branch-a 
```
# Git error: Encountered 7 file(s) that should have been pointers, but weren't
```
git rm --cached -r .
git reset --hard
git rm .gitattributes
git reset .
git checkout .

也要rm  .gitattributes
```
# How to add file to existing stash in git?
No you can't

# How can I git stash a specific file?
```
git stash push <path>
git stash push -m welcome_cart app/views/cart/welcome.thtml
```
# How to compare files from two different branches
```
git diff mybranch master -- myfile.cs
```
# How would I extract a single file (or changes to a file) from a git stash
```
$ git checkout stash@{0} -- <filename>
```

# Add only non-whitespace changes
```
git diff -U0 -w --no-color | git apply --cached --ignore-whitespace --unidiff-zero -
```

# Checkout Git Tag
```
$ git checkout tags/<tag> -b <branch>
```

# GIT 查找两个 commit 之间的 commit
```
good 1d4712d3394e1373
bad 03d3c1355d 
git rev-list --ancestry-path 1d4712d3394..03d3c1355d
```

# Stage only deleted files with git add
```
git ls-files --deleted | xargs git add
```

# Bisect 用法
```
good 1d4712d33 
bad 03d3c1355d 
git bisect start --first-parent 03d3c1355d 1d4712d33
git bisect good
git bisect reset bisect/bad
```
# Git modify old commit
```
git rebase --interactive commit^
# In the default editor, modify pick to edit in the line mentioning commit.
# Save the file and exit. git will interpret and automatically execute the commands in the file. You will find yourself in the previous situation in which you just had created commit modifying
git commit --all --amend --no-edit
git rebase --continue
```

# Git compare modified files under a folder btwn 2 branches
```
git diff --name-status branch1..branch2 -- Testing/test.cs Testing/Automation/
```
# 恢复某文件到某个branch or commit
git restore -s r2022/trunk -- Testing/test.cs
# git log pretty
```
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
```
# git log duration
```
git log --after="2014-02-12T16:36:00-07:00"
git log --before="2014-02-12T16:36:00-07:00"
git log --since="1 month ago"
git log --since="2 weeks 3 days 2 hours 30 minutes 59 seconds ago"
```
# remove local branches that have gone remotes:
```
git fetch -p && for branch in $(git branch -vv | grep ': gone]' | awk '{print $1}'); do git branch -D $branch; done
```
# HEAD^ HEAD~ 的区别
Here is an illustration, by Jon Loeliger. Both commit nodes B and C are parents of commit node A. Parent commits are ordered left-to-right. (N.B. The git log --graph command displays history in the opposite order.)
```
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A
A =      = A^0
B = A^   = A^1     = A~1
C = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```
From <https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git> 