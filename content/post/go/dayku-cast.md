+++
title = "Go-Cast库"
description = "Cast用于以一致且简单的方式在不同的go类型之间进行转换"
tags = [
    "go 库"
]
date = "2021-03-01"
categories = [
	"Go"
]
+++
##### github 地址
https://github.com/spf13/cast
##### 简短介绍
Cast是一个库，用于以一致且简单的方式在不同的go类型之间进行转换。
Cast提供了简单的函数，可以轻松地将数字转换为字符串、将接口转换为bool等。当可以进行明显的转换时，Cast会智能地执行此操作。它不会试图猜测您的意思，例如，当字符串是int（如“8”）的字符串表示形式时，您只能将其转换为int。Cast是为Hugo开发的，Hugo是一个使用YAML、TOML或JSON作为元数据的网站引擎。
##### 使用场景
在Go中处理动态数据时，通常需要将数据从一种类型转换为另一种类型。Cast不仅仅是使用类型断言（尽管它在可能的情况下使用类型断言）来提供一个非常简单和方便的库。
如果您使用接口来处理动态内容之类的事情，那么您需要一种将接口转换为给定类型的简单方法。这是你的图书馆。
如果您从YAML、TOML或JSON或其他缺少完整类型的格式中获取数据，那么Cast就是您的库。
##### 使用方式
Cast提供了一些To____ 方法。这些方法将始终返回所需的类型。如果提供的输入不会转换为该类型，则将返回该类型的0或nil值。
Cast还提供了与To___E 相同的方法。这些方法返回的结果与To____ 方法相同，另外还有一个错误，告诉您是否成功转换。使用这些方法，您可以区分输入何时与零值匹配，或者转换失败何时返回零值。

下面的例子仅仅是可用的示例。请检查代码是否完整。
###### toString
```go
cast.ToString("mayonegg")         // "mayonegg"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("one time")) // "one time"
cast.ToString(nil)                // ""

var foo interface{} = "one more time"
cast.ToString(foo)                // "one more time"
```
###### toInt
```go
cast.ToInt(8)                  // 8
cast.ToInt(8.31)               // 8
cast.ToInt("8")                // 8
cast.ToInt(true)               // 1
cast.ToInt(false)              // 0

var eight interface{} = 8
cast.ToInt(eight)              // 8
cast.ToInt(nil)                // 0
```

##### 内部实现
###### 反射reflect
通过反射包 拿到数据类型
```go
func indirect(a interface{}) interface{} {
	if a == nil {
		return nil
	}
	if t := reflect.TypeOf(a); t.Kind() != reflect.Ptr {
		// Avoid creating a reflect.Value if it's not a pointer.
		return a
	}
	v := reflect.ValueOf(a)
	for v.Kind() == reflect.Ptr && !v.IsNil() {
		v = v.Elem()
	}
	return v.Interface()
}

```
###### switch case
```go
func ToInt64E(i interface{}) (int64, error) {
	i = indirect(i)

	switch s := i.(type) {
	case int:
		return int64(s), nil
	case int64:
		return s, nil
	case int32:
		return int64(s), nil
	case int16:
		return int64(s), nil
	case int8:
		return int64(s), nil
	case uint:
		return int64(s), nil
	case uint64:
		return int64(s), nil
	case uint32:
		return int64(s), nil
	case uint16:
		return int64(s), nil
	case uint8:
		return int64(s), nil
	case float64:
		return int64(s), nil
	case float32:
		return int64(s), nil
	case string:
		v, err := strconv.ParseInt(s, 0, 0)
		if err == nil {
			return v, nil
		}
		return 0, fmt.Errorf("unable to cast %#v of type %T to int64", i, i)
	case bool:
		if s {
			return 1, nil
		}
		return 0, nil
	case nil:
		return 0, nil
	default:
		return 0, fmt.Errorf("unable to cast %#v of type %T to int64", i, i)
	}
}

```
