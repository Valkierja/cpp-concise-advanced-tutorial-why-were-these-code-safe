# side-effect 副作用

说实话, 本小节可能更适合大家去看英文维基百科和stackoverflow, 如果英文有困难或者是觉得看不进去的话,

我简化一下: **有N个副作用**这句话的意思就是一个运算会改变N个变量的值,比如

```cpp
a = 10 + b;
```

这个表达式有一个副作用, 这个副作用作用于a, 他使得a的值改变

至于为什么要把这个东西叫做副作用, 试想下面的例子:

```cpp
A = B + C;
```

其中`A B C` 分别是三个对象

A是一个定位符(下一小节会讲到), 他定位的内存空间如果与B 或者 C 有交集的话, 这个运算会产生一些有意思的不良后果, 其中至少有一点可以肯定, 那就是这次运算的结果肯定是非预期的(后文简称UB)

在下一节会讲到, A只是一个别名, 如果我们在需要用到B + C这个结果的地方(比如一个print函数)直接传递B+C 而非先计算B+C的结果, 存储到A里面, 再传递A, 那么我们就可以规避上述的非预期问题

总结, 开辟栈空间来进行A 的**赋值操作并不是必要的, 而是一个副作用**, 如果我们直接使用`foo(B+C)` 也是完全可以的

不过`foo(B+C)` 这里其实也有副作用, 因为实参赋值给了形参, 但是我相信这个意思各位读者已经get到了,即:

**对定位符的赋值操作称为副作用, 这种赋值操作可能造成运算结果错误**

至于什么是定位符, 请看下一小节

## 拓展阅读

[What exactly is a 'side-effect' in C++?](https://stackoverflow.com/questions/9563600/what-exactly-is-a-side-effect-in-c)

[在程序开发中，++i 与 i++的区别在哪里？](https://www.zhihu.com/question/19811087)

[What is the difference between ++i and i++ in C++?](https://www.tutorialspoint.com/What-is-the-difference-between-plusplusi-and-iplusplus-in-Cplusplus)
