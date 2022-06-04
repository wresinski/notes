## C++面向对象高级开发笔记 ##
> Big three:拷贝构造、拷贝赋值、析构（带堆内存类）
1. 数据在`private`中

2. 参数尽量是引用（是否是`const`另说）

3. 返回值尽量是引用

4. 应该加`const`的成员要加（全局函数不可加const）
```cpp
{
  const complex c1(2,1);
  cout << c1.real();
  cout << c1.imag();
  //const对象只能调用const成员函数
}
```
| | const object(data members不得变动) | non-const object(data members可变动) |
| ---- | ---- | ---- |
| const member function(保证不更改data members) | ✔ | ✔ |
| non-const member functions(不保证data members不变) | ✖ | ✔ |

5. 尽量使用构造函数的变量初始化

6. 在类定义里实现的成员函数是`inline`的

7. `+=`执行顺序从右往左，`<<`执行顺序从左往右
| 操作类型       | 符号                               | 执行顺序 | 优先级 |
|----------------|------------------------------------|----------|--------|
| 表达式         | [] () . -> 前缀++ 前缀--           | 从左到右 | 最高   |
| 一元操作符     | 后缀++ 后缀-- sizeof & * + - ~ !   | 从右到左 |        |
| 一元操作符     | type(类型说明)                     | 从右到左 |        |
| 乘除法         | * / %                              | 从左到右 |        |
| 加减法         | + -                                | 从左到右 |        |
| 位移操作       | << >>                              | 从左到右 |        |
| 大小关系       | <> <= >=                           | 从左到右 |        |
| 相等关系       | == !=                              | 从左到右 |        |
| 位与运算符     | &                                  | 从左到右 |        |
| 位异或运算符   | ^                                  | 从左到右 |        |
| 位或运算符     | \|                                 | 从左到右 |        |
| 逻辑 与        | &&                                 | 从左到右 |        |
| 逻辑 或        | \|\|                               | 从左到右 |        |
| 条件判断表达式 | ? :                                | 从右到左 |        |
| 简单混合操作   | = *= /= %= += -= <<= >>= &= ^= \|= | 从右到左 |        |
| 顺序赋值       | ,                                  | 从左到右 | 最低   |

8. class with pointer members必须有copy ctor和copy op= 
* copy ctor:
```cpp
String s2(s1);
String s2 = s1;
```
注意：传值参和return值的时候会调用拷贝构造
* copy op=:
```cpp
String s2;
s2 = s1;
```

9. op=拷贝赋值函数内部要检测是否自我赋值：
```cpp
if (this == &str)
  return *this;
```

10. `delete`忘记`[]`也会回收全部内存，但不会调用全部析构函数；`new()`是`malloc()`+`ctor()`，`delete()`是`dtor()`+`free()`;  
`operator new(size_t size)`、`operator new[](size_t size)`、`operator delete(void* pdead, size_t size)`、`operator delete[](void* pdead, size_t size)`可以被重载，全局（不可声明于一个namespace内）或成员  
注意：`size`是具体字节数  
`::new`和`::delete`用于强制调用全局函数  
编译器会在数组前面加上一个4字节的counter
还可以重载placement operator new和placement operator delete，例如`Foo* pf = new(300,'c')Foo;`；而且可以重载很多个版本，前题是每一个版本的声明都必需有独特的参数列，且第一参数必需是size_t  
我们也可以重载 class member `operator delete()`，写出多个版本；但它们绝不会被 delete 调用；只有当 new 所调用的 ctor 抛出 exception ，才会调用这些重载版的`operator delete()`；它只可能这样被调用，主要用来归还未能**完全创建成功**的object所占用的memory  
当我们想偷偷地多分配一些东西时，可以选择重载placement operator new

11. `const char*`=`char const*`，内容不可改变；`char *const`指针不可改变

12. 每个非`static`成员函数都有隐含的`this`指针

13. `static`成员变量记得在外部初始化（定义）

14. 单例模式（singleton）：把构造函数放在`private`以避免直接实例化
```cpp
class A {
public:
  static A& getInstance() { 
	static A a;
	return a;
  }
  setup() { ... }
private:
  A();
  A(const A& rhs);
  ...
};
```

15. `:virtual`虚继承：为了解决菱形继承下基类成员有两份的问题

16. 函数模板实例不需要明确指出type，因为编译器会自动进行类型推导

17. `operator type()`：重载类型转换，不需要写返回值，通常应该加`const`

18. 复合类的构造由内而外`Container::Container(...): Component() { ... };`，析构由外而内`Container::~Container(...) { ... ~Component() };`；继承同理  
子类构造函数会自动调用父类构造函数，子类析构函数同样也会自动调用父类析构函数

19. 父类析构函数必需是`virtual`

20. 构造顺序，父类->成员变量->本体

21. 虚函数非纯虚函数保证子类不必需重新定义

22. 原型&克隆
> parameter是形参，argument是实参

23. 当类型不符合，编译器会自动寻找函数将其转化为所需类型

24. 注意类型转换的ambiguous（二义性），单参数构造函数加上`explicit`关键字可拒绝隐式构造

25. `->`操作符会继续作用于作用结果（用于解释`->`操作符重载）

26. 智能指针必需重载`*`和`->`操作符

27. 仿函数：重载了`operator()`的类
> TODO:仿函数往往继承了`unary_function`和`binary_function`，why？

28. 模板的分类（可嵌套）：
* class template
* function template
* member template（可用在构造函数模拟up-cast）

29. 模板偏特化绑定参数时要从左往右不能跳  
`vector<bool>`并不是stl容器，因为`operator[]`无法返回一个对应元素的引用：
```cpp
vector<bool> c{ false, true, false, true, false }; 
bool b = c[0]; 
auto d = c[0]; //d的类型不是bool，而是vector<bool>中的一个内部类 
```

