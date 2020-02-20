---
title: Galaxy 生信平台（三）：xlsx 上传与识别
type: post
topic/tags: ["others", "tech"]
date: 2020-01-02T03:11:41.000Z
category: tech
published: true
---

我在《[**Firefox Quantum 向左，Google Chrome 向右**](https://www.yuque.com/shenweiyan/cookbook/firefox-quantum)**》**中，曾经吐槽过在 Firefox 中使用 Galaxy 上传本地的 Excel 文件时，会出现 xlsx 无法识别异常的问题。今天，我们来聊一聊原因。


## 背景

关于 Galaxy 生物信息平台，这里就不多说了，感兴趣的可以参考本专栏的前两篇文章。对于数据上传，Galaxy 不仅可以支持 NCBI SRA、EBI SRA、UCSC main table browser 等数据库数据的无缝对接。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1577937325706-b6378e9f-be06-4da9-bbc1-30315efbe348.png#align=left&display=inline&height=508&name=image.png&originHeight=508&originWidth=347&size=23558&status=done&style=none&width=347)

在本地文件的上传中，Galaxy 支持包括 ab1、arff、fasta、fastq、xlsx 在内 100 多种常见的格式数据上传。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1577935879824-f6ede4fa-0d96-4e78-bcfb-43b62526c6fd.png#align=left&display=inline&height=432&name=image.png&originHeight=432&originWidth=723&size=34902&status=done&style=none&width=723)

对于不太熟悉命令行操作的科研工作者，Excel 是他们进行批量订单提交和处理最喜欢也是最熟悉的一个数据格式，因此，我们以 Galaxy 为基础开发一部分定制化工具中，有很大的一部分都是基于 excel 文件进行处理的工具。但随之而来的问题是，所有的这些工具在 Google Chrome 下可以运行良好，但是在 Firefox 下却出现了问题。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1561788526653-5fec525f-ec2e-4aa6-829f-5f0034bc5fc9.png#align=left&display=inline&height=335&originHeight=335&originWidth=288&status=done&style=none&width=288)




## xlsx 文件上传

一开始，在办公环境下，我在内网环境部署的 Galaxy 和 [https://usegalaxy.org/](https://usegalaxy.org/) 中分别对 xlsx 格式的文件进行上传测试，发现：

- 在 Chrome 中两个 Galaxy 都能正常上传文件，没有任何错误。
- 只有在 Firefox 中两个 Galaxy 才会出现如上截图的相同 Warning。

于是，下意识的，我开始怀疑，是不是 Firefox 会针对 Excel 的文件进行了特殊处理？还是 Galaxy 的 xlsx 文件识别存在 bug？针对前一个问题，我一开始并不知道如何去验证，但对于后一个问题，我开始了另外的尝试。

- **start_cgi_http_server.sh**

```bash
#!/bin/bash

mkdir ./cgi-bin/
cp upload.cgi ./cgi-bin/
chmod +x ./cgi-bin/upload.cgi
mkdir ./upload/
python -m http.server --cgi 8080
```

- **upload.cgi**

**
```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

import cgi, cgitb, os, sys

UPLOAD_DIR = './upload'

def save_uploaded_file():
    print 'Content-Type: text/html; charset=UTF-8'
    print
    print '''
<html>
<head>
  <title>Upload File</title>
</head>
<body>
'''

    form = cgi.FieldStorage()
    if not form.has_key('file'):
        print '<h1>Not found parameter: file</h1>'
        return

    form_file = form['file']
    if not form_file.file:
        print '<h1>Not found parameter: file</h1>'
        return

    if not form_file.filename:
        print '<h1>Not found parameter: file</h1>'
        return

    uploaded_file_path = os.path.join(UPLOAD_DIR, os.path.basename(form_file.filename))
    with file(uploaded_file_path, 'wb') as fout:
        while True:
            chunk = form_file.file.read(100000)
            if not chunk:
                break
            fout.write (chunk)
    print '<h1>Completed file upload</h1>'

    print '''
<hr>
<a href="../upload.html">Back to upload page</a>
</body>
</html>
'''

cgitb.enable()
save_uploaded_file()
```

- **upload.html**

```python
<html>
<head>
  <title>Upload File</title>
</head>
<body>
  <h1>Upload File</h1>
  <form action="cgi-bin/upload.cgi" method="POST" enctype="multipart/form-data">
    File: <input name="file" type="file">
    <input name="submit" type="submit">
  </form>
</body>
</html>
```

通过这三个程序，就可以在 Linux 下启动一个简单文件上传小网站。

[![2020-01-02 15-54-41.mp4 (14.54MB)](https://note.bioitee.com/yuque/0/2020/jpeg/126032/1577957172406-365eda9f-c949-4b38-aca1-c8004374716c.jpeg?x-oss-process=image/resize,h_450)](https://www.yuque.com/shenweiyan/bio-cloud/galaxy-xlsx-issues?_lake_card=%7B%22status%22%3A%22done%22%2C%22name%22%3A%222020-01-02+15-54-41.mp4%22%2C%22size%22%3A15245946%2C%22percent%22%3A0%2C%22id%22%3A%22nXnfR%22%2C%22videoId%22%3A%225f91d7d6cc7e440683ca0df6e80732fe%22%2C%22aliyunVideoSrc%22%3Anull%2C%22taobaoVideoId%22%3A%22249884374144%22%2C%22uploaderId%22%3A126032%2C%22authKey%22%3A%22YXBwX2tleT04MDAwMDAwMTImYXV0aF9pbmZvPXsidGltZXN0YW1wRW5jcnlwdGVkIjoiMTczODE3N2U1YzY2YTQyYzU5OGIwNGRlNjNjN2Y4YWUifSZkdXJhdGlvbj0mdGltZXN0YW1wPTE1NzgwMzMwNzA%3D%22%2C%22docUrl%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fshenweiyan%2Fbio-cloud%2Fgalaxy-xlsx-issues%22%2C%22coverUrl%22%3A%22https%3A%2F%2Fnote.bioitee.com%2Fyuque%2F0%2F2020%2Fjpeg%2F126032%2F1577957172406-365eda9f-c949-4b38-aca1-c8004374716c.jpeg%22%2C%22card%22%3A%22video%22%7D#nXnfR)


## xlsx 文件识别

通过 python cgi 上传完文件后，在使用 python 模块进行处理的时，发现通过 Firefox 上传的文件开始出现问题了：

```python
In [1]: import pandas as pd
In [2]: pd.read_excel("upload/upload.xlsx")
---------------------------------------------------------------------------
XLRDError                                 Traceback (most recent call last)
<ipython-input-8-9e85b7330a4e> in <module>
----> 1 pd.read_excel("upload/upload.xlsx")

......

XLRDError: Unsupported format, or corrupt file: Expected BOF record; found b'b\x14#e\xa9\x01W\x00'
```

于是，开始回去看 Galaxy 的源码，想要搞明白 Galaxy 对于 xlsx 文件上传到底是怎么进行识别的，终于在 `packages/data/galaxy/datatypes/binary.py` 中发现 Galaxy 正是使用了 python 的 `zipfile` 模块 ：

![](https://note.bioitee.com/yuque/0/2020/png/126032/1577953605841-778111ab-4e64-416d-81d8-5bae0a0163dd.png#align=left&display=inline&height=657&name=image.png&originHeight=657&originWidth=734&size=70032&status=done&style=none&width=734)

于是，我也开始使用 `zipfile` 来对先前 python cgi 上传的文件进行测试：

```python
In [9]: import zipfile

In [10]: zipfile.ZipFile("upload/upload.xlsx")
---------------------------------------------------------------------------
BadZipFile                                Traceback (most recent call last)
<ipython-input-10-3793f2363956> in <module>
----> 1 zipfile.ZipFile("upload/upload.xlsx")

......

BadZipFile: File is not a zip file
```

同样的操作，我在 Chrome 重复了一遍，但是却神奇的发现，不管是 `panda` 还是 `zipfile` 模块，竟然一切都表现正常！ 似乎，Firefox 的确有点不正常。



## 真正原因

针对这个问题，我最开始向 Galaxy Project 团队咨询过，但一直没有从根本解决掉这个问题，他们建议考虑非 xlsx 格式数据的工具开发。

![](https://note.bioitee.com/yuque/0/2019/jpeg/126032/1562808631934-a67f49e6-a306-4e12-acdf-e8cb3d176bd2.jpeg#align=left&display=inline&height=1395&originHeight=1395&originWidth=1080&status=done&style=none&width=1080)

直到前几天，突发奇想在 Firefox 社区中重新提起这个事情，一个热心网友的回复才让我意识到了问题的所在，也就是哈希——文件完整性校验。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1577955329338-cb12b774-b017-4eae-9376-95aff28914af.png#align=left&display=inline&height=489&name=image.png&originHeight=489&originWidth=690&size=41835&status=done&style=none&width=690)

我把文件上传前的 MD5 和文件上传后的 MD5 重新进行了计算比较，这才发现：

- 使用 Firefox 上传前后文件的 MD5 是一致，Python 却不能识别为有效的 zip 文件；
- 使用 Chrome 上传的文件前后 MD5 是不一致的，Python 却能正常识别为有效的 zip 文件。

很明显，我的原始 xlsx 文件是有问题的！！吐血中！！！但是在办公环境中，这个原始的 xlsx 文件不管是 Office 2016 还是 WPS 都能正常打开，正常编辑保存。唯一不同的是文件中多了一个锁的标志。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1577955786229-5e2c4184-4712-417f-8c9a-ba695b81d5d5.png#align=left&display=inline&height=108&name=image.png&originHeight=108&originWidth=632&size=10347&status=done&style=none&width=632)


其实，这就是企业企业办公文档 Office Excel 软件加密的一种效果。

1. 安装加密软件：安装加密服务端和管理端，客户端安装在被加密的电脑中;
1. 设置加密策略：打开管理端，选择文件加密—加密策略，勾选需要加密的 office Excel，然后保存策略，如下图;重启被加密员工电脑。

![](https://note.bioitee.com/yuque/0/2020/jpeg/126032/1577956204996-bf0c004b-0552-40e2-b5b9-5ca3184a0a3e.jpeg#align=left&display=inline&height=654&originHeight=654&originWidth=967&size=0&status=done&style=none&width=967)

**测试加密效果：**被加密电脑重启后，打开 word 文档，新建文档并编辑保存，保存后的文件会显示“加锁”标志，如下图示，显示已成功加密。 

![](https://note.bioitee.com/yuque/0/2020/jpeg/126032/1577956244695-d279b5a9-242e-4569-a62f-d6874457b079.jpeg#align=left&display=inline&height=91&originHeight=91&originWidth=91&size=0&status=done&style=none&width=91)

**实现效果**：员工编辑后的文档自动加密，加密后的文档未经许可，私自通过 QQ，电子邮件，U盘等任何方式传输到公司以外，都将无法打开使用；不改变编辑操作习惯，在企业内部相互流通编辑，不受影响。彻底从源头保障数据的安全性。此外还可实现如需外发文件，可通过申请解密流程授权解密后方可外发；同时还可对其他软件进行加密，比如办公软件，设计软件，工程软件，编程软件，研发软件等。

最后，把未加密的 xlsx 文件进行重新测试，一切问题迎刃而解。



## 总结一下

这是一个企业文档加密引发的填坑记录，从问题的发现，问题的思考，到解决的思路值得探讨记录一下。

- MD5，MD5，MD5，重要的事情要说三遍！！！
- 了解 Galaxy 文件上传和识别的一些思路，对特定格式文件的定制化处理有更深理解。
- 接触了 Python CGI 的一些简单应用，总体而言，挺好玩的。
- 多多交流，学会搜索，善用资源，事半功倍。

Chrome 为什么能绕开部分企业文档加密的枷锁，还原文件，这是一个有待后面学习的问题，mark 一下，同时期待老师们指点迷津。



## 参考资料

1. 易控王，[如何加密excel表格？企业excel表格加密方法](http://www.ekongsoft.com/a/xinwenzhongxin/wenjianjiami/41.html)，易控王新闻中心
1. istevenshen，[xlsx upload failed in usegalaxy.org](https://help.galaxyproject.org/t/xlsx-upload-failed-in-usegalaxy-org/1346)，Galaxy Help
1. 史提芬先生，[火狐上传 Excel bug](https://support.mozilla.org/zh-CN/kb/%E8%8E%B7%E5%BE%97%E7%A4%BE%E5%8C%BA%E6%94%AF%E6%8C%81/discuss/8254?utm_campaign=kbforums-post&utm_medium=email&utm_source=notification#post-18529)，Mozilla 技术支持
