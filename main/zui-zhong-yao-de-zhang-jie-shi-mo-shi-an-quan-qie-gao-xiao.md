# 最重要的章节: 什么是安全且高效

在探讨什么是安全且高效之前, 我们先来探讨指针和引用的区别, 以及什么时候使用传值, 什么时候使用传引用

**请耐心看完本节, 这应该是本系列中让你收获最大的一节**

首先我想让读者回忆一下自己写C++课设时的应该遇到过的一个问题,&#x20;

在写C++课设的时候, 我相信大部分人都自己实现过分数类或者有理数类, 或者类似的东西, 如果没有实现过也没关系, 我们来想象这样一个类, 他有一些私有的成员,还有唯一一个有效的public函数: 一个加法运算符的重载, 这个加法运算符需要接收两个类对象, 计算结果返回一个类对象, 例子如下:

```
class myClass{
private:
    int num;
public:
    friend myClass operator+(myClass& left, myClass& right);
    //result.num = left.num + right.num
};
```

这里可能会发生了一个让人很难受的现象, 那就是可能很多人会这样写, 来让函数“优化”

```
friend myClass* operator+(myClass& left, myClass& right);
    //result->num = left.num + right.num
    //左右操作数都在用点操作符, 而result是指针操作符-> , 这种不对称性其实暗示了这是一个bad practice
```

究其原因, 可能有的人会回答到, 返回myClass 对象需要复制整个对象, 而返回myClass 指针只需要往eax寄存器里面放一个地址即可, 开销小了一两个数量级

我相信有不少人这样写过(即, 函数返回一个对象指针或者对象引用)

**让我们来看看你是否做到了安全且高效**



为了进一步讨论, 我们把上面的代码写全一点, 这样方便我们接下来的讨论

```
class myClass{
private:
    int num;
public:
    friend myClass operator+(myClass& left, myClass& right);//正确实现
    
    friend myClass& operator+(myClass& left, myClass& right);//错误实现
};
myClass operator+(myClass& left, myClass& right){//正确实现
    return myClass(left.num + right.num);
}

myClass& operator+(myClass& left, myClass& right){//错误实现, 有多种情况
    
}
```

首先我们需要说明的是这个讨论不单单局限于友元函数, 也不单单局限于重载运算符, **只要是一个函数需要返回"新的"对象, 就只能定义为"按值返回" 而不能定义为 "按引用返回" 而不能定义为 "按引用返回" 而不能定义为 "按引用返回"（不确切，请看文章末尾的补充说明一）**

首先我们发现, 我们这个函数计算的结果需要一个新的空间存放, 存放的对象是一个新的myClass instance

那么我们就需要问一个问题, 如果按引用返回, 我们这个新的实例对象需要放在哪里?

二进制安全经验丰富, 反应快的读者可能已经意识到重大问题了, 如果你还没有理解, 请跟着我的思路继续走下去； 先回答上面的问题, 这个新返回的对象必须要放在内存里, 要么是stack 要么是 heap(放在数据库里面之类的情况暂时不讨论)

首先是stack

```
myClass& operator+(myClass& left, myClass& right){//错误实现, 情况一
    myClass result(left.num + right.num);
    return result;
}
```

如果你使用的是VS之类的IDE, IDE应该会立刻报warning表示, 不应当返回一个指向栈上的对象的引用, 因为当函数结束返回时, 会做出退栈操作, 这个操作会销毁(生命周期意义上的销毁, 而非数据内容上的立刻销毁)栈上对象

然后你可能会改而采用这样的方法

```
myClass& operator+(myClass& left, myClass& right){//错误实现, 情况一
    return myClass(left.num + right.num);
}
```

此时IDE会立刻弹出Error(更严重了!) 表示编译无法进行, 因为我们打算把一个没有名字的右值`myClass(left.num + right.num)`绑定到一个non-const左值上(`myClass&`)

然后你可能会想, oh 那么我把函数原型签名为const返回值呢?

