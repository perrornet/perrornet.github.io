---
title: cast Type Conversion Library - Recommended Go Library
date: 2023-02-18 17:53:10
tags:
    - golang
---
In Go language, type conversion is a frequently used operation, but converting values between different types requires importing different libraries, which can be relatively cumbersome. However, the `cast` library solves this problem and provides many type conversion functions, usually only requiring the import of this one library.

The `cast` library not only provides conversions between common types, such as int to string, but also provides more complex type conversions, such as converting a time string to the `time.Time` type. In addition, it provides two conversion functions, one only returns the zero value of that type without returning an error, and the other returns an error, so developers can choose to use it according to their actual needs.

Here are some commonly used APIs:

- `cast.StringToDate`: Converts common time strings to the `time.Time` type.

```
fmt.Println(cast.StringToDate("2006-01-02 15:04:05"))
// output: 2006-01-02 15:04:05 +0000 UTC <nil>

```

- `cast.ToBoolSlice`: Converts an array of any type to the `[]bool` type.

```
fmt.Println(cast.ToBoolSlice([]interface{}{1,2,0,true, false}))
// output: [true true false true false]

```

- `cast.ToBool`: Converts any type to the `bool` type.

```
fmt.Println(cast.ToBool(1)) // output:true
fmt.Println(cast.ToBool(0)) // output: false
fmt.Println(cast.ToBool("")) // output: false
fmt.Println(cast.ToBool("0")) // output: false
fmt.Println(cast.ToBool("1")) // output: true
fmt.Println(cast.ToBool("true")) // output: true
fmt.Println(cast.ToBool("false")) // output: false

```

- `cast.ToString`: Converts any type to a string type.

```
cast.ToString("mayonegg")         // "mayonegg"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("one time")) // "one time"
cast.ToString(nil)        // "nil"
var foo interface{} = "one more time"
cast.ToString(foo)                // "one more time"

```

- `cast.ToStringMap`: Converts a map of any type to `map[string]interface{}`.

```
fmt.Println(cast.ToStringMap(map[interface{}]interface{}{
    1:2,
    "2":3,
    true:1,
}))
 // output: map[1:2 2:3 true:1]

```

More APIs can be found in its documentation: [https://github.com/spf13/cast](https://github.com/spf13/cast)
