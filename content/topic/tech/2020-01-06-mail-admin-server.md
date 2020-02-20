---
title: Galaxy 生信平台（四）：邮件与管理员配置
type: post
topic/tags: ["others", "tech"]
date: 2020-01-06T07:25:19.000Z
category: tech
published: true
---

我在《QQ 邮箱设置自定义域名邮箱》中给大家展示了，如何通过自己申请的域名+ QQ 邮箱打造一个高端大气的个人专业邮箱。今天来了一下如何把自定义的 QQ 域名邮箱应用到 Galaxy 生信分析平台中。
## 1. Galaxy 邮箱与管理员

Galaxy Project 是一个基于云计算背景下的开源生物信息分析项目，而 Galaxy 平台则是这个项目的集大成者。作为一个用于生产环境中，可以满足多用户使用的在线平台，Galaxy 的邮箱服务可谓是至关重要，尤其对于普通用户常见的登录限制、注册激活、密码重置与修改找回、页面错误报告等琐碎问题的处理上，它可以大大减轻系统管理员的工作量。

Galaxy 收发邮件的核心代码主要位于以下两个程序，其核心模块则是 `util.send_mail`，一个基于 `email` 和 `smtplib` 库进行封装的函数，具体源码，感兴趣的可以去研究一下。

- `lib/galaxy/webapps/galaxy/controllers/user.py`
- `lib/galaxy/util/__init__.py`

所以，在设置 Galaxy 邮箱服务前，我们先来了解一下如何在 Python 中基于 `email` 和 `smtplib` 库进行 QQ 邮件的发送。

## 2. 基于 Python 的邮件收发

首先我们要知道用 python 代理登录 qq 邮箱发邮件，是需要更改自己 qq 邮箱设置的。在这里大家需要做两件事情：邮箱开启 SMTP 功能 、获得授权码，步骤如下。

首先，在打开的 QQ 邮箱中,进入**"设置"。**

