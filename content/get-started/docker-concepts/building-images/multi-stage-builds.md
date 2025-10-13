---
title: 多阶段构建
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍多阶段构建的目的及其带来的收益。
summary: |
  通过将构建环境与最终运行环境分离，你可以显著减少镜像体积并降低攻击面。
  本指南将带你掌握多阶段构建的用法，构建精简高效的 Docker 镜像，
  以便在生产环境中降低开销并提升部署效率。
weight: 5
aliases: 
 - /guides/docker-concepts/building-images/multi-stage-builds/
---

{{< youtube-embed vR185cjwxZ8 >}}

## 概念解析

在传统构建中，所有构建指令会在同一个构建容器中按顺序执行：下载依赖、编译代码、打包应用……这些步骤产生的所有层最终都会进入你的镜像。此方式可行，但会导致镜像臃肿、携带不必要的内容，也会增加安全风险。多阶段构建正是为了解决这一问题。

多阶段构建允许你在一个 Dockerfile 中定义多个“阶段”，每个阶段都有明确的职责。可以理解为：在多个不同的环境中分别完成构建流程的不同部分，并在需要时进行协作。通过把“构建环境”和“运行环境”分离，你能够显著降低最终镜像体积与攻击面，尤其适用于依赖较多、构建体积较大的应用。

多阶段构建适用于各类应用：

- 对解释型语言（如 JavaScript、Ruby、Python）：可在一个阶段完成构建与代码压缩/产物整理，再将产物拷贝到更小的运行时镜像中，优化用于部署的镜像。
- 对编译型语言（如 C、Go、Rust）：可在一个阶段完成编译，再把编译好的二进制拷贝到最终运行镜像中，无需把编译器一并打包进最终镜像。

下面是一个多阶段构建的简化结构（伪代码）。注意包含多个 `FROM`，以及 `AS <stage-name>` 为阶段命名；第二个阶段中的 `COPY` 使用了 `--from` 从前一阶段拷贝产物。

```dockerfile
# 阶段 1：构建环境
FROM builder-image AS build-stage 
# 安装构建工具（例如 Maven、Gradle）
# 拷贝源代码
# 执行构建命令（例如编译、打包）

# 阶段 2：运行环境
FROM runtime-image AS final-stage  
#  从构建阶段拷贝应用产物（例如 JAR 文件）
COPY --from=build-stage /path/in/build/stage /path/to/place/in/final/stage
# 定义运行时配置（例如 CMD、ENTRYPOINT） 
```

该 Dockerfile 使用了两个阶段：

- 构建阶段：基于包含构建工具的镜像，用于编译应用；在此阶段安装工具、拷贝源码并执行构建命令。
- 最终阶段：基于较小的运行时镜像，只用于运行应用；从构建阶段拷贝编译产物（例如 JAR 文件），并用 `CMD` 或 `ENTRYPOINT` 定义应用的启动方式。

## 动手试试

本实践将以一个基于 Spring Boot 和 Maven 的“Hello World”示例，演示如何使用多阶段构建创建精简高效的 Java 应用镜像。

1. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。

