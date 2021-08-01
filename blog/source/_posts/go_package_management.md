---
title: go的依赖包管理   
date: 2021-04-01 16:02:13   
comments: true   
tags: go   
categories: Go基础   
excerpt: go的依赖包管理一直被人们所诟病，好在一直在改进；现如今，已经十分方便了，要是你还不了解，不妨现在来了解下吧.   
---

> Go的依赖包管理方式有很多，一般是大厂开源出来的解决方案，例如：[Dep](https://github.com/golang/dep)、[GoDep](https://github.com/tools/godep)、[GoVendor](https://github.com/kardianos/govendor)、GoModule等；目前市面上用的比较多的还是GoVendor+GoModule相结合的方式，具体我们来看看是如何使用的吧.

## Dep方式

### 安装
```shell script
$ curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
$ go get -u github.com/golang/dep/cmd/dep
$ brew install dep  # macOS
$ brew upgrade dep  # macOS
```

### 初始化
```shell script
$ dep init
$ ls
Gopkg.toml Gopkg.lock vendor/
```

### 日常操作
```shell script
dep ensure -add github.com/pkg/errors
dep ensure -update github.com/foo/bar
dep ensure -update                          # update all dependencies
dep status                                  # 用于查看当前项目依赖了哪些包，以及包的版本号
```

## GoDep方式

### 安装
```shell script
go get github.com/tools/godep
```

### 初始化
```shell script
godep save
```
_该命令会在根目录下自动生成一个Godeps和vendor目录，并将项目所依赖的第三方包信息写入Godeps/Godeps.json，同时复制包源码到vendor目录。注意：godep save并不会自动从远程下载依赖包，需要我们通过go get或godep get手动下载，godep save只是将下载的包源码复制到vendor目录_

### 日常操作
```shell script
go get -v -u github.com/gin-gonic/gin # 下载
godep update github.com/gin-gonic/gin
gedep get github.com/gin-gonic/gin
```

## GoVendor方式

### 为什么用vendor目录?
_不使用vendor目录的时候，我们代码库的所有依赖包都是安装到GoPath目录下的，也就是说，所有的代码库共用同一个依赖包库，然而实际上，每个代码库很可能依赖的包版本是不一样的；使用vendor目录的话，便可以很好地做到项目的隔离，它允许不同的代码库拥有自己的依赖包._

### 安装
```shell script
go get -u -v github.com/kardianos/govendor
```

### 初始化
```shell script
govendor init
```
命令执行完后，在我们的服务目录下，便会多出一个vendor目录，里面包含了服务所有依赖的包，与此同时，vendor目录下还有个vendor.json文件，它描述了各个依赖包的版本信息

### 常用命令
- 将已被引用且在 $GOPATH 下的所有包复制到 vendor 目录
```shell script
govendor add +external
```
- 仅从 $GOPATH 中复制指定包
```shell script
govendor add github.com/gin-gonic/gin
```
- 列出代码中所有被引用到的包及其状态
```shell script
govendor list
govendor list -v fmt  // 显示fmt包被哪些包引用
```
- 从远程仓库添加或更新某个包(如果不在 $GOPATH 下，也会存一份)
```shell script
govendor fetch github.com/gin-gonic/gin
```
- 根据vendor.json拉取并更新包
```shell script
govendor sync
```

## GoModule方式
> 从 Go 1.11 版本开始，官方已内置了更为强大的 Go modules 来一统多年来 Go 包依赖管理混乱的局面(Go 官方之前推出的 dep 工具也几乎胎死腹中)，并且将在 1.12 版本中正式默认开启。此方式受到社区的看好和强烈推荐，建议新项目采用 Go modules

### 开启
```shell script
export GO111MODULE=on
```

### 初始化
```shell script
go mod init <module-name> # 模块名可选，默认和目录名保持一致
```

### 检测&安装
```shell script
go mod tidy
```
_tidy会检测该文件夹目录下所有引入的依赖，写入go.mod文件；届时会发现go.mod文件发生改变_

### 常用命令
```shell script
go mod init <module-name> # 初始化go.mod
go mod tidy  # 更新依赖文件
go mod download  # 下载依赖文件
go mod vendor  # 将依赖转移至本地的vendor文件
go mod edit  # 手动修改依赖文件
go mod graph  # 打印依赖图
go mod verify  # 校验依赖
go build -mod=vendor # 编译时使用 vendor 目录
```

### go.mod文件
> 经常使用GoModule方式管理项目依赖的朋友，应该有注意到go mod文件的配置项都有如下这些：
```go
module <module-name> 指明项目的模块名
require ( //指定的依赖项模块,可选；如果只有单个，括号可省略，和go里面var/const声明多个变量是一样的
  github.com/BurntSushi/toml v0.3.1   //配置依赖模块地址和版本，多个配置多行
  github.com/DataDog/zstd v1.3.5
)
replace ( //替换依赖项模块,可选；如果只有单个，括号可省略，和go里面var/const声明多个变量是一样的
)
exclude ( //忽略依赖项模块，可选；如果只有单个，括号可省略，和go里面var/const声明多个变量是一样的
)
```
