## 使用 .tgz Tarball 在 Debian 上安装 MongoDB 社区



> 笔记:
>
> **MongoDB atlas**
>
> [MongoDB atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是云中托管的 MongoDB 服务选项，无需安装开销，并提供免费套餐以供入门。

### 概述

使用本教程使用下载的`.tgz`tarball 在 Debian Linux 上手动安装 MongoDB 7.0 Community Edition。

#### MongoDB 版本

本教程安装 MongoDB 7.0 Community Edition。要安装不同版本的 MongoDB Community ，请使用此页面左上角的版本下拉菜单选择该版本的文档。

#### 安装方法

虽然 MongoDB 可以通过下载的`.tgz` tarball 手动安装，如本文档所述，但建议尽可能使用系统上的 `apt`包管理器来安装 MongoDB。使用包管理器会自动安装所有需要的依赖项，提供一个示例`mongod.conf`文件来帮助您入门，并简化未来的升级和维护任务。

➤有关说明，请参阅[使用 apt 包管理器安装 MongoDB](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-debian/) 。

## 注意事项

### MongoDB shell，`mongosh`

使用`.tgz`包安装服务器时，需要按照[mongosh安装说明](https://www.mongodb.com/docs/mongodb-shell/install/)下载并安装[mongosh](https://www.mongodb.com/docs/mongodb-shell/)。

### 平台支持

[MongoDB 7.0 Community Edition 在x86_64](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-supported-platforms-x86_64)架构上支持以下 **64 位**Debian 版本 ：

- Debian 12“Bullseye”
- Debian 11“Buster”
- Debian 10“Stretch”

MongoDB 仅支持这些平台的 64 位版本。

有关详细信息，请参阅[平台支持](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-supported-platforms)。

### 制作说明

在生产环境中部署 MongoDB 之前，请考虑 [生产说明](https://www.mongodb.com/docs/manual/administration/production-notes/)文档，其中提供了生产 MongoDB 部署的性能注意事项和配置建议。

### 安装 MongoDB 社区版

#### 先决条件

使用以下命令安装 MongoDB社区`.tgz`tarball所需的依赖项： 

**Debian11(Bullseye) **

```
sudo apt-get install libcurl4 openssl liblzma5
```

**Debian10(Buster) **

```
sudo apt-get install libcurl4 openssl liblzma5
```

### 过程

按照以下步骤从 .tgz 手动安装 MongoDB Community Edition。

#### 下载压缩包。

安装所需的先决条件包后，从以下链接下载 MongoDB社区tarball:

➤ [MongoDB 下载中心](https://www.mongodb.com/try/download/community?tck=docs_server)

1. 在**版本**下拉列表中，选择要下载的 MongoDB 版本。
2. 在**平台**下拉列表中，选择您的操作系统版本和体系结构。
3. 在**包**下拉列表中，选择**tgz**。
4. 单击**下载**。

#### 从下载的存档中提取文件。

例如，从系统 shell 中，您可以使用以下`tar`命令提取：

```
tar -zxvf mongodb-linux-*-7.0.tgz
```

#### `PATH`确保二进制文件位于环境变量中列出的目录中。

MongoDB 二进制文件位于`bin/`tarball 的目录中。您可以：

- 将二进制文件复制到变量中列出的目录中`PATH` ，例如`/usr/local/bin`（根据需要更新 `/path/to/the/:/`安装目录）

  ```
  sudo cp /path/to/the/mongodb-directory/bin/* /usr/local/bin/
  ```

- 从变量中列出的目录创建指向二进制文件的符号链接`PATH`，例如`/usr/local/bin`（根据需要更新 `/path/to/the/mongodb-directory/`安装目录）：

  ```
  sudo ln -s  /path/to/the/mongodb-directory/bin/* /usr/local/bin/
  ```


#### 安装 MongoDB Shell ( `mongosh`)。

[安装](https://www.mongodb.com/docs/mongodb-shell/install/) `mongosh`然后使用 MongoDB Shell 连接到您安装的服务。

`mongosh`从您需要 的版本下载包[MongoDB 下载中心](https://www.mongodb.com/try/download/community?tck=docs_server)并解压缩包。

### 运行 MongoDB 社区版

ulimit 注意事项

大多数类 Unix 操作系统限制进程可能使用的系统资源。这些限制可能会对 MongoDB 操作产生负面影响，应该进行调整。有关为您的平台推荐的设置，请参阅[UNIX`ulimit`设置。](https://www.mongodb.com/docs/manual/reference/ulimit/)

> 笔记
>
> `ulimit`从 MongoDB 4.4 开始，如果打开文件数的值小于 ，则会生成启动错误 `64000`。

目录

默认情况下，MongoDB 实例存储：

* 它的数据文件在`/var/lib/mongodb`

* 它的日志文件在`/var/log/mongodb`

如果您通过包管理器安装，这些默认目录是在安装过程中创建的。

如果您通过下载 tarball 手动安装，则可以使用`mkdir -p <directory>`或`sudo mkdir -p <directory>`取决于将运行 MongoDB 的用户来创建目录。`mkdir`（有关和的信息，请参阅您的 linux 手册页`sudo`。）

默认情况下，MongoDB 使用`mongodb`用户帐户运行。如果更改运行 MongoDB 进程的用户，则还**必须**修改对`/var/lib/mongodb`和`/var/log/mongodb` 目录的权限，以授予该用户访问这些目录的权限。

要指定不同的日志文件目录和数据文件目录，[`systemLog.path`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-systemLog.path)请[`storage.dbPath`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.dbPath)编辑`/etc/mongod.conf`. 确保运行 MongoDB 的用户有权访问这些目录。

### 程序

按照以下步骤运行 MongoDB社区版。这些说明假定您使用的是默认设置。

#### 创建数据和日志目录。

创建 MongoDB 实例存储其数据的目录。例如：

```
sudo mkdir -p /var/lib/mongo
```

创建 MongoDB 实例存储其日志的目录。例如：

```
sudo mkdir -p /var/log/mongodb
```

启动MongoDB进程的用户必须对这些目录有读写权限。例如，如果您打算自己运行 MongoDB：

```
sudo chown `whoami` /var/lib/mongo     # Or substitute another user
sudo chown `whoami` /var/log/mongodb   # Or substitute another user
```

#### 运行 MongoDB。

要运行 MongoDB，请在系统提示符下运行该[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)进程。

```shell
mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --fork
```

有关命令行选项[`--dbpath`](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--dbpath)和的详细信息[`--logpath`](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--logpath)，请参阅 [选项。](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-label-mongod-options)

#### 验证 MongoDB 是否已成功启动。

通过检查日志文件中以下行的进程输出来验证 MongoDB 是否已成功启动`/var/log/mongodb/mongod.log`：

```
[initandlisten] waiting for connections on port 27017
```

您可能会在过程输出中看到非严重警告。只要看到上面显示的日志行，就可以在初始评估 MongoDB 期间安全地忽略这些警告。

#### 开始使用 MongoDB。

开始一个[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)与 .在同一台主机上的会话 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)。你可以跑[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh) 没有任何命令行选项来连接到 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)在默认端口 27017 上运行的本地主机上。

```
mongosh
```

有关使用连接的更多信息[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)，例如连接到[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)在不同主机和/或端口上运行的实例，请参阅 [mongosh文档。](https://www.mongodb.com/docs/mongodb-shell/)

为了帮助您开始使用 MongoDB，MongoDB 提供了各种驱动程序版本的[入门指南](https://www.mongodb.com/docs/manual/tutorial/getting-started/#std-label-getting-started)。有关驱动程序文档，请参阅[开始使用 MongoDB 进行开发。](https://api.mongodb.com/)

## 附加信息

### 默认绑定本地主机

默认情况下，MongoDB 启动时[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp)设置为 `127.0.0.1`，绑定到本地主机网络接口。这意味着`mongod`只能接受来自运行在同一台机器上的客户端的连接。远程客户端将无法连接到`mongod`，并且`mongod`将无法初始化[副本集](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-replica-set)，除非此值设置为有效的网络接口。

该值可以配置为：

- 在 MongoDB 配置文件中使用[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp), 或
- 通过命令行参数[`--bind_ip`](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--bind_ip)

> 警告
>
> 在绑定到非本地主机（例如可公开访问的）IP 地址之前，请确保您已保护集群免受未经授权的访问。有关安全建议的完整列表，请参阅 [安全清单](https://www.mongodb.com/docs/manual/administration/security-checklist/)。至少，考虑 [启用身份验证](https://www.mongodb.com/docs/manual/administration/security-checklist/#std-label-checklist-auth)和 [增强网络基础设施。](https://www.mongodb.com/docs/manual/core/security-hardening/)

有关配置的详细信息[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp)，请参阅 [IP 绑定。](https://www.mongodb.com/docs/manual/core/security-mongodb-configuration/)



原文链接 -https://www.mongodb.com/docs/v7.0/tutorial/install-mongodb-on-debian-tarball/

译者：韩鹏帅

