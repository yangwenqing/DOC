---
id: e6on2py35o5rtrh74cv07in
title: rust_study
desc: ''
updated: 1721887173247
created: 1721887173247
---

- [**rust入门**](#rust入门)
  - [**New a rust program**](#new-a-rust-program)
  - [**镜像源配置**](#镜像源配置)
      - [**字符**](#字符)
# **rust入门**
## **New a rust program**
-   `cargo new project_name` *新建一个rust项目*
-   `cargo run`         *编译并运行rust项目,包括以下两个步骤*
    + `cargo build` *编译rust文件*
    + `./target/debug/project_exe` *以debug模式运行可执行程序（编译快，运行慢）*
    + 如果需要高性能运行rust项目 在上述的命令后添加 `--release`
-   `cargo check`    *不编译运行项目，对代码进行快速检查*

## **镜像源配置**

- `$HOME/.cargo/config.toml` *如果不存在就创建一个*  
   **对rust的官方镜像源进行替代**
   ```
   [source.crates-io]  
   replace-with = 'ustc'
   [source.ustc]
   registry = "git://mirrors.ustc.edu.cn/crates.io-index"
   ```
   or

   ```
   [source.crates-io]
   replace-with = 'tuna'
   [source.tuna】
   registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
    ```
## **rust编程**
### **变量**
+ `let`绑定的变量是不可变的，如果需要变量可变，用`let mut variables`进行修改
+ 可以利用`:i32`类似的格式对变量进行类型定义  
    - `let a:i32 = 10;`
    - `let a = 10i32;`
    - `let a = 10_i32;`
+ 如果一个变量只是声明而未使用，利用下划线作为变量开头
+ **变量遮蔽**：利用`let`对变量重新绑定，后声明的变量可以对前面的变量进行遮蔽（*作用域需一致，否则后声明的变量仅在作用域内生效,如下代码段*）。
   ```
   let x = 5;
   {
      let x ='a';
   }
   ```
### **数值类型**
#### **整数**
|长度|有符号类型|无符号类型|
|:---:|:---:|:---:|
|8 位|i8|u8 | 
|16 位|i16|u16|  
|32 位|i32|u32| 
|64 位|i64|u64|  
|128 位|i128|u128|  
|视架构而定|isize|usize|
#### **浮点数**
+ 单精度浮点 f32  
+ 双精度浮点 f64
  -  <font color=#dd0000>避免在浮点数上测试相等性</font><br/>
  -  当结果在数学上可能存在未定义时，需要格外的小心
####  **位运算**
|运算符|说明|
|:---:|:---:|
|& 位与|相同位置均为1时则为1，否则为0|  
|\| 位或|相同位置只要有1时则为1，否则为0|  
|^ 异或|相同位置不相同则为1，相同则为0|  
|! 位非|把位中的0和1相互取反，即0置为1，1置为0|  
|<< 左移|所有位向左移动指定位数，右位补零|  
|>> 右移|所有位向右移动指定位数，左位补零|
#### **序列**
+ 序列中仅支持字符和数字（*可连续且可判断是否为空*）
  ```rust
  for i in range 1..5 {
    println!("{}",i);
  }
  for i in range 1..=5 {
    println!("{}",i);
  }
  for i in 'a'..'z' {
    println!("{}",i);
  }
  ```
#### **字符**
+ rust的字符支持Unicode编码，单个字符占用4个字节。
+ 

