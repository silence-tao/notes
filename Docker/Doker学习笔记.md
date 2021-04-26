---
typora-root-url: ..\img
---

# 1.安装Docker

## 1.1 环境准备

CentOS7

## 1.2 环境查看

``` shel
# 系统环境必须是3.10以上的
[root@VM_0_9_centos /]# uname -r
3.10.0-862.el7.x86_64
```

``` shell
[root@VM_0_9_centos /]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

## 1.3 安装

帮助文档：https://docs.docker.com/

1.卸载旧的版本

``` shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2.需要的安装包

``` shel
sudo yum install -y yum-utils
```

3.设置镜像的仓库

``` shel
# 默认是境外的仓库，很慢不建议使用
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 阿里云镜像仓库，推荐使用
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
# 更新软件包索引
yum makecache fast
```

4.安装docker docker-ce 社区版 ee 企业版

``` shel
sudo yum install docker-ce docker-ce-cli containerd.io
```

5.启动docker

``` she
sudo systemctl start docker
```

6.使用 `docker version` 查看是否安装成功

![image-20210427003114853](../img/image-20210427003114853.png)

7.hello world

``` shel
sudo docker run hello-world
```

![image-20210427004608665](../img/image-20210427004608665.png)

8.查看一下下载的hello-world镜像

``` shel
docker images
```

![image-20210427004854294](../img/image-20210427004854294.png)

## 1.4 卸载docker

``` shel
# 卸载依赖
sudo yum remove docker-ce docker-ce-cli containerd.io

# 删除docker资源目录
# /var/lib/docker	docker的默认工作路径
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

# 2.Docker的常用命令

## 2.1 帮助命令

``` shel
docker version		# 显示docker的版本信息
docker info			# 显示docker的系统信息，包括镜像和容器的数量
docker <命令> --help	# 万能命令
```

帮助文档地址：https://docs.docker.com/reference/

## 2.2 镜像命令

### 2.2.1 `docker images`

查看所有本地主机上的镜像

``` she
[root@VM_0_9_centos /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    d1165f221234   7 weeks ago   13.3kB

# 解释
REPOSITORY		镜像的仓库源
TAG				镜像的标签
IMAGE ID		镜像的id
CREATED			镜像的创建时间
SIZE			镜像的大小

# 可选项
-a, --all             # 列出所有的镜像
-q, --quiet           # 只显示镜像的id
```

### 2.2.2 `docker search`

搜索镜像

``` she
[root@VM_0_9_centos /]# docker search mysql
NAME     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql    MySQL is a widely used, open-source relation??  10777     [OK]       
mariadb  MariaDB Server is a high performing open sou??  4058      [OK]

# 可选项，通过收藏来过滤
-f, --filter filter   Filter output based on conditions provided
    --format string   Pretty-print search using a Go template
    --limit int       Max number of search results (default 25)
    --no-trunc        Don't truncate output

--filter=STARS=3000	  # 搜索出来的镜像就是STARS大于3000
[root@VM_0_9_centos /]# docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation??  10777     [OK]       
mariadb   MariaDB Server is a high performing open sou??  4058      [OK]       
[root@VM_0_9_centos /]# docker search mysql --filter=STARS=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation??  10777     [OK]       
```



## 2.3 容器命令

