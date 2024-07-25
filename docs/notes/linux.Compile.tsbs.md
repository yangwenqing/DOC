---
id: otm12m6ckxvq9n50g1o6y0h
title: tsbs
desc: ''
updated: 1721889263994
created: 1721889198665
---
#loongarch
#golang
## 准备工作
### golang切换国内镜像
```shell
go env -w GOPROXY=https://goproxy.cn,direct #换源
```
### tsbs下载编译
```shell
git clone https://github.com/timescale/tsbs.git
cd ./tsbs
make
```
## 问题解决
[![](http://223.76.216.188:50201/uploads/images/gallery/2024-07/scaled-1680-/image-1721875149138.png)](http://223.76.216.188:50201/uploads/images/gallery/2024-07/image-1721875149138.png)

**问题在于tsbs使用的sys的unix包中不存在loongarch的架构，需要更新最新的sys包**
```shell
go install go install golang.org/x/sys@latest
go mod edit -replace=golang.org/x/sys=golang.org/x/sys@v0.22.0
go clean -modcache
go mod tidy
```
[![](http://223.76.216.188:50201/uploads/images/gallery/2024-07/scaled-1680-/image-1721875491688.png)](http://223.76.216.188:50201/uploads/images/gallery/2024-07/image-1721875491688.png)

**同上，更新go-sysconf**
```shell
go get -u github.com/tklauser/go-sysconf
```