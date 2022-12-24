# const 常量

在C++中，常量一般有四种位置，为了叙述方便我临时将他们加上编号：

```
int const0 a=0;//与 const0 int a=0 等价

int const1 foo(int const2 bar) const3{
}
```

const0 意味着对左值`a` 的声明为一个常量，该常量的本质是只能绑定一次的定位符

const1意味着返回值是一个常量，这种写法在C11后逐渐变得不再推荐（这个说法是在英文互联网中搜到的），比如：

```
const int foo() {
   return 3;
}
int main() {
   int x = foo();  // copies happily
   x = 4;// copies happily
}
```

可以正常编译并运行

如果你觉得const1的这种情况不好理解的话，请看上一小节最后部分中，[Mohan](https://stackoverflow.com/a/38559710/1433373)提到的例子，如果你还是不太理解的话，记住这种写法已经逐渐过时了即可，**如果您对这种看法有异议，请不要着急，本系列的正文部分我会讲解这个点，事实上，我认为这个点的扩展内容（const的扩展内容）是EC这本书中最重要的五点内容之一**

const2意味着形参是一个只能初始化一次后无法再被重新定位的定位符，也就是说，实参可以是non-const型变量，关于const2我们后面的小节还会展开来讲，并提出多角度的观点同时附上足够有支持力度的佐证，我一共会描述四种情况，以扩展原书中的单一情况

const3意味着函数内部的所有操作只会对当前函数栈帧空间上的内容进行修改，而不会修改任何外部内容，这个是一个编译时的检查和保证，而非运行时的，所以如果你在编译结束后进行patch，也仍然是可以正常运行的（除非把主要逻辑破坏了），但是这个运行结果肯定就和预期的不一样了

对于const3，我建议能用则用，只要你判断函数逻辑是不修改外部实体的，你就加上const3，最为一个逻辑bug的编译器级检查，还能在某种程度上帮助编译优化



{% embed url="https://stackoverflow.com/questions/3247285/const-int-int-const" %}

{% embed url="https://stackoverflow.com/questions/117293/use-of-const-for-function-parameters" %}
