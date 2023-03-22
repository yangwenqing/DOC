---
id: dkzfawgvrieibhob8gnhpd9
title: Constants
desc: 'what need to modify'
updated: 1679454293255
created: 1679453927251
---
** 该文件指定了tsbs测试支持的数据库列表，如果需要添加新的数据库，需要仿照上述格式进行数据库的名录增加 **
```go
const (
	FormatCassandra       = "cassandra"
	FormatClickhouse      = "clickhouse"
	FormatInflux          = "influx"
	FormatMongo           = "mongo"
	FormatSiriDB          = "siridb"
	FormatTimescaleDB     = "timescaledb"
	FormatAkumuli         = "akumuli"
	FormatCrateDB         = "cratedb"
	FormatPrometheus      = "prometheus"
	FormatVictoriaMetrics = "victoriametrics"
	FormatTimestream      = "timestream"
	FormatQuestDB         = "questdb"
	FormatTDengine        = "TDengine"
	FormatTDengineSML     = "TDengineSML"
)

func SupportedFormats() []string {
	return []string{
		FormatCassandra,
		FormatClickhouse,
		FormatInflux,
		FormatMongo,
		FormatSiriDB,
		FormatTimescaleDB,
		FormatAkumuli,
		FormatCrateDB,
		FormatPrometheus,
		FormatVictoriaMetrics,
		FormatTimestream,
		FormatQuestDB,
		FormatTDengine,
		FormatTDengineSML,
	}
}
```