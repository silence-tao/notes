# 1.怎么修改 Docker 容器的时区？

基本上所有的容器镜像都会使用 UTC，而基于 Linux 的容器中的时区通常是通过 `/etc/localtime` 符号链接。大多数 Linux 的基础镜像不会安装 tzdata 文件来节省空间，当前教程主要用于解决基于 CentOS 镜像时区的问题，CentOS 基础镜像中已经安装了 tzdata 包。

## 1.1 使用 `date` 命令

改变容器时区最简单的方式就是在连接上容器之后使用 `date` 命令：

``` bash
$ docker exec -it <容器id|容器name> date +%T -s "10:00:00"
```

尽管时区更改通常会立即反映，但在某些情况下，容器需要重新启动才能更改时间。

## 1.2 在启动容器的时候使用环境变量

容器的时区可以在创建时使用 docker 容器中的环境变量设置：

``` bash
$ docker run -e TZ=Asia/Shanghai centos centos
```

## 1.3 使用 Dockerfile

在托管环境或需要启动太多相同容器的情况下，最简单的管理方法是使用 Dockerfile。Dockerfile 包含每个容器的基本配置设置，要更改 Docker 容器中的时间，可以在相应的 Dockerfile 中完成更改，Dockerfile 中的设置将在重新创建或添加新容器时反映出来。只需要在 Dockerfile 中添加下面这个环境变量即可：

``` dockerfile
ENV TZ="Asia/Shanghai"
```

## 1.4 使用容器卷挂载

Docker 容器的一个主要问题是容器中的数据在重启后不是持久的。 为了解决这个问题，我们使用 Docker 数据卷。Docker 机器中的数据卷是包含特定于容器的数据的共享目录。 卷中的数据是持久的，不会在容器重建过程中丢失。Docker 中的目录 `/usr/share/zoneinfo` 包含可用的容器时区。 可以将此文件夹中的所需时区复制到 `/etc/localtime` 文件，以设置为默认时间。主机的这个时区文件可以在 Docker 卷中设置，并通过在 Dockerfile 中配置它在容器之间共享，配置如下所示：

``` yaml
volumes:
	- "/etc/timezone:/etc/timezone:ro"
	- "/etc/localtime:/etc/localtime:ro"
```

## 1.5 制作带当地默认时区的基础镜像

当容器过多时，手动更改时区是不可行的。 为了创建更多具有相同时区的 Docker 实例，可以制作一个带当地默认时区的基础镜像。在 Docker 容器中设置所需的时区后，退出它并使用 `docker commit` 命令将容器提交为一个新的镜像：

``` bash
$ docker commit <容器name> <镜像name>
```

通过这个基础镜像，我们可以创建很多基于这个镜像的同一个时区的镜像。