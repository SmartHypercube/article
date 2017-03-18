---
title: 在Windows上使用SSHFS
tags:
  - Windows
  - SSH
  - 文件系统
date: 2017-02-28
---

我们经常有远程连接一台Linux服务器并调整它的配置，甚至是现场调试程序的需求，这就需要通过SSH连接编辑文件。生产环境中一般不会有很好用的开发工具链，尤其是像我这样喜欢把所有服务都搭建在Docker容器中的人，只得硬着头皮用难用的vi编辑器，或者基本上一样难用的未定制好的vim编辑器。

这个问题的一种解决方案是做一个一键安装和配置个性化开发环境的脚本，只要`curl ... | bash`，就能给任何一台机器装上自己喜欢的vim。但是无论如何，通过SSH使用vim仍然并不怎么愉快——鼠标的支持非常有限，剪贴板也不好用，而且小键盘数字键和中文字符也很难搞好（或许有人知道怎么写`.vimrc`能让小键盘数字键正常工作？请告诉我）。更别提给每个docker容器都装一个vim有多浪费了。

另一种更优雅的解决方案是利用SSH，将远程的文件系统映射到本地，在本地使用任何自己喜欢的工具修改文件，并实时上传更改。有点类似于NFS，只不过既然已经有能传输文件的SSH了，何必再在远程开一个NFS服务器呢？

经过一番摸索，我总结在Windows上安装SSHFS的步骤如下（Windows 10 x64）：

<!--more-->

1. 下载[win-sshfs](https://github.com/Foreveryone-cz/win-sshfs)工具的[1.5.12.8版本](https://github.com/Foreveryone-cz/win-sshfs/releases/tag/1.5.12.8)中的[压缩包](https://github.com/Foreveryone-cz/win-sshfs/releases/download/1.5.12.8/Release1.5.12.8.zip)。这不是最新版本，是最新稳定版本。最新开发版本确实无法使用。
2. 下载[Dokan](https://github.com/dokan-dev/dokany)项目的[0.7.4版本](https://github.com/dokan-dev/dokany/releases/tag/v0.7.4)中的[安装程序](https://github.com/dokan-dev/dokany/releases/download/v0.7.4/DokanInstall_0.7.4.exe)。上一步下载的win-sshfs依赖于这个版本，更新的版本无法使用。
3. 安装Dokan，解压win-sshfs，运行解压出来的程序。
4. 在右下角通知区域中点win-sshfs的图标，在弹出的管理器中点`Add`，填写配置，`Save`并`Mount`即可。

![管理器](管理器.png)

![Bingo~](效果.png)

P.S. 借助SSHFS，我终于可以在Windows系统中有一个好用的Linux开发环境了（bash on Windows有太多问题，不行）。现在采用的做法是安装一个不带图形界面的裸Debian虚拟机，添加一块Host-only网卡，配置开机自动启动SSH服务器，然后最小化虚拟机窗口，用自己喜欢的SSH客户端操作它、用SSHFS在本地挂载其中的文件，用gVim舒适地编辑程序。

补充1：Dokan依赖于x86版本的VC++2013运行时库，如果没有的话安装Dokan时它会提示你下载，点Yes并去下载安装即可，然后取消掉Dokan的安装程序，重新启动一次并完成安装。

补充2：Dokan的卸载程序有点问题，卸载不干净。卸载后需要手动删除`C:\Windows\System32\drivers\dokan.sys`这个文件，否则无法重新安装。
