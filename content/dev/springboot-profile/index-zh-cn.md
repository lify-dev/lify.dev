---
title: SpringBoot激活profiles方式
date: 2021-04-28
tags: [springboot, profile]
categories: [spring]
---



多环境是最常见的`配置隔离`方式之一，可以根据不同的运行环境提供不同的配置信息来应对不同的业务场景，在`SpringBoot`内支持了多种配置隔离的方式，可以激活单个或者多个配置文件。



------

**激活的`profiles`要在项目内创建对应的配置文件，格式为`application-{profile}.yml`。**



### 命令行方式

`命令行方式`是一种外部配置的方式，在执行`java -jar`命令时可以通过`--spring.profiles.active=test`的方式进行激活指定的`profiles`列表。

使用方式如下所示：

```shell
java -jar order-service-v1.0.jar --spring.profiles.active=dev &> order-service.log &
```

### 系统变量方式

**Mac/Linux系统配置环境变量**

编辑环境变量配置文件`/etc/profile`，添加名为`SPRING_PROFILES_ACTIVE`的环境变量，如下所示：

```shell
# spring 环境激活
export SPRING_PROFILES_ACTIVE=dev

```

**Windows系统配置环境变量**

环境变量的配置方式请参考Java环境变量配置，新建一个名为`SPRING_PROFILES_ACTIVE`的系统环境变量，设置变量的值为`dev`即可。



> 系统变量的方式适用于系统下所部署统一环境的`SpringBoot`应用程序，如统一部署的都是`prod`环境的应用程序。



### Java系统属性方式

`Java系统属性方式`也是一种外部配置的方式，在执行`java -jar`命令时可以通过`-Dspring.profiles.active=test`的方式进行激活指定的`profiles`列表。

 使用方式如下所示：

```shell
java -Dspring.profiles.active=dev -jar order-service-v1.0.jar &> order-service.log &
```

> 注意：`-D`方式设置`Java系统属性`要在`-jar`**前**定义。



### 配置文件方式

`配置文件方式`是最常用的方式，不过灵活性不强，局限性比较大，不建议使用这种方式来激活配置文件

 我们只需要在`application.yml`配置文件添加配置即可，使用方式如下所示：

```yaml
spring:
  profiles:
    # 激活profiles
    active: dev
```

### 优先级

> 命令行方式 > Java系统属性方式 > 系统变量方式 > 配置文件方式



经过测试**`命令行方式`的优先级最高，而内部`配置文件方式`则是最低的**。



### 激活多个profile

如果需要激活多个`profile`可以使用逗号隔开，如：`--spring.profiles.active=dev,test`



------

每一个应用项目都会用到大量的配置文件或者外部配置中心，而配置信息的`激活`是必不可少的一步，**尤为重要**。

建议大家使用`系统环境变量`的方式来激活指定`profile`的配置，这种方式比较简单，系统全局都可以使用（`注意：系统全局代表着该系统下所运行的全部SpringBoot应用都会采用该配置`），当然也可以采用`优先级替换的规则`进行单独指定。

