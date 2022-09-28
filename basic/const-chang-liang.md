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

如果你觉得const1的这种情况不好理解的话，请看上一小节最后部分中，[Mohan](https://stackoverflow.com/a/38559710/1433373)提到的例子

{% embed url="https://stackoverflow.com/questions/3247285/const-int-int-const" %}
