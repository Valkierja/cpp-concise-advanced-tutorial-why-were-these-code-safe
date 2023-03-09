# 杂谈与补充1

讲一个off topic的话题，我感觉很有启发作用，但是由于博客那边没啥人看（虽然这边也没啥人看），于是暂时写到这里了

考虑写一个性能占用较小，要求可移植性的，打log的宏函数

这里有一个基础例子

```cpp
#define LOG(msg) \
    std::cout << __FILE__ << "(" << __LINE__ << "): " << msg << std::endl
```

考虑以下实例可能的性能问题:

```
string file = "foo.txt";
int errorCode = 123;
...
LOG("Read failed: " << file << " (" << errorCode << ")");

// Outputs:
// test.cpp(5): Read failed: blah.txt (123)
```

可能没法一眼看出来, 笔者第一次看到的时候大概前前后后跑各种性能测试和性能分析, 加上查资料, 大约五小时才理解整个前因后果, 其实最本质的问题在这里, 考虑如下例子:

```
cout<<a<<b<<c<<d<<e<<....;
```

对于log这种, 数据富集型, 或者说是数据密集型的cout输出, 如果不考虑使用printf(以避免潜在的安全问题), 也不考虑使用std::format(C++20才有这个, 向前兼容和移植性差, 而且很多项目不仅仅是懒得升级编译器版本, 甚至是必须要编译在C++11下), 那么我们就会遇到cout加上一大串的operator<<

这会导致:

```
ostream::operator((ostream::operator(cout,a),b)//诸如此类的一系列调用
```

这会**硬**编译为:

<pre><code>push a
push cout
<strong>call operator
</strong>push b
push eax
call operator
</code></pre>

代码相似和重复的部分太多, 而且每个打log的地方都会静态代码扩展太多

在内存足够的情况下(不考虑嵌入式), 我们还是更愿意把这种细碎的操作放在内存里, 考虑如下解决方案

```
#define LOG(...) LogWrapper(__FILE__, __LINE__, __VA_ARGS__)

// Log_Recursive wrapper that creates the ostringstream
template<typename... Args>
void LogWrapper(const char* file, int line, const Args&... args)
{
    std::ostringstream msg;//==cout时向屏幕输出
    Log_Recursive(file, line, msg, args...);
}

// "Recursive" variadic function
template<typename T, typename... Args>
void Log_Recursive(const char* file, int line, std::ostringstream& msg, 
                   T value, const Args&... args)
{
    msg << value;
    Log_Recursive(file, line, msg, args...);
}

// Terminator
void Log_Recursive(const char* file, int line, std::ostringstream& msg)
{
    std::cout << file << "(" << line << "): " << msg.str() << std::endl;
}
```

这样就只有一个循环,而不是硬编码了多个loop, 尤其是log地方较多时, 每个地方都会扩展出来多个loop

上面的改进方法可以让我们只需要一个函数Log\_Recursive, 然后每个打log的地方只需要一个Log\_Recursive的前导函数

```
Log_Recursive:
    push arg
    call Log_Recursive
    
```

但是由于我一开始没有理解问题的关键在递归

我以为`msg << value`单次调用operator<<就可以减小代码大小

因此去又专门再看了一下operator<<实现, 看到了很多老帖子下的大佬写的一些鞭辟入里的评论, 附上

{% embed url="https://www.zhihu.com/question/22821783" %}

[https://www.zhihu.com/question/27353203](https://www.zhihu.com/question/27353203)



尤其是:



![](<../.gitbook/assets/image (1).png>)

![](../.gitbook/assets/image.png)

还有一条, 但是找不到了, 大致的意思就是线程控制的相关函数就不能返回值, 而应该返回引用, 因为复制值是无意义的, 并不会增加线程数量

