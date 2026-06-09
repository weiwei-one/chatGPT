# Go 函数签名、返回值、error 与 nil, err

## 1. 不要把返回值当黑魔法

Go 里一个函数返回几个值，由函数签名决定。

函数签名格式：

```go
func 函数名(参数列表) 返回值列表
```

例如：

```go
func A() error
```

表示返回一个值，类型是 `error`。

调用：

```go
err := A()
```

再如：

```go
func B() (string, error)
```

表示返回两个值：

1. string
2. error

调用：

```go
name, err := B()
```

## 2. 为什么 ReadInConfig 只接收一个 err

Viper 的 `ReadInConfig` 签名类似：

```go
func (v *Viper) ReadInConfig() error
```

它只返回一个 `error`。

所以调用时写：

```go
err := v.ReadInConfig()
```

或者常见写法：

```go
if err := v.ReadInConfig(); err != nil {
	return nil, err
}
```

不能写：

```go
cfg, err := v.ReadInConfig()
```

因为它没有返回 cfg。

## 3. if err := xxx(); err != nil 是什么语法

这段：

```go
if err := v.ReadInConfig(); err != nil {
	return nil, err
}
```

等价于：

```go
err := v.ReadInConfig()

if err != nil {
	return nil, err
}
```

区别是：

```go
if err := ...; err != nil
```

里面定义的 `err` 只在这个 if 语句范围内有效。

## 4. 为什么有时候 return nil, err

看外层函数签名：

```go
func Load() (*Config, error) {
	// ...
}
```

这个函数声明自己返回两个值：

1. `*Config`
2. `error`

所以函数内部 return 时必须返回两个值。

失败时：

```go
return nil, err
```

含义：

```text
第一个返回值：没有可用的 Config，所以返回 nil
第二个返回值：具体错误原因 err
```

成功时：

```go
return &cfg, nil
```

含义：

```text
第一个返回值：cfg 的地址
第二个返回值：没有错误，所以返回 nil
```

## 5. nil 是什么

`nil` 可以理解为 Go 中的“空引用”。

常见可以为 nil 的类型：

- pointer：指针
- slice
- map
- channel
- function
- interface

例如：

```go
var cfg *Config = nil
```

表示当前没有指向任何 Config 对象。

## 6. 为什么 return &cfg, nil

代码：

```go
var cfg Config
return &cfg, nil
```

`cfg` 是一个 Config 变量。

`&cfg` 表示取它的地址，类型是：

```go
*Config
```

因为函数签名是：

```go
func Load() (*Config, error)
```

第一个返回值必须是 `*Config`，所以返回 `&cfg`。

## 7. 为什么 Unmarshal 要传 &cfg

```go
var cfg Config
err := v.Unmarshal(&cfg)
```

这里传 `&cfg`，是因为 `Unmarshal` 要把配置写入 cfg。

如果只传：

```go
v.Unmarshal(cfg)
```

相当于传了一个副本，函数无法修改外面的 cfg。

所以需要传指针：

```go
v.Unmarshal(&cfg)
```

## 8. 看 Go 代码的核心习惯

看到：

```go
x := Foo()
```

马上去看：

```go
func Foo() ?
```

看到：

```go
x, err := Foo()
```

马上去看：

```go
func Foo() (?, error)
```

看到：

```go
return nil, err
```

马上看当前函数签名：

```go
func 当前函数() (*Something, error)
```

## 9. 常见模式

### 只返回错误

```go
func Save() error {
	if err := doSave(); err != nil {
		return err
	}
	return nil
}
```

### 返回结果 + 错误

```go
func FindUser(id int64) (*User, error) {
	user, err := queryUser(id)
	if err != nil {
		return nil, err
	}
	return user, nil
}
```

### 配置加载函数

```go
func Load() (*Config, error) {
	var cfg Config

	if err := readFile(); err != nil {
		return nil, err
	}

	if err := parse(&cfg); err != nil {
		return nil, err
	}

	return &cfg, nil
}
```

## 10. 记忆重点

```text
函数签名决定返回几个值
error 通常放最后一个返回值
失败时通常 return nil, err
成功时通常 return result, nil
&cfg 表示把 cfg 的地址传出去
Unmarshal 要修改变量，所以传指针
```
