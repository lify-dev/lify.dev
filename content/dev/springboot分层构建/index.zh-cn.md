---
title: springboot分层构建
date: 2021-05-26
tags: [springboot, docker bulid]
categories: [spring,springboot]
---


### 构建容器镜像

Spring Boot 应用程序可以 [使用 Dockerfiles](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.container-images.building.dockerfiles)进行容器化, 或者可以 [使用 Cloud Native Buildpacks 去创建在任何地方都可以运行的与docker兼容的容器镜像](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.container-images.building.buildpacks).

####  Dockerfiles

虽然可以在Dockerfile中仅包含几行代码的情况下将Spring Boot fat jar 转换为 docker 镜像, 我们将使用分层功能来创建和优化docker镜像. 当你创建一个包含分层索引文件时,  `spring-boot-jarmode-layertools` jar包将作为依赖加到你的jar包中.将此jar包放到classpath上, 你可以在特殊模式下启动应用程序，该模式允许引导代码运行与应用程序完全不同的内容, 例如，提取层的内容。

> `layertools`模式不能与包含启动脚本的[完全可执行的Spring Boot存档一起使用](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.installing)。当使用`layertools`构建jar包时，请禁用启动脚本的配置。

使用以下命令查看 `layertools` 可以使用的模式:

```shell
$ java -Djarmode=layertools -jar my-app.jar
```

会输出以下内容:

```
Usage:
  java -Djarmode=layertools -jar my-app.jar

Available commands:
  list     List layers from the jar that can be extracted
  extract  Extracts layers from the jar for image creation
  help     Help about any command
```

`extract` 命令可以很轻松的将应用程序拆分为多个层添加到dockerfile中。

使用 `jarmode` 的Dockerfile 示例

```dockerfile
FROM adoptopenjdk:11-jre-hotspot as builder
WORKDIR application
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM adoptopenjdk:11-jre-hotspot
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

假设上述Dockerfile位于当前目录中，则可以使用`docker build .`或可选地指定应用程序jar的路径来构建docker镜像，如以下示例所示：

```shell
$ docker build --build-arg JAR_FILE=path/to/myapp.jar .
```

这是一个多阶段的dockerfile. 构建器阶段提

的目录。每个`COPY`命令都与jarmode提取的层有关。

当然，无需使用jarmode即可编写Dockerfile。你也可以使用`unzip`和`mv`的某种组合将内容移至正确的层，但是jarmode简化了这一过程。