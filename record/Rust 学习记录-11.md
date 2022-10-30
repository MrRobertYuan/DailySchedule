# 多线程 - 学习

## 多线程

### 闭包

有三种 `FnOnce` `Fn` `FnMut`

其中 `FnOnce` 会获得环境变量的所有权，也可以通过 `move || {}` 来强制其获得所有权，通常用在环境变量的生命周期比闭包短的情形。

闭包可以匹配任何一种形式（不过要看其内部有无对环境变量的修改）

闭包如果作为函数的返回值，可以使用 `Box<dyn Fn(i32) -> i32>`

### 创建线程

```
thread::spawn(|| {})
```

内部是用闭包的形式来执行

`main` 线程一旦结束，就立刻结束，所以如果想要子线程完成任务，需要保持`main`线程的存活

`thread::sleep(Duration::from_millis(1))` 可以让线程休眠指定的时间



### 等待线程

线程名`.join()` 可以让当前线程阻塞，直到他等待的这个线程结束



### 使用 move 将当前线程中的变量转移到另一个线程

在闭包前面加 `move` 关键字，可以让这个闭包变成 `FnOnce` 获得里面使用的环境变量的所有权



### 屏障 (Barrier)

引入 `use std::sync::{Arc, Barrier}`

创建 `let barrier = Arc::new(Barrier::new(6))`

复制 `let b = barrier.clone()`

暂停 `b.wait()`



### 线程局部变量

`thread_local!()` 可以定义线程局部变量，在线程内部通过 `with` 方法获取变量值

```
thread_local!(static Foo: RefCell<u32> = RefCell::new(1));
Foo.with(|f| {})
```

每个线程都会有一个单独的变量，互不干扰

（这里有点难）



### 用条件控制线程的挂起和执行

条件变量：`Condvar` 

锁：`Mutex`

条件变量一般和锁一起使用

条件变量可以发出指令 `.notify_one()` 也可以接受 `cvar.wait()`

```
use std::thread;
use std::sync::{Arc, Mutex, Condvar};

fn main(){
	let pair = Arc::new((Mutex::new(false), Condvar::new()));
	let pair2 = pair.clone()；
	
	thread::spawn(move || {
		let &(ref lock, ref cvar) = &*pair2;
		let mut started = lock.lock().unwrap();
		
		*started = true;
		cvar.notify_one();
	});
	
	let &(ref lock, ref cvar) = &*pair;
	let mut started = lock.lock().unwrap();
	while !*started {
		started = cvar.wait(stated).unwrap();
	}
	
}
```

查了 rust 的手册

`fn wait<'a T>(&self, guard: MutexGuard<'a, T>) -> LockResult<MutexGuard<'a, T>>`

`guard` 表示释放的互斥锁，最后也会得到一个互斥锁。

条件变量是用来阻塞线程的，如果这个值发生了改变，改变的线程就会发出一个通知，如果收到了这个通知，并且获得的时候满足条件了，就可以不再阻塞了，这里也会自动释放掉锁，而如果条件仍然不满足的话，就会继续wait，把锁还回去。

所以这里的锁起到了对环境变量中的 判断值 的保护作用。



### 只被执行一次的函数

在多线程中，无论写在几个线程里面，只有第一个人会执行

```
use std::sync::Once;

static INIT: Once = Once::new();

INIT.call_once(|| {})
```



## 线程间的消息传递

Rust 在标准库中提供了消息通道 `channel`

### 多发送者，单接收者

`use std::sync::mpsc` (multiple producer single consumer)

创建：`let (tx, rx) = mpsc::channel()`

发送： `tx.send(T).unwrap()`

接收：`rx.recv().unwrap()`

返回的类型是 `Result<T, E>`，这里 `unwrap()` 相当于快速的错误处理。

`send` 只能发送同样类型的数据，同时也只能收这种类型的数据。



#### 不阻塞的 try_recv()

如果没有收到不会阻塞，而是直接返回一个错误

`Err(Empty)` 表示通道没有消息

`Err(Disconnected)`表示通道已经被关闭



#### 传输是会带所有权的

如果没有实现 `Copy` 就会把所有权转移



#### 多个消息如何接收

```
for received in rx{

}
```

可以通过 `for` 循环，阻塞的从 `rx` 中接收消息，直到`tx`被`drop` 掉才会结束

多个消息是 `FIFO` 的



#### 如果实现多个发送者

```
let (tx, rx) = mpsc::channel();
let tx1 = tx.clone();
```

只有所有的 `tx` 都被 `drop` 了，接收者才会结束



#### 同步与异步

之前都是异步的，无论如何，发送方都不会阻塞

还有同步的：

```
let (tx, rx) = mpsc::sync_channel(0);
```

做完发送之后会被阻塞，要等待接收方接收到才会继续。

