# Day-7

## 方法

#### 定义

```
struct Circle{
	x: f64,
	y: f64,
	radius: f64,
}

impl Circle{
	// new 的第一个参数不是 self，通常用来初始化一个结构体的实例
	fn new(x: f64, y: f64, radius: f64) -> Circle {
		Circle{
			x,
			y,
			radius,
		}
	}
	
	// &self 表示借用(引用)当前的 Circle 结构体，所以可以访问 .radius
	pub fn area(&self) -> f64{	// 可以使用pub表示是公有的方法
		std::f64::consts::PI * (self.radius * self.radius)
	}
}
```

方法的定义都是放在 `impl` +`structname` 里面的，函数之间不需要逗号隔开

方法和字段是完全隔开的

同时也可以用多个`impl` 来把不同的方法分类分块放

#### self 指针的使用

注意， `self` 是有所有权概念的，所以通常我们是使用 `&self` 和 `&mut self` 来对 `self` 做引用和可变引用，而不会直接把所有权传进方法里

几乎只有一种方法会需要把所有权传进来，就是把当前的对象转成另一个对象的时候，就可以把所有权释放了

【&self 其实是 `self: &Self`的缩写，也就是说 `Self` 就是表示当前结构体的类型】

#### 方法名可以和字段名相同

例如都叫`width`，使用方法时为`p.width()`使用字段为`p.width`

那这种方法通常用于做 `get()` 方法来获取一个字段的值（通常是字段私有的时候），而不用加`getXXX`这样的名字了

#### 用 . 代替 -> 

在 Rust 中不管是对对象的引用还是对象本身，都是通过 . 来访问方法的。

`p1.distance()`和 `(&p1).distance()` 是等价的

这里之所以可以实现这个的原因是因为 distance 中有规定 self 的类型是 &self 还是 &mut self 还是 self，所以它可以自动为对象添加 `&` `&mut` 或者 `*` 来让变量与函数匹配。【不过也可以注意到，这里 `&p` 还是不能直接转到 `&mut p` 的，反过来是可以的】

```
fn main() {
    struct Node{
        data: i32,
        next: i32,
    }
    impl Node{
        fn new(data:i32, next:i32) -> Node{
            Node { data, next }
        }
        fn test(&mut self){
            println!("{},{}", self.data, self.next);
        }
    }
    let mut a = Node::new(1,1);
    a.test();
    let b = &a;
    b.test();	// 这样就会报错
}
```

#### 关联函数

关联函数 不是方法，但是也是定义在 `impl` 里面的，它们是不带`self`参数的函数

一般我们约定使用 `new()` 作为初始化的关联函数

对于关联函数我们都是使用 `结构体名::关联函数` 来调用的 

#### 枚举也可以实现方法

和结构体一样，也是通过`impl` + 枚举类型名 就可以为枚举变量实现方法

#### take-home的问题

```
#[derive(Debug)]
enum TrafficLightColor {
    Red,
    Yellow,
    Green,
}

// 为 TrafficLightColor 实现所需的方法
impl TrafficLightColor {
    fn color(&self) -> &str{
        match *self{			// 我一开始这里没有写 *self, 而是写的 &self 或者 self 也能过，不知道为什么
            Self::Red => "red",
            Self::Yellow => "yellow",
            Self::Green => "green"
        }
    }
}

fn main() {
    let c = TrafficLightColor::Yellow;

    assert_eq!(c.color(), "yellow");

    println!("{:?}",c);
}
```

为什么 `self` 和 `&self`和 `*self` 都能匹配上？



## 泛型和特征

### 泛型

就是类似 C++ 中的 Template 模板，但是这里会在编译的时候就检查是不是所有的可能的类型 T 都可以正确执行

如果不是的话，就会报错；修复的方法是加上对应的特征（trait）来对这个 `T` 进行限制：

限制的方法和之前学的模式匹配类似，也是前面的是类型，用冒号连接它的特征：`<T: trait>`

如果想要用 < 比较大小：`std::cmp::PartialOrd`

如果想使用 + ：`std::ops::Add<Output = T>` （这里是限制它使用Add的返回值是T） `std::ops::Add<T, Output = T>`

如果想通过 `{:?}` 输出：`std::fmt::Debug`



泛型使用时需要在对应的函数名/结构体名后面声明使用的泛型 `<T>`

#### 一个泛型函数类型

```
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T{
	let mut largest = list[0];
	
	for &item in list.iter() {
		if item > largest
			largest = item;
	}
	largest
}
```

#### 结构体中使用泛型

```
struct Point<T>{
	x: T,
	y: T,
}
fn main(){
	let integer = Point{x:5, y:10};
	let float = Point{x:5.0, y:4.0};	// 这里说是float，但是自动推断的 T 是 f64
}
```

这种情况下，x 和 y 在结构体实例化的时候类型必须相同，不然就需要使用多个泛型参数

