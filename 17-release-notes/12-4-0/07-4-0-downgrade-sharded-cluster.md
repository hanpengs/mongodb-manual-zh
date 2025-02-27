# 将4.0分片集群降级到3.6

在尝试降级之前，请熟悉本文档的内容。

## 降级路径

一旦升级到4.0，如果您需要降级，我们建议您降级到最新的3.6补丁版本。

### 更改流注意事项

MongoDB 4.0引入了新的十六进制编码字符串更改流[恢复令牌：](https://www.mongodb.com/docs/upcoming/changeStreams/#std-label-change-stream-resume-token)

恢复令牌`_data`类型取决于MongoDB版本，在某些情况下取决于更改流打开/恢复时的功能兼容性版本（fcv）（即fcv值的更改不会影响已打开的更改流的恢复令牌）：

| MongoDB版本             | 功能兼容性版本 | 恢复令牌`_data`类型    |
| :---------------------- | :------------- | :--------------------- |
| MongoDB 4.0.7及更高版本 | "4.0"或"3.6"   | 六角编码字符串（`v1`） |
| MongoDB 4.0.6及更早版本 | "4.0"          | 六角编码字符串（`v0`） |
| MongoDB 4.0.6及更早版本 | "3.6"          | BinData                |
| MongoDB 3.6             | "3.6"          | BinData                |

* 从MongoDB 4.0.7或更高版本降级时，客户端无法使用从4.0.7+部署返回的恢复令牌。要恢复更改流，客户端需要使用4.0.7之前的升级恢复令牌（如果有）。否则，客户将需要启动新的更改流。
* 从MongoDB 4.0.6或更低版本降级时，客户端可以使用从4.0部署返回的BinData恢复令牌，但不能使用`v0`令牌。

## 创建备份

*可选但推荐。*创建数据库的备份。

## 先决条件

在降级二进制文件之前，您必须降级功能兼容性版本，并删除与3.6或更低版本[不兼容](https://www.mongodb.com/docs/upcoming/release-notes/4.0-compatibility/#std-label-4.0-compatibility-enabled)的任何4.0功能，如下所述。只有当`featureCompatibilityVersion`被设置为`"4.0"`时，这些步骤才是必要的。

### 1、降级功能兼容性版本

1. 将mongo shell连接到mongos实例。

2. 降级`featureCompatibilityVersion`为`"3.6"`

   ```
   db.adminCommand({setFeatureCompatibilityVersion: "3.6"})
   ```

   [`setFeatureCompatibilityVersion`](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)命令执行对内部系统集合的写入，并且是幂等的。如果由于任何原因命令未能成功完成，请在[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例上重试该命令。

为了确保分片集群的所有成员都反映更新的`featureCompatibilityVersion`，请连接到每个碎片副本集成员和每个配置服务器副本集成员，并检查`featureCompatibilityVersion`：

> 提示;
>
> 对于启用了访问控制的分片集群，要对碎片副本集成员运行以下命令，您必须以[碎片本地用户](https://www.mongodb.com/docs/upcoming/core/security-users/#std-label-shard-local-users)的身份连接到该成员[。](https://www.mongodb.com/docs/upcoming/core/security-users/#std-label-shard-local-users)

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

所有成员都应返回一个结果，其中包括：

```
"featureCompatibilityVersion" : { "version" : "3.6" }
```

如果任何成员返回包含`"4.0"`的`version`值或`targetVersion`字段`featureCompatibilityVersion`，请等待成员反映版本`"3.6"`后再继续。

有关返回的featureCompatibilityVersion值的详细信息，请参阅 [Get FeatureCompatibilityVersion.](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)

### 2、移除向后不兼容的持久功能

删除所有与4.0[不兼容](https://www.mongodb.com/docs/upcoming/release-notes/4.0-compatibility/#std-label-4.0-compatibility-enabled)的持久功能。例如，如果您定义了使用4.0查询功能（如[聚合转换运算符）](https://www.mongodb.com/docs/upcoming/release-notes/4.0/#std-label-4.0-agg-type-conversion)的任何视图定义、文档验证器和部分索引过滤器，则必须删除它们。

如果您的用户只有`SCRAM-SHA-256`凭据，您应该在降级之前为这些用户创建`SCRAM-SHA-1`凭据。要更新仅具有`SCRAM-SHA-256`凭据的用户，请运行[`db.updateUser()`](https://www.mongodb.com/docs/upcoming/reference/method/db.updateUser/#mongodb-method-db.updateUser)，其`mechanisms`仅为`SCRAM-SHA-1`，并将`pwd`设置为密码：

```
db.updateUser(
   "reportUser256",
   {
     mechanisms: [ "SCRAM-SHA-1" ],
     pwd: <newpwd>
   }
)
```

## 程序

### 降级分片集群

> 警告：
>
> 在继续下调程序之前，请确保所有成员，包括分片集群中的延迟副本集成员，反映先决条件更改。也就是说，在降级之前，检查`featureCompatibilityVersion`并删除每个节点的不兼容功能。

> 笔记：
>
> 如果您使用包含`SCRAM-SHA-256`[`authenticationMechanisms`](https://www.mongodb.com/docs/upcoming/reference/parameters/#mongodb-parameter-param.authenticationMechanisms)运行MongoDB 4.0，请使用3.6二进制文件重新启动时省略`SCRAM-SHA-256`。

### 1、下载最新的3.6二进制文件。

使用软件包管理器或手动下载，获取3.6系列的最新版本。如果使用软件包管理器，请为3.6二进制文件添加一个新存储库，然后执行实际的降级过程。

一旦升级到4.0，如果您需要降级，我们建议您降级到最新的3.6补丁版本。

### 2、停用平衡器。

将mongo shell连接到分片集群中的 [`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例，并运行[`sh.stopBalancer()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.stopBalancer/#mongodb-method-sh.stopBalancer)禁用均衡器：

```
sh.stopBalancer()
```

> 笔记：
>
> 如果迁移正在进行中，系统将在停止平衡器之前完成正在进行的迁移。您可以runsh[`sh.isBalancerRunning()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.isBalancerRunning/#mongodb-method-sh.isBalancerRunning)来检查平衡器的当前状态。

要验证平衡器是否已禁用，请运行[`sh.getBalancerState()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState)），如果平衡器被禁用，则返回false：

```
sh.getBalancerState()
```

有关禁用平衡器的更多信息，请参阅[禁用平衡器。](https://www.mongodb.com/docs/upcoming/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily)

### 3、降级`mongos`实例。

降级二进制文件并重新启动。

### 4、降级每个分片，一次一个

一次降级一个分片。如果碎片是复制集，则对于每个分片：

1. 一次降级副本集的[次要](https://www.mongodb.com/docs/upcoming/core/replica-set-members/#std-label-replica-set-secondary-members)成员：

   * 关闭[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)实例，并将4.0二进制文件替换为3.6二进制文件。

   * 使用`--shardsvr`和`--port`命令行选项启动3.6二进制文件。包括适合您部署的任何其他配置，例如`--bind_ip`。

     ```
     mongod --shardsvr --port <port> --dbpath <path> --bind_ip localhost,<hostname(s)|ip address(es)>
     ```

     或者，如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请将文件更新为包括[`sharding.clusterRole: shardsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和适合您部署的任何其他配置，例如[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，然后开始：

     ```
     sharding:
        clusterRole: shardsvr
     net:
        port: <port>
        bindIp: localhost,<hostname(s)|ip address(es)>
     storage:
        dbpath: <path>
     ```

   * 请等待成员恢复到SECONDARY状态，然后再降级下一个辅助成员。要检查成员的状态，可以在mongo shell中发出 [`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)。


   对每个辅助成员重复。

2. 逐步关闭副本集主副本。

   将一个mongo shell连接到主服务器，并使用 [`rs.stepDown()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.stepDown/#mongodb-method-rs.stepDown)将主服务器降级，强制选举一个新的主服务器：

   ```
   c
   ```

3. 当[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)显示初选已下级，而其他成员已处于`PRIMARY`状态时，请降级降级初选：

   * 关闭降级的主制文件，用3.6二进制文件替换为[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)二进制文件。

   * 使用`--shardsvr`和`--port`命令行选项启动3.6二进制文件。包括适合您部署的任何其他配置，例如`--bind_ip`。

     ```
     mongod --shardsvr --port <port> --dbpath <path>  --bind_ip localhost,<hostname(s)|ip address(es)>
     ```

     或者，如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请将文件更新为包括[`sharding.clusterRole: shardsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和适合您部署的任何其他配置，例如[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，并启动3.6二进制文件：

     ```
     sharding:
        clusterRole: shardsvr
     net:
        port: <port>
        bindIp: localhost,<hostname(s)|ip address(es)>
     storage:
        dbpath: <path>
     ```

### 5、降级配置服务器。

如果配置服务器是副本集：

1. 一次降级副本集的[次要](https://www.mongodb.com/docs/upcoming/core/replica-set-members/#std-label-replica-set-secondary-members)成员：

   * 关闭二级[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)实例，并将4.0二进制文件替换为3.6二进制文件。

   * 使用`--configsvr`和`--port`选项启动3.6二进制文件。包括适合您部署的任何其他配置，例如`--bind_ip`。

     ```
     mongod --configsvr --port <port> --dbpath <path>  --bind_ip localhost,<hostname(s)|ip address(es)>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请更新文件以指定[`sharding.clusterRole: configsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和适合您部署的任何其他配置，例如[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，并启动3.6二进制文件：

     ```
     sharding:
        clusterRole: configsvr
     net:
        port: <port>
        bindIp: localhost,<hostname(s)|ip address(es)>
     storage:
        dbpath: <path>
     ```

     包括适合您部署的任何其他配置。

   * 请等待成员恢复到SECONDARY状态，然后再降级下一个辅助成员。要检查成员的状态，请在mongo shell中发出 [`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)。

     对每个辅助成员重复.

2. 逐步关闭副本集主副本。

   * 将一个mongo shell连接到主服务器，并使用 [`rs.stepDown()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.stepDown/#mongodb-method-rs.stepDown)将主服务器降级，强制选举一个新的主服务器：

     ```
     rs.stepDown()
     ```

   * 当[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)显示主制已下级，另一个成员已处于`PRIMARY`状态时，请关闭降级主制，并将[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)二进制文件替换为3.6二进制文件。

   * 使用`--configsvr`和`--port`选项启动3.6二进制文件。包括适合您部署的任何其他配置，例如`--bind_ip`。

     ```
     mongod --configsvr --port <port> --dbpath <path> --bind_ip localhost,<hostname(s)|ip address(es)>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请更新文件以指定[`sharding.clusterRole: configsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和适合您部署的任何其他配置，例如[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，并启动3.6二进制文件：

     ```
     sharding:
        clusterRole: configsvr
     net:
        port: <port>
        bindIp: localhost,<hostname(s)|ip address(es)>
     storage:
        dbpath: <path>
     ```

### 6、重新启用平衡器。

一旦分片集群组件的降级完成，[重新启用平衡器。](https://www.mongodb.com/docs/upcoming/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable)

> 笔记:
>
> MongoDB 3.6部署可以使用从对4.0部署打开的更改流返回的BinData恢复令牌，但不能使用`v0`或`v1`十六进制编码的字符串恢复令牌。





 参见

原文 - [Downgrade 4.0 Sharded Cluster to 3.6]( https://docs.mongodb.com/manual/release-notes/4.0-downgrade-sharded-cluster/ )

