# Learn_rust_OS_log

## 简介

这是我学习rust以及rcore相关的记录

语言相对口语化（反正也不是给别人看的），但是保证该有的都有。

另外很抱歉现在（2022.7.10）才开始写，主要是之前在准备昨天的公务员考试。

关于内容，操作系统和RISC-V我都有详细的学过，前者包括ucore的实验，以及912真题的洗礼，后者我在zju的时候有专门修那门risc-v的计组课程，还设计了相应的CPU，当然与操作系统相关的系统级的指令和特权级这些，用的不是很多，因此记录不会很多，但如果有之前未曾涉及或者有遗忘的，还是会加以描述。

首先是rust的学习部分，这部分不算很难。

Day6吐槽一句，这个为什么还要把那个I AM NOT DONE的注释去掉啊，我真是醉了。。。我说怎么一直不行

## DAY1

### variables和data type

首先是Shadowing，类似于C在子作用域下的变量overlap，但是rust在同一作用域也让他可以覆盖掉了，我认为这样的实现和SSA只赋值一次，应该有很大的关系，也许这样可以增加编译优化的速度。

至于data type里面的i32这样的写法，我估计，怕不是设计者看见LLVM这么方便，直接就拿过来用了吧，当然，这样肯定方便于前端直接转换为LLVM的IR了。

不过没有C和go里面complex这样的基础类。我觉得干事情很悬。我去查了一下，emmm，用的第三方库

没有位域bit-field，或者类似什么的东西，听上去不错，额，我谢谢你。

看上去这门语言的设计者对于Compound Types的期望是，希望我们这些使用者不要关心诸如alignment这样的事情，而且位域也不支持，我开始对使用rust来构建一个OS失去信心了。

### Function

我看了一圈，觉得唯一值得记录的只有statement和expr的分开表示。

### Ownership

这应当类似于垃圾收集机制。

```
Each value in Rust has an owner.
There can only be one owner at a time.
When the owner goes out of scope, the value will be dropped.
```

所以rust设计的机制就是如果一个持有者不再持有某个memory（确切的说是以走出作用域为标准的），那么就使用drop释放掉他。这对于悬空指针的安全使用是非常重要的。

对于string类，存储的不是实际的内容，而是一个结构，用一个指针指向存储内容，该结构还包括len什么的，所以如果string进行一个assign，就相当于一个软拷贝，实际上是该结构被复制过去。但是现在assign，有两个指针指向他，就不安全，就让前面一个不再生效，同样的，对于多个mut的指向同一个s的引用会出错，也应该是这样的一个原因。要让前面一个生效，那么就要用clone。

确切的说，是下面的条件

```
Two or more pointers access the same data at the same time.
At least one of the pointers is being used to write to the data.
There’s no mechanism being used to synchronize access to the data.
```

我认为这是吸取了C/C++里面多个指针同时引用同一个对象，然后某个地方的改动导致另一个地方的错误的教训。这在某种情况下是安全的，但是总感觉缺少了一定的灵活性。

## Day2

### struct

struct 总共有三种，一种是传统C类型的，另一种是元组，最后一种好像就弄了个结构体，然后看他打印也只打印了一个结构体名称。好吧差别不大。

printIn里面打印结构体有两种格式：

```
{:#?}	// print one member per line
{:?}	// print in one line
```

另外，还有个dbg！宏，同样可以打印

在结构体中，可以像C++一样定义一些方法。第一个参数必须是&self。

### Test

没啥有用的

### enum

enum相比于C/C++不再是纯粹的int值，可以是string，tuple什么的。

Option枚举类型和null，大概就是为了解决NULL引起的一大堆问题吧

match，这不就是switch么，说是不要break引起的一堆问题，出发点并不错，可是，break不写可以让两个情况合并。match可以有返回值，但是所有的返回值必须一样的类型

对非枚举类型进行match，必须用个下划线

对于存在NULL类型的，需要考虑NULL情况

### if let

没啥用

## Day3

### package crate modules

一个package最多包涵一个库crate，可以包涵1或者更多的二进制crate

模块module

使用双冒号路径分割符

如果不加上pub，那么就是私有的，枚举类型也有类似的。大概这就是对C++的类属性的错综复杂做出一定的改变的结果吧。

use关键字

和其他语言的use差不多，不过这里面是对作用域而不是所谓的命名空间的使用，如果需要引用多个模块，用{}括起来或者干脆一个*

### colllection

主要是vector，string和hash map

实际上应该说是std::collection的应用吧，这个我就不多写了，成员方法和其他语言差不多。

### Error处理

可恢复错误用result<T,E>,不可恢复错误用Panic！

？操作符

(我也不知道为什么这个error6搞不定，我现在有点烦躁，很烦躁)

## Day4

接续Day3的那个error处理，好像这个添加的函数只要处理error即可，也就是输入是想要转换的error类型的源类型，输出则是目的类型

### Generic Types Traits and Lifetimes

泛型

这里需要注意impl后面跟<T>的情况

总而言之，用惯了C++各种template的人，看这个应该没什么难度

trait则规定了实现者必须实现的方法，而实现者则使用impl <trait> for <type>

每个impl块只能实现一个trait

trait作为传递参数的情况

## Day5

天气太热，去乡下消暑惹，这几天实在是没啥效率

回来继续做泛型的那个最后一个练习。看上去有点难度。

### Trait

trait 在structure上的实现——感觉就好像规定了公共的某些借口，然后这些接口必须实现。

至于泛型最后一个练习要求的trait bound syntax，简单来说，就是使用泛型的方法，让多个不同类型可以调用同一个trait的方法，而无需特意区分调用的类型，那不就是泛型用到接口上的意思么？非要创造一个词汇来形容？

好吧，我也不知道为什么稀里糊涂就过了

### Lifetime

这个生命周期其实可以和前面的ownship联系起来看，一个变量走出作用域就被释放掉

生命周期注释（反正我没看懂）大概是说，如果不声明生命周期可能一样，编译器，考虑到生命周期不一样会导致挂掉，不安全，所以不给过，但是声明了之后，那么就不考虑这个问题了。

### Command Line 相关

使用std::env::arg()来得到命令行参数

使用std::fs来读入文件

eprintln!宏来打印到标准错误而不是标准输出

## Day6

### Closure

这个概念不仅让我迷惑，也让我压根不想再碰rust了快。据说，对，据说这是为了实现函数式编程和lambda表达式所做的事情，啊直接翻译成lambda表达式不就很好理解了！当然，由于生命周期等其他的语言特性问题，需要注意这些。

闭包可以捕获外部变量，这是相对于fn的一个优点

### Threads

我也不想先学这个，毕竟这算是个很要紧的概念，但是standard_library_types说是13.2的部分，结果一看要用到16章，害。

创建：std::thread::spawn(|| )，注意这里面的闭包。

Join:这个join我就不多说了，大部分库都有类似的概念，等子线程搞完再跑主线程，不然主线程退出，子线程就挂了。

创建会得到一个句柄handle，使用handle.join().unwrap();

move让闭包获得使用值的所有权

线程交流：使用通道

std::sync::mpsc::channel()，例如示例中(tx, rx) = mpsc::channel();

tx用于发送，rx用于接受

sleep

使用互斥锁来进行并发Mutex

reference counted value锁 Rc 原子锁Arc

thread那个练习里面的代码，我觉得可以比较好的理解

### Box

创建：box::new

*cons list* 总感觉看上去就是个链表

### 迭代器

没啥好写的，C++迭代器还不简单么？
