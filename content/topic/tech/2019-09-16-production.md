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

出于这个原因，Galaxy 还支持 [PostgreSQL](https://www.postgresql.org/) 和 [MySQL](https://dev.mysql.com/)。 [PostgreSQL](https://www.postgresql.org/) 是首选，因为我们发现它在我们的数据库抽象层 [SQLAlchemy](https://www.sqlalchemy.org/) 中运行得更好。

要使用外部数据库，您需要自行去安装和设置。该过程超出了本文档的范围，但通常很简单。例如，在 Debian 和基于 Redhat 的 Linux 发行版上，可能已经安装了一个。如果没有，它应该是 `apt-get install` 或 `yum install` 。在 macOS 上，[PostgreSQL](https://www.postgresql.org/) 网站上提供了安装程序。CentOS 下安装 PostgreSQL，请参考：《[Linux 下 PostgreSQL 源码编译安装](https://www.yuque.com/shenweiyan/bioinit/linux-postgresql-install)》。

安装后，创建一个新用户和该新用户所属的新数据库。无需进一步设置，因为 Galaxy 会管理自己的架构。如果您使用 UNIX 套接字(UNIX socket)将应用程序连接到数据库（这是标准情况，如果 Galaxy 和数据库在同一系统上），运行 Galaxy 进程时您将要为数据库用户命名为与您所在的系统用户相同的名称。

要配置 Galaxy，请在 Galaxy 的配置文件 `config/galaxy.yml` 中设置 `database_connection` 。数据库 URL 相关的语法在 [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/latest/core/engines.html) 中有详细的描述。

这里是两个带有用户名和密码的示例数据库 URL：
```yaml
postgresql://username:password@localhost/mydatabase
mysql://username:password@localhost/mydatabase
```

值得注意的是，某些平台（例如，Debian/Ubuntu）将数据库套接字存储在数据库引擎默认目录以外的目录中。如果您在与 Galaxy 服务器相同的主机上连接到数据库服务器并且套接字位于非标准位置，则需要使用这些自定义参数（这些是 Debian/Ubuntu 的默认值，必要时可根据个人的安装进行更改）：
```yaml
postgresql:///mydatabase?host=/var/run/postgresql
mysql:///mydatabase?unix_socket=/var/run/mysqld/mysqld.sock
```

有关数据库 URL 可用选项的更多提示，参考：[SQLAlchemy documentation](https://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls)。

如果您使用 [MySQ](https://dev.mysql.com/)L 并遇到 "MySQL server has gone away" 的错误，请注意 `config/galaxy.ym` l中的 `database_engine_option_pool_recycle` 选项。如果这不能解决您的问题，请参阅 [Galaxy Development 邮件列表](https://galaxyproject.org/mailing-lists/)中的[这篇文章](http://gmod.827538.n3.nabble.com/template/NamlServlet.jtp?macro=print_post&node=2354941)。

如果您使用的是 [MySQL](https://dev.mysql.com/)，请确保数据库输出为 UTF-8，否则您可能会遇到 Python TypeErrors。

如果您使用 [MySQL](https://dev.mysql.com/) 和 [MyISAM 表引擎](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html)，当 Galaxy 处于多进程配置时，工作流程步骤可能出现无序执行（[get executed out of order](http://dev.list.galaxyproject.org/Job-execution-order-mixed-up-tt4662488.html)）并失败。请改用 [InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html) 引擎或切换到 [PostgreSQL](https://www.postgresql.org/)。



### 使用代理服务器

Galaxy 包含了一个独立的 Web 服务器，可以直接向客户端提供所有内容。但是，某些任务（例如提供静态内容）可以负载到更有效地处理这些任务的专用服务器。代理服务器还允许您使用代理支持的任何方法（例如，Kerberos 或 LDAP）在外部对用户进行身份验证，指示浏览器缓存内容以及压缩出站数据。此外，Galaxy 的内置 Web 服务器不支持字节范围请求（许多外部显示应用程序所需），但可以将此功能负载到代理服务器。除了释放 GIL 之外，压缩和缓存还可以减少页面加载的时间。

下载和上载数据也可以移动到代理服务器。详细介绍可以参考：[Make the proxy handle uploads and downloads](https://docs.galaxyproject.org/en/master/admin/production.html#make-the-proxy-handle-uploads-and-downloads)。

Galaxy 官方提供了提供了基于 Apache 和 Nginx 以下的配置示例：

- [Apache](https://docs.galaxyproject.org/en/master/admin/apache.html)，具有成熟代理功能，且广泛部署的通用 Web 服务器。
- [Nginx](https://docs.galaxyproject.org/en/master/admin/nginx.html)，一种高性能的反向代理，公共 Galaxy 网站目前在使用中。



### 使用计算集群

Galaxy 是一个运行命令行工具的框架，如果配置正确，可以在计算群集上运行这些工具。如果没有群集，您将受限于服务器中的核心数量，减去运行 Galaxy 本身所需的核心数量。Galaxy 目前支持 **TORQUE PBS**，**PBS Pro**，**Platform LSF** 和 **Sun Grid Engine** 集群，不需要专用或特殊的集群配置。只要在该平台上可以使用运行这些工具所需的依赖，Galaxy 的工具甚至可以在异构集群节点（不同的操作系统）上运行。

使用群集还可以为您带来附带好处：在本地运行工具时，它们是 Galaxy 服务器的子进程。这意味着如果重新启动服务器，则会失去与这些作业的联系，并且必须重新启动它们。但是在群集上，如果 Galaxy 服务器重新启动，作业将继续运行并完成。一旦 Galaxy 工作正常启动，它将恢复跟踪并完成工作，就好像什么也没发生过一样。

设置群集后，Galaxy 的配置并不困难。详细的集群相关配置信息，可以参考 [Galaxy cluster page](https://docs.galaxyproject.org/en/master/admin/cluster.html)。



### 清理数据集

从历史记录或库中删除数据集时，只会将其标记为已删除但未实际删除，因为以后可以取消删除。为了释放磁盘空间，可以运行一组脚本（例如， `cron/` 目录相关脚本）以删除本地策略指定的数据文件。有关说明，请参阅“[清除历史记录和数据集](https://galaxyproject.org/admin/config/performance/purge-histories-and-datasets/)”页面。



### 日志切换

要使用 logrotate 切换 Galaxy 日志文件，请将名为 `galaxy` 的新文件添加到 `/etc/logrotate.d/` 目录中，例如：
```yaml
PATH_TO_GALAXY_LOG_FILES {
  weekly
  rotate 8
  copytruncate
  compress
  missingok
  notifempty
}
```



### 本地数据

要开始设置本地数据，请参考：[Data Integration](https://galaxyproject.org/admin/data-integration/)。

- 所有本地参考基因组必须包含在 `builds.txt` 文件中。
- 某些工具（例如，Extract Genomic DNA）要求您缓存（可能很大）本地 .2bit 格式数据。
- 其他工具（例如，Bowtie2）要求您缓存 .fasta 数据和特定工具的索引文件。
- `galaxy_dist/tool-data/` 目录包含一系列，用于描述元数据以及本地数据和索引路径的 location 示例（ `<data_label>.loc` ）文件。
- 从 Galaxy [Tool Shed](https://galaxyproject.org/toolshed/) 中安装的工具包，也可能包含 location 文件。
- location 文件中的注释，说明了该工具所需要的数据格式，设置前可以参考。
- [Data Integration](https://galaxyproject.org/admin/data-integration/) 文档解释了如何获取、创建或重新编写许多常见数据和索引。有关更多详细信息，请参见各个Tool Shed 存储库的文档。

![](https://note.bioitee.com/yuque/0/2019/png/126032/1568612403485-e662cfe2-2030-4742-a9de-ded2dac688a5.png#align=left&display=inline&height=268&name=image.png&originHeight=268&originWidth=962&size=32957&status=done&width=962)



### 通过 FTP 启用上传

由于快速提升的测序技术，文件大小已经变得非常大，并且通过浏览器上传这些文件并不总是可行的。值得庆幸的是，一个简单的解决方案是允许 Galaxy 用户通过 FTP 上传它们并将这些文件导入其历史记录中。详细的的配置请参考 [File Upload via FTP page](https://docs.galaxyproject.org/en/master/admin/special_topics/ftp.html) 中的说明。



## 高级配置



### 负载平衡和 Web 应用程序扩展

如前所述，由于 Python 全局解释器锁（GIL，[Global Interpreter Lock](https://docs.python.org/c-api/init.html#thread-state-and-the-global-interpreter-lock)），从 Galaxy 进程中 unloading work 非常重要。GIL 是 Python 确保线程安全的方式，它通过一次只允许一个线程控制执行来实现这一点。这意味着无论服务器中的核心数量如何，Galaxy 都只能使用一个核心。但是，有一个解决方案：运行多个 Galaxy 进程并使用代理服务器来平衡所有这些进程。实际上，Galaxy 分为作业处理程序和 Web 服务器进程。作业处理程序不直接通过 Web 为任何用户请求提供服务。相反，他们会在数据库中查看新作业，并在找到它们后，处理它们的准备，监视，运行和完成。同样，Web 服务器进程可以自由处理服务内容和文件到 Web 客户端。

有关如何配置扩展和负载平衡的完整详细信息，请参阅 [the scaling](https://docs.galaxyproject.org/en/master/admin/scaling.html)[ ](https://docs.galaxyproject.org/en/master/admin/scaling.html)文档。



### 调整数据库

PostgreSQL 可以比 Galaxy 更有效地存储结果，从而减少 Galaxy 的内存占用。进行查询时，结果将保留在 Postgres 服务器上，Galaxy 可以只需要检索所需的行。要启用此功能，请在 Galaxy 配置中设置：

`database_engine_option_server_side_cursors:true` 。

如果服务器记录有关数据库连接池大小的错误，则可能需要增加默认的最小和最大池连接数，即 5 和 10。这些配置文件选项为 `database_engine_option_pool_size` 和 `database_engine_option_max_overflow` 。

最后，如果您使用的是 Galaxy <= release_2014.06.02，我们建议您指示 Galaxy 为每个线程使用一个数据库连接，以避免连接开销和过度使用。这可以通过 `database_engine_option_strategy:threadlocal` 启用。



### 使代理处理上传和下载

默认情况下，Galaxy 从代理服务器接收文件流上传，然后将此文件写入磁盘。同样，它将文件作为流发送到代理服务器。这占据了 Galaxy 流程中的 GIL，并且会降低该流程中其他操作的响应能力。要解决此问题，您可以将代理服务器配置为直接提供下载，仅涉及授权用户有权读取数据集的任务。如果使用 nginx 作为代理，您可以将其配置为接收上传的文件并将其写入磁盘本身，只有在完成后才通知 Galaxy。有关如何配置这些内容的所有详细信息都可以在 [Apache](https://docs.galaxyproject.org/en/master/admin/apache.html) 和[ nginx ](https://docs.galaxyproject.org/en/master/admin/nginx.html)代理指令页面上找到。



## 总结

以上就是 Galaxy 生信平台基于生产环境下部署的一些建议，其中涉及了很多数据库、系统，以及服务器、集群相关的知识。从个人而言，这是一个极好的实战学习项目；从公司而言，也是从传统命令行生信分析向可视化界面处理的一个很好开源案例，值得借鉴与参考。

目前，关于 Galaxy Project 在国内的深入研究相关介绍与文档不是很多，如果您有任何的想法，欢迎留言交流。
