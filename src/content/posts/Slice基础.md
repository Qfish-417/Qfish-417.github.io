---
title: Slice的小细节
published: 2025-04-07
description: Slice的小细节
image: ./cover.jpg
tags: [易错, go]
category: 后端
draft: false
pinned: false
lang: zh-CN      # 仅当文章语言与 config.ts 中的站点语言不同时设置
---
------

# 2.1 slice的底层结构是怎样的？

```go
type sliceHeader struct {
    ptr *[0]byte  // 底层数组的指针
    len int       // slice 当前长度
    cap int       // slice 容量
}
```

Go slice 本质是一个轻量级结构体，包含三个字段：底层数组指针 ptr、长度 len、容量 cap。slice 自身不存数据，而是对数组的窗口。slice 可以共享底层数组，也可以通过 append 扩容时分配新数组。访问 slice 的元素实际上是通过 ptr + i 访问内存。

# 2.2 Go语言里slice是怎么扩容的？

Go slice 扩容发生在 append 时容量 cap 不够的情况下。Go 会申请新的底层数组，把原数组数据复制过去，然后返回新的 slice。小于 1024 个元素的 slice 扩容会翻倍，大于 1024 的 slice 每次扩容约 25%。扩容可能导致原 slice 与新 slice 不再共享底层数组，需要注意性能和并发问题。

 **为什么大 slice 不翻倍** → 减少内存浪费

**append 返回新 slice** → “旧 slice 是否改变”→旧 slice 的结构体本身（ptr、len、cap）不会改变，但底层数组可能共享或不共享，取决于是否触发扩容。

# 2.3 从一个切片截取出另一个切片，修改新切片的值会影响原来的切片内容吗

一个切片截取出的新切片默认会共享原切片的底层数组，因此修改新切片的元素会影响原切片中对应位置的值。只有当新切片触发扩容并分配了新的底层数组时，修改才不会影响原切片。

# 2.4 slice作为函数参数传递，会改变原slice吗？

slice 作为参数是值传递，但 slice 本身包含一个指向底层数组的指针，因此函数内部修改 slice 的元素会影响外部 slice；如果在函数里 append 并触发扩容，使 slice 指向新数组，那么外部 slice 不会受影响。