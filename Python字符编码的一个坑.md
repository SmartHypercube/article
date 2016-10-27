---
title: Python字符编码的一个坑
tags:
  - Python
  - Unicode
  - 笔记
categories:
  - 底层
date: 2016-07-22 13:05:51
---

如果Python的标准输出不是终端而是管道的话，其文件编码默认是ascii，测试时我们常常在终端下运行程序，很难想到一个测试时完全没问题的程序假如被放在管道里，试图输出Unicode字符时居然就会出现字符编码错误。之前我做微信公众号服务器时就发生了这样的情况，一段Python代码要被PHP的`system`函数调用执行，本地调试完全没有问题，但是上线工作时系统就不能处理Unicode字符。我在网上搜索了很久，终于从若干篇stackoverflow文章中总结出了问题和解决方案。

<!--more-->Python2的解决方案很简单，因为其允许向文件写入bytes，也就是Unicode字符串调用encode方法后得到的字节数据流。因此只需要把所有输出字符串都`encode('utf8')`即可。

Python3的解决方案比较复杂，其不允许向ascii编码的文件写入bytes，因此我们需要重新以UTF-8编码打开标准输出。不罗嗦了，在要处理Unicode，且可能在管道中运行的程序开头加上这一段代码，即可确保安全：

    import sys
    import codecs
    sys.stdout = codecs.getwriter('utf8')(sys.stdout.detach())
    sys.stderr = codecs.getwriter('utf8')(sys.stderr.detach())

至于`stdin`是不是也有类似的问题，我还没尝试，但很可能也是需要类似处理一下的。

顺便再说一个小问题，在某些条件下（或许是“在管道中运行”），Python3的`open`函数默认编码居然会是ascii。觉得每次调用`open`时都指定`encoding='utf8'`很丑陋的话，或者代码中调用的某些库会自己`open`文件的话，可以在代码开头加上这个来改变`open`的默认编码：

    import functools
    open = functools.partial(open, encoding='utf-8')

这个问题我没有仔细研究，它的产生条件，以及在Python2下的情况尚不明确。

作为我最喜欢的编程语言，Python要写出能安全处理UTF-8的程序居然如此麻烦——要引入两个库，用wrapper换掉三个系统自带对象——实在是令我失望。
