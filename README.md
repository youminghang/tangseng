# Go-SearchEngine 重构中...

基于Go语言的搜索引擎

# 项目大体框架

![项目大体结构图](./doc/struct.png)

1、gin作为http框架，grpc作为rpc框架，etcd作为服务发现。\
2、总体服务分成`用户模块`、`收藏夹模块`、`搜索引擎模块`。\
3、用户模块和收藏夹模块使用共同的数据库。\
4、搜索引擎单独设立数据库，并且预留了一个redis作为缓存，存储搜索引擎数据采用读写分离模式，主要负责读，次要负责写，允许主从复制的延迟。

# 项目主要功能
## 1. 用户模块
- 登录注册

## 2. 收藏夹模块
- 创建/更新/删除/展示 收藏夹
- 将搜索结果的url进行收藏夹的创建/删除/展示

## 3. 搜索模块
- 重构中...

# 未来规划

## 1.架构相关
- [ ] 引入降级熔断
- [ ] 引入jaeger进行链路追踪
- [ ] 引入skywalking进行监控

## 2.功能相关
...

# 项目主要依赖
- gin
- gorm
- etcd
- grpc
- jwt-go
- logrus
- viper
- protobuf

# 项目结构

## 1.grpc_todolist 项目总体
```
grpc-todolist/
├── app                   // 各个微服务
│   ├── favorite          // 收藏夹
│   ├── gateway           // 网关
│   ├── search-engine     // 新版搜索微服务
│   ├── search-engine-old // 旧版搜索微服务
│   └── user              // 用户模块微服务
├── bin                   // 编译后的二进制文件模块
├── config                // 配置文件
├── consts                // 定义的常量
├── doc                   // 接口文档
├── idl                   // protoc文件
│   └── pb                // 放置生成的pb文件
├── loading               // 全局的loading，各个微服务都可以使用的工具
├── logs                  // 放置打印日志模块
├── pkg                   // 各种包
│   ├── ctl               // 用户信息相关
│   ├── discovery         // etcd服务注册、keep-alive、获取服务信息等等
│   ├── es                // es 模块
│   ├── jwt               // jwt鉴权
│   ├── logger            // 日志
│   ├── res               // 统一response接口返回
│   ├── util              // 各种工具、处理时间、处理字符串等等..
│   └── wrappers          // 熔断
└── types                 // 定义各种结构体
```

## 2.gateway 网关部分
```
gateway/
├── cmd                   // 启动入口
├── internal              // 业务逻辑（不对外暴露）
│   ├── handler           // 视图层
│   └── service           // 服务层
│       └── pb            // 放置生成的pb文件
├── logs                  // 放置打印日志模块
├── middleware            // 中间件
├── routes                // http 路由模块
└── rpc                   // rpc 调用
```

## 3.user && favorite 用户与收藏夹模块
```
user/
├── cmd                   // 启动入口
└──internal               // 业务逻辑（不对外暴露）
   ├── service            // 业务服务
   └── repository         // 持久层
       └── db             // 视图层
           ├── dao        // 对数据库进行操作
           └── model      // 定义数据库的模型
```

## 4.search-engine 搜索引擎模块
重构中...


# 项目文件配置

将config文件夹下的`config.yml.example`文件重命名成`config.yml`即可。

```yaml
server:
  port: :4000
  version: 1.0
  jwtSecret: 38324-search-engine

mysql:
  driverName: mysql
  host: 127.0.0.1
  port: 3306
  database: search_engine
  username: search_engine
  password: search_engine
  charset: utf8mb4

redis:
  user_name: default
  address: 127.0.0.1:6379
  password:

etcd:
  address: 127.0.0.1:2379

services:
  gateway:
    name: gateway
    loadBalance: true
    addr:
      - 127.0.0.1:10001 

  user:
    name: user
    loadBalance: false
    addr:
      - 127.0.0.1:10002 # 监听地址

  favorite:
    name: favorite
    loadBalance: false
    addr:
      - 127.0.0.1:10003 # 监听地址

  searchEngine:
    name: favorite
    loadBalance: false
    addr:
      - 127.0.0.1:10004 # 监听地址

domain:
  user:
    name: user
  favorite:
    name: favorite
  searchEngine:
    name: searchEngine
```


# 项目启动
## makefile启动

启动命令

```shell
make env-up         # 启动容器环境
make user           # 启动用户摸块
make task           # 启动任务模块
make gateway        # 启动网关
make env-down       # 关闭并删除容器环境
```

其他命令
```shell
make proto # 生成proto文件，如果proto有改变的话，则需要重新生成文件
```
生成.pb文件所需要的工具有`protoc-gen-go`,`protoc-gen-go-grpc`,`protoc-go-inject-tag`


## 手动启动

1. 利用compose快速构建环境

```shell
docker-compose up -d
```

2. 保证mysql,etcd活跃, 在 app 文件夹下的各个模块的 cmd 下执行

```go
go run main.go
```

# 导入接口文档

打开postman，点击导入

![postman导入](doc/1.点击import导入.png)

选择导入文件
![选择导入接口文件](doc/2.选择文件.png)

![导入](doc/3.导入.png)

效果

![postman](doc/4.效果.png)


# 项目启动
- 在各模块下进行

```go
go mod tidy
```

- 在各模块下的cmd目录

```go
go run main.go
```