"幸运的是", 这次可以编译通过了, 似乎结果也很正确

![](<../.gitbook/assets/image (1) (1) (1).png>)

但还是一样的, 我们正在尝试把一个栈上的对象返回其引用

![](<../.gitbook/assets/image (3) (1).png>)

可以看到这个对象被放在了ebp-0C8h这个位置 ,这里还是栈上

那么有"聪明的人"可能会想到, 那么我们把这个对象new到heap上然后返回不就解决了吗

我相信这也是大多数对heap只有表面了解, 而没有深入研究的程序员会在这种情形下做出的判断,&#x20;

一言以蔽之, 请问这个new出来的对象该由谁, 在什么时候进行delete操作?

例子如下

<pre><code><strong>myClass&#x26; operator+(myClass&#x26; left, myClass&#x26; right){//错误实现, 情况一
</strong>    auto result = new myClass(left.num + right.num);
    return *result;
}

int main(){
<strong>    c = a + b + e + d; //类型均为myClass
</strong>}
</code></pre>

我们无法提前获知这种连续的运算情况下 总共额外生成了多少个result对象, 这个对象具有临时性,只需要从assign符号右边到左边的赋值完成就可以立刻被delete, (而且在使用结束后delete越早越好, 因为这可以让其他线程获得更大的潜在内存空间) 但是我们却不知道该delete几个

可能你会说, 我们只需要用一个数组去记录不就可以了吗,

但是你还记得我们的初衷吗, 我们的初衷是把一个按值返回的对象优化为按引用返回或者是按指针返回的对象, 用一个数组去记录的方式会让我们于最初的初衷背道而驰

可能有人会说, 既然我们的result是临时的, 且一个实例的大小是固定的, 那我们直接开一个static去存储.data段或者.bss段到会不会更好一些?

实际上这也是一个bad practice, 因为这会造成多线程安全的问题, 和伪比较问题 例子如下

```
myClass& operator+(myClass& left, myClass& right){//错误实现, 情况一
    static myClass result;
//...
    return ...
}
```

多线程安全的例子我应该不用举例了, 大家可以想象到, 所有线程实例都是访问相同的bss段, 这样会导致多线程安全问题, 总共有三种情况, 分别是先来的线程的结果等于后来的线程, 后来的线程结果等于先来的线程, 以及二者的结果都等于某一个UB值

另外就是类似下面的判断永远会被定义为真

```
if ( (a+b) == (c+d) )
```

至此,我们已经讨论完了所有关于引用和指针的可能性, 还包括了静态变量的例子, 我们会发现,这些实现都是bad practice  都是不安全的实现,  所以我们可以得出一个结论高效且安全是不可能实现的, 安全才是第一要务, 安全且高效是可以被实现的,  比如像在这个例子中, 会牺牲函数的开销(因为是按值返回), 但是我们依旧要这样做:

```
myClass operator+(myClass& left, myClass& right){//正确实现
    return myClass(left.num + right.num);
}
```

本小节以原书的原句结尾:

虽然你采用这个实现需要承担多次构造和拷贝的开销, 但这只是你为了达成正确目的而采取的正确手段所需的小小成本罢了

你, 身为一个商业软件的程序员, 的首要工作是找到正确的实现, 而非高效的实现, 如果正确的实现意味着高开销的实现, 那么你应该把担子丢给编译器的维护人员与操作系统的维护人员, 你可以继续享受你的生活



补充说明一 : 并非所有函数(或者重载的运算符)都不能返回引用, 而是一旦返回的对象并非**已存在**的对象, 或者没有恰当方式捕获每一个对象的delete时机, 那么你只能按值返回, 返回后由callee处理, 将其放在callee的调用栈里面。当返回一个已存在的对象时，只需要返回指向它的指针或者引用，当可以用恰当的方式捕获每一个新对象时，你可以通过new的方式返回指针，然后再利用另一个函数去确定他们的delete时机





