# Day6

## 复合类型

### 元组

定义就是用括号括起来的不同类型的变量，取值两种方法：一是模式匹配一一对应就可以了，二是用.加下标取值。

```
let tup:(i32, f64, u8) = (500, 6.4, 1);
let (x,y,z) = tup;
let x = tup.0;
let y = tup.1;
```

它的使用场景：例如可以用来让函数返回多个值

#### take-home

用 println!("{:?}", tup) 最多输出包含12个元素的tup，元素可以是递归的tup，但是一个tup中最多12个元素



##### take-home 疑问

```
fn main() {
    let (x, y, z);

    // 填空
    (y, z, x) = (1, 2, 3);
    
    assert_eq!(x, 3);
    assert_eq!(y, 1);
    assert_eq!(z, 2);
}
```

这个先用 let 再直接赋值是为什么？

如果写的是 `let (y, z, x) = (1,2,3)` 反而会报错



### 结构体

#### 定义：

```
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

所有字段都要有类型，不允许某个字段为可变类型

#### 实例化：

```
    let [mut] user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
```

每个字段都需要初始化

#### 访问字段

```
user.email
user.email = "" // 要求 user 是可变的。
```

#### 简化构造

当函数参数名和结构体字段名同名时，可以用缩略的方法进行初始化

这里也不一定要是函数参数名，而是随便一个临时变量名相同就可以用

```
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

#### 简化复制构造

```
  let user2 = User {
        email: String::from("another@example.com"),
        ..user1		// 注意这里后面没有,了
    };
```

如果只需要修改一个字段，可以用`..user1`来代替，它只能放在构建结构体的末尾。

不过要注意，这里如果 user1 中有一些没有自动做复制的类型，例如 `string` 类型，就会发生所有权转移，而就不再能使用它，但是其他的字段不会受到影响。

这是因为例如 `String` `Vec<u8>` 这种类型它们的本质是一个指针加一个大小加一个容量组成的，所以赋值的时候把指针给出去了所有权就没有了。

#### 元组结构体

字段是可以没有名称的（这个应该只适用于元组结构体的情况，就是所有的字段都没有名称，如果有的有有的没有是不行的），如果所有的字段都没有名称，就称为元组结构体。

```
struct Point(i32, i32, i32);
let x = Point(1,2,3);
```

这个适用于每个意义都很清楚的字段，例如Color的 RGB，Point的xyz。

但是也要注意一点，它和元组是不同的，所以你需要用对应的类型才能做模式匹配。

```
let (x,_,_) = Point(1,2,3); 	// wrong
let Point(x,_,_) = Point(1,2,3) // right
```



#### 单元结构体

就是没有字段的结构体。

不关心它的内容，只关心它的行为的时候使用。

#### 暂时不要在结构体中使用引用

需要声明周期才能只用`&str`这种类型

声明周期大概就是说明了一个变量的生存时间，那么在结构体中需要注意，不要让一个字段的声明周期短过这个结构体的声明周期，

