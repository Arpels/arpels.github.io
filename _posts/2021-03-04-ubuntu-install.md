---
title: 'Ubuntu 20.04 虚拟机安装及环境配置记录'
date: 2021-03-04
permalink: /posts/2021/03/ubuntu_install/
tags:
  - Ubuntu
---

Ubuntu 20.04 装机记录。

*  目录
{:toc}

前段时间在虚拟机上装 SEAL，可能是 16.04 太过古老，gcc 更新到最新 SEAL 安装依然会报各种奇怪的错误，懒得折腾，遂顺势换 20.04 了。

## 1. 主要流程
（1） 下载吾爱破解的 VMware Workstation 15 pro 和 Ubuntu 官网最新的 20.04 镜像。

（2） 添加新的虚拟机，分配合适的内存和磁盘空间，完成后安装 Ubuntu。

（3） Ubuntu 一系列设置，划分几个分区，以及语言时间等等的选择。

（4） 更改一些虚拟机设置，提升运行速度。

（5） 装机。

（6） 配置 SEAL 安装需要的环境。

（7） 安装 SEAL。

## 2. 虚拟机安装
虚拟机安装是老生常谈了，随便一搜就有很多例子可以参照。

内存分配看宿主机的配置而定，不超过 VMware 定的上限即可。

语言我习惯上设置成英文。

如果装好虚拟机但感觉有点卡顿，可以尝试修改一些设置。可以参考[这篇文章](https://zhuanlan.zhihu.com/p/265868395)。


## 3. 装机
### （1） apt-fast
简而言之就是一个多线程并行的 `apt-get`。可以提高安装速度。

### （2） Qv2ray
在国内，没有梯子寸步难行。先在 `v2ray` 官网上下载 `Linux` 版的 `v2ray-core`，`chmod+x` 给 `v2tcl` 和 `v2ray` 可执行权限，然后在 `Qv2ray` 的首选项里指定内核所在路径即可。

### （3） 开发环境
可以直接 `sudo apt-get install build-essential`。

如果需要最新或特殊版本可以去官网找，一般官网会给一键脚本。

### （4） Terminator+zsh+ohmyzsh
用功能更强大的 `Terminator` 取代自带的 `bash`。

设置 `Terminator` 和 `zsh` 为默认：
```shell
gsettings set org.gnome.desktop.default-applications.terminal exec /usr/bin/gnome-terminal
gsettings set org.gnome.desktop.default-applications.terminal exec-arg "-x"
sudo chsh -s /bin/zsh
```

关于 theme 上，`Terminator` 我用的是 `dracula`，`zsh` 用的 `ys`。

有一点需要注意的是目前官网上的 `dracula` 关于 `font` 的一行代码是有问题的，直接用会让字体变得很奇怪，把那行删掉就行。

此外，开了一些常用的插件。常用路径、代码高亮、代码自动补全等等，这几个都非常好用。还可以用一行代码，让每次开启终端时出现一个随机动物说一句名言。
```shell
quote | cowsay -f $(ls /usr/share/cowsay/cows/ | shuf -n1)
```

我的终端最终效果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_c9e654e34affdf923d82ef507158c237.png)


### （5） 软件源与终端开代理
国内一般会选择换国内的源。但我觉得开梯子直接用国外的源更方便，也能避免以前用国内源遇到的一些不必要的问题。

由于 `Qv2ray` 不会影响终端，所以我们需要自己在 `~/.zshrc` 里加几行代码。

而且，`git` 等有些命令是单独的，不会受到环境变量影响，对这种要单独写一行代码，如下。

另一种方法是用 `proxychains`。
```shell
# proxy list
alias proxy='git config --global http.proxy http://127.0.0.1:8889; git config --global https.proxy https://127.0.0.1:8889; export http_proxy=http://127.0.0.1:8889; export https_proxy=https://127.0.0.1:8889; export ftp_proxy=ftp://127.0.0.1:8889; curl cip.cc'
alias unproxy='git config --global --unset http.proxy; git config --global --unset https.proxy; unset http_proxy; unset https_proxy; unset ftp_proxy; curl cip.cc'
```

### （6） 浏览器与输入法
输入法我装的是 `Google拼音`。浏览器比起火狐还是更习惯 `chrome`，很多插件太香了。

现在很多功能只需要浏览器就能实现了，而且 `chrome` 还有很方便的同步功能，完全可以满足很多需求，大大减轻我们装机的烦恼。

### （7） vscode 与 sublime
我不怎么用 `vim`，我装的是 `vscode` 和 `sublime`。`sublime` 各种插件网上一搜就有。

### （8） 写文档与 markdown
我也曾用过一段时间 `Typora`，但我感觉没什么必要装。

写文档直接开浏览器用 `notion` 或者 `hackmd` 就够了，本地的话 `sublime` 装一个 `markdown` 插件就行。

### （9） alias与软链接
这两个小技巧可以带来极大的方便。

### （10） 其他
其他花哨的东西还有很多，一搜就能找到，比如 `Gnome Extensions` 就有许多有意思的，比如 `eDEX-UI`。

不过我之所以用 `Linux` 就是为了简洁，我觉得一个好用的终端，一个好用的浏览器，一张好看的壁纸，再加上 vscode 就已经足够了。

最终效果图：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_95535da9c542f1fc01807badeb9e653b.png)


## 4. SEAL安装
SEAL 的安装对着 `github` 里的文档安装即可。

我安装的时候主要碰到两个坑点。

一个是环境要求，`Clang++ (>= 5.0) or GNU G++ (>= 6.0), CMake (>= 3.12)`。一开始我在16.04上把 `gcc` 和 `cmake` 都更新到最新了，但安装的时候依然报了一堆奇怪的错误。懒得折腾，直接换 20.04 就解决了。

另一个是 `zstandard`。主要是 `GFW` 或者运营商的原因，不开梯子的话会在 `Zstandard install...` 这里一直卡住。如果强制暂停有时重新安装时还会报错，主要是 `git` 的问题：
```shell
GIT fatal: ambiguous argument 'HEAD': unknown revision or path not in the working tree
```

解决方法：

可以参考[这个问题](https://stackoverflow.com/questions/12267912/git-fatal-ambiguous-argument-head-unknown-revision-or-path-not-in-the-workin)。
```shell
git commit --allow-empty -n -m "Initial commit".
```

然后打开梯子重新安装就好了。

随便建一个 `cpp` 就可以试试有没有安装好了。

安装好的效果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_a82adf8dfb28f89be6d029e41c5c98e1.png)

***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
