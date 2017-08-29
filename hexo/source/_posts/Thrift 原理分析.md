---
title: Thrift 原理分析
date: 2017-08-29
tags: 分布式服务框架
categories: 分布式服务框架
---

由浅入深，我们先定义一个基本的接口文件，最最基本的：userService.thrift

```
namespace java com.xxx.userservice
struct User{
  1: i64 id,
  2: string name,
  3: i32 age
}

service UserService{
  User getUserById(1:i64 id)
}
```

`struct`只支持简单的数据类型，不支持复杂的Java类，这也是为了兼容更多的语言而做出的妥协。由struct生成的对象`User`实现了
