# 到底什么是封装：类方法都应当被定义为成员函数吗?

这个问题我觉得可能很多人都切身经历过，只要你写C++课设的时候，每一行代码都是自己写的话，你肯定会发出这样一个疑问：xxx函数该定义在某某类里面吗？

我们给一个例子吧，这里我觉得原书例子非常好，就不自己写例子了

首先假设我们有一个浏览器类

其中已经实现了三个成员函数，分别是清除浏览记录，清除cookie，清除cache，这三个函数都是操作类实例数据的函数，且他们可以被恰当的定义在类内

```cpp
+
public:
  ...
  void clearCache();
  void clearHistory();
  void removeCookies();
  ...
};
```

同时我们还想设计一个一键清除所有记录的函数clearAll

**请问这个函数该定义在哪里，请先自己思考一下，并在心里给出一到三个理由，再接着往下看**



















准备好了吗？我们继续讲下去。

首先这个问题不是很适合只学过C++的学生来回答，他们有更大可能选不出来正确答案，但是如果你学过软件工程和数据库原理，你有更大概率选对，蒙对的概率也大于百分之五十。

揭晓答案：这个clearAll函数不应当定义为成员函数，首先是最简单化的理由：

我们已经有三个public函数（或者说是可以被使用的三个接口）对外暴露了，我们再定义一个暴露接口是毫无意义的，而且她增加了一个可以被无限访问的对外接口，这有点类似于数据库原理里面的第一范式：数据库表的每一列都是不可分割的基本数据项，我们已经有三个数据项了，但是又在数据库内部合并了这三个数据项，这样做对于数据库内部的数据独立性只有坏处，另外，我们这里的clearAll职能比较简单，如果这是一个复杂函数，他的主要功能是按照某种复杂的顺序和执行次数来执行其他public函数，并额外调用其他复杂的东西，那么我们就更有动力把他声明为非成员函数了，因为一个成员函数可以无限访问私有变量，这样的函数越少封装性越好；**封装性的本质是看得到数据的人、函数越少越好，**因为这样可以让我们在需要修改封装内的逻辑的时候，受到影响的人和函数较少，大部分人和函数都是在操作封装后的东西，而不会强耦合封装前的底层数据。

这是第一个最简单的理由，我个人觉得这个理由不是很有说服力，但是他足够简单，足够让我们有动力这样做，即：封装**必要**函数到public后，**其他**调用public的函数都应当被定义为utility函数，然后放在类外面

第二点原因更加有说服力一些，我们先来看看STL的代码组织结构

大家回想一下我们使用STL的容器是如何操作的，首先我们需要`#include<vector>` ...欸，对了，我们不是`#include<STL>`

实上，如果我们把STL的组织方式放到我们的浏览器项目中的话，他看起来是这样子的：

<pre class="language-cpp"><code class="lang-cpp">
<strong>//In webbrowser.h
</strong>namespace MyBrowser{
class WebBrowser //...
}

//In webbrowserutility.h
namespace MyBrowser{
void clearAll()//...
}
</code></pre>

因为可能有的用户或开发人员并不关心clearAll功能，他们只需要最基本的一个浏览器类，基于这点，我们把上述函数放到成员函数里面也是不合适的



总结：

由于以下三个原因（实际原因更多），我们建议把一个唯一作用是调用其他public成员函数的函数声明为non-mem non-fri（非成员非友元）：

1. 这样的函数就好比数据库原理当中的合并列，他们只会破坏封装
2. 这样的组织方式与STL相似，十分简洁，清晰，并且易于拆分成多个较小的代码文件
3. 精简类的核心功能是有益的，不论是从运行效率还是开发效率的角度



