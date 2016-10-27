---
title: 检查RSS是否有更新的小工具
tags:
  - Python
  - RSS
  - XML
  - 作品
  - 博客
categories:
  - 网络
date: 2016-02-09 12:40:20
---

似乎如今没有自动检查RSS更新并发邮件提醒的服务了？因为我很希望能在关注的几个大大更新博客时及时获得通知，于是我自己写了一个小脚本来做这件事，打算部署到freeshell持续运行。发出来分享一下～

<!--more-->需要在同目录下有一个list.txt，里面每行格式为：`<名称>: <RSS URL>`。例如：<del>`0x01: http://0x01.me/feed`</del>。_由于不明原因，我的博客的RSS貌似挂掉了，暂且用xkcd举例：`xkcd: https://xkcd.com/rss.xml`_

运行fetch.py会创建data目录，其中每个文件为一个源的数据。`eval(open('data/0x01').read())[1]`是一个dict，其key为各文章的guid，value为存储文章信息的dict。

每次运行fetch.py会输出上次fetch以来的更新。若读取源失败会报错并跳过这个源。

    import os, time, datetime
    import urllib.request
    import xml.etree.ElementTree as ET

    TFORM = '%a, %d %b %Y %H:%M:%S GMT'

    def parseItem(item):
        result = dict()
        for k in item.getchildren():
            result[k.tag] = k.text
        return result['guid'], result

    try:
        data = set(os.listdir('data'))
    except FileNotFoundError:
        os.mkdir('data')
        data = set()

    with open('list.txt') as f:
        os.chdir('data')
        for rss in f:
            name, _, url = rss.partition(': ')

            # get the last info
            if name in data:
                with open(name) as df:
                    lastmodi, last = eval(df.read())
            else:
                lastmodi, last = 0, dict()

            # fetch the new info
            changed = False
            try:
                with urllib.request.urlopen(url) as http:
                    t = http.getheader('Last-Modified')
                    if t:
                        t = time.strptime(http.getheader('Last-Modified'), TFORM)
                        newmodi = datetime.datetime(t.tm_year, t.tm_mon, t.tm_mday, t.tm_hour, t.tm_min, t.tm_sec, tzinfo=datetime.timezone.utc).timestamp()
                        if newmodi == lastmodi:
                            continue
                    else:
                        newmodi = 0
                    for item in ET.parse(http).iterfind('./channel/item'):
                        guid, detail = parseItem(item)
                        if guid not in last:
                            changed = True
                            last[guid] = detail
                            print(name + ': ' + guid)
            except Exception as e:
                print('fail to fetch <%s>: %s' % (name, str(e)))
                continue

            # write to the file
            with open(name, 'w') as df:
                df.write(repr((newmodi, last)))
