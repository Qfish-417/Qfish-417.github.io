---
title: go-zero的kafka配置
published: 2025-06-08
description: 配置kafka
image: ./cover.jpg
tags: [Go, 消息队列]
category: Golang
draft: false
pinned: false
lang: zh-CN

---


---
配置主要分为三个部分：<u>***配置文件定义***</u>、<u>***生产者发送***</u>、<u>***消费者监听***</u>

# 1.定义配置文件

需要定义 Brokers 地址、Topic 名称和消费者组 ID。

## 在yaml文件里面加上配置

```yaml
  Kafka:
    Brokers:
      - "127.0.0.1:9092"
      - "127.0.0.1:9093"
    # 业务 Topic
    OrderTopic: "chat"
    # 消费者组 ID (多个实例部署时，相同的 GroupId 会负载均衡消费)
    OrderGroupId: "chats"
```

## 在config里面配置

```go
输入 Config struct {
    rest.RestConf
    //
    Kafka       KafkaConf `json:"kafka"`
}
type KafkaConf struct {
	Brokers []string `json:"brokers"` // Kafka 集群地址列表，例如 ["192.168.1.10:9092", "192.168.1.11:9092"]
	Topic   string   `json:"topic"`   // 默认 Topic 名称
	Group   string   `json:"group"`   // 消费者组 ID (如果是纯生产者服务，此字段可选)

	// 可选的高级配置 (根据需求添加)
	// Timeout       int `json:"timeout,optional,default=3000"` // 超时时间 ms
	// RetryCount    int `json:"retryCount,optional,default=3"` // 重试次数
}
```

## 2初始化以及分装

`go get github.com/zeromicro/go-queue`

然后就是svc

```go
输入 ServiceContext struct {
	Config       config.Config
	//...
	OrderProducer *kq.Pusher
	// 注意：消费者通常在 main.go 或独立的 worker 程序中启动，不一定要放在 ServiceContext 里，
	// 但如果是 API 服务同时兼做消费者，也可以在此初始化。
}
```

```go
func NewServiceContext(c config.Config) *ServiceContext {
	//...
	pusher := kq.NewPusher(c.Kafka.Brokers, c.Kafka.Topic)
	return &ServiceContext{
		Config:       c,
		//...
        OrderProducer: pusher,
	}
}

```

### 3定义生产者和消费者

## 生产者

只有三步：准备消息、序列化、发送消息

## 消费者

通常是一个独立的进程

需要：配置消费者、启动监听队列、启动消费循环
