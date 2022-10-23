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