30. 范围偏特化：
```cpp
template <typename T>
class C
{
  ...
};
//假如是指针
template <typename U>
class C<U*>
{
  ...
};
```

31. template template parameter（模板模板参数）:
```cpp
template <typename T, template <typename T> class Container>
class XCls
{
private:
  Container<T> c;
public:
  ......
};
```
——模板参数为模板
```cpp
template<typename T>
using Lst = list<T, allocator<T>>;
```
~`XCls<string, list> mylst1;`~编译不过因为`list`需要两个模板参数  
`XCls<string, Lst> mylst2`正确

32. 这不是template template parameter：
```cpp
template <class T, class Sequence = deque<T>>
class stack {
  friend bool operator== <> (const stack&, const stack&);
  friend bool operator< <> (const stack&, const stack&);
protected:
  Sequence c; //底层容器
......
};
```
`stack<int> s1;`自动生成第二参数  
`stack<int,list<int>> s2;`第二参数为确定类型  
模板参数为模板（不确定类型）的才是template template parameter

33. c/c++负数取模采用的是向零取整：向0方向取最接近精确值的整数

34. variadic template:
```cpp
void print() {}
template <typename T, typename... Types>
void print(const T &firstArg, const Types &...args) {
  cout << firstArg << endl;
  print(args...);
}
```

35. ranged-base for:
```cpp
vector<double> vec;
...
for ( auto elem : vec ) {
  cout << elem << endl;
} //传值，不会改变容器内元素
for ( auto& elem : vec ) {
  elem *= 3;
} //传引用，会改变容器内元素
```

36. reference通常不用于声明变量，而用于参数类型(parameters type)和返回类型(return type)的描述

37. 以下被视为“same signature”（所以二者不能同时存在）：
```cpp
double imag(const double& im) { ... }
double imag(const double  im) { ... } //Ambiguity
```

38. const是函数签名的一部分：优先调用非const，除非是const对象（否则如果非const函数可以被const函数重载，矛盾）
```cpp
charT operator[] (size_type pos) const
{ ....../*不必考虑COW*/ }
//const对象只会调用上者
reference operator[] (size_type pos)
{ ....../*必需考虑COW*/ }
//COW:Copy On Write
```
当成员函数的const和non-const版本同时存在，const object只会（只能）调用const版本，
non-const object只会（只能）调用non-const版本

39. 对于父类子类同名非虚函数，调用父类成员函数时，通过对应的类名调用或者用父类对象或指针调用即可，虚函数则必需通过类名或者父类对象，不可通过父类指针调用；虚函数调用的c形式：`(*(p->vptr)[n])(p)`或`(* p->vptr[n] )(p)`  

动态绑定虚机制（多态）调用的条件：
* 通过指针来调用
* up-cast(向上转型）
* 调用的是虚函数  
其他的（通过对象调用、非虚函数调用）都是静态（编译时写死），即通过指针调用的虚函数区决于具体实例

40. 要让父类中的函数调用子类中的函数，必需在父类中声明为虚函数，并在子类中重载

41. 避免使用`A a();`来构造对象，而尽量使用`A a;`，因为如果A定义没有对应的无参数构造函数，则前者会被当作函数声明（如果已经写了构造函数，编译器便不会再给一个默认构造函数）

42. `volatile`关健字：表明该变量随时可能会被其他程序修改，因此每次应该从变量所在的内存地址读取而不是从临时的寄存器中读取

43. 三种继承(public protected private)
| | public继承 | protected继承 | private继承 |
| --- | --- | --- | --- |
| public成员 | public | protected | private |
| protected成员 | protected | protected | private |
| private成员 | 不可用 | 不可用 | 不可用 |

* public继承
> 从语义角度上来说，public继承是一种接口继承（可以理解为子类对象可以调用父类的接口，也就有可能实现多态了）；从语法角度上来说，public继承后，关系见上图；很明显父类public成员在子类中仍然是public，所以子类对象可以调用父类的接口

* protected继承
> 从语义角度上来说，protected继承是一种实现继承；从语法角度上来说，protected继承后，父类public和protected成员都变成子类的protected成员了，也就是说子类对象无法调用父类的接口，只能将父类的函数当作子类的内部实现，当然也就不符合“Liskov替换原则（LSP）”了

* private继承
> 从语义角度上来说，private继承是一种实现继承；从语法角度上来说，private继承后，父类public和protected成员都变成子类的private了，它比protected继承更严格；也就说这些父类的成员只能被继承一次，再继续继承，父类的成员就不可见了；private继承更不符合“Liskov替换原则（LSP）”了

44. 指向类成员的指针
* 指向类数据成员的指针
> //定义和赋值（和类相关）  
> 数据类型 类名:: *指针名 = &类名::成员名  
> //使用（和对象相关）  
> //由于类不是运行时 存在的对象。因此，在使用这类指针时，需要首先指定类的一个对象，然后，通过对象来引用指针所指向的成员。  
> 类对象名.*指向非静态数据成员的指针  
> 类对象指针->*指向非静态数据成员的指针  
* 指向类成员函数的指针
> //定义和赋值（和类相关）  
> 函数返回类型名 （类名::*函数指针名）（参数列表） = &类名::成员函数名  
> //使用（和对象相关）  
> //由于类不是运行时存在的对象。因此，在使用这类指针时，需要首先指定类的一个对象，然后，通过对象来引用指针所指向的成员。  
> (类对象名.*指向非静态成员函数的指针)(参数列表)  
> (类对象指针->*指向非静态成员函数的指针)(参数列表)  