这里括号中的数字表示这个消息通道的缓冲

如果设置为 `1` 那么第一条语句就不会阻塞，第二条就会阻塞。



而异步的方法的缓冲是无上限的，可能内存会有问题，所以可以通过实现一个有缓冲的同步来实现异步以阻止内存过大。



#### 关闭通道

在所有发送者 或者 所有接收者 `drop` 之后通道就会关闭。



### 传输多种类型的数据

可以使用枚举来实现，不过枚举的缺点是，Rust 会按照其中最大内存的元素进行对齐，所以对于那些小的元素，就会造成内存上的浪费。



## 线程同步：锁、条件变量和信号量

### 互斥锁

同一时间只允许一个线程访问这个值。

#### 单线程中使用

Mutex 中会自带一个 `T` 类型的数据

```
let m = Mutex::new(5);
{
let mut num = m.lock().unwrap();
*num = 6;
}
// 超过作用域后，锁自动被 Drop
// drop(num)
```

`m.lock()` 会阻塞当前线程直到获取到锁。

也可能会返回一个和错误，例如现在拥有锁的线程 `panic()` 了。

`Mutex<T>` 是一个智能指针，`m.lock()` 会返回一个指针指针 `MutexGuard<T>` 它实现了 `Deref` 特征，会自动解引用后获得一个引用类型，这个引用指向`Mutex` 内部的数据，同时也实现了`Drop` 特征，超出了作用域之后，自动释放锁。

#### 多线程中使用

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
	let counter = Arc::new(Mutex::new(0));
	let mut handles = vec![];
	
	for _ in 0..10{
		let counter = Arc::clone(&counter);
		let handle = thread::spawn(move || {
			let mut num = counter.lock().unwrap();
			*num += 1;
		})
		handles.push(handle);
	}
	for handle in handles{
		handle.join().unwrap();
	}
	println!("{}", *(counter.lock().unwrap()));
}
```

`Mutex` 如果需要在多线程之间传输，必须使用线程安全的 `Arc`。

如果想避免死锁的情况，可以使用 `.try_lock()`



### 读写锁 RwLock

支持并发读 `.read()` 可以获得一个不可变的引用

单独写`.write()` 可以获得一个可变的引用，同时都是支持`.try_xxx()`



### 信号量

比较复杂，书上也没有讲的很清楚



## 原子类型与内存顺序



### 使用 Atomic 作为全局变量

可以设置需要修改的值为全局的原子变量

```
static R: AtomicU64 = AtomicU64::new(0);

R.fetch_add(1, Ordering::Relaxed);
R.load(Ordering::Relaxed);
R.store(0, Ordering::SeqCst)
```

这种方法比 `mutex` 要快很多

`Atomic` 默认就是 `mut` 不用特意声明

一直使用 `Ordering::Relaxed` 是控制原子操作的 内存顺序



### 内存顺序

CPU 访问内存时的顺序，该顺序可能受以下因素的影响：

1. 代码中的先后顺序
2. 编译阶段：编译器的优化
3. 运行阶段：因CPU的缓存机制导致顺序被打乱

#### 五种规则

1. `relaxed`  无规则
2. `release` 之前的操作永远在它之前
3. `acquire` 之后的访问永远在它之后
4. `acqRel` 以前的操作永远在之前，之后的操作永远在之后
5. `SeqCst` 加强版`acqRel` 

### 多线程中使用 Atomic

需要使用 `Arc` 一起使用



## 基于 Send 和 Sync 的线程安全

`Send` 和 `Sync` 是标记特征，该特征未定义任何行为。

实现 `Send` 的类型可以在线程间安全的传递所有权

实现 `Sync` 的类型可以在线程间安全的共享（通过引用）

`Sync` 还有一个潜在的依赖，一个类型要在线程间安全的共享的前提是，指向它的引用必须能在线程间传递。

如果 `&T` 是 `Send` ，那么`T` 是 `Sync`



### 实现 `Send` 和 `Sync` 的类型

几乎所有类型都默认实现了，除了

1. 裸指针
2. `Cell` 和 `RefCell`
3. `Rc` 因为它内部的计数器不是线程安全的

手动实现 `Send` 和 `Sync` 是不合理的



# 宏编程 - 学习

分为两大类： 声明式宏 和 过程宏

## 宏和函数的区别

宏本质上从一种代码生成另一种代码

可变参数： 宏的参数是可变的，函数则是固定的

宏展开：宏在编译前就会展开，因此宏可以为指定的某个类型实现某个特征，而对应的特征会在编译的时候实现

宏的缺点：宏的语法复杂，可读性不高，难以理解和维护

## 声明式宏 marco_rules!

宏类似 match 也是把一个值和对应的模式进行匹配，而且这个模式会和特定的代码相关联。

输入是一段代码，匹配这个结构，如果有合适的话就会实现宏展开

#### 简化的 vec!

```
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

