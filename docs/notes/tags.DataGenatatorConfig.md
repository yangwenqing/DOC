---
id: 6kzvclh0id8lj2gnsfildez
title: DataGenatatorConfig
desc: ''
updated: 1679453927536
created: 1679452572680
---
# **结构**
## DataGeneratorConfig
```go
type DataGeneratorConfig struct {
	BaseConfig            `yaml:"base"`
	Limit                 uint64        `yaml:"max-data-points" mapstructure:"max-data-points"`
	InitialScale          uint64        `yaml:"initial-scale" mapstructure:"initial-scale" `
	LogInterval           time.Duration `yaml:"log-interval" mapstructure:"log-interval"`
	InterleavedGroupID    uint          `yaml:"interleaved-generation-group-id" mapstructure:"interleaved-generation-group-id"`
	InterleavedNumGroups  uint          `yaml:"interleaved-generation-groups" mapstructure:"interleaved-generation-groups"`
	MaxMetricCountPerHost uint64        `yaml:"max-metric-count" mapstructure:"max-metric-count"`
}
```
## BaseConfig
```go
type BaseConfig struct {
	Format string `yaml:"format,omitempty" mapstructure:"format,omitempty"`
	Use    string `yaml:"use-case" mapstructure:"use-case"`

	Scale uint64

	TimeStart string `yaml:"timestamp-start" mapstructure:"timestamp-start"`
	TimeEnd   string `yaml:"timestamp-end" mapstructure:"timestamp-end"`

	Seed  int64
	Debug int    `yaml:"debug,omitempty" mapstructure:"debug,omitempty"`
	File  string `yaml:"file,omitempty" mapstructure:"file,omitempty"`
}
```
# **作用**
从`BaseConfig`继承所有的通用配置参数并添加几个数据生成所特有的参数配置。
## 接口
`BaseConfig`和`DataGeneratorConfig`均有两个接口：
1. `@AddToFlagSet(fs *pflag.FlagSet)` 负责将参数配置传入到pflag参数集中
2. `Validate() error` 负责校验参数输入中的错误

* 在校验时使用了`targets/constants`包中的`constants.go`，需要检验是否支持该类型的数据库。
#Constants