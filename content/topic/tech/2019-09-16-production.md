---
title: Galaxy 生信平台（二）：生产环境部署
author: 沈维燕
type: post
topic/tags: ["others", "tech"]
date: 2019-09-16T01:43:11.000Z
category: tech
published: true
---

在上一篇文章中，我们介绍了适合单个用户进行使用和开发的 Galaxy 在线平台，今天我们来聊一下在为多用户生产环境设置 Galaxy 时，我们应采取的一些可以让 Galaxy 获得最佳性能的额外步骤。


## 原因

默认情况下的 Galaxy 会启用下面的服务：

- 使用 [SQLite](https://www.sqlite.org/)（无服务器数据库），因此您无需为快速或基本开发运行/配置额外的数据库服务器。 但是，虽然SQLite 支持并发访问（[concurrent access](https://sqlite.org/lockingv3.html)），但它不支持多个并发写入，这会降低系统吞吐量。
- 使用基于 Python 编写的内置 HTTP 服务器。此服务器执行的大部分工作可以移动到 [nginx](https://docs.galaxyproject.org/en/master/admin/nginx.html) 或 [Apache](https://docs.galaxyproject.org/en/master/admin/apache.html)，这将提高性能。
- 在本地运行所有工具。如果将 Galaxy 迁移到群集（[cluster](https://docs.galaxyproject.org/en/master/admin/cluster.html)）将大大增加其服务和计算的能力。
- 在单个进程中运行，这是 CPython 中的性能问题。

Galaxy 附带以上的默认配置，以确保在进行基本开发时可以实现最简单，最防错的配置。正如您很快就会看到的，目标是尽可能多地从 Galaxy 流程中删除工作，因为这样做会大大加快其剩余职责的执行速度。这是由于Python全局解释器锁（GIL），详细说明可以参考 [Galaxy 的](https://docs.galaxyproject.org/en/master/admin/production.html#advanced-configuration)[高级配置部分文档](https://docs.galaxyproject.org/en/master/admin/production.html#advanced-configuration)。



## 基础

实现 Galaxy 可扩展性的基础就是， **我们需要一个干净的环境！** 以下许多说明是适用于任何生产应用程序的最佳实践。

- 创建一个名为 galaxy 的 **非 ROOT 用户** 。当您要授予或限制对数据的访问权限时，作为现有用户运行将导致问题。
- 从一个全新版本的 Galaxy 开始，不要尝试从以前使用过的版本中进行切换以用于开发。下载并安装在 galaxy 用户主目录中。
- 应该由 galaxy 用户运行 Galaxy 托管的一系列系统服务（如 Apache，邮件服务器，数据库服务器等）。发布的的 `contrib/` 目录中提供了初始化脚本，OS X 启动定义和 Solaris SMF manifests。您还可以在 `run.sh` 中使用 `--daemon` 和 `--stop-daemon` 参数来手动启动和停止 Galaxy 的服务，但仍然需要分离进行操作。当作为守护进程运行时，服务器的输出日志将写入 `galaxy.log` 而不是终端，除非使用 `--log-file` 参数进行自定义说明。
- 为 Galaxy 提供自己的数据库用户和数据库，以防止 Galaxy 的架构与数据库中的其他表冲突。此外，限制 Galaxy 的数据库用户，以便它只能访问自己的数据库。
- 确保 Galaxy 使用干净的 Python 解释器。 `$PYTHONPATH` 或解释器的 `site-packages/` 目录中的冲突可能会导致其他的一些问题。对于 Galaxy 自身的框架依赖项 Galaxy 可以自行进行管理，因此您不必担心。最简单的方法是使用 `virtualenv` ：
```bash
nate@weyerbacher% pip install virtualenv
nate@weyerbacher% virtualenv --no-site-packages galaxy_env
nate@weyerbacher% . ./galaxy_env/bin/activate
nate@weyerbacher% cd galaxy-dist
nate@weyerbacher% sh run.sh
```

- Galaxy 可以放在一个集群/网络文件系统中（它已经过 NFS 和 GPFS 测试），如果你要在集群上运行它，可以参考：[https://docs.galaxyproject.org/en/master/admin/cluster.html](https://docs.galaxyproject.org/en/master/admin/cluster.html)。



### 基础配置

安装 Galaxy 的步骤大多遵循常规说明的步骤（[regular instructions](http://getgalaxy.org/)）。不同之处在于，在执行上述基础工作之后，您应该初始化配置文件（ `cp config/galaxy.yml.sample config/galaxy.yml` ）并在启动服务器之前按照下面的描述对其进行修改。如果在服务器运行时对此配置文件进行任何更改，则必须重新启动服务器才能使更改生效。



### 禁用开发人员设置

配置示例文件 `config/galaxy.yml` 中设置了两个选项，这些选项不应在生产服务器上启用。您应该将两者都设置为 `false` ：

- `debug:false` - 禁用在内存中加载整个响应的中间件，以便在页面中显示调试信息。如果保持启用状态，代理服务器可能会超时等待响应，或者如果 Galaxy 进程正在为大文件提供服务，则可能会耗尽内存。
- `use_interactive:false` - 禁止通过 Web 显示和实时调试回溯。保持启用状态将暴露您的配置（数据库密码，id_secret 等）。
- 禁用 `filter-with:gzip` 。启用 gzip 过滤器将导致 UI 失败，因为一旦 `debug` 设置为 `False` ，模板就会流式传输。您仍然可以（并且鼓励）在代理服务器中启用 `gzip` 。

在部署期间，您可能会遇到失败作业的问题。默认情况下，Galaxy 会删除与作业执行相关的文件。您可以指示 Galaxy 保留失败作业的文件： `cleanup_job:onsuccess` 



### 切换到数据库服务器

最重要的建议是切换到实际的数据库服务器。默认情况下，Galaxy 将使用 [SQLite](http://www.sqlite.org/)，它是一个无服务器的简单文件数据库引擎。由于它是无服务器的，因此所有数据库处理都在 Galaxy 进程中进行。这有两个缺点：它占用前面提到的GIL（意味着该进程不能自由地执行其他任务），并且它不如专用数据库服务器那么高效。还有其他缺点：当多个用户的负载增加时，事务锁的风险也会增加。锁定将导致（除其他外）超时和作业错误。如果从 SQLite 开始，然后意识到需要数据库服务器，则需要迁移数据库或重新开始。Galaxy 没有提供从 SQLite 迁移数据的内部方法，虽然 Web 上提供了免费的转换工具，但这个过程并非易事。

