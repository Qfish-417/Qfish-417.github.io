---
title: String类型
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

# 

意味着它的value是字符串，不过根据字符串的格式不同，又可以分为**string**、**int**、**float**。无论哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型空间不超过512m

# 基础命令

## SET

设置一个值（覆盖已有值）

```css
SET name "Qfish"
```

## GET

取值

```css
GET name
```

# 批量操作

## MSET

一次设置多个值

```css
MSET a 1 b 2 c 3
```

## MGET

```css
MGET a b c
```

# 设置时间

## SETEX

设置值 + 过期时间

秒级过期

```nginx
SETEX code 60 "123456"
```

## PSETEX

毫秒级过期

```nginx
PSETEX temp 500 "hi"
```

## SETNX key value

仅当 key 不存在时才设置（常用于分布式锁）

```csharp
SETNX lock 1
```

# 数字操作

## INCR

自增加1

```nginx
INCR count
```

## DECR

自减1

```nginx
DECR count
```

## INCRBY

自增n

```nginx
INCRBY count 10
```

## DECRBY

自减n

```nginx
DECRBY count 10
```

## INCRBYFLOAT

自增浮点数

```nginx
INCRBYFLOAT price 0.8
```

# 字符串追加与长度

### **APPEND **

往原值后追加内容

```nginx
APPEND msg " world"
```

### **STRLEN **

获取字符串长度

```nginx
STRLEN name
```

# 子串操作

### **GETRANGE key start end**

按范围取子串

```nginx
GETRANGE title 0 4
```

### **SETRANGE key offset value**

从偏移位置开始覆盖写入（不会删除之后的内容）

```nginx
SETRANGE name 2 "fish"
```

# GETSET

### **GETSET key newValue**

返回旧值，同时设置新值

```nginx
GETSET token "new_token"
```

# set key value nx==setnx

setex同理