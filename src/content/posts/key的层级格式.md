---
title: key的层级结构
published: 2025-04-22
description: Redis
image: ./cover.jpg
tags: [新手, Redis]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置




---

------



# 结构

key允许有多个单词形成层级结构，多个单词之间用:隔开，格式如下：

```
项目名:业务名:类型:id
```

可以将对象序列化为JSON字符串后存储

避免id冲突