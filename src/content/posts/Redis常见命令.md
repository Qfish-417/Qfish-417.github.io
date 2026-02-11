---
title: Redis的常见命令
published: 2025-04-21
description: Redis
image: ./cover.jpg
tags: [新手, Redis]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置



---

# 

他是key-value数据库，key一般是字符串型，value可以是其他的

String、Hash、List、、Set、SortedSet（五个基本类型）GEO、BitMap、HyperLog

官方文档[Commands | Docs](https://redis.io/docs/latest/commands/)

# 通用命令

## MSET

可以一次设置多个键值对的命令，属于原子性操作，要么成功，要么失败

## KEYS

查看符合末班的所有key（不建议生产环境）

## DEL

有中括号，可以删除多个

## EXISTS

也可以一对多

判断是否存在

## EXPIRE

给key设置有效期，有效期到期时key会被删除

## TTL

查看剩余有效期

-2相当于已经删除了

-1相当于永久有效