如果这个周期比它短，也就是说这个指针还存在，但是它指向的内容已经消亡了，这样就会出现野指针，所以有这样的要求，我们在定义的时候也需要加上 <`a> 来对这样的变量声明它的生命周期，但是实例化的时候不要自己填写，而是编译器可以自动填写。

#### 结构体的输出

在代码前面加上

```
#[derive(Debug)]
```

就可以通过 `{:?}` 来输出了

如果结构体比较大，希望输出多行，则可以使用`{:#?}`



如果想要使用 `{}` 来输出，就需要实现 `Display` 这个特征



### 枚举

枚举是一个类型，声明是包含了所有可能的**枚举成员**，枚举值就是其中的某一个**枚举成员的实例**

使用 `::` 来访问具体的成员

```
enum PokerSuit {
  Clubs,
  Spades,
  Diamonds,
  Hearts,
}
let heart = PokerSuit::Hearts;	// 这里 heart 就是 PokerSuit 类型的
```

枚举中每个值都可以翻译成一个对应的数字，

声明时：

```
enum PokerSuit {
  Clubs = 0,
  Spades = 1,
  Diamonds = 2,
  Hearts = 3,
}
```

使用时

```
PokerSuit::Clubs as u8
```



#### 枚举除了基础值还可以做拓展

```
enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(u8),
    Hearts(u8),
}

fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds(13);
}
```

这样就可以代替多一个结构体的麻烦。

任何类型的数据都可以放入枚举类型中。这样就可以在相同类型的枚举变量中，处理不同类型的数据了

```
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

#### Option 枚举 来处理空值

```
enum Option<T>{
	Some(T),
	None,
}
```

这个枚举内置在rust的预定义中了，所以可以直接在代码中使用它们

```
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

这里 Some 会和一个 T 类型的数据绑在一起，表示它是一个 Option<T> 类型的枚举类型

这里 None 因为没有办法推断，所以需要显式在前面声明它是一个 Option<i32> 类型的枚举类型。

这个好处是，当我们想要对一个 Option<T> 类的变量进行运算的时候，先要把它翻译成 <T> 类，而此时就可以帮我们判断它是不是空的。

在应用一些需要有一些值可能为空的情况的时候，就可以把这个类型定义成 Option<T> 来防止空值泛滥。

#### 枚举实现列表

```

// 填空，让代码运行
use crate::List::*;

enum List {
    // Cons: 链表中包含有值的节点，节点是元组类型，第一个元素是节点的值，第二个元素是指向下一个节点的指针
    Cons(u32, Box<List>),
    // Nil: 链表中的最后一个节点，用于说明链表的结束
    Nil,
}

// 为枚举实现一些方法
impl List {
    // 创建空的链表
    fn new() -> List {
        // 因为没有节点，所以直接返回 Nil 节点
        // 枚举成员 Nil 的类型是 List
        Nil
    }

    // 在老的链表前面新增一个节点，并返回新的链表
    fn prepend(self, elem: u32) -> List {
        Cons(elem, Box::new(self))
    }

    // 返回链表的长度
    fn len(&self) -> u32 {
        match *self {
            // 这里我们不能拿走 tail 的所有权，因此需要获取它的引用
            // 这个 ref 相当于让 tail 变成一个对后半部分的引用
            Cons(_, ref tail) => 1 + tail.len(),
            // 空链表的长度为 0
            Nil => 0
        }
    }

    // 返回链表的字符串表现形式，用于打印输出
    fn stringify(&self) -> String {
        match *self {
            Cons(head, ref tail) => {
                // 递归生成字符串
                format!("{}, {}", head, tail.stringify())
            },
            Nil => {
                format!("Nil")
            },
        }
    }
}

fn main() {
    // 创建一个新的链表(也是空的)
    let mut list = List::new();

    // 添加一些元素
    list = list.prepend(1);
    list = list.prepend(2);
    list = list.prepend(3);

    // 打印列表的当前状态
    println!("链表的长度是: {}", list.len());
    println!("{}", list.stringify());
}
```



### 数组

这里主要讲的是 `arr` 定长数组

#### 定义

```
let a = [1,2,3,4,5];
let b:[i32; 5] = [1,2,3,4,5];
let c = [3; 5];					// [3,3,3,3,3]
```

#### 使用

```
a[0]
```

一旦越界访问就会报错

#### 切片

```
let b:&[i32] = a[1..3] 	// 类型中不再需要包含长度
```

#### 二维数组

```
let arrays: [[u8;3];4] = [[0;3];4]
for a in &arrays {
	for n in a {	//也可以是 in a.iter()
	
	}
	for i in 0..a.len(){
	
	}
}
```



## 流程控制

### 分支控制

`if` 是表达式，可以在每个分支中放表达式，会根据condition返回表达式

每个condition中的表达式的类型要一致

```
if condition1{
	expr1
} else if condition2{
	expr2
} else {
	expr3
}
```

和其他语言一样，从上往下，找到第一个满足的运行，其他的就算满足也不会执行

### 循环控制

for, while, loop. continue, break

#### for

最基础的还是

```
for i in 1..-5{
	
}
```

但是最实用的是类似 python 中的

```
for item in &container{
	// 不能修改集合中的元素
}
for item in &mut container{
	// 可以修改集合中的元素
}
```

| 使用方法                      | 等价使用方式                                      | 所有权     |
| ----------------------------- | ------------------------------------------------- | ---------- |
| `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
| `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
| `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

如果同时希望有一个索引：

```
for (i,v) in a.iter().enumerate(){
	
}
```

#### while

```
while condition{

}
```

#### loop

无条件循环

```
loop{

}
```

它一般都是搭配 break 使用，rust 中的 break 是可以带一个返回值的，和 return 一样，同样的，loop 也是一个表达式，返回值等于 break 的返回值。



`for` 可以不用 index 去访问数组，所以效率更高，因为实用 index 会需要做边界检查

#### continue 和 break 对外层循环生效

当有多层循环嵌套的时候，我们可以在外层的循环上做上标签，这样在使用 continue 和 break 的时候加上这个标签，就可以实现对外层循环的控制了。



## 模式匹配

### match

match 和 switch case 很像：

```
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {	// | 表示或
            println!("South or North");
        },
        _ => println!("West"),						// _ 表示 default
    };
}
```

但是也有不同，就是在输入的 变量 和下面的 条件 比较时，使用的是模式匹配。

第二个不同就是 match 本身是一个表达式，它是带返回值的。

第三个不同就是 match 必须要匹配所有的可能性，如果有遗漏就会报错。



match 使用模式匹配的形式，就可以帮忙取出一些绑定的值，例如 enum 中所绑定的值。

```
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

