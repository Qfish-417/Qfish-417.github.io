---
title: Redis的下载
published: 2025-04-21
description: Redis
image: ./cover.jpg
tags: [新手, Redis]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置


---

------

# 

# 一般建议在Linux或者虚拟机下载

下载

```
sudo apt update
sudo apt install redis-server
```

启动：

```
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

验证：

```
redis-cli ping
```

# 设置密码

1. 打开配置文件：

```
sudo nano /etc/redis/redis.conf
```

1. 找到这一行（大约在文件开头附近）：

```
# requirepass foobared
```

1. 去掉 `#`，并把 `foobared` 改成你想设置的密码，例如：

```
requirepass MyRedisPass123
```

1. 保存并退出：
   - 在 nano 中：按 `Ctrl+O` 保存，然后 `Enter`，再按 `Ctrl+X` 退出
2. 重启 Redis 服务让配置生效：

```
sudo systemctl restart redis-server
```

