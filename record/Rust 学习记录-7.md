# Day-8

# 特征

## 特征 - take home

###  在特征中使用 Self 指代获取特征的结构体类型

```
trait Animal{
	fn new() -> Self;
}
impl Animal for Sheep{
	fn new() -> Sheep{
	
	}
}
let a = Animal::new(); // error
let b:Sheep = Animal::new(); // right
let c = Sheep::new(); // right
```

### [derive()]

注意这里的不同的特征之间是用逗号隔开，而不是`+`

### 神奇的Box

```
fn random_animal(random_number: f64) -> Box<dyn Animal> {
    if random_number < 0.5 {
        Box::new(Sheep {})
    } else {
        Box::new(Cow {})
    }
}
```

### 函数泛型

```
struct Cacher<T: Fn(u32) -> u32> {
   calculation: T,
   value: Option<u32>,
}
```

`Fn(u32) -> u32` 就是一个函数泛型，形参为 `|x| x+1`



## 特征对象

rust 中的对象是没有继承的，如果想要遍历多个不同的对象，并对它们做相同的操作，就需要特征对象。

所以这个对象其实是针对特征而言的，能调用的方法也只有特征中所定义的方法

### 定义

假如 `A` 和 `B`都实现了`Draw`特征，那么有个地方需要传入一个实现了 `Draw` 特征的某个结构体的时候，就可以写：

`x: Box<dyn Draw>`	或者 `x:&dyn Draw`

传入时

`Box::new(a)`或者 `&a`

它们传进去之后，想执行都是直接 `x.draw()`就可以了

Box操作：在堆上分配一个对象，然后用一个栈上的指针指向它。



### 特征对象的动态分发

静态分发 是指在编译器完成了对泛型等的操作

动态分发 则是在说特征对象在运行的时候才知道需要调用什么方法，`dyn` 关键字也是动态的意思

根据动态分发的特点，也可以得到特征对象的特点：

1. 大小不固定
2. 几乎总是使用引用方式（引用大小是固定的，都是两个字）



### Self 和 self

`self` 是对象

`Self` 是对象类型



### 对象安全

只有对象安全的特征才能拥有特征对象

对象安全需要满足：

1. 返回类型不能是 `Self`
2. 方法没有任何泛型函数

【这里我的理解是说，因为特征对象是动态分发的，而泛型和Self是静态分发的，所以不能一起使用】

例如 `Clone` 因为要返回 `Self` 类型，所以不是对象安全的。



### Take-home

如果想要声明一个包含多个类型的数组，那么就一定要声明这个数组的类型，并使用特征对象。

`    let birds:[Box<dyn Bird>; 2] = [Box::new(Duck), Box::new(Swan)];`



如果一个特征使用了 `Self` 你可以把它改成 `Box<dyn Trait>` 并对应的改。



## 高阶特征

### 关联类型

有点像关联函数的使用方法。

是可以在特征中定义的类型：

```
pub trait Iterator {
	type Item;
	fn next(&mut self) -> Option<Self::Item>;
}
```

这个是定义特征

```
impl Iterator for Counter {
	type Item = u32;
	
	fn next(&mut self) -> Option<Self::Item> {
		//
	}
}
```

这个是在要应用这个特征的结构体上的实现，需要对每一个用这个特征的结构体都需要定义自己的关联类型。

可以在某种程度上减少泛型的复杂度：

例如一个需要两个泛型的特征：

```
trait Container<A, B> {
	fn contains(&self, a: A, b: B) -> bool;
}
```

如果需要定义一个涉及这个特征的函数，就必须把这两个泛型加上：

```
fn difference<A,B,T>(contianer: &T) -> i32
where T: Container<A,B>{}
```

这里类型 T 需要是一个满足 Container 特征的类型，但是 Container 自己带两个泛型参数，所以写起来很麻烦，就可以把这两个泛型类型作为关联类型放在里里面：

```
trait Container{
	type A;
	type B;
	fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
}
fn difference<T: Container>(container: &T){}
```

### 默认泛型类型参数

当给特征加一个泛型参数的时候，可以给他一个默认的具体类型，例如标准库中的加法

```
trait Add<RHS=Self> {
	type Output;
	
	fn add(self, rhs: RHS) -> Self::Output;
}
```

这里就是你可以为加法设置一个加数的类型，如果不设置，就会默认加自己

应用的时候，如果不需要设置其他的加法

```
impl Add for struct{

}
impl Add<SOME_TYPE> for struct{

}
```



### 调用同名的方法

如果不同的特征中有同名的方法，甚至结构体自己有自己的同名方法，会默认调用方法

如果想要调用特征中的方法，则需要使用 `Trait::func(&struct)` 这种形式取调用

这种是对于函数中有引用 `Self` 的方法可以这样使用

如果对于没有 `Self` 的方法，例如关联函数，就需要使用完全限定语法：

```
<Type as Trait>::function()
```



### 在特征定义上加特征约束

```
use std::fmt::Display;

trait OutlinePrint: Display{
	// ... .to_string()
}
```

这个 `OutlinePrint` 就是要求其同时具有 `Display` 的特征



### newtype

之前有一个孤儿原则，就是不能对一个标准库的类型实现一个特征

但是可以定义一个新的 type，只包括一个已知类型本身，然后对这个包装后的类型实现特征