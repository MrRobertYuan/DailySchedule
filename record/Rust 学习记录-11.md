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

这里还有点懵，等会看锁章节的时候仔细看



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