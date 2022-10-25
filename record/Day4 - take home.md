# Day4 - take home

## Practice

### BOX

如果想用 `str` 必须借助 box

```
let s:Box<str> = "hello".into();
// 此时 &s : &str
```

Box 这个还有点奇怪，昨天的问题也涉及到了这个

```
let s:Box<&str> = "hello".into();
// 此时 *s: &str
```



### 空字符串

```
let s = String::new();
```

这个函数可以生成初始大小为 n 的String

```
let s = String::with_capacity(n);
```



### 转义字符串

```
\u{}	// 用来写 Unicode 字符
```

```
"......\		// 可以用来写多行字符串
........"
```



易错点：

```
    let long_delimiter = r###"Hello, "##""###;
    assert_eq!(long_delimiter, "Hello, \"##\"")
```

下面这个式子其实比较的是 long_delimiter 和 Hello, "##"

只是因为 " 需要转义，所以才这样的。



### 切片

```
fn main() {
    let arr: [char; 3] = ['中', '国', '人'];

    let slice = &arr[..2];
    
    println!("{:?}", slice);
    
    // 修改数字 `8` 让代码工作
    // 小提示: 切片和数组不一样，它是引用。如果是数组的话，那下面的 `assert!` 将会通过： '中'和'国'是char类型，char类型是Unicode编码，大小固定为4字节，两个字符为8字节。
    assert!(std::mem::size_of_val(&slice) == 16);
}
```

注意这里是字符数组而不是字符串！

然后这里求的也是 `&slice` 而不是 `slice` 的大小，`slice` 的大小是 2*4 = 8，而一个切片引用的大小是两个字(usize)

```
fn main() {
    let arr = ['中', '国', '人'];

    let slice = &arr[0..2];
    
    // 修改数字 `8` 让代码工作
    // 小提示: 切片和数组不一样，它是引用。如果是数组的话，那下面的 `assert!` 将会通过： '中'和'国'是char类型，char类型是Unicode编码，大小固定为4字节，两个字符为8字节。
    assert!(std::mem::size_of_val(slice) == 8);
}
```

所以这样也是对的。



### String & &str

String 转为 &str 有两种方法

```
str = &string
str = string.as_str()
```



记得可以使用可变引用来改变字符串的值

```
let slice3: &mut String = &mut s;
```



遍历一个 UTF-8 字符串中的所有字符

```
for (i,c) in s.chars().enumerate()
```



如果想按照切片一样对 UTF-8 字符取段，使用`utf8_slice`

```
use utf8_slice;
fn main() {
   let s = "The 🚀 goes to the 🌑!";

   let rocket = utf8_slice::slice(s, 4, 5);
   // Will equal "🚀"
}
```



如果想通过字节数组转成 String，使用`from_utf8()` 

```
let v = vec![104, 101, 108, 108, 111];

// 将字节数组转换成 String
let s1 = String::from_utf8(v).unwrap();    
```

**疑问：unwrap() 函数的作用**



### 字符串的函数

```
s.as_mut_ptr();
s.len();
s.capcacity();
```