2. 打开这个[预初始化项目](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.4.0-M3&packaging=jar&jvmVersion=21&groupId=com.example&artifactId=spring-boot-docker&name=spring-boot-docker&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.spring-boot-docker&dependencies=web) 以生成一个 ZIP。如下所示：

    ![Spring Initializr 截图，选择了 Java 21、Spring Web 与 Spring Boot 3.4.0](images/multi-stage-builds-spring-initializer.webp?border=true)

    [Spring Initializr](https://start.spring.io/) 是一个 Spring 项目脚手架生成器，提供可扩展的 API 来生成基于 JVM 的项目，内置 Java、Kotlin、Groovy 等语言的常见模板。

    选择 **Generate** 生成并下载该项目的 ZIP。

    在本示例中，我们选择了 Maven 作为构建工具，添加 Spring Web 依赖，并使用 Java 21。

3. 浏览项目结构。解压后，你将看到类似如下的目录：

    ```plaintext
    spring-boot-docker
    ├── HELP.md
    ├── mvnw
    ├── mvnw.cmd
    ├── pom.xml
    └── src
        ├── main
        │   ├── java
        │   │   └── com
        │   │       └── example
        │   │           └── spring_boot_docker
        │   │               └── SpringBootDockerApplication.java
        │   └── resources
        │       ├── application.properties
        │       ├── static
        │       └── templates
        └── test
            └── java
                └── com
                    └── example
                        └── spring_boot_docker
                            └── SpringBootDockerApplicationTests.java
    
    15 directories, 7 files
    ```

   `src/main/java` 用于存放源码，`src/test/java` 存放测试代码，`pom.xml` 是项目的 POM（Project Object Model）。

   `pom.xml` 是 Maven 项目的核心配置文件，包含构建定制所需的大部分信息。它可能看起来很庞大，但你无需立刻掌握所有细节即可高效使用。

4. 创建一个返回 “Hello World!” 的 RESTful Web 服务。

    在 `src/main/java/com/example/spring_boot_docker/` 目录下，将 `SpringBootDockerApplication.java` 修改为如下内容：

    ```java
    package com.example.spring_boot_docker;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;


    @RestController
    @SpringBootApplication
    public class SpringBootDockerApplication {

        @RequestMapping("/")
            public String home() {
            return "Hello World";
        }

    	public static void main(String[] args) {
    		SpringApplication.run(SpringBootDockerApplication.class, args);
    	}

    }
    ```

    该文件声明了 `com.example.spring_boot_docker` 包并导入必要的 Spring 组件，创建了一个简单的 Spring Boot Web 应用：当访问首页时返回 "Hello World"。

### 创建 Dockerfile

现在已有项目代码，可以创建 `Dockerfile`。

 1. 在项目根目录（包含 src、pom.xml 等）创建名为 `Dockerfile` 的文件。

 2. 在 `Dockerfile` 中指定基础镜像：

     ```dockerfile
     FROM eclipse-temurin:21.0.8_9-jdk-jammy
     ```

  3. 使用 `WORKDIR` 指令设置工作目录，后续命令将在该目录下运行，同时文件也会复制到镜像内的此路径：

     ```dockerfile
     WORKDIR /app
     ```

  4. 将 Maven 包装器脚本与项目的 `pom.xml` 拷贝到容器内的 `/app`：

     ```dockerfile
     COPY .mvn/ .mvn
     COPY mvnw pom.xml ./
     ```

  5. 在容器内执行命令，使用 Maven Wrapper 预拉取依赖（不构建最终 JAR），以便加速后续构建：

     ```dockerfile
     RUN ./mvnw dependency:go-offline
     ```

  6. 将宿主机项目的 `src` 目录拷贝到容器内 `/app`：

     ```dockerfile
     COPY src ./src
     ```

  7. 设置容器启动时的默认命令，运行 Maven Wrapper 执行 `spring-boot:run` 来启动应用：

     ```dockerfile
     CMD ["./mvnw", "spring-boot:run"]
     ```

    至此，你应得到如下 Dockerfile：

    ```dockerfile 
    FROM eclipse-temurin:21.0.8_9-jdk-jammy
    WORKDIR /app
    COPY .mvn/ .mvn
    COPY mvnw pom.xml ./
    RUN ./mvnw dependency:go-offline
    COPY src ./src
    CMD ["./mvnw", "spring-boot:run"]
    ```

### 构建容器镜像

 1. 执行以下命令构建镜像：

    ```console
    $ docker build -t spring-helloworld .
    ```

 2. 使用 `docker images` 查看镜像大小：

    ```console
    $ docker images
    ```

    例如：

    ```console
    REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
    spring-helloworld   latest    ff708d5ee194   3 minutes ago    880MB
    ```

    可见镜像约为 880MB，包含完整的 JDK、Maven 工具链等。在生产环境中，这些内容不必存在于最终镜像中。

### 运行 Spring Boot 应用

1. 既然镜像已构建完毕，运行容器：

    ```console
    $ docker run -p 8080:8080 spring-helloworld
    ```

    你将在容器日志中看到类似如下输出：

    ```plaintext
    [INFO] --- spring-boot:3.3.4:run (default-cli) @ spring-boot-docker ---
    [INFO] Attaching agents: []
    
         .   ____          _            __ _ _
        /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
       ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
        \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
         '  |____| .__|_| |_|_| |_|\__, | / / / /
        =========|_|==============|___/=/_/_/_/
    
        :: Spring Boot ::                (v3.3.4)
    
    2024-09-29T23:54:07.157Z  INFO 159 --- [spring-boot-docker] [           main]
    c.e.s.SpringBootDockerApplication        : Starting SpringBootDockerApplication using Java
    21.0.2 with PID 159 (/app/target/classes started by root in /app)
     ….
     ```

2. 通过浏览器访问 [http://localhost:8080](http://localhost:8080)，或使用 curl：

    ```console
    $ curl localhost:8080
    Hello World
    ```

### 使用多阶段构建

1. 考虑如下 Dockerfile：

    ```dockerfile
    FROM eclipse-temurin:21.0.8_9-jdk-jammy AS builder
    WORKDIR /opt/app
    COPY .mvn/ .mvn
    COPY mvnw pom.xml ./
    RUN ./mvnw dependency:go-offline
    COPY ./src ./src
    RUN ./mvnw clean install

    FROM eclipse-temurin:21.0.8_9-jre-jammy AS final
    WORKDIR /opt/app
    EXPOSE 8080
    COPY --from=builder /opt/app/target/*.jar /opt/app/*.jar
    ENTRYPOINT ["java", "-jar", "/opt/app/*.jar"]
    ```

    可以看到该 Dockerfile 被拆分为两个阶段：

    - 第一阶段与前文类似，提供 JDK 构建环境用于构建应用，此阶段命名为 `builder`。
    - 第二阶段是新的 `final` 阶段，使用更精简的 `eclipse-temurin:21.0.2_13-jre-jammy` 运行时镜像，仅包含运行应用所需的 JRE。该镜像足以运行已编译的 JAR。

   > 生产环境建议使用 `jlink` 创建自定义的精简运行时。Eclipse Temurin 为各版本提供了 JRE 镜像，但借助 `jlink` 可以只包含应用所需的 Java 模块，从而进一步缩小体积并提升安全性。详见[此页面](https://hub.docker.com/_/eclipse-temurin)。

   借助多阶段构建，Docker 会使用一个基础镜像完成编译、打包与单元测试，再使用另一个更小的镜像承载应用的运行时。最终镜像不再包含开发或调试工具，因此体积更小、安全性更高。

2. 重新构建镜像，获得可直接用于生产的构建产物：

    ```console
    $ docker build -t spring-helloworld-builder .
    ```

    该命令会基于当前目录中的 `Dockerfile` 构建镜像，并以最终阶段生成名为 `spring-helloworld-builder` 的镜像。

    > [!NOTE]
    >
    > 在多阶段 Dockerfile 中，最后一个阶段（此处为 `final`）是构建的默认目标。如果未显式通过 `--target` 指定阶段，`docker build` 会默认构建最后一个阶段。若只想构建含 JDK 的构建阶段，可使用：`docker build -t spring-helloworld-builder --target builder .`

3. 使用 `docker images` 对比镜像大小：

    ```console
    $ docker images
    ```

    可能的输出如下：

    ```console
    spring-helloworld-builder latest    c5c76cb815c0   24 minutes ago      428MB
    spring-helloworld         latest    ff708d5ee194   About an hour ago   880MB
    ```

    可见最终镜像仅 428 MB，相比最初的 880 MB 明显更小。

    通过对各阶段进行有针对性的优化、只保留必要内容，你在保证功能一致的前提下显著降低了镜像体积。这不仅提升性能，也让镜像更轻量、更安全、更易维护。

## 延伸阅读

* [多阶段构建](/build/building/multi-stage/)
* [Dockerfile 最佳实践](/develop/develop-images/dockerfile_best-practices/)
* [基础镜像](/build/building/base-images/)
* [Spring Boot Docker](https://spring.io/guides/topicals/spring-boot-docker)

