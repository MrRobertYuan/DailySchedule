# Rustlings

## Collections

### HashMap

1. 新建的方法，`let basket:HashMap<String, i32> = HashMap::new()`而不是 `let basket = HashMap<String, i32>:new()`
2. 查询和插入的方法 `entry().or_insert()`可以一步搞定

### Vec

1. 新建 Vec 并让它等于某个 [] 的方法，可以使用 `Vec::from()` 也可以 `vec!{}` 也可以一个个加
2. 遍历一个 `Vec` 的方法，可以通过 `for i in v.iter_mut{ *i }` 



## String

只考察了 String 和 &str 的区别



## quiz2

也是对 String 和 &str 的考察

1. to_owned() 通过切片生成原本的，所以返回 String
2. format 返回 String
3. .into() from的逆操作，返回 String
4. trim() 去掉两边的空格，不改变自己，返回切片
5. replace() 返回 String
6. to_lowercase() 返回 String



## Error_Handling

1. Result<T1, T2> 可以返回 Ok(T1) 和 Err(T2)
2. string.parse::<i32>() 的返回值是一个 Result<i32, ParseInt> 类型
3. `?` 方法可以自动进行错误级提升，例如将`io::Error` 转为 `error::Error`，`?` 的正确值需要一个人接住，它会自动返回错误值，但是不能返回正确值，特别的如果这个函数是 main 函数，必须返回`()`，则可以返回类型设置为 `Result<(), Box<dyn Error>>` ，记得需要 `use std::error::Error`
4. 常常需要和 Match 搭配使用



`map` `map_err`

都是传入一个可以转换一种类型到另一种类型的函数

`map`可以`fn (T) -> F`转 `Some(T)``None``Ok(T)` 到 `Some(F)``None``Ok(F)`

`map_err` 可以`fn (T) -> F` 转`Err(T)`到`Err(F)`



## Option

1. 类型记得匹配
2. if let 和 while let 用来解绑 Option 非常合适
3. 在 `match`的时候成功匹配了，会根据你设置的值转移所有权，那么你可以设置 `let Some(ref p)=option` 这样可以取引用而不转移所有权了



## Trait

1. 给结构体实现特征，就是实现对应的函数
2. 特征中写 `self` ，实现对应特征的时候可以改成 `mut self`



## Test & quiz3

稍微不太知道它让我们做什么，如果只是填空也太简单了

1. 使用 assert!(true)
2. 使用 assert_eq!(a, b)
3. 用 assert! 判断函数是否正确



## standard_library types

### Rc 和 Arc

Rc(Reference Counting)

多个引用指向一个数（不可变引用），并对引用计数，归零时表示这个数不再可用，适合多人引用而不知道谁最后结束

创建的方法：`let a = Rc::new(String::from("hello"));`

复制引用的方法： `let b = Rc::clone(&a);`

查看计数的方法：`Rc::strong_count`



Rrc(Atomic Rc)

 API 相同，但是线程安全



### Box

用 Box 可以放一些不知道大小的变量，因为它本身是一个指针，大小是一定的，而它指向的数据是堆上的，可以不限大小。



### Iterators

生成迭代器`arr.into_iter()`

迭代器的特征：

```
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```

手动实现迭代器一定要声明为可变的，才可以用`next`跳到下一个，而且返回类型是 `Option`

`into_iter()` 夺走所有权

`iter()` 借用

`iter_mut()` 可变借用

能使用 `into_iter()` 的特征：

```
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```

使用 `.collect()` 可以将 `iter` 变回数组



这里的第二题做的有点丑（写法比较C）

第三题有一问是一个个加进去的，第二问用了 `。collect()`

第四题很巧妙，首先需要找到和 `sum()` 一样的`product()`累乘，然后要知道序列可以生成迭代器

`(1..=num).into_iter().product()`



还是决定好好学习一下迭代器再开始做



## 迭代器 - 学习

迭代器允许我们迭代一个连续的集合，例如数组、Vec、HashMap，只需要关心对每个迭代器的处理，不用关心何时开始何时结束，按照什么样的索引去访问的问题

### For 循环与迭代器

使用 `for` 对数组访问其实是语法糖，是因为数组实现了 `IntoIterator` 的特征，同样的还有序列

实际等于 `for a in arr.into_iter()`

```
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```

### 惰性初始化

创建和使用是分开的，创建时不需要性能消耗，使用的时候才会开始

### next 方法

```
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```

迭代器实现了 `Iterator` 特征，可以用 `next` 取值，for的本质也是不停的使用 `next` 取值

这种方法是消耗性的，直到返回 `None` 为止

### 消费者和适配器

消费者是迭代器上的方法，会消费掉迭代器中的元素，然后返回其类型的值

都是通过 `next()` 方法来消费元素的

#### 消费者适配器

只要迭代器的某个方法在内部使用了 `next` 方法，那么它就是一个消费性适配器。

例如 `sum()` 就是不断调用 `next` 来进行求和，在使用完 `sum` 之后，该迭代器的所有权就没有了

`skip(n)` 就是相当于执行`n`次`next()`

#### 迭代器适配器

这个方法会返回一个新的迭代器，这也是实现链式调用的关键

不过迭代器适配器是惰性的，需要一个消费适配器来收尾，将迭代器转为一个具体的值

例如一个很常见的 `.collect()` 可以把迭代器变回数组

不过 collect() 需要等式左边显式的声明类型，例如 `Vec<_>`

`zip()` 是一个迭代器适配器，它是把两个迭代器合在一起变成 `Iterator<Item=(ValueA,ValueB)>`的新的迭代器，这样就可以通过 `collect()` 变成一个 `HashMap`

`map()` 也是一个迭代器适配器，可以对值进行操作，变成另一个值，可以通过闭包作为参数

`filter()` 也是一个迭代器适配器，也可以传闭包，闭包可以捕获环境值（即迭代器之外的值进行筛选）

### 实现 Iterator 特征

定义好 `Item` 和 `fn next(&mut self) -> Option<Self::Item>{}`就可以让一个结构体拥有 `Iterator` 特征了

而很多其他的函数都是依据`next` 做的，所以会自动实现 `zip()`，`map()`,`filter()`,`sum()`等函数

#### enumerate()

可以通过这个生成一个迭代时的索引，有点像`zip()`也是一个迭代器适配器



## Iterator 重做

首先解决第五题变得简单了：

首先需要学一个 `.count()` 是一个消费适配器，可以计算迭代器的数量

对于 `HashMap` 就可以直接用 `filter()` 加 `.count()` `map.into_iter().filter(|&(k,&v)| v == value).count()`，对于 `collection` 就可以先 `map` 将`HashMap` `map` 成一个`count`的值，然后取 `.sum()` 非常方便。

重写第二题，`for` 循环改成使用 `iterator()` 看起来厉害了很多

`words.into_iter().map(|&word| capitalize_first(word)).collect()`

注意 `sum()`方法只能对数字进行计算，而不能对 `String` 进行计算，所以这里用了一个 `for_each`

`words.into_iter().map(|&word| capitalize_first(word)).for_each(|word| ans += &word);`

重写第三题，对错误的处理一般般，但是也写成了只用迭代器的方式：

`let division_results:Vec<i32> = numbers.into_iter().map(|n| divide(n, 27)).map(|n| match n{Ok(n) => n, Err(e) => 0}).collect();`

这里如果除法出错了就只是填上了0，而没有返回错误，比`for`的写法要差一些