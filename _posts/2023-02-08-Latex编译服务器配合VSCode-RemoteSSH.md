---
layout: post
tags: [LaTex]
categories: [CentOS7]
---
## 如何自己搭一个LaTex编译服务器 

要使用latex编辑论文等，但是Overleaf用起来不够方便，家里的Homelab反正也是能用起来，直接开搞，直接用的root用户

### 安装必要的依赖包

```
yum install perl-Digest-MD5 fontconfig-devel libXmu-devel libpng-devel libjpeg-devel libtiff-devel gd-devel ghostscript-devel libXaw-devel ncurses-devel gtk2-devel perl-Tk perl-ExtUtils-MakeMaker perl-Digest-SHA1 perl-ExtUtils-ParseXS perl-ExtUtils-CBuilder perl-ExtUtils-Embed perl-Test-Harness perl-Test-Simple
```
### 开始拉texlive的iso，拉完以后就挂载，挂载后执行安装，国内玩家选一个国内的源
```
wget https://mirrors.cqu.edu.cn/CTAN/systems/texlive/Images/texlive.iso

mount -o loop texlive.iso /mnt/tex
cd /mnt/tex
./install-tl -repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
```

主菜单中可以选择需要安装的宏包，分为scheme（大方案）和collection（更精细的集合）两种方式。我的建议是进入collection，去掉所有除中文、英文的其他外语种的宏包（有需要的可自我保留，之后也可随时下载回来），再去除不常用的LuaLaTeX编译引擎相关的宏包，最终能省大约1个G的空间。其他的宏包建议全部选择，最终大概占用空间7G。

按照最后提示的instruction，添加环境变量

```
export MANPATH=${MANPATH}:/usr/local/texlive/2022/texmf-dist/doc/man
export INFOPATH=${INFOPATH}:/usr/local/texlive/2022/texmf-dist/doc/info
export PATH=${PATH}:/usr/local/texlive/2022/bin/x86_64-linux
```
生效环境变量 `source  ~/.bash_profile`

没用的文件可以删除了
```
umount ~/mnt/
rm -f texlive.iso
```

测试一下
```
xelatex -v
```
然后vscode ssh到机器上(此处步骤略去，就是使用RemoteSSH插件)，安装`LaTex Workspace`插件

创建或者打开LaTex project文件夹，在vscode中新建一个workspace的setting.json，拷贝进去一下部分，[参考来源](https://zhuanlan.zhihu.com/p/166523064)
```json
{
    "latex-workshop.latex.autoBuild.run": "never",
    "latex-workshop.showContextMenu": true,
    "latex-workshop.intellisense.package.enabled": true,
    "latex-workshop.message.error.show": false,
    "latex-workshop.message.warning.show": false,
    "latex-workshop.latex.tools": [
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "-outdir=%OUTDIR%",
                "%DOCFILE%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
    "latex-workshop.latex.recipes": [
        {
            "name": "XeLaTeX",
            "tools": [
                "xelatex"
            ]
        },
        {
            "name": "PDFLaTeX",
            "tools": [
                "pdflatex"
            ]
        },
        {
            "name": "BibTeX",
            "tools": [
                "bibtex"
            ]
        },
        {
            "name": "LaTeXmk",
            "tools": [
                "latexmk"
            ]
        },
        {
            "name": "xelatex -> bibtex -> xelatex*2",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex"
            ]
        },
        {
            "name": "pdflatex -> bibtex -> pdflatex*2",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
        },
    ],
    "latex-workshop.latex.clean.fileTypes": [
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk"
    ],
    "latex-workshop.latex.autoClean.run": "onFailed",
    "latex-workshop.latex.recipe.default": "lastUsed",
    "latex-workshop.view.pdf.internal.synctex.keybinding": "double-click"
}
```

Viola! 点击侧边栏的LaTex，编译就好啦