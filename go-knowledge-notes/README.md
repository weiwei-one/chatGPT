# Go 知识笔记

这个目录用于整理 Go 后端学习过程中的薄弱知识点。

当前主题：

1. Go 配置管理库选择：Viper / Koanf / cleanenv
2. Viper 的推荐封装方式：只在 `config.Load()` 内部使用
3. YAML 配置文件基础语法
4. Go 函数签名、返回值、`error`、`nil, err`
5. `ReadInConfig()`、`Unmarshal(&cfg)`、指针传参的含义

## 学习原则

- 先能看懂团队项目里的主流写法，再考虑个人偏好的简洁写法。
- 配置库不要污染业务层，业务层只依赖 `Config` 结构体。
- 看 Go 代码时，优先看函数签名，函数签名决定参数和返回值。

## 目录

- [01-config-library-choice.md](./docs/01-config-library-choice.md)：Go 配置库选择
- [02-viper-load-pattern.md](./docs/02-viper-load-pattern.md)：Viper 推荐封装方式
- [03-yaml-basics.md](./docs/03-yaml-basics.md)：YAML 基础语法
- [04-go-function-return-error.md](./docs/04-go-function-return-error.md)：Go 函数签名、返回值和错误处理
