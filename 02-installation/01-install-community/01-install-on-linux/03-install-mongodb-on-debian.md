# 在 Debian 上安装 MongoDB 社区版



> 笔记
>
> **MongoDB atlas**
>
> [MongoDB atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是云中托管的 MongoDB 服务选项，无需安装开销，并提供免费套餐以供入门。

## 概述

使用本教程使用`apt`包管理器安装 MongoDB 7.0 Community Edition 。

### MongoDB 版本

本教程安装 MongoDB 7.0社区 版。要安装不同版本的 MongoDB Community ，请使用此页面左上角的版本下拉菜单选择该版本的文档。

## 注意事项

### 平台支持

[MongoDB 7.0 Community Edition 在x86_64](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-supported-platforms-x86_64)架构上支持以下 **64 位**Debian 版本 ：

- Debian 12“Bullseye”
- Debian 11“Buster”
- Debian 10“Stretch”

MongoDB 仅支持这些平台的 64 位版本。

有关详细信息，请参阅[平台支持](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-supported-platforms)。

### 制作说明

在生产环境中部署 MongoDB 之前，请考虑 [生产说明](https://www.mongodb.com/docs/manual/administration/production-notes/)文档，其中提供了生产 MongoDB 部署的性能注意事项和配置建议。

### 官方 MongoDB 包

要在您的Debian系统上安装 MongoDB Community ，这些说明将使用由 MongoDB Inc 维护和支持的官方`mongodb-org`包。官方`mongodb-org`包始终包含最新版本的 MongoDB，并且可以从其自己的专用 repo 获得。



> 重要
>
> Debian提供的`mongodb`包不是 MongoDB Inc.维护的，与官方`mongodb-org`**包**冲突。如果您已经在Debian系统 上安装了`mongodb`软件包，则**必须**先卸载`mongodb`软件包，然后才能继续执行这些说明。
>

看[MongoDB 社区版包](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-debian/#std-label-debian-package-content)官方软件包的完整列表

## 安装 MongoDB 社区版

按照以下步骤使用`apt`包管理器安装 MongoDB Community Edition 。



### 导入包管理系统使用的公钥。

从终端发出以下命令以从中导入 MongoDB 公共 GPG 密钥[https://www.mongodb.org/static/pgp/server-7.0.asc](https://www.mongodb.org/static/pgp/server-7.0.asc)

```
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
```

该操作应以`OK`.

但是，如果您收到指示`gnupg`未安装的错误消息，您可以：

1. 使用以下命令安装`gnupg`及其所需的库：

   ```
   sudo apt-get install gnupg
   ```

   

2. 安装后，重试导入密钥：

   ```
   wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
   ```




### 为 MongoDB创建一个`/etc/apt/sources.list.d/mongodb-org-7.0.list`文件。

使用适合您的 Debian 版本的命令创建列表文件：

Debian 10“Buster”

```
echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/7.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

Debian 9“Stretch”

```
echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/7.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

### 重新加载本地包数据库。

发出以下命令以重新加载本地包数据库：

```
sudo apt-get update
```



### 安装 MongoDB 包。

您可以安装最新稳定版本的 MongoDB 或特定版本的 MongoDB。

**安装最新版本的 MongoDB。**

要安装最新的稳定版本，请发出以下命令

```
sudo apt-get install -y mongodb-org
```

**安装特定版本的 MongoDB。**

````
sudo apt-get install -y mongodb-org=7.0 mongodb-org-database=7.0 mongodb-org-server=7.0 mongodb-mongosh=7.0 mongodb-org-mongos=7.0 mongodb-org-tools=7.0
````

如果您只安装`mongodb-org=7.0`而不包含组件包，则无论您指定哪个版本，都将安装每个 MongoDB 包的最新版本。

可选的。虽然您可以指定任何可用的 MongoDB 版本，但 `apt-get`会在更新版本可用时升级包。为防止意外升级，您可以将软件包固定在当前安装的版本上：

```
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-database hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-mongosh hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```



## 运行 MongoDB 社区版

**ulimit 注意事项**

大多数类 Unix 操作系统限制进程可能使用的系统资源。这些限制可能会对 MongoDB 操作产生负面影响，应该进行调整。有关为您的平台推荐的设置，请参阅[UNIX`ulimit`设置。](https://www.mongodb.com/docs/manual/reference/ulimit/)笔记`ulimit`从 MongoDB 4.4 开始，如果打开文件数的值小于 `64000`，则会生成启动错误 。

**目录**

默认情况下，MongoDB 实例存储：

* 它的数据文件在`/var/lib/mongodb`

* 它的日志文件在`/var/log/mongodb`

如果您通过包管理器安装，这些默认目录是在安装过程中创建的。

如果您通过下载 tarball 手动安装，则可以使用`mkdir -p <directory>`或`sudo mkdir -p <directory>`取决于将运行 MongoDB 的用户来创建目录。`mkdir`（有关和的信息，请参阅您的 linux 手册页`sudo`。）

默认情况下，MongoDB 使用`mongodb`用户帐户运行。如果更改运行 MongoDB 进程的用户，则还**必须**修改对`/var/lib/mongodb`和`/var/log/mongodb` 目录的权限，以授予该用户访问这些目录的权限。

要指定不同的日志文件目录和数据文件目录，[`systemLog.path`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-systemLog.path)请[`storage.dbPath`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.dbPath)编辑`/etc/mongod.conf`. 确保运行 MongoDB 的用户有权访问这些目录。

### 程序

按照以下步骤在您的系统上运行 MongoDB Community Edition。这些说明假定您使用的是官方软件包——而不是Debian提供 `mongodb-org` 的非`mongodb`官方软件包——并且使用的是默认设置。

**初始化系统**

要运行和管理您的[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)进程，您将使用操作系统的内置[初始化系统](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-init-system)。最新版本的 Linux 倾向于使用**systemd**（使用`systemctl`命令），而旧版本的 Linux 倾向于使用**System V init**（使用`service`命令）。

如果您不确定您的平台使用哪个 init 系统，请运行以下命令：

```
ps --no-headers -o comm 1
```



然后根据结果选择下面适当的选项卡：

- `systemd`- 选择下面的**systemd (systemctl)**选项卡。
- `init`- 选择下面的**System V Init（服务）**选项卡。

系统（系统控制）System V 初始化（服务）

### **系统 (systemctl)**选。

#### 1、启动 MongoDB。

[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)您可以通过发出以下命令来启动该过程：

```
sudo systemctl start mongod
```



如果您在启动时收到类似以下的错误 [`mongod`：](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)

```
Failed to start mongod.service: Unit mongod.service not found.
```

首先运行以下命令：

```
sudo systemctl daemon-reload
```



然后再次运行上面的启动命令。



#### 2、验证 MongoDB 是否已成功启动。

```
sudo systemctl status mongod
```



您可以选择通过发出以下命令确保 MongoDB 将在系统重启后启动：

```
sudo systemctl enable mongod
```



#### 3、停止 MongoDB。

[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)根据需要，您可以通过发出以下命令来停止该过程：

```
sudo systemctl stop mongod
```



#### 4、重新启动 MongoDB。

您可以[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)通过发出以下命令来重新启动该过程：

```
sudo systemctl restart mongod
```



您可以通过查看文件中的输出来跟踪错误或重要消息的过程状态`/var/log/mongodb/mongod.log`。



#### 5、开启使用 MongoDB。

开启一个[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)与 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod).在同一台主机上的会话 。你可以跑[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh) 没有任何命令行选项来连接到 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)在默认端口 27017 上运行的本地主机上。

```
mongosh
```

### System V Init（服务）

### 1、启动 MongoDB。

发出以下命令来启动[`mongod`：](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)

```
sudo service mongod start
```

#### 2、验证MongoDB是否启动成功

验证[`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)进程是否已成功启动：

```
sudo service mongod status
```

您还可以检查日志文件以了解进程的当前状态 ，默认情况下[`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)位于： `/var/log/mongodb/mongod.log`正在运行的 [`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)实例将指示它已准备好使用以下行进行连接：

```
[initandlisten] waiting for connections on port 27017
```

#### 3、停止 MongoDB

根据需要，您可以[`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)通过发出以下命令来停止该进程：

```
sudo service mongod stop
```

#### 4、重新启动 MongoDB

发出以下命令重新启动[`mongod`：](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)

```
sudo service mongod restart
```

#### 5、开始使用 MongoDB

开始一个[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)会话与 [`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod). 你可以运行[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh) 没有任何命令行选项来连接到 [`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)在本地主机上运行的默认端口 27017。

```
mongosh
```

有关使用[`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)连接的更多信息，例如连接到[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)在不同主机和/或端口上运行的实例，请参阅 [mongosh文档。](https://www.mongodb.com/docs/mongodb-shell/)

为了帮助您开始使用 MongoDB，MongoDB 提供了各种驱动程序版本的[入门指南](https://www.mongodb.com/docs/manual/tutorial/getting-started/#std-label-getting-started)。有关驱动程序文档，请参阅[开始使用 MongoDB 进行开发。](https://api.mongodb.com/)

## 卸载 MongoDB 社区版

要从系统中完全删除 MongoDB，您必须删除 MongoDB 应用程序本身、配置文件以及任何包含数据和日志的目录。以下部分将指导您完成必要的步骤。



> 警告
>
> 此过程将*完全*删除 MongoDB、其配置和*所有* 数据库。此过程不可逆，因此请确保在继续之前备份所有配置和数据。



### 关闭 MongoDB。

[`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)通过发出以下命令停止进程：

```
sudo service mongod stop
```



### 删除包。

删除您之前安装的所有 MongoDB 包。

```
sudo apt-get purge mongodb-org*
```



### 删除数据目录。

删除 MongoDB 数据库和日志文件。

```
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```



## 附加信息

### 默认绑定本地主机

默认情况下，MongoDB 启动时[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp)设置为 `127.0.0.1`，绑定到本地主机网络接口。这意味着`mongod`只能接受来自运行在同一台机器上的客户端的连接。远程客户端将无法连接到`mongod`，并且`mongod`将无法初始化[副本集](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-replica-set)，除非此值设置为有效的网络接口。

该值可以配置为：

- 在 MongoDB 配置文件中使用[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp), 或
- 通过命令行参数[`--bind_ip`](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--bind_ip)



>  警告
>
> 在绑定到非本地主机（例如可公开访问的）IP 地址之前，请确保您已保护集群免受未经授权的访问。有关安全建议的完整列表，请参阅 [安全清单](https://www.mongodb.com/docs/manual/administration/security-checklist/)。至少，考虑 [启用身份验证](https://www.mongodb.com/docs/manual/administration/security-checklist/#std-label-checklist-auth)和 [强化网络基础设施。](https://www.mongodb.com/docs/manual/core/security-hardening/)
>

有关配置的详细信息[`bindIp`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp)，请参阅 [IP 绑定。](https://www.mongodb.com/docs/manual/core/security-mongodb-configuration/)



### MongoDB 社区版包

MongoDB Community Edition 可从其自己的专用存储库获得，并包含以下官方支持的包：

| 包名字                 | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `mongodb-org`          | `metapackage`自动安装下面列出的组件包。                      |
| `mongodb-org-database` | `metapackage`自动安装下面列出的组件包。包裹名字描述`mongodb-org-server`包含[`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod)守护程序、关联的 init 脚本和[配置文件](https://www.mongodb.com/docs/v7.0/reference/configuration-options/#std-label-conf-file)( `/etc/mongod.conf`)。您可以使用初始化脚本来启动[`mongod`](https://www.mongodb.com/docs/v7.0/reference/program/mongod/#mongodb-binary-bin.mongod) 配置文件。有关详细信息，请参阅上面的“运行 MongoDB 社区版”部分。`mongodb-org-mongos`包含[`mongos`](https://www.mongodb.com/docs/v7.0/reference/program/mongos/#mongodb-binary-bin.mongos)守护进程。 |
| `mongodb-mongosh`      | 包含 MongoDB Shell ([`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)）。 |
| `mongodb-org-tools`    | `metapackage`自动安装以下组件包：包名字描述`mongodb-database-tools`包含以下 MongoDB 数据库工具：[`mongodump`](https://www.mongodb.com/docs/database-tools/mongodump/#mongodb-binary-bin.mongodump)[`mongorestore`](https://www.mongodb.com/docs/database-tools/mongorestore/#mongodb-binary-bin.mongorestore)[`bsondump`](https://www.mongodb.com/docs/database-tools/bsondump/#mongodb-binary-bin.bsondump)[`mongoimport`](https://www.mongodb.com/docs/database-tools/mongoimport/#mongodb-binary-bin.mongoimport)[`mongoexport`](https://www.mongodb.com/docs/database-tools/mongoexport/#mongodb-binary-bin.mongoexport)[`mongostat`](https://www.mongodb.com/docs/database-tools/mongostat/#mongodb-binary-bin.mongostat)[`mongotop`](https://www.mongodb.com/docs/database-tools/mongotop/#mongodb-binary-bin.mongotop)[`mongofiles`](https://www.mongodb.com/docs/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)`mongodb-org-database-tools-extra`包含[`install_compass`](https://www.mongodb.com/docs/v7.0/reference/program/install_compass/#std-label-install-compass)脚本 |



原文链接 -https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/ 

译者：韩鹏帅

