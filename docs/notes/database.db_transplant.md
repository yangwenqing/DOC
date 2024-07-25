---
id: zukv8204827vaabovfirzk2
title: Db_transplant
desc: ''
updated: 1721890208717
created: 1721890191937
---
#loongarch
# **龙芯平台数据库移植指南**


* 
	1. **龙芯平台已移植的数据库**

**关系型数据库：**

PostgreSQL、MySQL、HighgoDB（瀚高）、VastBase（海量）、ShenTong（神通）、KingBase（金仓）、OceanBase（分布式）、DMDB（达梦）、GreatSQL（万里）、HungHuDB（鸿鹄）、SinoDB（星瑞格）

**非关系型数据库：**

**时序数据库：**

TDengine（涛思）、CTS（水脉）、TimescaleDB（pg扩展）、GoldenData（庚顿）、Prometheus

**图数据库：**

Ultipa（赢图）、Neo4j

**键值数据库**：

etcd、Redis、rocksdb

# **数据库移植**


* 
	1. **环境准备与尝试编译**

本次移植教程的针对对象为开源数据库

**2.1.1 必备组件准备**

对于数据库编译所需的其他组件，首先考虑从龙芯源内直接安装，服务器系统（centos系）使用yum，桌面端系统（debian系）使用apt，组件的包名在不同的系统上也有细微的差别。具体所需的组件可以从README文件中获取，亦或是前往官网查看开发者手册如何从源码编译安装数据库，当然也可以在编译报错后查看报错信息确认缺失的组件。

以postgresql-12.5为例：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080627.2189374.png)

但并不是所有的组件都一定能够在龙芯的官方源中都能够寻找到对应的包，有的数据库编译对相应组件包的版本有着严格的需求。

以oceanbase的编译为例，首先准备编译环境；

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080627.4320507.png)

由于oceanbase的编译对cmake的版本有要求，故而首先准备3.23.2版本的cmake

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080627.7065492.png)

oceanbase的分布式组件对于isa-l也有需求，因此也要手动编译isa-l

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080627.92344.png)

对于部分第三方库。其本身可能不支持龙芯平台，故而在使用这些库的时候，需要对其进行手动修改并编译。

**2.1.2 准备数据库源码**

首先在网络上拉取所需的数据库源码，使用wget、git或其他方法均可。

以postgresql-12.5为例：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080628.1406496.png)

对源码包进行解包处理。

以postgresql-12.5为例：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080628.3572528.png)

**2.1.3 编译安装**

对于不同语言编写的数据库管理系统，有着不同的编译安装方法。本文着重讨论关于CMAKE/MAKE组织的项目架构。以postgresql-12.5为例：![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080628.574227.png)

如果项目是以CMAKE进行组织的，前期需要利用CMAKE将CmakeList.txt转化为Makefile文件才能进行后续的编译工作。

额外需要说明的是如果在龙芯平台上编译使用go语言架构的项目，需要进行以下的操作。

go env -w GOPROXY=https://goproxy.cn,direct //中国区域的go源

# go env -w GOPROXY= http://goproxy.loongnix.cn:3000,direct //龙芯go源 

# go env -w GOSUMDB=off //如果设置龙芯官方源作为go源的话，需要关闭go的校验功能

# 若关闭校验和后仍出现校验失败，则可以删除go.sum文件，通过go mod tidy命令重新生成。若生成失败，可以尝试手动对go.sum中的校验和进行校正。


* 
	1. **问题定位**

对于编译过程中出现的问题，一般通过以下途径进行查看：


1. **configure阶段**

configure阶段出现的问题一般可以直接在命令行的界面看到错误提示，例如必要库缺失，常常反映为：

*Error：Package requirements （xxx） was not met；No module named xxx found.*

如果数据库在微架构层次不支持编译，也会configure阶段进行反馈，以postgresql-12.5的源码编译为例

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080628.7924988.png)

更确切的错误信息可以通过config.log进行查看。

如果项目使用Cmake进行编译，如果架构不支持，那么在编译过程也会出现架构相关的错误，以TDengine 2.6 版本的编译为例：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080629.0081806.png)

出现了无法识别的编译选项‘-msse4.2’,该选项实际为X86架构进行crc32校验时的选项，表明编译时机器没有走loongarch的编译路径，需要对源代码进行修改以添加loongarch支持。

对于上述错误，在文件的目录下利用’grep -rn ‘sse4.2’进行匹配，可以定位到错误出现的位置

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080629.2252855.png)


1. **make阶段**

make阶段的错误多为ld出现错误，如动态链接库缺失或者包内组件头文件缺失，亦或是编译时错误的使用了系统自带的库文件而非使用数据库指定版本的库文件。当然，也存在缺少loongarch架构支持的状况。make阶段的错误在命令行有直接输出，不过为了保存现场，一般将make过程的记录通过管道符重定向到新的文件中进行查看。


* 
	1. **问题解决**

**2.3.1 包含configure.guess和configure.sub的编译**

