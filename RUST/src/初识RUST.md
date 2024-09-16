# 初识RUST

> 本篇主要是在初学完《RUST圣经》基础篇后，对RUST语言的初认知。总结出我认为对于一个C/C++的程序员，在刚接触RUST之后的大体印象（即RUST的特点？），旨在一文让人对RUST有初步了解。


## 1、关于RUST需要明确的几个概念

- **变量相关**
 - `let a = "It's a &str"`，在rust语言中这个语句被称作为**变量绑定**，这个过程如何理解呢？感觉可以看作`"It's a &str"`这个常量是存在栈内存上，上述语句是先创建了一个变量`a`，再把常量对象（对应内存）的**所有权**给到了变量a。主要是为了内存安全;
 - 可变与不可变，在变量声明时用`mut`区分;
 - 变量解构：类似于匹配

- **所有权**
 - Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者;
 - 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者;
    ```rust
    let s1 = String::from("hello");//变量s1是String类型的hello的所有者
    //String类型是可变长的字符串，它是存在堆上的
    let s2 = s1;//这里hello的所有权发生了转移
    println!("{}, world!", s1);//s1是个无效引用，所以报错
    ```
    这么做的好处就在于一定程度上避免了内存二次释放的风险;
    ```rust
    let x = 5;
    let y = x;//这里并没有发生所有权的转移，而是在栈上对值5做了拷贝
    println!("x = {}, y = {}", x, y);
    ```
 - 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop);

- **引用与借用**
 - 获取变量的引用称之为借用，允许你使用值，但是不获取**所有权**（因此引用的变量离开作用于之后也不会释放引用指向的值）;
 - 可变引用与不可变引用的使用规则
 - **可变引用**同时只能存在一个：**同一作用域，特定数据只能有一个可变引用**，避免多个指操作一个数据的数据竞争问题
 - 可变引用与不可变引用**不能同时存在**：对于引用的作用域而言的。好处是避免数据污染，不可变引用肯定是不希望自己指向的值被改变的。
    ```rust
    let mut s = String::from("hello");
    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    let r3 = &mut s; // 大问题
    println!("{}, {}, and {}", r1, r2, r3);
    //但s被改了其实，r1和r2也就被改了
    ```
 - 编译器会保障引用的作用域结束前，数据是不会先释放的（不然会编译错误）有效避免空指针的情况；

## 2、对RUST几个关键特点的认知与联系
> 通过泛型、特性、方法三者的概念与彼此间的关系来对rust有个初步认识
- 类型：许多类型和其他编程语言大差不差，这里对有所区别的情况做个说明；
 - 数值类型：NaN，is_nan(),as做类型转换
 - 字符类型：中文在UTF-8中占三个字节
 - 切片：对集合类型的部分连续的元素序列的引用，所以对中文字符串的切片引用要注意索引
 - 元组：`let x: (i32, f64, u8) = (500, 6.4, 1);let six_point_four = x.1;`
 - **结构体**：结构体中的字段所有权转移出去后，则无法访问该字段，但可以访问其他字段
    ![alt text](image.png)
 - **枚举**：枚举的功能非常强大，任何数据类型都可以放到枚举成员中。它是一种创建一组相关值的类型。每个枚举成员都可以包含不同类型的数据，但这些成员都属于同一个枚举类型。
    ```rust
    enum Message {
      Quit,//没有任何关联数据
      Move { x: i32, y: i32 },//包含一个匿名结构体
      Write(String),//包含一个String字符串
      ChangeColor(i32, i32, i32),//包含三个i32
    }

    fn main() {
      let m1 = Message::Quit;
      let m2 = Message::Move{x:1,y:1};
      let m3 = Message::ChangeColor(255,255,0);
    }
    ```
    还可以为枚举实现方法，特性等，感觉极大加强枚举类型的功能
 - 泛型：和其他语言的语法也大差不差，区别在于可以对泛型指定对应的特征。如下：
    ```rust
    fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
      //fn 函数名<T:指定特征>(函数参数:T类型)->返回类型T
      a + b//不带分号，标识直接返回
    }
    ```
- 方法：
 - 通过`impl`来定义方法：类似于其他语言的Class，只不过方法和数据结构体是分离的
    ```rust
      struct Circle {
         x: f64,
         y: f64,
         radius: f64,
      }

      impl Circle {
      // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
      // 这种方法往往用于初始化当前结构体的实例
      fn new(x: f64, y: f64, radius: f64) -> Circle {
         Circle {
            x: x,
            y: y,
            radius: radius,
         }
      }

      // Circle的方法，&self表示借用当前的Circle结构体
      fn area(&self) -> f64 {
            std::f64::consts::PI * (self.radius * self.radius)
         }
      }

    ```
 - `self、&self、&mut self`：**所有权**
 - `self` 表示 `Circle` 的所有权转移到该方法中，这种形式用的较少
 - `&self` 表示该方法对 `Circle` 的不可变借用
 - `&mut self` 表示可变借用
 - 可以用为一个结构体定义多个impl块，便于把相关的方法放在一起
 - 关联函数：在`impl`中与结构体关联紧密，但不使用`self`，因此他是函数而不是方法（**无法x.area()这么调用、而是用Circle::new()**）
 - 可以为枚举实现方法====>很好用的特殊结构体？
