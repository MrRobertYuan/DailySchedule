# 类型转换

## as 转换

`as + 类型名` 就可以转换

比较特别的转换有内存指针的转换：

```
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize;
let second_address = first_address + 4;
let p2 = second_address as *mut i32
```

这里先把 `p1` 转成整数，然后+4，然后转回指针。



还要注意转换时候，从大的类型转成小的类型会容易发生错误。

同时转换不具有传递性，如果`a` 可以转换成 `b` 类型，`b` 类型可以转换成 `c` 类型，并不能说明`a`可以直接转成`c`类型



下面的语句可以阻止溢出的报错，但是溢出还是会发生，和C一样是取模。浮点数不同，如果浮点数超过了范围，就会直接变成最大值或者最小值

```rust
#[allow(overflowing_literals)]

    assert_eq!(300.1_f32 as u8, 255);
    assert_eq!(-100.1_f32 as u8, 0);
```



## TryInto 转换

使用 `try_into()` 转换必须要引入对应的特征：
```
use std::convert::TryInto
```



`try_into()` 会根据等式左边尝试进行转换，并返回一个 `Result`，也就是一个带 Ok 和 Err 的枚举变量

```
let b: i16 = 1500;
let b_: u8 = match b.try_into(){
	Ok(b1) => b1,
	Err(e) => {
		0
	}
}
```



和 From / Into 一样，想要实现 TryInto 可以通过给对应的结构体实现 `TryFrom` 特征来实现

```
impl TryFrom<T> for Struct{
	type Error = ();
	
	fn try_from() -> Result<Self, Self::Error>{
	
	}
}
```





## 通用类型转换

### 点操作符

之前学过点操作符会做一些帮你 加引用、解引用的方法，下面是它的判断步骤

1. 如果可以直接调用，就是正好符合类型，**值方法调用**
2. 如果加一个引用可以使用，**引用方法调用**
3. **解引用方法**
4. 如果T是一个定长类型，会尝试将 T 转为**不定长类型**，例如从 [i32;2] 转为 [i32] （注意，数组无法通过索引访问，只有数组切片才可以，也就是[i32;2]不能使用.index()但是[i32]可以）



一个神奇的例子：

```
fn do_stuff<T: Clone>(value: &T) {
    let cloned = value.clone();
}
// cloned 会是 T 类型，因为 T 有 clone 函数

fn do_stuff<T>(value: &T) {
    let cloned = value.clone();
}
// cloned 会是 &T 类型，因为 T 没有 clone 函数，但是引用有
```



### transmute

类似 Union 的只要字节相同的转换

```
mem::transmute<T, U>
```



## From/Into

From 和 Into 是配对的，实现了前者，后者就会自动实现。

`impl From<T> for U`

就可以使用

`U::from(T)`	`let u:U = T.into()`

同样的，如果你希望给一个自定义的结构体实现 From 和 into 的功能，就可以使用

```
impl From<T> for Struct{
	fn from(x: T) -> Struct{
	
	}
}
```



# 包和模块

## 包 crate

包是一个独立的可编译单元

`main.rs` 和 `lib.rs` 都是编译单元，所以都是包



### 项目 Package

奇怪的翻译，可以是二进制项目，也可以是库项目

包含独立的 `Cargo.toml` 文件



## 模块

构成包的基本单元，可以将代码按照功能重组，实现更好的可读性和易用性

```
// 餐厅前厅，用于吃饭
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();		// wrong，从包根访问这个是 Private 的，父无法访问子，子可以访问父，如果需要改，不仅要给模块加pub还需要递归的给里面的模块，以及函数都加上才可以

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```



```
crate
 └── eat_at_restaurant
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```



`crate` 从包根开始绝对路径

`self` 当前

`super` 父模块



### 模块与文件分离

`src/front_of_house.rs`

```
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

`src/lib.rs`

```
mod front_of_house;	// 和模块名同名的文件名去找

pub use crate::front_of_house::hosting;	// 用绝对路径引用

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```



## use

可以用 `as` 来取别名，以区分同名函数

`use` 之后，可见性会自动被设置为私有，如果希望它可以继续被外部使用就使用 `pub use`

如果有相同的组成方式，可以用 `{}` 一起引入进来

```
use std::collections::{HashMap,BTreeMap,HashSet};
use std::{cmp::Ordering, io};

use std::io::{self, Write};
```

可以用 `*` 代替所有项

```
use std::collections::*;
```



`pub `可以加`(in crate::mod::mod)` 的形式来限制公开权限，也可以用`self,super,crate`



# 注释和文档

有三类注释，一类是代码注释，一类是文档注释，一类是包和模块注释

## 代码注释

`//`

`/* */`

## 文档注释

`///`

`/** */`

`cargo doc` 可以在 `target/doc` 下生成 `HTML` 文件

## 包和模块的注释

`//!`

`/*! */`