## 官方资料

全面了解请访问[官方网站](https://dataease.io/)，[项目地址](https://github.com/dataease/dataease)以及[源码部署文档](https://dataease.io/docs/v2/installation/deployment_installation)

## 源码编译

**DataEase 需要编译和打包 core-backend 得到一个 CoreApplication.jar**

不想单独安装 maven，可以使用 IDEA 自带的 Maven 进行编译打包 jar 包
在终端使用 IDEA 自带 maven，需在～/.bash_profile 中添加 Maven 环境：

```
export MAVEN_HOME=/Applications/IntelliJ\ IDEA.app/Contents/plugins/maven/lib/maven3
export PATH=$PATH:$MAVEN_HOME/bin
```

不想破坏现有环境变量，即不在配置文件或 IDEA 中修改环境，影响已有项目，可以在 maven 命令前添加环境

相关配置条件在文档中已有说明
运行命令（JAVA_HOME 需对应 jdk-21 实际位置，/usr/libexec/java_home -V 命令查看各 jdk）：
编译：`JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home mvn clean install -Dmaven.compiler.source=21 -Dmaven.compiler.target=21`
打包：`JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home mvn clean package -Pstandalone -U -Dmaven.test.skip=true`

## docker 部署

要求：mysql >= 8.0.5
此 mysql 数据库用于存储运行数据，而非用于展示的数据源

dataease 镜像默认是`registry.cn-qingdao.aliyuncs.com/dataease/dataease:v2.3-rc1`
dataease 通过`application.yml`获取 mysql 的连接参数，可以通过将本地运行目录（默认是`/opt/dataease2.0`）下的 conf 目录映射到 dataease 工作目录`/opt/apps/`下的 config 目录：
卷映射命令`- /opt/dataease2.0/conf:/opt/apps/config`，若`application.yml`实际位置不同，需同步修改

`application.yml`配置：
注意修改 mysql 的参数，需要与实际的 mysql 容器参数对应，例如：`jdbc:mysql://de_mysql:3306/dataease`，`de_mysql`为容器名称

原文档 docker 运行虽然使用`docker compose`，但没有配置 mysql 容器的依赖，即没有在 dataease 的服务中设置`depends_on`为 mysql
因此 dataease 和 mysql 容器没有运行在同一个网络，会因为无法连接 mysql 导致而 dataease 容器关闭，通过设置网络可以使两个容器处于同一个网络，并通过容器名称作为连接地址访问
若 mysql 不在容器中，而是在宿主机，url 可以设置为`host.docker.internal`，如此可以不需要配置 mysql 依赖

从源码制作 dataease 镜像代替默认镜像：

1. 在 Dockerfile 中修改镜像为`image: dataease`
2. 在运行目录中创建`application.yml`（复制源码中的相同文件）
3. 添加`COPY ${workdir}$/application.yml /opt/apps/config/`, `${workdir}`为`application.yml`所在目录（此步骤是非常规操作，将`application.yml`直接复制到镜像中，而不是通过卷映射）
4. 在项目根目录下（即 Dockerfile 所在目录）运行镜像构建命令：`docker build -t dataease .`

运行：

1. cd ${workdir}
2. 在目录下创建对应的`.env`、`docker-compose.yml`文件（复制源码中的相同文件）
3. 运行命令：`docker compose up`或`docker compose up -d`，-d 表示 docker 后台运行，日志不输出到终端
