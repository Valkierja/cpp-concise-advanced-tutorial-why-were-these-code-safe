# 实现委托的技术细节：引用折叠、万能引用和完美转发

写了一点 但是看到了这篇 感觉不可能写的比po主好，故直接贴上

[https://zhuanlan.zhihu.com/p/50816420](https://zhuanlan.zhihu.com/p/50816420)

另附评论区一句

Q：请问完美转发的应用场景是啥？

A：有一个非常明显应用场景，就是你的调用过程出现了“委托”，必须要把参数性质原封不动的转发给对应实际干活的函数。

例如std::make\_unique(args...)，因为构造std::unique\_ptr的时候，这个函数不知道那个构造函数接受的是左值还是右值还是引用之类的，所以必须要把这里接收到的参数，原封不动的"转发"给对应的构造函数，还必须得是完美转发（字面意思）。

同时还有const T&这个参数，我没记错的话，如果接收的是右值时，会构造对象，这样也是有性能开销的，这样引用啥的就没啥意义了，用T&&可以解决这个问题。