**![](https://note.bioitee.com/yuque/0/2020/png/126032/1578292378668-8948f35d-2b3e-4afd-b944-c127f2f50a57.png#align=left&display=inline&height=195&name=image.png&originHeight=195&originWidth=821&size=24711&status=done&style=none&width=821)**
第二，在邮箱设置界面,打开**“帐户”**。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578292437426-d3419a60-6375-4c0d-9862-660f3c69da2c.png#align=left&display=inline&height=349&name=image.png&originHeight=349&originWidth=972&size=39686&status=done&style=none&width=972)

第三，往下找到 POP3/IMAP/...服务,点击 POP3/SMTP 所在行，右侧的**“开启”**。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578292851877-236be38e-d100-4d3f-9d8f-b13fed0163a2.png#align=left&display=inline&height=291&name=image.png&originHeight=291&originWidth=828&size=31265&status=done&style=none&width=828)

如果是第一次开启 POP3/SMTP 这时系统会要求通过密保手机发送一条特别内容的短信，待手机上短信发送完成后，点击**“我已发送”**。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578292906922-11102102-5128-40f9-ace9-f365be6daeb5.png#align=left&display=inline&height=405&name=image.png&originHeight=405&originWidth=825&size=47215&status=done&style=none&width=825)

这样就可以得到 POP3/SMTP 服务的授权码了。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578293068784-4958f676-2517-4dbb-b4c4-c66c20f45fc7.png#align=left&display=inline&height=433&name=image.png&originHeight=433&originWidth=888&size=56881&status=done&style=none&width=888)

回到上页，可以看到 POP3/SMTP 服务已经处于开启状态。同样的，如果忘记了授权码，可以点击温馨提示中"生成授权码"的链接重新再次生成。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578293143597-dff55415-2710-4e96-8765-f2afbd3341f3.png#align=left&display=inline&height=331&name=image.png&originHeight=331&originWidth=881&size=40621&status=done&style=none&width=881)


SMTP 服务和授权码都有了，接下来我们要基于 python3 的 **email **和 **smtplib **库实现一个简单的发邮件功能。

```python
from email.mime.text import MIMEText
from email.header import Header
from smtplib import SMTP_SSL

#qq 邮箱 smtp 服务器
host_server = 'smtp.qq.com'
#sender_login 为发件人的域名邮箱(或者 QQ 号码)
sender_login = 'shenweiyan@bioitee.com'
#pwd 为 qq 域名邮箱的授权码
pwd = 'r***h***b'
#发件人的邮箱
sender_mail = 'shenweiyan@bioitee.com'
#收件人邮箱
receiver_mail = 'ishenweiyan@gmail.com'
#邮件的正文内容
mail_content = '你好，我是沈维燕，现在在进行一项用 python 登录 qq 邮箱发邮件的测试'
#邮件标题
mail_title = '沈维燕的邮件'

#ssl登录
smtp = SMTP_SSL(host_server)
#set_debuglevel() 是用来调试的。参数值为 1 表示开启调试模式，参数值为 0 关闭调试模式
smtp.set_debuglevel(1)
smtp.ehlo(host_server)
smtp.login(sender_login, pwd)

msg = MIMEText(mail_content, "plain", 'utf-8')
msg["Subject"] = Header(mail_title, 'utf-8')
msg["From"] = sender_mail
msg["To"] = receiver_mail
smtp.sendmail(sender_mail, receiver, msg.as_string())
smtp.quit()
```

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578294621933-2d5f314d-f0eb-4529-a606-fa4895911c9d.png#align=left&display=inline&height=325&name=image.png&originHeight=325&originWidth=684&size=26483&status=done&style=none&width=684)


## 3. 配置 Galaxy 邮件服务

有了上一步 python 发送邮件的经验，接下来我们只需要修改一下 Galaxy 的主配置文件 config/galaxy.yml 即可：

```yaml
galaxy:
    smtp_server: 'smtp.qq.com'
    smtp_username: 'shenweiyan@bioitee.com'
    smtp_password: 'h**z***j'
    smtp_ssl: true
    error_email_to: 'shenweiyan@bioitee.com'
    email_from: 'shenweiyan@bioitee.com'
```

配置完成后，重启 Galaxy 服务即可。

## 4. 管理员与强制登录

如果你只想把你个人部署的 Galaxy 提供给特定的人使用，例如同一个实验室的其他小伙伴，你可以把 Galaxy 的注册功能关闭，并设置只能通过管理员创建新账号。
```yaml
galaxy:
    # 设置 Galaxy 管理员账号，如有多个管理员，请用逗号分隔
    admin_users: 'shenweiyan@bioitee.com'
    # 强制所有用户均需要注册登录才能使用
    require_login: false
    # 开启用户激活功能(如果设置为"false"，则帐户自注册以来一直处于活动状态)
    user_activation_on: true
    # 设置激活有效时长(小时)
    activation_grace_period: 3
    # 允许未注册的用户创建新帐户(否则，必须由管理员创建)
    allow_user_creation: false
    # 允许管理员删除已经注册的账号
    allow_user_deletion: true
    # 允许管理员模拟其他用户登录，进行问题追踪
    allow_user_impersonation: true
```
 

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578300288534-b06fda6b-f5fe-442b-a90e-c5e957f27a87.png#align=left&display=inline&height=689&name=image.png&originHeight=689&originWidth=1006&size=107517&status=done&style=none&width=1006)

## 5. 一些测试

配置好了 Galaxy 邮箱服务，我们来测试一下注册激活验证和重置密码。

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578303073127-101150dd-d70f-4d1f-a844-a58806a773c4.png#align=left&display=inline&height=397&name=image.png&originHeight=397&originWidth=1006&size=68346&status=done&style=none&width=1006)

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578303049973-dafe5e6b-8b45-4dd9-a517-4cd1dd82c978.png#align=left&display=inline&height=394&name=image.png&originHeight=394&originWidth=977&size=47011&status=done&style=none&width=977)

![](https://note.bioitee.com/yuque/0/2020/png/126032/1578303303678-1b26f86f-7743-4ee9-a75a-014a19c139bb.png#align=left&display=inline&height=347&name=image.png&originHeight=347&originWidth=994&size=37096&status=done&style=none&width=994)

到这里，Galaxy 的邮件服务和管理员设置就完成了，如果想要更改 Galaxy 默认的邮件内容，可以仔细去研究一下它的源码。最后总结一句，就是，QQ 域名邮箱的本质还是 QQ 邮箱，因此，它收发邮箱的服务器设置依然是 qq.com。