```
struct Point<T, U>{
	x: T,
	y: U,
}
```

这种情况下，x 和 y 的类型就又可以相同又可以不同了。

#### 枚举类型

最经典的使用泛型的枚举类型：

```
enum Option<T>{
	Some(T),
	None,
}
```

以及：
```
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

T 是正确返回类型，E 是错误类型

函数返回时，如果正确了，就会得到它原本需要的信息，如果不正确就会给出对应的错误类型

#### 方法中使用泛型

这里首先要强调的是，如果结构体本身有泛型参数，那么在声明它的方法的时候，如果想使用其中的泛型，就需要在 `impl` 后面也加上需要使用的泛型。

这个是对于结构体的大框架而言的

而对于在方法的块中定义的各个函数，都是和函数的泛型参数一样也是可以在函数名后增加对应的函数需要的泛型参数的。

##### 为具体的泛型类型实现方法

```
struct Point<T>{
	x: T,
	y: T,
}
impl<T> Point<T>{
	// 这是所有的泛型 T 组成的Point结构体都可以使用的方法
}
impl Point<i32>{
	// 这是只有在 T 为 i32 时才有的方法
}
```

#### 常数泛型

第一种情况是类似常变量的形式

```
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T;N]){
	println!("{:?}",arr);
}
```

这个`N` 是基于值的泛型参数，基于值的类型是 `usize`

泛型是写的时候不知道，但是在编译前都可以计算好的，而常泛型就是写的时候不知道，但是在编译前就计算好的常量。

#### const 泛型表达式

可以通过 `where` 限制参数占用的内存大小

```
fn somthing<T>(val: T)
where Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
{

}
pub enum Assert<const CHECK: bool>{

}
pub trait IsTrue{

}
impl IsTrue for Assert<true>{

}
```

这个表述还挺迷的（不过也说了只能在 Nightly 版本下使用）

大概就是在枚举变量 `Assert` 的泛型中定义了一个 const 类型的泛型参数叫做 CHECK，类型是 bool

CHECK 表达式的值为 `{ core::mem::size_of::<T>() < 768 }`



#### 泛型的性能

实在编译和最后的代码中把相应的类型生成好，运行时是零成本的

#### take-home 泛型

如果 `A` 是空的结构体。

`let a = A;` 可以设置一个 `A` 类型的结构体变量 `a`

可以通过 `struct S(A);` 来定义一个包含一个A类型成员的结构体 S



注意泛型只有在定义的时候需要使用，在使用的时候则只需要自动推断

#### take-home const 泛型

const 泛型只支持以下的形参：

- 一个单独的 const 泛型参数
- 一个字面量 (i.e. 整数, 布尔值或字符).
- 一个具体的 const 表达式( 表达式中不能包含任何 泛型参数)

```
fn foo<const N: usize>() {}

fn bar<T, const M: usize>() {
    foo::<M>(); // ok: 符合第一种
    foo::<2021>(); // ok: 符合第二种
    foo::<{20 * 100 + 20 * 10 + 1}>(); // ok: 符合第三种
    
    foo::<{ M + 1 }>(); // error: 违背第三种，const 表达式中不能有泛型参数 M
    foo::<{ std::mem::size_of::<T>() }>(); // error: 泛型表达式包含了泛型参数 T
    
    let _: [u8; M]; // ok: 符合第一种
    let _: [u8; std::mem::size_of::<T>()]; // error: 泛型表达式包含了泛型参数 T
}

fn main() {}
```



`&str` 的大小 是 两个字，一个是指针一个是字符串大小

`String`的大小是三个字，一个指针，一个大小，一个容量



### 特征

特征类似其他语言中的接口，定义了一个可以被共享的行为，只要实现了特征，你就能使用该行为

#### 定义特征

```
pub trait Summary{
	fn summerize(&self) -> String;
}
```

特征只定义行为看起来是怎么样的，而不定义具体行为应该怎么做，所以只是一个声明

所有实现这个特征的类型都需要具体实现这个特征中相应的方法。

你也可以同时规定一个默认的执行方法，其他使用这个特征的结构体可以选择重载这个方法或者直接使用你的默认方法：

```
pub trait Summary{
	fn summerize(&self) -> String{
		String::from("Hello")
	}
}
```

同时在一个特征内，允许这些函数相互调用，尽管可能有点函数没有默认执行方法。



#### 为类型实现特征

```
pub trait Summary{
	fn summerize(&self) -> String;
}
pub struct Post{
	pub title: String;
	pub author: String;
	pub content: String;
}
// 之前方法的写法是 impl Post{} 现在是 impl 特征名 for Post
impl Summary for Post{
	fn summerize(&self) -> String{
		println!("{},{}", self.title, self.size);
	}
}
```



可以看到引入特征会需要修改结构体的方法（因为它需要增加相应的函数）

所以有一个**孤儿规则**，就是如果想要给一个结构体加上一个特征，那么至少有一方是当前作用域下的（就是你自己写的），而不能给标准库里的一个结构体加一个标准库里的特征，以免破坏了别人的代码或者别人破坏了你的代码。



#### 用特征修饰函数参数

```
pub fn notify(item: &impl Summary) -> () {
	println!("{}",item.summersize());
}
pub fn notify<T: Summary>(item: &T) -> (){
}
```

这样就是说 item 的类型是实现了 Summary 特征的引用类型。

它其实就是底下这种写法的语法糖

其中 `T:Summary` 称为特征约束

但是这种语法糖不是所有情况都适用，例如下面这个例子

```
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

