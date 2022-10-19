# Day2

## 变量

### 命名规范

| 条目                               | 惯例                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| 包 Crates                          | [unclear](https://github.com/rust-lang/api-guidelines/issues/29) |
| 模块 Modules                       | `snake_case`                                                 |
| **类型 Types**                     | **`UpperCamelCase`**                                         |
| 特征 Traits                        | `UpperCamelCase`                                             |
| 枚举 Enumerations                  | `UpperCamelCase`                                             |
| 结构体 Structs                     | `UpperCamelCase`                                             |
| **函数 Functions**                 | **`snake_case`**                                             |
| 方法 Methods                       | `snake_case`                                                 |
| 通用构造器 General constructors    | `new` or `with_more_details`                                 |
| 转换构造器 Conversion constructors | `from_some_other_type`                                       |
| **宏 Macros**                      | **`snake_case!`**                                            |
| **局部变量 Local variables**       | **`snake_case`**                                             |
| 静态类型 Statics                   | `SCREAMING_SNAKE_CASE`                                       |
| **常量 Constants**                 | **`SCREAMING_SNAKE_CASE`**                                   |
| 类型参数 Type parameters           | `UpperCamelCase`，通常使用一个大写字母: `T`                  |
| 生命周期 Lifetimes                 | 通常使用小写字母: `'a`，`'de`，`'src`                        |
| Features                           | [unclear](https://github.com/rust-lang/api-guidelines/issues/101) but see [C-FEATURE](https://course.rs/practice/naming.html#c-feature) |



### 变量绑定

`let a = "hello world"`

把一个**内存对象**绑定给一个**变量**，会把这个对象的所有权给这个变量。



### 变量可变性

变量默认是**不可变**的，只有使用 `mut` 关键字才能让它变成可变的。

```
let mut x = 5;
x = 6;
```

在原本内存对象的位置修改值。



### 未使用的变量

RUST 会对未使用的变量做出警告，可以通过在变量名前加 "_" 来让RUST 不对它发出警告。

但是  `let _x = 5`这样还是会进行值绑定的操作的。

而如果 `let _ = s` 则完全不会绑定。所以可以用 "_" 来忽略单个值，也可以通过 ".." 来忽略剩余值，例如

```
struct Point {
	x: i32,
	y: i32,
	z: i32,
}
let origin = Point {x:0, y:0, z:0};
match origin {
	Point {x, .. } => println!("x is {}", x),
}
```



### 变量解构

```
fn main() {
    let (a, mut b): (bool,bool) = (true, false);
    // a = true,不可变; b = false，可变
    println!("a = {:?}, b = {:?}", a, b);

    b = true;
    assert_eq!(a, b);
}
```

从元组中直接赋值，这里如果不事先定义类型也可以，RUST 可以自行推断类型。

还可以进行一些更复杂的模式匹配的解构：
```
struct A {
    e: i32
}

fn main() {
    let (a, b, c, d, e);

    (a, b) = (1, 2);
    // _ 代表匹配一个值，但是我们不关心具体的值是什么，因此没有使用一个变量名而是使用了 _
    [c, .., d, _] = [1, 2, 3, 4, 5];
    A { e, .. } = A { e: 5 }; # 这里是对A类型的结构体进行赋值，使用了:

    assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
}
```



### 常量

必须指定类型，使用 `const` 、通常是全部大写的蛇形命名。

常量可以在全局域声明（相反的 let 不能在全局域使用）。



### 变量覆盖

```rust
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;

    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

这里有一个变量域的问题，在 {} 中定义的变量会取代之前的变量，然后出了 {} 就会变回去，所以这里有两个不同的变量，不同的内存对象空间

```rust
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let mut x = x + 1;
    {
        x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
    println!("The value of x is: {}", x);
}
```

如果这样的话，就是两个12了，只有一个变量。



## 基本类型

数值类型：`i8,i16,i32,i64,isize;u8,u16,u32,u64,usize;f32,f64`

字符串：`&str`

布尔：`true, false`

字符：Unicode

单元：`()`

### 类型推断

所有变量都必须在编译时有类型，但是如果不显式声明，RUST会自动推断。

`parse()` 可以帮忙做字符串的类型转换，但是parse()的返回类型并不是翻译出来的，需要加一个 `.expect()` 才可以得到正确的结果

```
let guess = "42".parse::<i32>().expect("Not a number!");
```

也可以在等式左边设定类型转换的最终需要的类型

```
let guess::i32 = "42".parse().expect("Not a number!");
```

也可以用 `as + 你需要的类型` 来做一些类型转换



### 整数类型

范围和 C 中基本一致，有符号整数也是少一位，默认为 i32

类似C中`1ll`的写法，可以写`1i64`来给立即数显示声明类型

同时可以用 i32::MAX 来指代 2^31 - 1

**不一样的地方**：在debug模式下，Rust可以检测整数溢出（编译的时候就能看出来？）。

测试了一下，如果用的函数的形式，cargo build 的时候还是检查不出来的：

```
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    let a = 2_000_000_000;
    let b = 2_000_000_000;
    let x = add(a, b);
    println!("{}", x);
}
```

但是直接写可以：

```
fn main() {
    let a = 2_000_000_000;
    let b = 2_000_000_000;
    let x = a + b;
    println!("{}", x);
}
```

但是这两个在运行的时候会给出溢出的错误，比C要好一些。



### 浮点数

默认是 `f64`，标准也是 IEEE-754，和 C 应该也是一样的。（不过这个应该是和硬件绑定的标准）

要注意的点和 C 语言使用浮点数一样，也是注意误差，注意 NaN，浮点数对象有判断NaN的方法： `.is_nan()`

如果需要按位输出

```
println!("{:.2}, x"); # 保留两位输出
```





### 算术运算符

`+, -, *, /, %`



### 位运算符

`& | ^ ! >> <<`



### 序列

类似python中的语法，可以用简洁的方法生成连续的序列：

```
for i in 1..=5 {
	println!("{}", i);
}
```

目前只支持数字或者字符，也不支持等差数列那种形式。

这里加了 '=' 表示包括后面这个，如果不加 '=' 表示不包括这个

```
Range{start:1, end:5} # 1..5
RangeInclusive::new(1,5) # 1..=5
```





### 有理数和复数

这是一个包：`num`，添加的方法需要在 Cargo.toml 的 [dependencies] 下加入 `num = "0.4.0"`

支持1. 任意大小的有理数和复数，2. 固定大小的整数和任意精度的浮点数 3.固定精度的十进制小数



### 字符

RUST 使用的是 Unicode 编码，而不是 ASCII 编码，每个 Unicode 是四个字节



### 单元类型

只有一个（），对于函数的返回值而言，它也是有返回值的，称为单元类型

没有返回值的函数称为 发散函数（diverge function）



对着昨天没有看懂的代码，把其中包含的一些 RUST 的知识给搞懂，这里还是没有搞懂（sad

### String-Match

```
fn main() {
    let penguin_data = "\
    common name, lenth (cm)
    Little penguin,33
    Yellow-eyed penguin,65
    Fiordland penguin,60
    Invalid,data
    ";
    
    let records = penguin_data.lines();

    for (i, record) in records.enumerate(){
        if i == 0 || record.trim().len() == 0{
            continue;
        }

        let fields: Vec<_> = record
            .split(',')
            .map(|field| field.trim())
            .collect();
        
        if cfg!(debug_assertions){
            eprintln!("debug:{:?} -> {:?}", record, fields);
        }

        let name = fields[0];

        if let Ok(length) = fields[1].parse::<f32>(){
            println!("{}, {}cm", name, length);
        }
    }
}
```



### String 类型

#### 切片

```
fn main() {
	let s = String::from("hello world");
	int len = s.len();
	let hello = &s[0..5];
	let world = &s[6..11];	# 左闭右开
	let a = &s[..2];		# 左边默认为 0
	let b = &s[4..];		# 右边默认为 len
	let c = &s[..];
	let chinese = "中国人";
	let a = &[0..2];		# wrong 因为一个中文字符占3个字符，无法提取
	
}
```

#### 关于 UTF-8 字符串

如果想要以 Unicode 字符的方式遍历字符串，最好是使用 chars 方法

```
for c in "中国人".chars() {
}
```

如果按字节输出

```
for b in "中国人".bytes() {
}
```

如果想要在复杂的 UTF-8 中获取字串，需要使用 utf8_slice 这样的库才可以

