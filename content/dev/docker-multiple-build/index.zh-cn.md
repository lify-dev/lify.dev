---
title: 多阶段构建 docker 镜像
date: 2021-09-29
tags: [docker,docker bulid]
categories: [docker]
---

# 一、概述

spring boot 2.3 开始`spring-boot-maven-plugin` 插件默认开启了分层构建 `jar包` 或者 `war包`，本文主要根据官方文档记录一下如何使用分层jar构建镜像。

# 二、`spring-boot-maven-plugin` 分层构建 jar 包

配置 spring boot 项目继承自 `spring-boot-starter-parent`，设置 `<parent>` 如下：



```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.1</version>
</parent>
```

继承自 `spring-boot-starter-parent` 的项目有如下的默认配置：

- 默认编译jdk：JDK 1.8
- 默认编码：UTF-8
- 默认依赖项管理部分继承自 `spring-boot-dependencies POM`，管理公共依赖项的版本，这种依赖管理允许您在自己的 POM 中使用时省略那些依赖项的`<version>`标记
- 默认配置的 excution goal为：`repackage`

当在项目中引入 `spring-boot-maven-plugin` 插件默认就开启了分层构建 `jar包`，我们可以使用如下方式进行验证：

- 在项目的 pom.xml 文件中添加如下插件配置：



```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>getting-started</artifactId>
    <!-- ... -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

- 执行如下命令进行打包：



```shell
mvn clean package 
```

在 target 目录下面生成对应的 jar 包，解压 jar 包目录结构如下：

![img](https://git.poker/peacePiz/image-hosting/blob/master/20220729/image.6gx0i5ga1mc0.webp?raw=true)


jar 分别在 `BOOT-INF/classes` 和 `BOOT-INF/lib` 中包含应用程序的类和依赖项。 类似地，可执行 war 包含 `WEB-INF/classes` 中的应用程序类和 `WEB-INF/lib` 和`WEB-INF/lib-provided` 中的依赖项。 对于需要从 jar 或 war 的内容构建 docker 镜像的情况，需要分离这些目录以便将它们写入不同的层。

其中的 `BOOT-INF/layers.idx` 文件就是用来定义分层 jar 的构建顺序的，层的顺序很重要，因为它决定了在应用程序的一部分发生更改时，尽可能将发生更改的内容放在后面，应该先添加最不可能更改的内容，然后再添加最可能更改的层，默认顺序是：

- `dependencies`（用于存放不包含 snapshot 的依赖）
- `spring-boot-loader`（用于存放类加载器）
- `snapshot-dependencies`（用于存放包含 snapshot 的依赖）
- `application`（用于存放应用程序的类和资源）

![img](https://git.poker/peacePiz/image-hosting/blob/master/20220729/image.3mykkfk690w0.webp?raw=true)


此分层旨在根据应用程序构建之间更改的可能性来分离代码，项目的依赖不太可能在内部版本之间经常更改，因此将其放置在单独的层`dependencies`中，以允许工具重新使用缓存中的层，应用程序代码更可能在内部版本之间进行更改，因此将其隔离在单独的层`application`中。

使用下面的命令查看分层jar 的目录顺序：



```shell
java -Djarmode=layertools -jar probedemo-0.0.1-SNAPSHOT.jar list

dependencies
spring-boot-loader
snapshot-dependencies
application
```

使用下面的命令将jar包按照分层的目录结构进行解压以便创建分层镜像：



```undefined
java -Djarmode=layertools -jar probedemo-0.0.1-SNAPSHOT.jar extract
```

关于 `java -Djarmode=layertools -jar application.jar` 官方的说明如下：



```tsx
Usage:
  java -Djarmode=layertools -jar probedemo-0.0.1-SNAPSHOT.jar

Available commands:
  list     List layers from the jar that can be extracted
  extract  Extracts layers from the jar for image creation
  help     Help about any command
```

要禁用此特性，可以采用以下方式



```xml
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <layers>
                        <enabled>false</enabled>
                    </layers>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

# 三、多阶段构建 docker 镜像

知道了上面分层 jar 包的构建目录之后，我们可以使用多阶段来构建 docker 镜像，`Dockerfile` 的内容如下：



作者：惜鸟
链接：https://www.jianshu.com/p/8b8a5bd34547
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```dockerfile
# 指定基础镜像，这是多阶段构建的前期阶段
FROM openjdk:11-jre-slim as builder
# 指定工作目录，目录不存在会自动创建
WORKDIR /app
# 将生成的 jar 复制到容器镜像中
COPY target/*.jar application.jar
# 通过工具spring-boot-jarmode-layertools从application.jar中提取拆分后的构建结果
RUN java -Djarmode=layertools -jar application.jar extract

# 正式构建镜像
FROM openjdk:11-jre-slim
# 指定工作目录，目录不存在会自动创建
WORKDIR /app
# 前一阶段从jar中提取除了多个文件，这里分别执行COPY命令复制到镜像空间中，每次COPY都是一个layer
COPY --from=builder app/dependencies ./
COPY --from=builder app/spring-boot-loader ./
COPY --from=builder app/snapshot-dependencies ./
COPY --from=builder app/application ./
# 指定时区
ENV TZ="Asia/Shanghai"
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 定义一些环境变量，方便环境变量传参
ENV JVM_OPTS=""
ENV JAVA_OPTS=""
# 指定暴露的端口，起到说明的作用，不指定也会暴露对应端口
EXPOSE 8080
# 启动 jar 的命令
ENTRYPOINT ["sh","-c","java $JVM_OPTS $JAVA_OPTS org.springframework.boot.loader.JarLauncher"]
```

用下面的命令构建 docker 镜像：



```css
docker build -t demo:1.0.0 . 
```

使用如下命令查看镜像的分层信息：



```bash
docker history demo:1.0.0
```

![img](https://git.poker/peacePiz/image-hosting/blob/master/20220729/image.31ntyfetuoc0.webp?raw=true)

image.png

如上图，整个 jar 的内容，例如 class、依赖库、依赖资源等，分多次 COPY 到镜像空间中，所以以后如果只改了class，在更新镜像的时候，只需要下载 class 的 layer 即可（其他 layer 可以直接用之前缓存到本地的）。

![img](https://git.poker/peacePiz/image-hosting/blob/master/20220729/image.3p9312ex08o0.webp?raw=true)

