# 集合类型

## Vec

### 新建动态数组

`Vec::new()` 创建空的

`Vec![]` 使用宏

### 更新

在后面添加一个元素 `v.push()` 但是修改需要先设置其为 `mut` 才可以修改

还可以增加一个数组 `v.extend([])`

### 从 Vector 中读取元素

`v[i]` 如果越界会直接报错

`v.get(i)` 返回是 Option 类型，需要用 `match` 拆

### 同时借用多个数组元素

```
let mut v = vec![1,2,3,4,5];
let first = &v[0];
v.push(6);
println!("{}", first);
```

`v.push()` 需要使用 可变引用，但是可变引用和不可变引用只会用一个，所以这个会报错

### 储存不同类型的元素

就是对之前知识的引用：
可以通过枚举，也可以通过特征对象。

其中特征对象使用比较多，因为可以动态增加类型。

### 使用 from/into 将一些常见类型转成 Vec

可以使用的有 `[T; N]` `String` `str` `.into_iter().collect()`

只要为 `Vec` 实现了 `From<T>` 特征，那么 `T` 就可以被转换成 `Vec`。



## Key-Value 存储 HashMap

### 创建

```
use std::collections::HashMap;

let mut mymap = HashMap::new();
mymap.inset("x", 1);
```

因为HashMap不在prelude中，所以需要用到 `std::collection::HashMap`，上图中`mymap`类型是 `HashMap<&str, i32>`

在一个Map中，所有的K必须是同一类型，V也必须是同一类型。

这里 Key 的类型是所有实现了 `Hash` 和 `Eq` 两个特征的类型，包括：
1. bool
2. int, uint
3. String, &str

注意 浮点数没有实现 `Hash`

如果一个集合类型的所有字段都实现了`Eq` 和 `Hash`那么这个集合类型会自动实现`Eq`和`Hash`

如果一个结构体类型的所有字段都实现了 `Eq` 和 `Hash`，还需要对它做一下派生，`[derive(PartialEq, Eq, Hash)]`



还可以使用迭代器来把一个 元组的 Vec 转成 HashMap

```
let team_map: HashMap<_,_> = team_list.into_iter().collect()
```

还可以直接用`from`

```
let team_map = HashMap::from(team_list);
```



### 所有权转移

和 `Vec` 一样，也是有 `Copy` 就不转移，其他情况就会转移

如果将引用类型放入`HashMap`，那么这个引用的对象需要比`HashMap`更长或者相等的生命周期

### 查询

使用 `get(Key) `来查询，会返回一个 `Option<&V>` 如果有则返回 Some(&) 【注意这里是一个引用】，没有则返回 None

同时也支持遍历

```
for (key, value) in &map{
}
```

### 更新

如果 key 相同， insert 就会变成覆盖

还有一种查询并插入的方法

```
let v = scores.entry(key).or_insert(value)
```

如果有这个 key 就返回，如果没有就插入 (key, value)

这个 v 得到的是一个指针，*v 可以改变和查询这个 key 下的 value

```
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();
// 根据空格来切分字符串(英文单词都是通过空格切分)
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

例如这个查询单词出现次数的例子

### 哈希函数

HashMap 使用的是数学安全的哈希函数，如果需要效率更高，需要寻找其他的哈希实现

```
use std::hash::BuildHasherDefault;
use std::collections::HashMap;
// 引入第三方的哈希函数
use twox_hash::XxHash64;

// 指定HashMap使用第三方的哈希函数XxHash64
let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
hash.insert(42, "the answer");
assert_eq!(hash.get(&42), Some(&"the answer"));
```

这里就是一个设置默认哈希的例子

### 容量

```
with_capacity()	// 初始化容量
shrink_to() 	// 缩小容量
shrink_to_fit()	// 自动缩小容量到合适
```



