---
typora-root-url: ..\img
typora-copy-images-to: ..\img
---

# 1.升级内核

[CentOS 7/6系统升级内核版本](https://cloud.tencent.com/developer/article/1472857)

## 1.1 查看系统内核

``` shell
[root@VM_0_9_centos /]# uname -r
3.10.0-862.el7.x86_64
```

## 2.2 升级软件包

在升级内核之前，先升级软件包（非必要）

``` shell
yum update -y
```

## 3.3 升级内核

1.先导入elrepo的key，然后安装elrepo的yum源

``` shell
[root@VM_0_9_centos /]# rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@VM_0_9_centos /]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```

2.仓库启用后，可以使用下面的命令列出可用的内核相关包

``` shell
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

![image-20210428163457592](/image-20210428163457592.png)

上图可以看到，长期维护版本lt为5.4，最新主线稳定版ml为5.12，接下来使用命令安装最新稳定内核

``` shell
yum -y --enablerepo=elrepo-kernel install kernel-ml
```

# 2.运行EasyFaaS服务

使用docker-compose在本地运行easyfaas服务

## 2.1 拉取EasyFaaS源码

github地址：https://github.com/baidu/EasyFaaS

``` shell
# 创建EasyFaaS代码目录
[root@VM_0_9_centos github]# mkdir EasyFaaS
[root@VM_0_9_centos github]# cd EasyFaaS/
# 初始化本地仓库
[root@VM_0_9_centos EasyFaaS]# git init
Initialized empty Git repository in /root/github/EasyFaaS/.git/
# 绑定远程GitHub仓库地址
[root@VM_0_9_centos EasyFaaS]# git remote add origin https://github.com/baidu/EasyFaaS.git
# 拉取代码
[root@VM_0_9_centos EasyFaaS]# git pull
```

![image-20210428171925915](/image-20210428171925915.png)

## 2.2 设置运行参数

设置easyfaas服务组件容器运行参数

主要配置在./scripts/docker-compose/compose-local/.env 文件，其中

- **CONTROLLER_EXPORT_PORT**: controller组件暴露的服务端口
- **REGISTRY_EXPORT_PORT**： mock本地函数代码仓库服务组件的服务端口
- **FUNCTION_DATA_DIR**: 本地函数代码目录
- **easyfaas_REPO**： 镜像仓库repo
- **CONTROLLER_TAG**： controller组件镜像tag, 完整镜像ID为${easyfaas_REPO}/controller:{$CONTROLLER_TAG}
- **FUNCLET_TAG**： funclet组件镜像tag, 完整镜像ID为${easyfaas_REPO}/mini-funclet:{$CONTROLLER_TAG}
- **RUNTIME_TAG**: runner-runtime组件镜像tag, 完整镜像ID为${easyfaas_REPO}/runner-runtime:{$CONTROLLER_TAG}
- **REGISTRY_TAG**: 本地mock函数仓库组件tag, 完整镜像ID为${easyfaas_REPO}//func-registry:{$CONTROLLER_TAG}

``` shell
# env文件配置 示例
$ cat ./scripts/docker-compose/compose-local/.env 
CONTROLLER_EXPORT_PORT=8899
REGISTRY_EXPORT_PORT=8002
FUNCTION_DATA_DIR=/tmp/funcData
easyfaas_REPO=registry.baidubce.com/easyfaas-public
CONTROLLER_TAG=demo1.0
FUNCLET_TAG=demo1.0
RUNTIME_TAG=demo1.0
REGISTRY_TAG=demo1.0
```

备注：可以跳过此步骤，直接运行脚本快速体验（确保端口号不能冲突）

## 3.3 启动服务

运行如下命令启动服务，并运行docker ps查看组件是否正确运行： 其中controller组件容器，funclet组件容器及func-registry组件容器为常驻容器， 查看容器是否正常运行

``` shell
$ cd ./scripts/docker-compose/compose-local/
$ sh app_control.sh start
```

查看服务是否正常运行

![image-20210428173014849](/image-20210428173014849.png)

## 3.4 操作运行

操作运行(使用easyfaas函数计算服务)

### 3.4.1 创建函数

使用stub组件构建本地代码仓库其创建函数接口API详见接口API文档, 可以使用curl或者其他工具如Paw发起创建函数请求。

#### 1.编写函数代码

首先编写一个简单的nodejs10的hello world 函数代码,代码存储在index.js文件中 备注： 您也可以直接跳过此步骤，直接运行1.1和1.2，直接运行1.3

```shell
$ cat index.js 
exports.handler = (event, context, callback) => {
    callback(null, "Hello world!");
};
```

#### 2.打包函数代码并获得其base64编码

```shell
$ zip code.zip index.js 
$ base64 code.zip 
UEsDBBQAAAAIAAxDX00vNEyNUAAAAFgAAAAIABwAaW5kZXguanNVVAkAA/ie2VuKQthfdXgLAAEE
6AMAAAToAwAAS60oyC8qKdbLSMxLyUktUrBV0EgtS80r0VFIzs8rSa0AMRJzcpISk7M1FWztFKq5
FIAAJqSRV5qTo6Og5JGak5OvUJ5flJOiqKRpzVVrDQBQSwECHgMUAAAACAAMQ19NLzRMjVAAAABY
AAAACAAYAAAAAAABAAAApIEAAAAAaW5kZXguanNVVAUAA/ie2Vt1eAsAAQToAwAABOgDAABQSwUG
AAAAAAEAAQBOAAAAkgAAAAAA
```

#### 3.调用func-registry创建函数接口创建函数

请求body中Code字段填入上一步骤中获得的base64编码

```shell
$ curl -X POST "http://<YOUR_IP>:<YOUR_FUNC_REGISTRY_PORT>/v1/functions/testHelloWorld" -d '{"Version":"1","Description":"stubs create","Runtime":"nodejs10","Timeout":5,"MemorySize":128,"Handler":"index.handler","PodConcurrentQuota":10,"Code":"UEsDBBQAAAAAAHCjX00AAAAAAAAAAAAAAAAJABUAX19NQUNPU1gvVVgIALSf2Vu0n9lbVVQFAAG0n9lbUEsDBBQACAAIAAyjX00AAAAAAAAAAAAAAAATABUAX19NQUNPU1gvLl9pbmRleC5qc1VYCACwn9lb+J7ZW1VUBQAB+J7ZW2JgFWNnYGJg8E1MVvAPVohQgAKQGAMnAwODEQMDQx0DA5i/gYEo4BgSEgRlgnQsYGBgEEBTwogQl0rOz9VLLCjISdXLSSwuKS1OTUlJLElVDggGKXw772Y0iO5J8tAH0YAAAAD//1BLBwgOCcksZgAAALAAAABQSwMEFAAIAAgAAAAAAAAAAAAAAAAAAAAAAAgAAABpbmRleC5qc0qtKMgvKinWy0jMS8lJLVKwVdBILUvNK9FRSM7PK0mtADESc3KSEpOzNRVs7RSquRQUFOBCGnmlOTk6CkoeqTk5+Qrl+UU5KYpKmtZctdaAAAAA//9QSwcILzRMjVUAAABYAAAAUEsBAhQDFAAAAAAAcKNfTQAAAAAAAAAAAAAAAAkAFQAAAAAAAAAAQP1BAAAAAF9fTUFDT1NYL1VYCAC0n9lbtJ/ZW1VUBQABtJ/ZW1BLAQIUAxQACAAIAAyjX00OCcksZgAAALAAAAATABUAAAAAAAAAAECkgTwAAABfX01BQ09TWC8uX2luZGV4LmpzVVgIALCf2Vv4ntlbVVQFAAH4ntlbUEsBAhQAFAAIAAgAAAAAAC80TI1VAAAAWAAAAAgAAAAAAAAAAAAAAAAA+AAAAGluZGV4LmpzUEsFBgAAAAADAAMA2AAAAIMBAAAAAA=="}' -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5'   -H 'Content-Type: application/json; charset=utf-8'
请求示例
POST /v1/functions/testHelloWorld HTTP/1.1
Content-Type: application/json; charset=utf-8
Host: 172.22.170.33:8002
Connection: close
User-Agent: Paw/3.1.10 (Macintosh; OS X/10.15.4) GCDHTTPRequest
Content-Length: 990

{"Version":"1","Description":"stubs create","Runtime":"nodejs10","Timeout":5,"MemorySize":128,"Handler":"index.handler","PodConcurrentQuota":10,"Code":"UEsDBBQAAAAAAHCjX00AAAAAAAAAAAAAAAAJABUAX19NQUNPU1gvVVgIALSf2Vu0n9lbVVQFAAG0n9lbUEsDBBQACAAIAAyjX00AAAAAAAAAAAAAAAATABUAX19NQUNPU1gvLl9pbmRleC5qc1VYCACwn9lb+J7ZW1VUBQAB+J7ZW2JgFWNnYGJg8E1MVvAPVohQgAKQGAMnAwODEQMDQx0DA5i/gYEo4BgSEgRlgnQsYGBgEEBTwogQl0rOz9VLLCjISdXLSSwuKS1OTUlJLElVDggGKXw772Y0iO5J8tAH0YAAAAD//1BLBwgOCcksZgAAALAAAABQSwMEFAAIAAgAAAAAAAAAAAAAAAAAAAAAAAgAAABpbmRleC5qc0qtKMgvKinWy0jMS8lJLVKwVdBILUvNK9FRSM7PK0mtADESc3KSEpOzNRVs7RSquRQUFOBCGnmlOTk6CkoeqTk5+Qrl+UU5KYpKmtZctdaAAAAA//9QSwcILzRMjVUAAABYAAAAUEsBAhQDFAAAAAAAcKNfTQAAAAAAAAAAAAAAAAkAFQAAAAAAAAAAQP1BAAAAAF9fTUFDT1NYL1VYCAC0n9lbtJ/ZW1VUBQABtJ/ZW1BLAQIUAxQACAAIAAyjX00OCcksZgAAALAAAAATABUAAAAAAAAAAECkgTwAAABfX01BQ09TWC8uX2luZGV4LmpzVVgIALCf2Vv4ntlbVVQFAAH4ntlbUEsBAhQAFAAIAAgAAAAAAC80TI1VAAAAWAAAAAgAAAAAAAAAAAAAAAAA+AAAAGluZGV4LmpzUEsFBgAAAAADAAMA2AAAAIMBAAAAAA=="}

响应结果:
HTTP/1.1 200 OK
Server: fasthttp
Date: Fri, 11 Dec 2020 03:07:50 GMT
Content-Length: 0
Connection: close
```

``` shell
# 请求示例
curl -X POST "http://127.0.0.1:8002/v1/functions/testHelloWorld" -d '{"Version":"1","Description":"stubs create","Runtime":"nodejs10","Timeout":5,"MemorySize":128,"Handler":"index.handler","PodConcurrentQuota":10,"Code":"UEsDBBQAAAAAAHCjX00AAAAAAAAAAAAAAAAJABUAX19NQUNPU1gvVVgIALSf2Vu0n9lbVVQFAAG0n9lbUEsDBBQACAAIAAyjX00AAAAAAAAAAAAAAAATABUAX19NQUNPU1gvLl9pbmRleC5qc1VYCACwn9lb+J7ZW1VUBQAB+J7ZW2JgFWNnYGJg8E1MVvAPVohQgAKQGAMnAwODEQMDQx0DA5i/gYEo4BgSEgRlgnQsYGBgEEBTwogQl0rOz9VLLCjISdXLSSwuKS1OTUlJLElVDggGKXw772Y0iO5J8tAH0YAAAAD//1BLBwgOCcksZgAAALAAAABQSwMEFAAIAAgAAAAAAAAAAAAAAAAAAAAAAAgAAABpbmRleC5qc0qtKMgvKinWy0jMS8lJLVKwVdBILUvNK9FRSM7PK0mtADESc3KSEpOzNRVs7RSquRQUFOBCGnmlOTk6CkoeqTk5+Qrl+UU5KYpKmtZctdaAAAAA//9QSwcILzRMjVUAAABYAAAAUEsBAhQDFAAAAAAAcKNfTQAAAAAAAAAAAAAAAAkAFQAAAAAAAAAAQP1BAAAAAF9fTUFDT1NYL1VYCAC0n9lbtJ/ZW1VUBQABtJ/ZW1BLAQIUAxQACAAIAAyjX00OCcksZgAAALAAAAATABUAAAAAAAAAAECkgTwAAABfX01BQ09TWC8uX2luZGV4LmpzVVgIALCf2Vv4ntlbVVQFAAH4ntlbUEsBAhQAFAAIAAgAAAAAAC80TI1VAAAAWAAAAAgAAAAAAAAAAAAAAAAA+AAAAGluZGV4LmpzUEsFBgAAAAADAAMA2AAAAIMBAAAAAA=="}' -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5' -H 'Content-Type: application/json; charset=utf-8'
```

### 3.4.1 查看函数

请求示例

```shell
$ curl -X GET "http://<YOUR_IP>:<YOUR_FUNC_REGISTRY_PORT>/v1/functions/brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1" -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5'   -H 'Content-Type: application/json; charset=utf-8' 

GET /v1/functions/testHelloWorld HTTP/1.1
Host: 127.0.0.1:8002
Connection: close
```

响应结果

```javascript
{
  "Code": {
    "_": {},
    "Location": "/var/faas/funcData/brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1/code.zip",
    "RepositoryType": "filesystem",
    "LogType": ""
  },
  "Concurrency": {
    "_": {},
    "ReservedConcurrentExecutions": null,
    "AccountReservedSum": 0
  },
  "Configuration": {
    "_": {},
    "CodeSha256": "VoLwl2uaH/cF0hsvKQMkLbbd7JKtGgG5MNExk8whT+M=",
    "CodeSize": 625,
    "DeadLetterConfig": null,
    "Description": "stubs create",
    "Environment": {
      "_": {},
      "Error": null,
      "Variables": null
    },
    "FunctionArn": "brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1",
    "FunctionName": "testHelloWorld",
    "Handler": "index.handler",
    "KMSKeyArn": null,
    "LastModified": "2021-02-28T11:11:07Z",
    "Layers": null,
    "MasterArn": null,
    "MemorySize": 128,
    "RevisionId": null,
    "Role": null,
    "Runtime": "nodejs10",
    "Timeout": 5,
    "TracingConfig": null,
    "Version": "1",
    "VpcConfig": null,
    "CommitId": "349dd8a4-db7d-4fc3-a4cd-8c8b482e00c3",
    "Uid": "df391b08c64c426a81645468c75163a5",
    "PodConcurrentQuota": 10
  },
  "LogConfig": {
    "LogType": "",
    "BosDir": "",
    "Params": ""
  },
  "Tags": {}
}
```

``` shell
# 请求示例
curl -X GET "http://127.0.0.1:8002/v1/functions/brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1" -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5' -H 'Content-Type: application/json; charset=utf-8'
```

### 3.4.3 运行函数

向controller发起函数调用请求

```shell
$ curl -X "POST" "http://<YOUR_IP>:<YOUR_CONTROLLER_PORT>/v1/functions/brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1/invocations"   -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5'      -H 'Content-Type: application/json; charset=utf-8'      -d $'{}'
请求示例
POST /v1/functions/brn:bce:cfc:bj:cd64f99c69d7c404b61de0a4f1865834:function:testHSF2:1/invocations HTTP/1.1
Content-Type: application/json; charset=utf-8
X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5
Host: 127.0.0.1:8001
Connection: close

响应示例：
Hello world!
```

``` shell
# 请求示例
curl -X POST "http://127.0.0.1:8899/v1/functions/brn:cloud:faas:bj:8f6e5a28c663957ea04522547a66d08f:function:testHelloWorld:1/invocations" -H 'X-easyfaas-Account-Id: df391b08c64c426a81645468c75163a5' -H 'Content-Type: application/json; charset=utf-8' -d $'{}'
```