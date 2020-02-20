---
title: CODING 静态网站
author: 沈维燕
type: post
topic/tags: ["others", "开发"]
date: 2019-12-04T06:16:59.000Z
category: 开发
published: true
---

在 cong.net 部署静态博客，跟 github 是不一样的，这里简单记录一下。以谢益辉的 [hugo-xmag](https://github.com/yihui/hugo-xmag) 博客主题为例。


## 1. 创建项目，进入代码浏览
![](https://note.bioitee.com/yuque/0/2019/png/126032/1575440647377-1c547ee2-4757-4339-9781-cb15263cdea6.png#align=left&display=inline&height=680&name=image.png&originHeight=680&originWidth=644&size=96913&status=done&style=none&width=644)



## 2. 在服务器中生成网站，并推送

```bash
shenweiyan@ecs-steven 14:25:22 /home/shenweiyan
$ mkdir shenweiyan.com
$ cd shenweiyan.com
$ mkdir themes
$ cd themes
$ git clone https://github.com/yihui/hugo-xmag.git
Cloning into 'hugo-xmag'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 547 (delta 2), reused 7 (delta 1), pack-reused 535
Receiving objects: 100% (547/547), 339.23 KiB | 53.00 KiB/s, done.
Resolving deltas: 100% (236/236), done.
$ cd /home/shenweiyan/shenweiyan.com
$ cp -r themes/hugo-xmag/exampleSite/* .
$ hugo   # 这一步会默认生成 public 静态博客目录

                   | EN
+------------------+----+
  Pages            | 41
  Paginator pages  |  3
  Non-page files   |  0
  Static files     |  2
  Processed images |  0
  Aliases          | 16
  Sitemaps         |  1
  Cleaned          |  0

Total in 129 ms
$ cd public
$ git init
Initialized empty Git repository in /home/shenweiyan/shenweiyan.com/public/.git/
$ git remote add origin https://username:passwd@e.coding.net/bioit/shenweiyan.com.git
$ git add --all
$ git commit -m "first commit"
$ git push origin master        # 完成 public 目录的所有文件推送
```


## 3. 开始构建静态网站

在导航栏的 “**构建与部署**” 中选择 “**静态网站**”，点击“**直接发布静态网站**”。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441226502-a901a77e-cbd1-4411-995e-4ee7797fdb5a.png#align=left&display=inline&height=680&name=image.png&originHeight=680&originWidth=1020&size=131262&status=done&style=none&width=1020)

填写网站名称，然后 "**保存**"。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441406356-59268eb4-5a9e-4730-8d3c-f47ef4e3139c.png#align=left&display=inline&height=680&name=image.png&originHeight=680&originWidth=1020&size=103511&status=done&style=none&width=1020)

"**保存**" 后，在部署页面点击 "**立即部署**"。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441668729-d77691e2-1b21-4f14-8754-6f1f5f09378d.png#align=left&display=inline&height=680&name=image.png&originHeight=680&originWidth=1020&size=113449&status=done&style=none&width=1020)

部署成功后，通过访问地址可以打开静态的网站。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441748045-41ec5429-1705-40eb-ad10-d5f3a49e4488.png#align=left&display=inline&height=680&name=image.png&originHeight=680&originWidth=1020&size=102889&status=done&style=none&width=1020)

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441846673-27751544-6bdc-4baf-a7d3-bcd7ff057312.png#align=left&display=inline&height=693&name=image.png&originHeight=693&originWidth=1020&size=204624&status=done&style=none&width=1020)



## 4. 绑定域名

在项目导航栏 "构建与部署" → "静态网站" 页面，点击右上角的 "设置" 按钮。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575441961453-8aea90a3-ea88-4e85-b2dc-4bbfca51f184.png#align=left&display=inline&height=693&name=image.png&originHeight=693&originWidth=1020&size=97335&status=done&style=none&width=1020)

在 "设置" 页面，填写需要绑定的域名，并选择强制 https 访问。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575442149333-7b6fed1e-c879-401c-8875-d40572a52f26.png#align=left&display=inline&height=693&name=image.png&originHeight=693&originWidth=1020&size=122835&status=done&style=none&width=1020)

在域名解析中添加一条 CNAME 记录。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575442326430-129f4240-1b0e-473b-b903-77f669736075.png#align=left&display=inline&height=467&name=image.png&originHeight=467&originWidth=657&size=24903&status=done&style=none&width=657)

域名解析 CNAME 添加后，等待几分钟，直至证书状态为“正常”。如果证书状态失败，检查你的 CNAME 记录，并再尝试多重新申请几次。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575442466874-5e6ef929-2cc2-430c-b65c-5065208fed36.png#align=left&display=inline&height=693&name=image.png&originHeight=693&originWidth=1020&size=131532&status=done&style=none&width=1020)

最后，通过域名访问，一切正常。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1575442724369-205d0da3-1284-4c36-8981-95eee6f24791.png#align=left&display=inline&height=693&name=image.png&originHeight=693&originWidth=1020&size=216060&status=done&style=none&width=1020)

