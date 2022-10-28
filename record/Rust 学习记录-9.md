# Rustling

首先需要根据 Readme 安装

## intro

熟悉一下操作，最方便的还是在 VS 里的终端运行，不然可能更新不同步

```
rustling watch
rustling verify
rustling run + name
rustling hint + name
rustling list
```



## Variable

非常基础的题目

1. 变量要有初始值
2. 常量要有类型
3. 可以同名覆盖变量



## Functions

也是非常基础的函数调用

1. 参数的类型
2. 返回的类型



## If

基本使用方法，`if, else if, else`



## generics

一些简单的类型转换，以及使用泛型来结构体定义

以及对于泛型的方法声明



## move_semantics

这章应该是关于所有权的一些题目

1. 注意Vec的可变性，才能实现`push`
2. 函数传递所有权，使用引用或者`clone`
3. 可以使用可变指针修改数组，并传递可变指针的所有权使得只有一个可变指针
4. 同时只能拥有一个可变指针，可变指针的生存域就是他的作用域
5. String 是可变的，有所有权的，&str是不可变的，有复制函数的



## Primitive_types

一些最基本的类型

1. bool 类型，只有 true 和 false
2. char，可以用 `is_alphabetic()` 或者 `is_numeric()`判断它是否是字符或者数字，不过`is_alphabetic(‘中’) = true`
3. 数组生成固定大小 `with_capacity`
4. 元组解构



## Structs

1. 一些基本类型：经典结构体，元组结构体，单元结构体
2. 一些方法的使用，`self` 的使用



## Enums

1. 枚举的定义
2. 枚举可以附加类型，这个类型可以直接写 `{x:i32,y:i32},(i32, i32)`这样



## Modules

1. 模块的定义
2. 模块的可见性
3. 模块引用使用 `{}`