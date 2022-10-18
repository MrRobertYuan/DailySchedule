[TOC]

# Rust 学习记录

比较了几个学习的教程，发现还是跟着 Rust 语言圣经的学习效率最高

https://course.rs/about-book.html

## Day0

### 下载并配置环境

下载 Rust：https://www.rust-lang.org/tools/install

配置 Rust 环境：https://www.bilibili.com/read/cv17841257

配置 VSCode 环境：https://www.rust-lang.org/tools/install 根据上一篇下载了 rust-analyzer 以及 Better TOML

RA 的使用方法：https://zhuanlan.zhihu.com/p/218098514

RA 的使用文档：https://rust-analyzer.github.io/manual.html#features

### 第一个 Rust 程序

Hello world

### 遗留的问题

Rust-analyzer 报错：找不到工作区

**解决方案 - day1：**

之所以找不到工作区是因为RA需要基于 cargo 生成的 TOML，而昨天的代码没有使用 cargo 来构建，所以无法找到工作区。

所以先使用 `cargo new hello-world` 命令创建 `hello-world` 文件夹，再使用 VSCode 打开这个文件夹就可以使用 RA 自带的很多功能了：

![image-20221018150440832](..\images\Hello-World-RA.png)



## Day1

先学一下 cargo 的基本使用方法：

官网英文教程：https://doc.rust-lang.org/rust-by-example/cargo.html

菜鸟中文教程：https://www.runoob.com/rust/cargo-tutorial.html

### Cargo

Cargo 是 Rust 的构建系统和包管理器，是用来管理项目的依赖的

#### 基本操作

```
cargo new		// 创建
cargo build		// 搭建
cargo run		// 运行
cargo check 	// 检查编译
```

可以加上 `--release` 来生成可发布的更高效的程序

#### 在 VSCode 中配置 Rust 工程

根据菜鸟教程上的方法配置 `tasks.json` 和 `launch.json` 两个文件。

就可以运行和调试了。

#### Cargo 生成的目录结构

```
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs								# 默认的库文件
│   ├── main.rs								# 默认的可执行文件
│   └── bin/								# 其他的可执行文件
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

#### Cargo.toml

在 Cargo.toml 中描述了项目的信息

```
[package]
name = "hello-world"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
clap = "2.27.1" # from crates.io
rand = { git = "https://github.com/rust-lang-nursery/rand" } # from online repo
bar = { path = "../bar" } # from a path in the local filesystem
```

Package 部分说的是 项目名，项目版本，项目作者 等项目的信息

Dependencies 部分描述的是项目需要满足的前置条件，可以是 crates.io（crate 可以是库文件也可以是可执行文件），github 上的项目，也可以是本地文件。

具体的使用可以查阅 Cargo Book：https://doc.rust-lang.org/cargo/reference/



### Hello-World+

![image-20221018170454668](..\images\Hello-World-Plus.png)

首先可以看到 RA 插件自带的类型推理，而且它不需要配置 .vscode 中的编译文件，就可以直接运行和调试，非常方便。

其次可以看到输出的字符串中可以使用各个地方的语言，这是因为Rust支持UTF-8来编码

再次可以看到 println 后的 ！，这是 宏 操作符。

同时可以看到输出的字符串中使用了 {} 来占位，它可以自动识别数据例子输出

同时可以看到 for 循环中不能直接对集合类型循环，需要变成迭代器才能循环。（for 可以隐式地把集合变成迭代器）