如果在编译过程中出现了诸如’can not guess build type;you must specify one’的错误，表明构建机的环境无法被识别。

对于这种错误，有两种解决方法。


1. 目前最新版的config.guess和config.sub中已经添加了loongrach的支持，能够在loongarch的平台上正确识别到构建机的环境。使用以下命令拉取新版的config.guess和config.sub.

curl -o config.guess "https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob\_plain;f=config.guess"

curl -o config.sub "https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob\_plain;f=config.sub"


1. 对已有的config.guess和config.sub进行修改，添加loongarch的架构支持。

修改config.guess

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080629.4366155.png)

修改config.sub

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080629.6507819.png)

**以上方法不仅适用于数据库的编译，对于其他软件包在龙芯平台上的编译同样适用。**

**2.3.2 其他简单的架构添加**

**以编译etcd为例**

首先从github中拉取etcd的源码

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080629.8527033.png)

尝试编译：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080630.052402.png)

出现了如下错误：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080630.2526367.png)

该错误表明ectd在构建时未能成功的获取到需要构建的目标架构，故而编译通过后在运行时会出现无法在未支持的的架构上运行的错误，需要对文件进行修改添加loongarch的支持**。**

通过查看build.sh的源码发现问题出现在GOARCH未指定上，所以对GOARCH的设置位置进行搜索：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080630.4623156.png)

由于go官方支持loongarch架构，架构名称为loong64，故而对etcd.go文件进行如下修改：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080630.6702745.png)

再次编译并运行，成功启动

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080630.887048.png)

**以编译TDengine为例：**

首先拉取TDengine的源码：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080631.1047385.png)

运行build.sh:

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080631.3125205.png)

该错误表明缺少loongarch支持，编译时进入了x86的分支。

对cmake文件夹内的文件进行阅读，添加loongarch的宏定义：

修改platform.inc

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080631.5213397.png)

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080631.7298074.png)

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080631.9386673.png)

修改define.inc

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080632.1475375.png)

重新编译，编译报错，报错位置来自./src/util/src/tcrc32.c，修改该文件：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080632.3645303.png)

在所有的\_TD\_ARM\_和\_TD\_MIPS\_宏定义后追加loongarch的定义，共计4处。

再次编译，jmelloc编译出错，原因在于config.guess和config.sub不支持loongarch，根据上文提供的解决方法拉取新版的config文件，可继续向下编译。

再次编译，在编译时报错，错误信息如下：

![error1](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080632.573003.png)

从报错信息可以明显得知，在构建包的过程中缺少了必要的头文件，从前面的引入信息可知引入的os.h和osFile.h,阅读文件源码，发现os.h中引入的os.inc.h的源码中缺少了loongarch的定义，作如下修改

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080632.7905126.png)

重新编译，出现如下错误信息：

![error2](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080633.0068524.png)

标准输出中提示/src/os/src/details/osSysinfo.c中出现了故障，阅读源码，发现该部分代码用于输出提示信息，对程序主体功能无影响，故而作如下更改，跳过输出：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080633.248864.png)

再次编译，编译通过，运行/debug/bin目录下的taosd可执行文件，启动成功，表明编译完成。

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080633.46565.png)

**2.3.3 功能源码修改**

当程序源码的部分功能模块不支持loonagrch架构时需要手动对该部分源码进行添加，相较于前面所做出的修改，这一部分改动耗费的时间更多，需要改动人对程序代码和龙芯架构都有不浅的认知，甚至有时loongarch架构暂时缺少对该功能的支持，所以在某些极端情况下，可以利用效率较低的备选方案代替首选方案。

**以postgresql-12.5为例**：

在前文作出config.guess和config.sub的修改之后，继续对postgresql进行编译，标准输出中提示如下错误：

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080633.683635.png)

postgresql使用的spinlock暂时不支持loongarch架构，如果想要继续编译，有以下两个选择


1. 禁用自旋锁，改用信号量在软件层面上代替自旋锁，但数据库性能会有所下降。

方法：configure时使用--disable-spinlock选项 

使用 ./configure --disable-spinlocks 命令，让pg使用信号量实现的自旋锁。当然这种自选锁性能会比较差。

pg 中硬件无关的自旋锁代码在 spin.h, spin.c中，使用的linux的信号量(semaphores)实现的。


1. 修改postgresql源码，为loongarch添加自旋锁支持。

硬件相关的自旋锁部分的代码在 s\_lock.h, s\_lock.c 里面定义,为了支持 loongarch 架构，需要在 src/include/storage/s\_lock.h 里面加上loongarch 相关架构的实现。一般加在 #endif arm 的后面即可。


* 
	1. 手写汇编实现自旋锁

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080633.9004703.png)


* 
	1. gcc自旋锁

![](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080634.116616.png)

完成该修改后，编译通过。

从15.1, 14.6, 13.9, 12.13, 11.18, and 10.23 等版本开始，pg 官方支持 loongarch64 使用 gcc 编译器提供的自旋锁, 不需要手动修改自旋相关代码

<https://github.com/postgres/postgres/commit/1c72d82c25849307416db140007674c674bdee10>