### if let

if let 通常是在分支数很少（通常是1）的情况下，又需要做 模式匹配 来取出绑定值的时候使用的

```
fn main() {
    let v = Some(3u8);
    match v {
        Some(3) => println!("three"),
        _ => (),
    }
}
```

```
if let Some(3) = v {
    println!("three");
}
```

### matches! 宏

```rust
matches!(n1, n2);
```

这个宏计算就是判断一下n2是否是n1的模式串，是就返回true，不是就返回false

```
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));

let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```

### 模式匹配时的变量覆盖

在一个 `{}` 的作用域里进行模式匹配时，用一个新的变量（通常指同变量名时）匹配了原本的值，那么在这个作用域里，这个变量名绑定的就是匹配得到的新的值



### 解构 Option

用 match 就很容易可以做到

需要注意就是这里还是需要对 match 的返回值的类型做一个声明，因为可能返回是 None，那么就需要一个类型Option<T>的声明。



### 总结 - 模式适用的场景

模式串包括：

字面值；解构的数组、枚举、结构体或者元组；变量；通配符；占位符

模式匹配要求：两边的变量的类型一定要相同

可以用到的地方：

match 分支中的每个分支都是一个模式

if let 的等号左边是一个模式串，右边是变量

while let 和 if let 一样，只要条件里的这个模式能匹配上就会一直循环

for 循环中，如果同时使用索引值和元素的话， `for (index, value) in v.iter().enumerate()` 这里就有元组的模式匹配

let 语句本身就是模式匹配

函数参数 也可以匹配元组来拆分元组

对于一些无法确定的进行模式匹配的情况，就不能使用 let ，必须使用 if let：

```
let Some(x) = some_option_value; 	// wrong
if let Some(x) = some_option_value{
}									// right
```



### 总结 - 全模式列表

#### 匹配字面值

相当于 = 的判断

```
let x = 1;
match x{
	1 => ,
	2 => ,
	_ =>
}
```

#### 匹配命名变量 / 解构并分解值

就是用一个变量去绑定一些复合类型中的元素的值

在这个生命周期内这些变量的名字会优先使用。

#### 单分支多模式

可以使用 `|` 来匹配多个模式

#### 使用序列 ..= 来匹配值的范围

```
let x = 5;
match x{
	1..=5 =>,
	_ =>
}
```

#### 嵌套解构

如果是一个嵌套的复合类型，那么也可以用嵌套的模式串来对它进行解构

#### 忽略模式中的值

`_` 可以忽略所有值，或者占某个位，忽略某个值，也可以放在变量名的前面，让编译器忽略未使用的变量

`..` 可以忽略所剩部分的值，也可以加上尾巴或者头部，省略中间部分的值，但是需要保证 `..` 是没有歧义的

#### 匹配守卫

可以在模式串后增加一个 `if` 来表示这个模式串的匹配守卫，例如：