注意这里的定义是 `vec`，！是使用的时候加的

#### 模式解析

`( $( $x:expr ), * )`

`expr` 会匹配任何 `rust` 表达式，并给对应的表达式一个名字 `$x`

`,` 表示前面的表达式匹配的分隔是靠`,` 隔开 

`*` 表示之前的模式可以出现零次也可以出现无数次

而后面的执行部分

`*` 就表示这段表达式会被扩展成之前匹配的次数多次

下面就是 `let v = vec!{1,2,3}` 转换成的代码

```
let v = {
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

## 用过程宏为属性标记生成代码

过程宏输出的代码不会替换之前的代码

过程宏的定义必须放在一个特定的单独的包内，因为其需要提前被编译

创建`derive` 类型的过程宏：（后面有具体过程）
```
use proc_macro;

#[proc_macro_derive(HelloMacro)]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

#### 自定义 derive 过程宏

假如我们有一个特征 `HelloMacro` 那么可以通过对每个需要它的类型手动实现它，也可以对每个类型标记上 `#[derive(HelloMacro)]`

如果想要实现对应的 `hello_macro` 宏，需要创建一个对应的包名`hello_macro_derive`

使用 `cargo new hello_macro_derive --lib`

```
hello_macro
├── Cargo.toml
├── src
│   ├── main.rs
│   └── lib.rs
└── hello_macro_derive
    ├── Cargo.toml
    ├── src
        └── lib.rs
```

同时在 `Cargo.toml`中添加相应的内容：

```
[dependencies]
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
# 也可以使用下面的相对路径
# hello_macro_derive = { path = "./hello_macro_derive" }
```

还需要在 `hello_macro_derive/Cargo.toml` 下增加：
```
[lib]
proc-macro = true	// 过程宏的开关打开

[dependencies]
syn = "1.0"
quote = "1.0"
```

接着在 `hello_macro_derive/src/lib.rs` 中添加下面的代码：

```
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // 基于 input 构建 AST 语法树
    let ast = syn::parse(input).unwrap();

    // 构建特征实现代码
    impl_hello_macro(&ast)
}
```

`syn::parse` 会返回一个 `DeriveInput` 的结构体来表示代码：

```
#[derive(HelloMacro)]
struct Sunfei;
```

```
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Sunfei",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

可以看到对单元结构体 `Sunfei` 做了这样的语法解释

```
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

`quote!` 可以定义想要返回的代码，不过还需要先 `.into()` 做转换

### 类属性宏

```
#[route(GET, "/")]
fn index() {}
```

当用户使用 `HTTP GET` 访问 `/` 根路径的时候，就会用 `index()` 来服务

其定义函数大概如下：

```
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

### 类函数宏

和声明宏很像，但是它的定义是和过程宏一样的，所以它可以处理一些更难的函数。

而声明宏更像是一个模式匹配，能处理的难度不会很高。



# Rustling - 练习

## Thread

之前没有用互斥锁来操作，只使用了智能指针`Arc`，智能指针是不可变的引用

但是我想要改变值，必须是可变引用，所以加一个互斥锁来存放，然后取出的时候加上`mut` 就可以改变其中的值了

## Macro + quiz4

这里只考察了声明宏

1. 使用宏需要加！
2. 在宏定义前面加上 `#[macro_export]` 这样其他包就可以把宏加入当前作用域中
3. 放在 mod 中记得加 pub
4. 如果一个宏有多种匹配模式，需要用 ；隔开，而不是 match 一样用 ，隔开
5. 简单的宏定义方法

## Clippy

是一个对Rust正确性的提升工具，会更严格的编译

1. 提示推荐使用 常数pi ， std::f32::consts::PI
2. 提示类型不匹配，建议使用 if let 来解构 Option

## Conversions

1. 如果对应类型有 `as_ref()` 函数，需要对其类型进行限制，对于 `&str`和 `String`的限制都为 `AsRef<str>`
2. 想要使用 T::from(U) 就需要 `impl From<U> for T{}` 并写上 `fn from(x:U) -> T{}`
3. from_into 和 from_str 都是按规定写出来就好，需要用到 `.split().collect()` 以及`Result`的相关知识
4. tryfrom 和 from 也基本一致，只是返回值不太一样，也是码代码的题

## Advanced_errors

1. ？可以实现自动转换成更抽象的错误，所以可以通过给高阶的错误增加 `From` 特征来让它可以从低阶错误转变过来，从而自动实现 `?`
2. 第二问也是一个代码题，就是对每种错误单独处理就好，对于自定义的错误，如果想让它可以自动转成`std::error::Error` 就需要给他这个特征 `impl std::error::Error for ParseClimateError {}` 里面不用实现

