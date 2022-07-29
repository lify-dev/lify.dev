---
title: dokcer安装并开启SQL Server代理
date: 2021-05-31 10:00:00
tags: [sqlserver, agent]
categories: [sqlserver, cdc]
---



## docker 安装命令

```shell
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=!_6t8iu%Io' -e 'MSSQL_AGENT_ENABLED=true' -p 1433:1433 -d --name sql1 -h sql1 mcr.microsoft.com/mssql/server:2017-CU24-ubuntu-16.04
```

**说明**

| 参数                                               | 说明                                                         |
| :------------------------------------------------- | ------------------------------------------------------------ |
| **-e "ACCEPT_EULA=Y"**                             | 将 ACCEPT_EULA 变量设置为任意值，以确认接受最终用户许可协议。 SQL Server 映像的必需设置。 |
| -e 'SA_PASSWORD=<YourStrong@Passw0rd>'             | 指定至少包含 8 个字符且符合 [SQL Server 密码要求](https://docs.microsoft.com/zh-cn/sql/relational-databases/security/password-policy?view=sql-server-ver15)的强密码。 SQL Server 映像的必需设置。 |
| -e 'MSSQL_AGENT_ENABLED=true'                      | 开启 sql server 代理                                         |
| **-p 1433:1433**                                   | 将主机环境中的 TCP 端口（第一个值）映射到容器中的 TCP 端口（第二个值）。 在此示例中，SQL Server 侦听容器中的 TCP 1433，并对主机上的端口 1433 公开。 |
| **--name sql1**                                    | 为容器指定一个自定义名称，而不是使用随机生成的名称。 如果运行多个容器，则无法重复使用相同的名称。 |
| -h sql1                                            | 用于显式设置容器主机名，如果不指定它，则默认为容器 ID，该 ID 是随机生成的系统 GUID。 |
| **mcr.microsoft.com/mssql/2017-CU24-ubuntu-16.04** | SQL Server 2017 Ubuntu Linux 容器映像。                      |

参考：[快速入门：使用 Docker 运行 SQL Server 容器映像](https://docs.microsoft.com/zh-cn/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&amp;pivots=cs1-bash)



## 开启SQL Server代理

想要让 sql server 自动备份数据库和启用数据变更捕捉(CDC)就必须要启动 sql server 代理服务



### windows启动SQL Server Agent服务

在cmd命令提示符下，输入下列命令

```bash
net start SQLSERVERAGENT
```

或

1. 在 **“对象资源管理器”** 中，单击加号以展开要管理 SQL Server 代理服务的服务器。
2. 右键单击“SQL Server 代理”，然后选择“启动”、“停止”或“重启”。
3. 在“用户帐户控制”对话框中，单击“是”。
4. 系统提示是否要执行该操作时，请单击 **“是”**

![image](https://cdn.jsdelivr.net/gh/peacePiz/image-hosting@master/20210419/image.2zcfku5fp9q0.png)



设置开机启动

![image](https://cdn.jsdelivr.net/gh/peacePiz/image-hosting@master/20210419/image.2oc7otcpsuw0.png)



将“启动类型”设置为 自动，那么在电脑重启或开机时， SQL Server代理（MSSQLSERVER）就会自动启动了。

如果“启动类型”为手动，那么在电脑重启时，这个服务就不会自动启动了，就需要手动启动了

参考：[sqlserver SQL Agent服务无法启动]([sqlserver SQL Agent服务无法启动如何破_Hehuyi_In的博客-CSDN博客](https://blog.csdn.net/Hehuyi_In/article/details/103228691))

参考：[启动、停止或暂停服务 - SQL Server Agent | Microsoft Docs](https://docs.microsoft.com/zh-cn/sql/ssms/agent/start-stop-or-pause-the-sql-server-agent-service?view=sql-server-ver15)

### linux 下开启SQL Server代理

对于SQL Server 2019和SQL Server 2017 CU4及更高版本，您只需要启用SQL Server代理即可。您不需要安装单独的软件包。

要启用SQL Server代理，请按照以下步骤操作。

```shell
sudo /opt/mssql/bin/mssql-conf set sqlagent.enabled true  

sudo systemctl restart mssql-server
```



对于SQL Server 2017 CU3和更早版本，必须安装SQL Server代理程序包。

### Install on RHEL

使用以下步骤在Red Hat Enterprise Linux上安装mssql-server-agent。

```bash
sudo yum install mssql-server-agent
sudo systemctl restart mssql-server
```



如果已经安装了mssql-server-agent，则可以使用以下命令将其更新到最新版本：

```bash
sudo yum check-update
sudo yum update mssql-server-agent
sudo systemctl restart mssql-server
```

### Install on Ubuntu

使用以下步骤在Ubuntu上安装mssql-server-agent。

```bash
sudo apt-get update 
sudo apt-get install mssql-server-agent
sudo systemctl restart mssql-server
```

### Install on SLES

使用以下步骤在SUSE Linux Enterprise Server上安装mssql-server-agent。

```bash
sudo zypper install mssql-server-agent
sudo systemctl restart mssql-server
```

如果已经安装了mssql-server-agent，则可以使用以下命令将其更新到最新版本：

```bash
sudo zypper refresh
sudo zypper update mssql-server-agent
sudo systemctl restart mssql-server
```

