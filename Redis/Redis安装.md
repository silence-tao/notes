# Redis安装

## 1.Redis在Mac环境下安装

### 1.1 通过HomeBrew安装Redis

**1.查看 Redis 版本**

使用下面的命令查看 Redis 的版本：

``` shell
brew search redis
```

示例：

``` bash
chentao10@SHMAC-C02GK0ZYM ~ % brew search redis
==> Formulae
hiredis             redis               redis@3.2           redir
iredis              redis-leveldb       redis@4.0           redo
```

> 通过 `@` 来指定版本，上面没指定版本的默认是最新版本

**2.安装 Redis 最新版本**

命令：

``` shell
brew install redis
```

安装信息：

``` bash
chentao10@SHMAC-C02GK0ZYM ~ % brew install redis
Running `brew update --preinstall`...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> New Formulae

### 省略 ###

==> Checking for dependents of upgraded formulae...
==> No broken dependents found!
==> Caveats
==> redis
To restart redis after an upgrade:
  brew services restart redis
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/redis/bin/redis-server /usr/local/etc/redis.conf
```

**3.启动 Redis**

通过 brew services 来启动（后台启动）推荐，命令如下：

``` shell
brew services start redis
```

通过 Redis 默认的命令 redis-server 来启动

由于 Home Brew 会帮我们配置好环境变量（创建相关可执行文件的符号链接 到 `/usr/local/bin` 目录)），所以我们可以在终端的任意目录下运行命令：

``` shell
# 指定配置文件，后台启动必须这样做
redis-server /usr/local/etc/redis.conf

# 默认启动参数，这样启动后关闭窗口服务就停止了
redis-server
```

**4.连接 Redis**

Redis 默认端口为 `6379`，通过以下命令连接：

``` shell
# 本机服务连接
redis-cli

# 远程服务连接
redis-cli -h <远程服务IP地址> -p <端口号（默认6379）>
```

**5.关闭 Redis 服务**

通过以下命令关闭 Redis：

``` shell
redis-cli shutdown
```

### 1.2 通过源码安装

[Redis 官网](https://redis.io/) 下载源码

``` shell
# 解压源码
tar -zxvf redis redis-7.0.0.tar.gz

# 编译源码
make

# 安装源码
make install
```