**以opengauss的编译为例：**

在编译 opengauss 的时候， arm 上面有128位原子指令实现下面的函数

__excl_compare_and_swap_u128

__lse_compare_and_swap_u128

龙芯上暂时没有128位的原子操作。下方是两个函数的arm汇编代码，其中ldxp、stlxp、caspal均是128位原子指令操作，无法利用loongarch汇编进行等价替换。

static inline uint128_u __excl_compare_and_swap_u128(volatile uint128_u *ptr, uint128_u oldval, uint128_u newval) 

 { 

 uint64_t tmp, ret;

 uint128_u old; 

asm volatile("1: ldxp %0, %1, %4n" 

 " eor %2, %0, %5n" 

 " eor %3, %1, %6n"

 " orr %2, %3, %2n" 

 " cbnz %2, 2fn" 

 "stlxp %w2, %7, %8, %4n" 

 " cbnz %w2, 1bn" 

 " b 3fn" 

 "2:" 

 " stlxp %w2, %0, %1, %4n" 

 " cbnz %w2, 1bn" 

 "3:" 

 " dmb ishn"

: "=&r"(old.u64[0]), "=&r"(old.u64[1]), "=&r"(ret), "=&r"(tmp), 

 "+Q"(ptr->u128) 

:"r"(oldval.u64[0]), "r"(oldval.u64[1]), "r"(newval.u64[0]), "r"(newval.u64[1]) 

 : "memory"); 

 return old; 

 } 

 

static inline uint128_u __lse_compare_and_swap_u128(volatile uint128_u *ptr, uint128_u oldval, uint128_u newval) 

 { 

 uint128_u old; 

 register unsigned long x0 asm ("x0") = oldval.u64[0]; 

 register unsigned long x1 asm ("x1") = oldval.u64[1]; 

 register unsigned long x2 asm ("x2") = newval.u64[0]; 

 register unsigned long x3 asm ("x3") = newval.u64[1]; 

 asmvolatile(".arch_extension lsen" 

 " caspal %[old_low], %[old_high], %[new_low], %[new_high], %[v]n" 

 :[old_low]"+&r"(x0),[old_high]"+&r"(x1), [v] "+Q" (*(ptr)) 

 :[new_low]"r"(x2),[new_high]"r"(x3) 

 :"x30","memory"); 

 old.u64[0]=x0; 

 old.u64[1] = x1; 

 return old; 

 } 

 

解决思路，

1. 去掉 128位原子指令相关的代码, 通过阅读openguass源码，128 位原子指令是为了优化 wal 日志写入, 可以去掉这个优化，回退到之前使用自旋锁的实现逻辑。（缺点：工程量大，要熟悉opengauss代码，要删除大量代码）

2. 利用 libatomic 库提供的 128 cas 功能编译。(需要测试正确性)

**2.3.4 其他解决方法**

**1. 运行缺少组件**

以图数据库neo4j在龙芯上的编译运行为例：

首先拉取数据库

https://neo4j.com/download-center/#community

选择 Linux / Mac Executable 下载 tar 包

neo4j-community-4.4.16-unix.tar.gz

tar xvzf neo4j-community-4.4.16-unix.tar.gz

cd neo4j-community-4.4.16

启动neo4j：

./bin/neo4j console

此时会出现错误：

Exception in thread "main" java.lang.UnsatisfiedLinkError: Native library (com/sun/jna/linux-loongarch64/libjnidispatch.so) not found in resource path

根据报错信息，确认jna包出现故障，从龙芯官方拉取最新的jna包并对项目内的同名包进行替换，再次启动启动成功

wget <https://github.com/loongson/jna/releases/download/5.12.1-loongarch64/jna-5.12.1.jar>

cp jna-5.12.1.jar neo4j-community-4.4.16/lib

**2. 重写编译脚本**

**以oceanbase的编译为例：**

对官方的build.sh进行重构，使其能够在龙芯平台上进行正常编译。具体修改见 [git@github.com:loongarch64/oceanbase的generic分支中的build.sh。](mailto:git@github.com:loongarch64/oceanbase的generic分支中的build.sh。)


* 
	1.  **数据库移植方法总结**

**流程图如下**

**![未命名文件](http://223.76.216.188:50201/uploads/images/gallery/media/202303//1680080634.3411124.jpeg)**


1. **参考资料**
2. **loonagrch ABI 规范**

**<https://loongson.github.io/LoongArch-Documentation/LoongArch-ELF-ABI-CN.html>**


1. **龙芯架构文档（包含指令集、处理器、桥片用户手册，工具链约定等）**

**<https://loongson.github.io/LoongArch-Documentation/README-CN.html>**


1. **龙芯开源社区**

**<http://www.loongnix.cn/index.php/%E9%A6%96%E9%A1%B5>**

**推荐书目：**


1. **《汇编语言编程基础基于Loongarch》**
2. **《龙芯应用开发标准教程》**

**也可通过loongdb@loongson.cn和我们进行交流，欢迎来信！**