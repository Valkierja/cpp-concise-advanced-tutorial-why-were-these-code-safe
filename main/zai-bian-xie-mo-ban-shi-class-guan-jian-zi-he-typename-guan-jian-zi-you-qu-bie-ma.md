# 在编写模板时，class关键字和typename关键字有区别吗？

先说总结，在定义模板的第一行，尖括号里面的typename和class是等价互换的，但是在一种情况下只能用typename

我们来看例子

```
template<typename T>
void test(){
T::iterator *it;
//...
}
```

请暂停一会儿, 思考一下这段代码的问题

































如果你脑筋急转弯做的足够多, 或者是编程经验足够丰富，大概率是能够一眼看出这段代码的问题的，这个问题来自于一个长久的槽点：C语言声明指针的符号是一个乘号

如果上述代码是一个函数的声明与定义，这无伤大雅，因为编译器会正确解析乘号的语义学正确所指（refer-to）

但是这个是一个模板，在编译前IDE并不能检查到这个模板要接收的类T里面有没有一个叫做 `iterator` 的成员, 同时, it可能是一个全局变量

也就是说这段语句完全有可能被编译器判断为一个乘法运算, 其结果没有保存在任何左值当中。

为了避免这个情况可以这样写， 这样IDE就会检查T::iterator是类型还是成员,并正确解释乘号的含义

<pre><code>template&#x3C;typename T>
void test(){
<strong>typename T::iterator *it;
</strong>//...
}
</code></pre>

这里的typename就不能换成class了, 算是固定用法



