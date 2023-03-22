---
id: v47m0pc77q55kg02andlwhk
title: DataGenerator
desc: ''
updated: 1679477774152
created: 1679454694103
---
# **结构**
```go
type DataGenerator struct {
	Out io.Writer  //数据的写端口
	config *common.DataGeneratorConfig  //数据生成的配置参数
	bufOut *bufio.Writer   //缓冲区写端口
}
```
# **方法与接口**
+ `@func (g *DataGenerator) init(config common.GeneratorConfig) error`
    + 作用：检验Out、config、bufOut是否成功初始化，并将out指定为os.out
+ `@func (g *DataGenerator) Generate(config common.GeneratorConfig, target targets.ImplementedTarget) error`
    + 作用：首先通过`@init`校验`generator`的可用性，然后从`pkg/data/usecases/common`中的`usecase.go`引入`@GetSimulatorConfig`方法完成模拟器参数的载入，通过从`simulator.go`引入`@NewSimulator`方法完成生成数据的模拟器的创建，然后调用`@getSerializer`获取序列化信息。本方法在项目中未被调用过。
    + #usecase
    + #simulator
    + #target
+ `@func (g *DataGenerator) CreateSimulator(config *common.DataGeneratorConfig) (common.Simulator, error)`
    + 作用：首先通过`@init`校验`generator`的可用性，然后从`pkg/data/usecases/common`中的`usecase.go`引入`@GetSimulatorConfig`方法完成模拟器参数的载入，通过从`simulator.go`引入`@NewSimulator`方法完成生成数据的模拟器的创建，
+ `@func (g *DataGenerator) runSimulator(sim common.Simulator, serializer serialize.PointSerializer, dgc *common.DataGeneratorConfig) error`
    + 作用：通过已经创建的simulator来写入测点数据，测点数据通过`/pkg/data`中的`point.go`进行生成。
    + #point
+ `@func (g *DataGenerator) getSerializer(sim common.Simulator, target targets.ImplementedTarget) (serialize.PointSerializer, error)`
    + 作用：将生成的数据序列化用于写入，对于CrateDB与Clickhouse，代码fallthrough，对于timescaledb，则需要调用`@wirteheader`对headers进行特殊处理。
+ `@func (g *DataGenerator) writeHeader(headers *common.GeneratedDataHeaders)`
    + 作用： 将生成数据的headers进行分割，并依据FieldKeys对keys进行升序排列，同时将tagKeys及对应的tagType写入缓冲区。