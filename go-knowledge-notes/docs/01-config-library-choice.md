# Go 配置库选择：Viper、Koanf、cleanenv

## 结论

面向就业，优先掌握 **Viper**。

原因不是 Viper 写法最优雅，而是：

- 国内 Go 后端项目里 Viper 更常见；
- Gin / Gorm / Zap / JWT 这类模板经常和 Viper 搭配；
- 面试和接手团队代码时，遇到 Viper 的概率更高；
- AI 生成 Go 后端模板时，也更容易默认使用 Viper。

但是，个人项目里可以了解 Koanf 或 cleanenv。

## 对比

| 维度 | Viper | Koanf | cleanenv |
|---|---|---|---|
| 国内常见度 | 高 | 中低 | 中低 |
| 就业友好度 | 高 | 一般 | 一般 |
| 写法简洁度 | 一般 | 较好 | 很好 |
| 功能完整度 | 高 | 较高 | 中 |
| 初学者理解成本 | 中等 | 中等 | 低 |
| 适合简历项目 | 推荐 | 可以 | 可以，但要会解释 |

## 实用选择

当前阶段建议：

```text
简历项目：Viper
自己练手小项目：cleanenv / Koanf
只使用环境变量的服务：caarlos0/env
进入公司后：按团队规范
```

## 面试中可以这样解释

如果项目使用 Viper：

```text
我使用 Viper 是因为它在 Go 后端项目里比较常见，团队接受度高。
但我不会在业务代码里直接调用 viper.GetXXX，而是统一封装在 config.Load() 中，
启动时解析到 Config 结构体，后续通过依赖传递使用。
```

如果项目使用 Koanf / cleanenv：

```text
我知道 Viper 是 Go 项目里更常见的配置库。
但这个项目配置需求比较简单，只需要 YAML + 环境变量覆盖，
所以选择了更轻的配置方案。核心设计仍然是统一解析到 Config struct。
如果团队规范是 Viper，可以平滑切换。
```
