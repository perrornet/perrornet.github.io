---
title: cast类型转换库—go库推荐
date: 2023-02-18 17:53:10
tags:
    - golang
---
golang 中各种类型之间的转换实在是太过于繁琐，但是类型转换偏偏又是我们在开发过程中最常用的操作。同类型之间转换还稍微好一点，可以使用内置类型函数进行转换，但是想要转换不同的类型却需要引入不同的库， 而 cast 库正好解决了这一痛点，它提供了很多种的类型转换函数同我们使用， 通常的类型转换我们只需要引入这一个库就可以解决。
<!-- more -->


通常 go 中的类型转换都需要 err 变量来提示类型不可转换，cast 库提供了两种方法：

1. 仅返回该类型的零值，不会返回 err
2. 使用函数 To__E 将会返回一个 err 变量来告诉我们是否发生错误这样分开提供函数，极大的方便我们在开发中根据当前所需选择函数。

常用 API 列表：

* cast.StringToDate: 将常见的时间字符串转换为 time.Time 类型
```
fmt.Println(cast.StringToDate("2006-01-02 15:04:05")) 
// output: 2006-01-02 15:04:05 +0000 UTC <nil>
```

* cast.ToBoolSlice: 将任意类型的数组转换为 []bool 类型
```
fmt.Println(cast.ToBoolSlice([]interface{}{1,2,0,true, false})) 
// output: [true true false true false]
```
* cast.ToBool: 将任意类型转换为 bool 类型
```
fmt.Println(cast.ToBool(1)) // output:true
fmt.Println(cast.ToBool(0)) // output: false
fmt.Println(cast.ToBool("")) // output: false
fmt.Println(cast.ToBool("0")) // output: false
fmt.Println(cast.ToBool("1")) // output: true
fmt.Println(cast.ToBool("true")) // output: true
fmt.Println(cast.ToBool("false")) // output: false
```
* cast.ToString: 将任意类型转换成 string
```
cast.ToString("mayonegg")         // "mayonegg"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("one time")) // "one time"
cast.ToString(nil)        // "nil"
var foo interface{} = "one more time"
cast.ToString(foo)                // "one more time"
```
* cast.ToStringMap: 将任意类型 map 转换成 map[string]interface{}
```
fmt.Println(cast.ToStringMap(map[interface{}]interface{}{
    1:2,
    "2":3,
    true:1,
}))
 // output: map[1:2 2:3 true:1]
```
更多的 API 可查看其文档：[https://github.com/spf13/cast](https://github.com/spf13/cast)