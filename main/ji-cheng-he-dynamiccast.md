# 继承和dynamic\_cast

## 本章节目前有较多纰漏, 请暂时无视本章, 笔者还在查资料





我们前面已经讲解了关于虚表和虚类的继承, 这样我们讲解dynamic\_cast就方便许多了

首先我们要划定一下dynamic\_cast的用途范围

笔者刚开始接触这个模板函数的时候, 曾经以为他会影响成员变量之类的东西

实际上这个函数只会操作一个东西, 那就是虚表地址, 他只会替换虚表地址为需要cast的类型的地址

在详细讲解这一切之前, 笔者还是想从别的东西作为切入点

笔者想从class这个关键字的本质含义讲起

class的本质其实是定义了一个任意字节宽度的类型

而继承就是按顺序拼接各个类的字节

可以运行以下例子

```
#include <iostream>
using namespace std;
class A {
public:
    int a;
    A() {
        a = 1;
    }
};
class B {
public:
    int b;
    B() {
        b = 2;
    }
};
class C:public A,public B {
public:
    int c;
    C() {
        c = 3;
    }
};

class D :public B, public A {
public:
    int d;
    D() {
        d = 4;
    }
};

int main()
{
    A* c1 = new C;
    B* c2 = new C;

    A* d1 = new D;
    B* d2 = new D;
    
    cout << c1->A::a << " " << c2->b << " " << d1->a << " " << d2->b << " ";

}
```

以c1和c2为例

他们的类型不同, 调用相同的new函数, 但是返回的地址不同:

![](<../.gitbook/assets/image (1).png>)

![](../.gitbook/assets/image.png)

![](<../.gitbook/assets/image (7).png>)

可以看到c2的偏移往后移动了4字节

笔者目前还没仔细了解这个字节偏移是谁实现的, 猜测是new运算符里面实现的

但是总的来说, 继承就是按顺序把不同域的成员拼接起来, 然后把虚表替换一下

对于成员来说, 他们的结构顺序可以静态确定, 并且在new的动态过程中, 可以保证指针域只会包含静态类型所覆盖的范围, 动态类型的范围可以安全的不被所指

也就是说, 动态类型为X的, 静态类型为Y(也就是说Y是基类)的一个对象(不论有没有虚函数), 可以被安全的静态转换为 动态和静态类型均为X的对象

但是如果要将上面的对象转换为动态类型和静态类型均为Y的对象, 则不可以安全的通过静态转换(static\_cast 或 c-like的括号强制转换)实现

形象一点来说

以C-like方式在C++的继承结构中, 向上移动是安全的(图源[g4g](https://www.geeksforgeeks.org/cpp-hierarchical-inheritance/))

![](<../.gitbook/assets/image (8).png>)

但是向下移动是不安全的

~~因为虚函数的工程含义是, 用继承树上的most delivered node的虚表中的函数实现, 代替基类的实现~~



### 参考文档

{% embed url="https://www.ibm.com/docs/en/zos/2.4.0?topic=expressions-dynamic-cast-operator-c-only" %}





