---
title: cast类型转换库—go库推荐
date: 2023-02-18 17:53:10
tags:
    - golang
---
在 Go 语言中，类型转换是一项经常需要进行的操作，但转换不同类型之间的值却需要引入不同的库，过程相对繁琐。不过，`cast` 库解决了这个问题，提供了很多种类型转换函数，而且通常只需要引入这一个库即可。

`cast` 库不仅提供了常见类型之间的转换，例如 int 转换为 string，还提供了更加复杂的类型转换，例如将时间字符串转换为 `time.Time` 类型。此外，它还提供了两种转换函数，一种仅返回该类型的零值，不会返回错误，另一种则会返回错误，以便开发人员可以根据实际需求选择使用。

以下是一些常用的 API：

- `cast.StringToDate`: 将常见的时间字符串转换为 `time.Time` 类型

```
fmt.Println(cast.StringToDate("2006-01-02 15:04:05"))
// output: 2006-01-02 15:04:05 +0000 UTC <nil>

```

- `cast.ToBoolSlice`: 将任意类型的数组转换为 `[]bool` 类型

```
fmt.Println(cast.ToBoolSlice([]interface{}{1,2,0,true, false}))
// output: [true true false true false]

```

- `cast.ToBool`: 将任意类型转换为 `bool` 类型

```
fmt.Println(cast.ToBool(1)) // output:true
fmt.Println(cast.ToBool(0)) // output: false
fmt.Println(cast.ToBool("")) // output: false
fmt.Println(cast.ToBool("0")) // output: false
fmt.Println(cast.ToBool("1")) // output: true
fmt.Println(cast.ToBool("true")) // output: true
fmt.Println(cast.ToBool("false")) // output: false

```

- `cast.ToString`: 将任意类型转换成字符串类型

```
cast.ToString("mayonegg")         // "mayonegg"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("one time")) // "one time"
cast.ToString(nil)        // "nil"
var foo interface{} = "one more time"
cast.ToString(foo)                // "one more time"

```

- `cast.ToStringMap`: 将任意类型的 map 转换成 `map[string]interface{}`

```
fmt.Println(cast.ToStringMap(map[interface{}]interface{}{
    1:2,
    "2":3,
    true:1,
}))
 // output: map[1:2 2:3 true:1]

```

更多的 API 可以在其文档中查看：[https://github.com/spf13/cast](https://github.com/spf13/cast)
