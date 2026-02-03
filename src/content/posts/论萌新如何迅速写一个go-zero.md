---
title: 论萌新如何迅速写一个go-zero
published: 2023-09-09
description: 萌新如何迅速入门go-zero
image: ./cover.jpg
tags: [新手, go]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置
---
*<!--作者也是一个小萌新，这只是一个学习的过程，如果觉得有问题欢迎私信纠正-->*

------

# 1、下载go

首先应该就是先下载一个go，如果连go都没有后面也就无法进行

点击链接直接去官网下载[下载go](https://go.dev/dl/)

## 查看go的版本

```
go version
```

# 2、安装goctl

在终端运行一下代码安装goctl，安装后，goctl 会在 GOPATH/bin 目录下

```
go install github.com/zeromicro/go-zero/tools/goctl@latest

```

## 查看GOPATH

```
go env GOPATH
```

通常是在C:\Users\你的用户名\go\bin

当然以上的go和goctl都是需要进行全局变量（环境变量）配置

# 3、检查和配置

```
goctl env check --install --verbose --force
```

这个命名主要用于检查和配置 goctl 开发环境，能够自动检查和安装 goctl 开发所需的所有依赖工具和环境配置。这不是安装在项目里，安装的**所有工具都在你的电脑里**（系统级）。

到这里准备环境的工作已经基本上搞定了

# 4、写一个api文件

## ① 新建 `.api` 文件（定义接口）

假设文件名是：user.api

```
syntax = "v1"    // 指定语法版本为 v1

type (
    // 请求体结构
    RegisterReq {
        Username string `json:"username"`  // 用户名字段，JSON 标签
        Password string `json:"password"`  // 密码字段，JSON 标签
    }

    // 响应体结构  
    RegisterResp {
        Id int64 `json:"id"`  // 返回的用户ID
    }
)

@server(
    group: user          // 路由分组
    prefix: /api/v1      // 路由前缀  
)
service user-api {       // 服务名称
    @handler Register    // 处理器名称
    post /user/register (RegisterReq) returns (RegisterResp)  //请求方法是post，路由路径是/user/register，请求体类型RegisterReq，响应体类型RegisterResp
}
```

以上的代码只是一个简单的注册模块

如果需要多个方法可以参考下面的代码

```
syntax = "v1"

//注册
type (
	RegisterReq {
		Username string `json:"username"`
		Password string `json:"password"`
	}
	RegisterResp {
		Id int64 `json:"id"`
	}
)

//登录
type (
	LoginReq {
		Username string `json:"username"`
		Password string `json:"password"`
		RememberMe bool   `json:"remember"`
	}
	LoginResp {
		Id int64 `json:"id"`
	}
)

//token
type RefreshResp {
	Token string `json:"token"`
}

@server (
	group:  auth
	prefix: /api/v1
)
service user-api {
	@handler RefreshToken
	post /user/refresh returns (RefreshResp)
}

@server (
	group:  user
	prefix: /api/v1
)
service user-api {
	@handler Register
	post /user/register (RegisterReq) returns (RegisterResp)

	@handler Login
	post /user/login (LoginReq) returns (LoginResp)
}
```

<!--注意：如果想要在type块之间注释最好要进行换行。即使后面运行命名的时候会自动帮你换行，但尽量可以避免这个小问题-->

## ② 用 goctl 生成 API 项目代码（必执行）

在 `.api` 文件所在目录执行：

```
goctl api go -api 项目名.api -dir .
```

执行完后，会生成一整套结构 👇

![image-20260203154248863](/src/assets/images/image-20260203154248863.png)

以及一个user.go的文件



goctl会根据 api 文件 **自动生成骨架代码**

------

# 5、生成后你「必须会改」的 3 个地方（核心开发区）

## 1、配置文件

### etc/user.yaml（配置文件）

```
Name: user-api
Host: 0.0.0.0
Port: 8888
```

以后会在这里加上

- MySQL 配置
- Redis 配置
- JWT 配置

## 2、internal/handler（接收 HTTP 请求）

按照我的第一份api文件，registerhandler.go`（一般不用大改），如果你需要用自己写报错日志的话可以进行修改

```
func RegisterHandler(svcCtx *svc.ServiceContext) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        var req types.RegisterReq
        if err := httpx.Parse(r, &req); err != nil {
            httpx.Error(w, err)
            return
        }

        l := logic.NewRegisterLogic(r.Context(), svcCtx)
        resp, err := l.Register(&req)
        if err != nil {
            httpx.Error(w, err)
        } else {
            httpx.OkJson(w, resp)
        }
    }
}
```

> HTTP → 参数解析 → 调用 logic → 返回 JSON

------

## 3、internal/logic（写业务的地方）

```
func (l *RegisterLogic) Register(req *types.RegisterReq) (*types.RegisterResp, error) {
    // 写你的业务逻辑
    return &types.RegisterResp{
        Id: 1,
    }, nil
}
```

**所有业务都写在这里，不要写 handler**

------

# 6、创建 API 后要执行的代码

## 1. 初始化 go module

```
go mod init user
go mod tidy
```

（如果 goctl 已经帮你生成了 `go.mod`，这一步可跳）

------

## 2. 启动 API 服务

```
go run user.go
```

看到类似输出说明成功：

```
Starting server at 0.0.0.0:8888...
```

# 7、避雷

 手写 handler
 把业务写进 handler
改生成文件再重复 goctl 覆盖（具体查看覆盖规则）
api 路由和 yaml 端口对不上
忘了 `go mod tidy`



