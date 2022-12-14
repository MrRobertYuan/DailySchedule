# Day3

## 语句和表达式

RUST 中的函数返回值不需要 return，而只需要最后一个表达式就可以

语句不会返回值，而表达式会返回值

所以不能把语句作为右值，而表达式可以，`let` 目前稳定版中是语句，不能 `let b = (let a = 10)`



调用一个函数是表达式，因为它会返回值

可以用大括号括出一段语句，并最后跟一个表达式，这样也是一个表达式

```
let y = {
	let x = 3;
	x + 1
};
```

注意这里 `x+1` 后面没有分号，**表达式都没有分号**

如果表达式不返回值，会默认返回一个 ()

也就是说如果上面的式子写成：

```
let y = {
	let x = 3;
	x + 1;
};
```

那么 y 就会等于 ()



### 函数

基本格式

```
fn add(i: i32, j: i32) -> i32 {
	i + j
}
```

括号内是参数，用冒号声明类型，-> 后是返回值类型

位置无关，在调用前后定义都可以。

之前学过函数是表达式，最后写一个式子就表示返回值，如果想要提前返回，可以使用 return + 表达式值



#### 特殊返回值

和表达式一样，如果最后不是一个表达式，例如一个语句结尾，就会返回一个 （）

你可以通过显式的方法来返回（）（就是指在定义的时候就设置返回的类型为（））



同时也可以用发散函数，此时的返回值类型为 ！

表示这个函数永远不返回，通常是用来写一个让程序崩溃的函数。

可以使用 `panic!("")`这个宏，也可以使用 `loop{}` ，也可以使用 `unimplemented!()`，或者`todo!()`

还有睡眠：`std::thread::sleep(std::time::Duration::from_secs(1))`



## 所有权

RUST 可以在编译的时候找到内存不安全的问题

原理就是 RUST 会跟踪堆上分配的空间的分配和回收，防止内存泄漏。（因为栈上的空间的分配和释放其他语言也可以做的很好，就没有强调）

### 所有权原则

1. 每个值都有一个所有者（一个变量）
2. 一个值只能有一个所有者
3. 当所有者离开作用域时，这个值会被丢弃



作用域和其他语言一样，也是以 {} 作为分割



### 字符串

RUST 中有硬编码的字符串类型 `&str` （可能类似常量），它的值是不可变的，还有 `String` 类型，它是存在堆上的，是动态的

```rust
let s = String::from("hello");	# 可以通过 `&str` 创建 `String`
s.push_str(", world!");			# 支持 push_str() 等动态修改
println!("{}",s);
```



String 是一个复杂类型，包括了 储存在栈中的堆指针、字符串长度（已经使用的大小）、字符串容量（分配空间的大小）

这里的这个 '&' 字符应该表示的是引用。



### 转移所有权

对于基本数据类型，会进行一个值的拷贝，而不是所有权的转移

```
let mut x = 5;
let y = x;
```

这些操作都是在栈上运行的

这些值需要满足的是他们的大小是确定的，而不是 String 这种的。

包括了：所有整数，布尔，所有浮点，字符，只包含前面的类型的元组，不可变的引用 &T

注意可变引用 &mut T 是不可以复制的。



而对于字符串

```
let s1 = String::from("hello");
let s2 = s1;
```

这样是不可以实现拷贝功能的，字符串类型没有内置的自定义的拷贝功能。

但是这样编译是没有问题的。

但是之后这样你想使用 s1 就会报错。也就是说 RUST 中没有浅拷贝，使用了浅拷贝就会直接转移所有权。



虽然对于 String 类型不可以，但是对于字符串常量，这个操作是可以的

```
fn main() {
    let x: &str = "hello, world";
    let y = x;
    println!("{},{}",x,y);
}
```

这样 y 是对 x 值的一个拷贝，拷贝了对于这个字符串的引用。



如果想要实现深拷贝，需要用到 `.clone()` 函数，可以复制字符串里的所有内容。



### 函数传递

RUST 好像没有形参和实参的分别，在传入参数的时候，就和 let 一样，会把值进行 转移 或者 复制。

如果一个字符串传入了函数，那么出了这个函数，这个字符串的变量就不能用了，和上面的 s1 一样。

函数里如果定义了 String ，也可以把所有权传出来，而不用担心会被释放。

所有权转移的时候，可变性也会发生变化：

```
let s = String::from("Hello")
let s1 = s; // s1 是不可变的
let mut s1 = s1; // s1 是可以变的
```



### 部分转移所有权

就是类似结构体或者序列中有一部分通过解构，将其中的所有权给了别的变量，那么这个原本的复合体就不能直接使用了

但是其中没有给出去的所有权的部分还是可以使用。



## 引用和借用

引用就是 `&` 取地址，提取引用类型的内存地址中的值，就是通过 `*` ，这里和 C 都是一样的。

获取变量的引用，称之为借用（borrowing）

不加 mut 的引用就是不可变的引用，你可以用它的值，但是你没有它的所有权，不能改变它。所以引用离开作用域，只会删引用不会删对象。



引用的类型就是 在你引用的类型前面加`&`，例如 `&String` 表示对`String`的引用。



### NLL **Non-Lexical Lifetimes**

在新版编译器中，引用的作用域是 其定义到最后一次使用的 范围。



### 可变引用

`&mut String` 类型就是一个可变引用，想要对一个变量取可变引用也是这样：`let y = &mut x`

在同一作用域，每个数据只能有一个可变引用。

对于一个变量，同一作用域内，可变引用和不可变引用不能同时存在，所以要么你有一个可变引用，要么你有多个不可变引用。

注意：和 C 一样，可变引用只能指向可变变量



### 悬垂引用

一个引用引用的对象消亡了，而这个引用还没有，这就是悬垂引用。

在 RUST 中，从编译器层面阻止了这种代码。



### ref

ref 和 `&` 都用来取引用，但是用法不同

```
let x = 5;
let x1 = &x;
let ref x2 = x;
```

相当于 `&` 是用在右边告诉你，我是一个引用，不需要你的所有权，`&mut` 表示这是一个可变引用

而 `ref` 是用在左边告诉你，我是一个引用，不需要你的所有权，`ref mut` 表示这是一个可变引用





## 今日疑问

Box 类型是什么 Box::new(); 是什么