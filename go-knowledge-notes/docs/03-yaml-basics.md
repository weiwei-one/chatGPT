# YAML 配置文件基础语法

## 1. key: value 表示键值对

```yaml
name: zhangsan
age: 18
```

等价于：

```json
{
  "name": "zhangsan",
  "age": 18
}
```

注意冒号后面通常要有空格。

正确：

```yaml
name: zhangsan
```

不推荐：

```yaml
name:zhangsan
```

## 2. 缩进表示层级

```yaml
server:
  port: 8080
```

表示：

```text
server.port = 8080
```

等价于 JSON：

```json
{
  "server": {
    "port": 8080
  }
}
```

## 3. 缩进用空格，不用 Tab

推荐两个空格：

```yaml
server:
  port: 8080
```

不要用 Tab：

```yaml
server:
	port: 8080
```

## 4. 同级字段要对齐

正确：

```yaml
server:
  port: 8080
  mode: debug
```

错误：

```yaml
server:
  port: 8080
    mode: debug
```

## 5. 字符串可以加引号

简单字符串可以不加：

```yaml
mode: debug
```

复杂字符串建议加引号：

```yaml
dsn: "host=localhost user=postgres password=postgres dbname=book sslmode=disable"
```

数据库连接字符串中有空格、等号、特殊字符，加引号更稳。

## 6. 数字和布尔值会自动识别

```yaml
server:
  port: 8080
  debug: true
```

对应 Go：

```go
type ServerConfig struct {
	Port  int  `mapstructure:"port"`
	Debug bool `mapstructure:"debug"`
}
```

## 7. 列表用 -

```yaml
cors:
  allow_origins:
    - "http://localhost:5173"
    - "https://example.com"
```

对应 Go：

```go
type CORSConfig struct {
	AllowOrigins []string `mapstructure:"allow_origins"`
}
```

## 8. 注释用 #

```yaml
server:
  # 服务监听端口
  port: 8080
```

## 9. 和 Go struct 的对应关系

YAML：

```yaml
server:
  port: 8080
  mode: debug

database:
  dsn: "host=localhost user=postgres password=postgres dbname=book sslmode=disable"
```

Go：

```go
type Config struct {
	Server   ServerConfig   `mapstructure:"server"`
	Database DatabaseConfig `mapstructure:"database"`
}

type ServerConfig struct {
	Port int    `mapstructure:"port"`
	Mode string `mapstructure:"mode"`
}

type DatabaseConfig struct {
	DSN string `mapstructure:"dsn"`
}
```

对应关系：

```text
server          -> Config.Server
server.port     -> Config.Server.Port
server.mode     -> Config.Server.Mode
database        -> Config.Database
database.dsn    -> Config.Database.DSN
```

## 记忆重点

```text
冒号表示 key-value
缩进表示层级
同级必须对齐
复杂字符串建议加引号
不要用 Tab
```
