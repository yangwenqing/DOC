---
id: hcfprgfngvrk1lnslwj123e
title: '2023-03-22 tsbs解析'
desc: 'tsbs source code'
updated: 1679454694346
created: 1679448630951
traitIds:
  - journalNote
---
<!-- TOC -->

- [**tsbs_generate_data**](#tsbs_generate_data)
    - [**main.go**](#maingo)

<!-- /TOC -->
# **tsbs_generate_data**
## **main.go**
```go
var (
	profileFile string
	dg          = &inputs.DataGenerator{}
	config      = &common.DataGeneratorConfig{}
)
```
1. 通过从`pkg/data/usecase/common`包中的`generator.go`引入`DateGeneratorConfig`结构和相应的方法，用于生成测试所需的初始化测试参数。 #DataGenatatorConfig
2. 通过从`pkg/internal/inputs`包的`generator_data.go`引入`DataGenerator`结构和相应的方法，完成创建测试所需要的数据。 #DataGenerator