虽然表达了它们俩都实现了 Summary 特征，但是无法表达这两个是相同类型的意思



#### 多重特征的表示形式

特征之间可以用 `+` 连接

```
fn notify<T: Summary + Display>(item: T){}
fn notify(item: &(impl Summary + Display)){}
```



#### where 约束

使用 `where` 可以把一些写在`<>`中比较复杂的特征用`where`写在后面：

```
fn somefunction<T: Clone + Display, U: Clone + Debug>(t: &T, u: &U){}
fn somfuntion<T, U>(t: &T, u: &U)
	where T: Clone + Display,
	U: Clone + Debug
{}
```



#### 使用特征约束有选择地生成方法或特征

特征约束指的是泛型变量的特征约束，可以通过这种方法，对一些有泛型定义的结构体，有选择性的获得方法或者特征

一个有选择性的获得特征的例子：

```
use std::fmt::Display;
struct Pair<T>{

}
// 只有在 T 满足这两个条件下，才会定义下面的方法
impl<T: Display + PatialOrd> Pair<T>{
	
} 
// 对所有满足 Display 特征的类 T 实现了 ToString 特征
impl<T: Display> ToString for T{

}
```



#### 在函数的返回中使用 impl Trait

这种情况只在你不知道如何表达函数的返回类型的时候，只知道它满足什么条件的时候使用

但是这个并不能代替泛型，因为它只能返回一种类型，如果有多种可能的返回类型，编译就无法通过

```
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        }
    }
}
```



#### 修复例子

这里的类型 `T` 需要用 `<` 大小，需要做一个复制，而不是通过一个引用转移所有权 `largest = list[0]`

所以需要 `std::cmp::PartialOrd`和`Copy`，不过`PartialOrd` 已经在 `prelude` 中了，所以不用加 `std::cmp` 的前缀

```
fn largest<T: PatialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

还有一种不要求实现`Copy`的方法，就是全程使用引用来进行计算

```
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];	// 这里 largest 是一个对最大值的引用

    for item in list.iter() {	// 注意这里也要改成 item = iter() 而不能是 &item = iter() ，因为这样 item 就会尝试等于 *iter() 去获取所有权
        if (*item) > (*largest) {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = *largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = *largest(&char_list);
    println!("The largest char is {}", result);
}
```



#### 使用 derive 派生特征

`#[derive(Debug)]` 如果之前对象被 `derive` 标记了，就会自动实现 `Debug` 特征中的默认代码，就可以通过`{:?}` 输出对象

`#[derive(Debug)]` 就会让它们自动实现事先默认的自我复制函数

加在对应的结构体的前面就可以生效了。



#### 调用特征的方法，需要把对应的特征引入作用域

```
use std::convert::TryInto;

fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  let b_ = b.try_into()
            .unwrap();

  if a < b_ {
    println!("Ten is less than one hundred.");
  }
}
```

这里的 `try_into()` 是`TryInto` 中定义的，所以需要引入。

rust 会默认将 `std::prelude` 中的模块提前引入。



#### 几个例子

```
use std::ops::Add;

// 为Point结构体派生Debug特征，用于格式化输出
#[derive(Debug)]
struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
    x: T,
    y: T,
}

// 对于满足加法的结果还是T的类型的 Point<T> 也实现 Add 特征
impl<T: Add<T, Output = T>> Add for Point<T> {
    type Output = Point<T>;

	// 对这类 Point<T> 实现 add 函数
    fn add(self, p: Point<T>) -> Point<T> {
        Point{
            x: self.x + p.x,
            y: self.y + p.y,
        }
    }
}

// 对满足 Add 的 T 实现 add 函数
fn add<T: Add<T, Output=T>>(a:T, b:T) -> T {
    a + b
}

fn main() {
    let p1 = Point{x: 1.1f32, y: 1.1f32};
    let p2 = Point{x: 2.1f32, y: 2.1f32};
    println!("{:?}", add(p1, p2));

    let p3 = Point{x: 1i32, y: 1i32};
    let p4 = Point{x: 2i32, y: 2i32};
    println!("{:?}", add(p3, p4));
}
```