- **特性Trait（非常重要）**:一组可以被共享的行为，只要实现了特征，你就能使用这组行为，让不同的类型能有共同的特征（这个特征只是定义了接口，具体的时间根据自己的类型来，有点类似于抽象类？）
 - 定义特征：特征只定义行为看起来是什么样的，而不定义行为具体是怎么样的。因此，我们只定义特征方法的签名，而不进行实现，此时方法签名结尾是;，而不是一个 {}。
    ```rust
    pub trait Summary {
    fn summarize(&self) -> String;
        }
    ```
 - 可以为类型实现特征，类似于结构体中实现方法，举个例子；
    ```rust
      pub trait Summary {
         fn summarize(&self) -> String;
      }
      pub struct Post {
         pub title: String, // 标题
         pub author: String, // 作者
         pub content: String, // 内容
      }

    impl Summary for Post {
    fn summarize(&self) -> String {
    format!("文章{}, 作者是{}", self.title, self.author)
        }
    }

    pub struct Weibo {
    pub username: String,
    pub content: String
    }

    impl Summary for Weibo {
    fn summarize(&self) -> String {
    format!("{}发表了微博{}", self.username, self.content)
        }
    }
    ```
 - 特征作为函数参数——**只要实现了这个特征的类型都能作为函数入参**；
 - 特征约束——类似`let b:i16 = 3;`这里的i16可以换成特征，作为变量的特征约束；
 - 特征相关的内容还特别多，感觉是rust中非常重要的特性，这里只是大致说明一下是什么就行，如何更好运用以及特点待后续更多的学习。

- 通过泛型、方法、特征三者均赋予类型极高的灵活性，让结构体、枚举这样的类型有了更多的可拓展性。目前对这些特性的初步认知，rust可以实现面向对象、多态、代码抽象、复用等特性。

## 3、基本语法形式

```rust
let a:i32 = 5; let mut b = 6;//可变与不可变变量，指定i32的类型；
let x = 5; let y = &x; assert_eq!(5,*y);//引用与解引用；
println!("{},{}",a,b);//打印，{}作为占位符，有Display特性就行；

let mut a = String::from("hi");
let b = &a;//不可变引用
let mut c = &a;//可变引用
```

```rust
#![allow(dead_code)] // 允许编译器忽略未使用的代码

use std::fmt; // 引入格式化库
use std::fmt::{Display}; // 引入Display特征

// 定义一个枚举类型，表示文件状态
#[derive(Debug,PartialEq)] // 自动派生Debug和PartialEq特性
enum FileState {
 Open, // 文件打开状态
 Closed, // 文件关闭状态
}

// 定义一个结构体，表示文件
#[derive(Debug)] // 自动派生Debug特性
struct File {
 name: String, // 文件名
 data: Vec<u8>, // 文件数据
 state: FileState, // 文件状态
}

// 为FileState实现Display特性，用于格式化输出
impl Display for FileState {
 fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
 match *self {
 FileState::Open => write!(f, "OPEN"), // 当状态为Open时，输出"OPEN"
 FileState::Closed => write!(f, "CLOSED"), // 当状态为Closed时，输出"CLOSED"
     }
   }
}

// 为File实现Display特性，用于格式化输出
impl Display for File {
 fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
 write!(f, "<{} ({})>", // 格式化输出文件名和状态
 self.name, self.state) // 使用self.state调用Display特性的fmt方法来格式化状态
   }
}

// 为File实现new方法，用于创建新的File实例
impl File {
 fn new(name: &str) -> File {
 File {
 name: String::from(name), // 将传入的文件名转换为String并存储
 data: Vec::new(), // 初始化一个空的Vec作为文件数据
 state: FileState::Closed, // 默认文件状态为Closed
    }
  }
}

fn main() {
 let f6 = File::new("f6.txt"); // 创建一个新的File实例
 //...
 println!("{:?}", f6); // 打印文件的详细信息（调试模式）
 println!("{}", f6); // 打印文件的简要信息（非调试模式）
}
```


## 4、项目代码文件构成
> - Packages:项目、软件包 一个 Cargo 提供的 feature，可以用来构建、测试和分享包
> - Crate：包，一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行
> - Module：模块，可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元，语法：`mod my_module;`
> - 对于`crate`和`module`关系的理解：https://zhuanlan.zhihu.com/p/164556350

典型的package结构
```txt
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```
- 唯一库包：src/lib.rs
- 默认二进制包：src/main.rs，编译后生成的可执行文件与 Package 同名
- 其余二进制包：src/bin/main1.rs 和 src/bin/main2.rs，它们会分别生成一个文件同名的二进制可执行文件
- 集成测试文件：tests 目录下
- 基准性能测试 benchmark 文件：benches 目录下
- 项目示例：examples 目录下
>rust里是通过use xxx来导入其他文件的模块，并简化写法；对于toml文件中引入的外部依赖，代码中可以直接用，也可以用use来简写