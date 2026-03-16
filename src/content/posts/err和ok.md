# 前言

我第一遇到这个问题是在go-zero里写jwt的时候，当时很自然的用err来获取错误信息

```go
claims, err := l.ctx.Value("claims").(jwt.MapClaims)//这是错误代码
```

然后出现了以下错误

```
无法在多个赋值中将 bool 赋给 err (类型 error)
类型未实现 error，因为缺少某些方法:
Error() string
```

这时便意思到要用ok来获取错误信息

所以在这里聊一聊ok和err的问题

# 区别

## err

err是error类型，更多的是获取函数返回的错误

## ok

ok是bool类型，表示是不是成功，有没有错误信息，用于 map 查找、类型断言、channel 接收等地方的“是否成功，在上面的代码中应该用ok，是因为返回的实际是`map[string]interface{}`,最主要的还是这是类型断言，所以这里应当使用ok。ps：本人使用err用魔怔了，这才意识到还有ok