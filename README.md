# Learn_rust_OS_log

## 简介

这是我学习rust以及rcore相关的记录

语言相对口语化（反正也不是给别人看的），但是保证该有的都有。

另外很抱歉现在（2022.7.10）才开始写，主要是之前在准备昨天的公务员考试。

关于内容，操作系统和RISC-V我都有详细的学过，前者包括ucore的实验，以及912真题的洗礼，后者我在zju的时候有专门修那门risc-v的计组课程，还设计了相应的CPU，当然与操作系统相关的系统级的指令和特权级这些，用的不是很多，因此记录不会很多，但如果有之前未曾涉及或者有遗忘的，还是会加以描述。

首先是rust的学习部分，这部分不算很难。

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
