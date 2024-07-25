---
id: 7w8scl7r7x61xdpnvcszujo
title: Blender
desc: ''
updated: 1721887313722
created: 1721887313722
---

#loongarch
# deb安装
**麒麟Blender版本为`2.8.2`**
**统信Blender版本为`2.7.9`（攀升反应Blender版本过低）**

剑锋建议将将麒麟Blender移植到统信,但是`libvpx`统信为`libvpx5`，缺少`libvpx6`
**[麒麟gotdot3](https://archive.kylinos.cn/kylin/KYLIN-ALL/pool/main/g/godot/ "麒麟gotdot3")**
**[麒麟Blender](https://archive.kylinos.cn/kylin/KYLIN-ALL/pool/main/b/blender/ "麒麟Blender")**
1. 从[github上拉取libvpx源码](https://github.com/webmproject/libvpx/tree/main "github上拉取libvpx源码")编译，使用`--enable-shared`生成动态库，V1.9.0生成`libvpx6`，但安装包无法检测到该依赖。
2. 使用[麒麟库](https://archive.kylinos.cn/kylin/KYLIN-ALL/pool/main/libv/libvpx/ "麒麟库")中的libvpx1.8.2，成功安装gotdot3和Blender-data，但是安装Blender时出现依赖冲突，冲突对象主要为libboost等核心库的版本过低，搁置该方案。

# Blender源码编译
1. **拉取Blender源码**
[github拉取Blender的源码](https://github.com/blender/blender?tab=readme-ov-file "github拉取Blender的源码")
2. **新建编译目录**
进入源码目录，`mkdir -p build`新建编译文件夹，然后进入编译文件夹。
3. **编译安装git-lfs**
尝试编译Blender
```
../build_files/build_environment/install_linux_packages.py
```
发现`git-lfs`未找到，[从github拉取git-lfs的源码](https://github.com/git-lfs/git-lfs "从github拉取git-lfs的源码")，使用如下命令尝试编译
```
sudo apt install golang
cd ./git-lfs
mkdir -p ./build
make
make install
```
`golang`在uos源里只有1.15版本，`git-lfs`需求大于1.17,从龙芯源上下载golang1.19
```
wget http://ftp.loongnix.cn/toolchain/golang/go-1.19/abi1.0/go1.19.11.linux-loong64.tar.gz
```
详情见[安装说明](http://docs.loongnix.cn/golang/install.html "安装说明")
然后执行[龙芯go源换源教程](http://docs.loongnix.cn/golang/goproxy.html "龙芯go源换源教程")后重新编译。编译成功后将`/blender/build_files/build_environment/install_linux_packages.py`的对应检测注释。

4. **替换libegl-dev**
再次编译Blender，发现缺少`libegl-dev`，将`install_linux_packages.py`进行如图修改
![](http://223.76.216.188:50201/uploads/images/gallery/media/202403/2024-03-27_165346_6825720.24254366535613103.png)

5. **编译安装libdecor-dev**
再次编译，缺少libdecor-dev，暂时注释跳过。

6. **编译安装libpystring**
```
git clone https://github.com/imageworks/pystring.git
sudo apt install libtool libtool-bin
make
make intsall
```
安装完成后将依赖文件中对libpystring的检测注释掉。

7. **编译安装libshaderc**
+ 再次编译，缺少libshaderc。
```
git clone https://github.com/google/shaderc.git
cd $SOURCE_DIR
./utils/git-sync-deps
mkdir -p ./build
cd ./build
cmake -GNinja -DCMAKE_BUILD_TYPE={Debug|Release|RelWithDebInfo} $SOURCE_DIR
ninja
```
`libshaderc`要求`cmake >= 3.17`，uos的版本为3.13。
+ 编译cmake高版本
```
git clone https://github.com/Kitware/CMake.git
cd $SOURCE_DIR
mkdir -p ./build
cd ./build
../bootstrap
make -j 4
sudo make install
```
+ 替换cmake
```
# 卸载低版本cmake
sudo apt remove cmake
# 建立链接
sudo update-alternatives --install /usr/bin/cmake cmake /usr/local/bin/cmake 0
# 查看cmake版本，显示3.29则正常
cmake --version
```
+ 解决编译问题
![](http://223.76.216.188:50201/uploads/images/gallery/media/202403/2024-03-27_170822_4726560.9526181805878313.png)