---
title: 阿里云 ECS 更换操作系统
author: 沈维燕
type: post
topic/tags: ["others", "运维"]
date: 2019-09-23T07:04:36.000Z
category: 运维
published: true
---

参考：[更换系统盘（公共镜像）](https://help.aliyun.com/document_detail/50134.html?spm=a2c4g.11186623.2.8.6a6447a0NgnzZM#concept-n4k-x3j-ydb)

![](https://note.bioitee.com/yuque/0/2019/png/126032/1569222859005-68ee5efd-ae82-4c8b-884d-c9c8cb337c76.png#align=left&display=inline&height=438&name=image.png&originHeight=438&originWidth=1125&size=72056&status=done&width=1125)

<a name="h2-url-3"></a>
## 操作步骤

1. 登录[ECS管理控制台](https://ecs.console.aliyun.com/)。
1. 在左侧导航栏，选择****实例与镜像** > **实例****。
1. 在顶部状态栏处，选择地域。
1. 找到需要更换系统盘的实例，在**操作**列中，单击****更多** > **实例状态** > **停止****，并按页面提示停止实例。

**说明** 对于按量付费的VPC类型实例而言，停机不收费模式下，更换系统盘后可能无法成功启动实例。建议您停止实例时关闭**停机不收费**。具体操作，请参见[按量付费实例停机不收费](https://help.aliyun.com/document_detail/63353.html#concept-js1-1fd-5db)。


1. 实例停止后，在**操作**列中，单击****更多** > **磁盘和镜像** > **更换系统盘****。
1. 在弹出的对话框里，仔细阅读更换系统盘注意事项后，单击**确定，更换系统盘**。
1. 在**更换系统盘**页面上，配置以下参数。
  - **镜像类型**：选择**公共镜像**，并选择需要的镜像版本。

**说明** 如果您需要使用其他镜像，请参见[更换系统盘（非公共镜像）](https://help.aliyun.com/document_detail/25448.html#concept-vbb-ckj-ydb)。
  - **系统盘**：不能更换系统盘的云盘类型，但是您能根据业务需求和新镜像的需求扩容系统盘，最大容量为500 GiB。扩容时能设置的最小容量与系统盘当前容量有关，如下表所示。
  - **安全设置。**
  - 确认**配置费用**：目前中国站所有公共镜像都不收费，这里的配置费用指系统盘的费用。系统盘价格，请参见[云产品价格页](https://www.aliyun.com/price/product#/ecs/detail)。
  - 确认无误后，单击**确定更换**。


<a name="h2-url-4"></a>
## 执行结果

登录ECS控制台监控系统状态，完成操作系统更换大概需要10分钟。完成后，实例会自动启动。


<a name="h2-url-5"></a>
## 后续步骤

更换系统盘后，您可能需要做以下操作：

- （可选）自动快照策略与磁盘ID绑定。更换了新的系统盘后，旧磁盘上应用的自动快照策略自动失效。您需要对新系统盘设置自动快照策略。具体操作，请参见[为新的系统盘设置自动快照策略](https://help.aliyun.com/document_detail/25457.html#concept-nyv-k3l-xdb)。
- 如果更换前后都是Linux系统，而且，实例上原来挂载了数据盘并设置了开机自动挂载分区。更换系统盘后，原来系统盘中的数据盘分区挂载信息丢失。您必须在新系统盘的/etc/fstab文件写入新分区信息，并挂载分区，不需要对数据盘格式化并分区。操作步骤如下，具体的操作命令，请参见[Linux格式化数据盘](https://help.aliyun.com/document_detail/25426.html#concept-jl1-qzd-wdb)。
  1. （建议）备份etc/fstab。
  1. 向etc/fstab写入新分区信息。
  1. 查看etc/fstab中的新分区信息。
  1. 运行`mount`命令挂载分区。
  1. 查看文件系统空间和使用情况：运行命令`df -h`。
- 挂载操作完成后，不需要重启实例即可开始使用新的数据盘。