```
match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

这个 `Some(x) if x < 5` 就是一个匹配守卫

对于使用 `|` 连接的多个模式，在其后面加的 `if` 的模式守卫也会作用于所有的模式，也就是只要通过这个模式守卫，得到的元素一定满足模式守卫中声明的条件

#### @ 绑定

使用的方法就是在一个模式串前面加一个变量名并加一个`@`

```
match p{
	Hello{id: 1..10} // 不加 @，不会绑定值，只会判定范围
	Hello{id: id_tmp @ 1..10} // 不仅判定范围，而且会把id的值给 id_tmp
}
```

它不仅可以用来做解构，也可以和解构一起使用

```
let point = Point{x: 10, y: 5};
if let p @ Point{x: 10, y} = point{		// 这里的 y 相当于 y:y
	// 这里你就可以同时使用 y 和 p，这里的p就等于匹配上的 Point
}
```

使用 @ 做匹配的时候，如果后面有多个模式串，需要把后面的模式串括起来

```
fn main() {
    match 1 {
        //num @ 1 | 2 => {		// wrong
        num @ (1 | 2) => {		// right
        	println!("{}", num);
        }
        _ => {}
    }
}
```





#### Take-home 难题

如何改正确

```
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;

    match r {
       &mut value => value.push_str(" world!") 
    }
}
```

官方改法：

```
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;
    
    match r {
       value => value.push_str(" world!") 
    }
}
```

理解：

`v` 是一个 `String` 类型的变量，所以它其实是一个栈上指针，指向一个堆中的区域

`r` 是一个可变引用，它的目标其实是 `v` ，是针对这个栈上的指针的，对`r` 进行的操作：例如`r.push_str(" world!")` 的本质其实是`(*r).push_str`也即是`v.push_str` 【这里也可以看到 RUST 中虽然有*取引用，但是没有 -> 和 . 的区别，r作为引用也是.】

那么原题中无法通过 `match` 的手段将 `&mut value = r` 来让 `value = v`对 `value` 的修改也不会影响到`v`

官方这样改就相当于

```
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;
    
    match r {
       _ => r.push_str(" world!") 
    }
}
```

不过也有少许的不同，就是官方的写法中 r 的所有权会给到value然后消失，而这种写法 r 的所有权不会消失

```rust
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;

    match *r {
       ref mut value => {value.push_str(" world!"); r.push_str(" world!")} 
    }
    
    println!("{:?}", (*r));
}
```

这种方法`r`也不会消失，甚至短暂的出现了对 v 的第二个可变引用 value ？

其实并不是BUG，这个是和 NLL 有关，就是 r 还没有对 v 进行操作之前不认为它已经创建出来了，所以之前也只有 value 一个可变引用，之后value再没有用过，所以value就消失了，而 r 就有了。

```rust
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;

    match *r {
       ref mut value => { r.push_str(" world!"); value.push_str(" world!")} 
    }
    
    println!("{:?}", (*r));
}
```

这样的话就会报错了，因为 r 和 value 的生命周期有重叠了。

同时也要注意这里的这个 `ref` 的使用，使用了 `ref` 系统就不会尝试做所有权的转移，也就是说 *r 的所有权不会转移到 value 上

```
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;

    match *r {
       mut value => value.push_str(" world!")
    }
    
    println!("{:?}", (*r));
}
```

而如果你去掉 ref，这里就是会报错的，因为它会尝试让 value = (*r) 而这里 *r 的所有权并不能通过 r 获得，所以所有权会发生错误



#### take-home 一些困惑

关于为什么会报错的问题：

```
fn main() {
    let mut v = String::from("hello,");
    let r = &mut v;

    match r {
       &mut value => ()
    }
}
```

上面这个程序是会报错的



```
fn main() {
    let mut a = 1;
    let r = &mut a;
    match r {
    // &mut value => value = 6 // wrong
    // &mut mut value => value = 6 // right 但是后面的a的值不会变
       &mut value => *r = 6
    }
    println!("{}", a);
}
```

下面这个不会报错



就是关于能否将一个数匹配成一个 `&mut value` 也是有讲究的：

1.  `r` 必须是一个可变的引用
2. `r` 不能是一个没有定义 Copy 类型的变量的可变引用

【解答】

这里就是相当于做了一次复制操作，让 `value = (*r)` ，那么对于 `String` 类型不会做 `Copy`，而对于`i32`类型就可以复制

对于一个 &mut 类型的 `r`

`match r { &mut value => ()}` 就等于 `match *r {value => ()}`

而这里的话因为 *r 是没有所有权的，因为 r 是一个可变指针，所以需要在 value 前加上 ref，表示我也是只取一个引用，而不需要你的所有权。



有一个 rust 官方的总结：

https://github.com/rust-lang/rfcs/blob/master/text/2005-match-ergonomics.md