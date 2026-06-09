# Viper 推荐写法：只封装在 config.Load()

## 核心原则

不要在业务代码里到处写：

```go
viper.GetString("database.dsn")
viper.GetInt("server.port")
```

这种写法会导致：

- 配置读取逻辑散落在各处；
- handler / service / repository 直接依赖 Viper；
- 测试困难；
- 以后换配置库成本高。

更好的方式是：

```text
Viper 只出现在 internal/config 包
业务层只接收 Config 结构体
```

## 推荐目录

```text
.
├── config.yaml
├── main.go
└── internal
    └── config
        └── config.go
```

## 示例配置

```yaml
server:
  port: 8080

database:
  dsn: "host=localhost user=postgres password=postgres dbname=book sslmode=disable"

redis:
  addr: "localhost:6379"
```

## Config 结构体

```go
package config

import (
	"strings"

	"github.com/spf13/viper"
)

type Config struct {
	Server   ServerConfig   `mapstructure:"server"`
	Database DatabaseConfig `mapstructure:"database"`
	Redis    RedisConfig    `mapstructure:"redis"`
}

type ServerConfig struct {
	Port int `mapstructure:"port"`
}

type DatabaseConfig struct {
	DSN string `mapstructure:"dsn"`
}

type RedisConfig struct {
	Addr string `mapstructure:"addr"`
}

func Load() (*Config, error) {
	v := viper.New()

	v.SetConfigFile("config.yaml")
	v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
	v.AutomaticEnv()

	if err := v.ReadInConfig(); err != nil {
		return nil, err
	}

	var cfg Config
	if err := v.Unmarshal(&cfg); err != nil {
		return nil, err
	}

	return &cfg, nil
}
```

## main.go 中使用

```go
cfg, err := config.Load()
if err != nil {
	panic(err)
}

fmt.Println(cfg.Server.Port)
fmt.Println(cfg.Database.DSN)
```

## 这套写法的好处

- 项目里仍然使用主流 Viper；
- 但业务代码看不到 Viper；
- 后续替换为 Koanf / cleanenv 时，只需要改 `internal/config`；
- 配置结构清晰，适合测试和维护。

## 关键流程

```text
config.yaml
    ↓
v.ReadInConfig()
    ↓
Viper 内部保存配置
    ↓
var cfg Config
    ↓
v.Unmarshal(&cfg)
    ↓
得到 Go 结构体 cfg
    ↓
return &cfg, nil
```

## ReadInConfig 和 Unmarshal 的区别

### `ReadInConfig()`

作用：

```text
从配置文件读取内容到 Viper 内部
```

示例：

```go
if err := v.ReadInConfig(); err != nil {
	return nil, err
}
```

它只负责读取配置文件，不负责生成 Go 结构体。

### `Unmarshal(&cfg)`

作用：

```text
把 Viper 内部的配置映射到 Go 结构体
```

示例：

```go
var cfg Config
if err := v.Unmarshal(&cfg); err != nil {
	return nil, err
}
```

这里必须传 `&cfg`，因为 Viper 要修改 cfg 的内容。
