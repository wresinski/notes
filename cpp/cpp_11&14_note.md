## C++11&14笔记 ##
* pack
  * 用于template parameters，就是template parameters pack（模板参数包）
  * 用于function parameters types，就是function parameters types pack（函数类型参数包）
  * 用于function parameters，就是function parameters pack（函数参数包）
----
1. Variadic Template

```cpp
void print() {}

template <typename T, typename... Types>
void print(const T &firstArg, const Types &...args) {
    cout << firstArg << endl;
    print(args...);
}

// 可与上方代码并存
template <typename Types>
void print (const Types&... args)
{ /*...*/ }
```

```cpp
// 4
#include <functional>
template <typename T>
inline void hash_combine(size_t& seed, const T& val) {
    seed ^= std::hash<T>()(val) + 0x9e3779b9
	+(seed<<6) + (seed>>2);
}

// 3
// auxiliary generic function
template <typename T>
inline void hash_val(size_t& seed, const T& val) {
    hash_combine(seed, val);
}

// 2
//hash function
template <typename T, typename... Types>
inline void hash_val(size_t& seed,
		     const T& val,const Types&... args) {
    hash_combine(seed, val);
    hash_val(seed, args...);
}

// 1
// auxiliary generic function
template <typename... Types>
inline size_t hash_val(const Types&... args) {
    size_t seed = 0;
    hash_val(seed, args...);
    return seed;
}

class CustomerHash {
public:
    std::size_t operator() (const Customer& c) const {
	return hash_val(c.fname, c.lname, c.no);
    }
};
```

```cpp
void printf(const char *s)
{
    while (*s)
    {
        if (*s=='%'&&*(++s)!='%')
	    throw std::runtime_error("invalid format string: missing arguments");
	std::cout << *s++;
    }
}

template<typename T, typename... Args>
void printf(const char* s, T value, Args... args)
{
    while (*s)
    {
        if (*s=='%'&&*(++s)!='%') {
	    std::cout << value;
	    printf(++s, args...);
	    return;
	}
	std::cout << *s++;
    }
    throw std::logic_error("extra arguments provided to printf");
}
```

```cpp
int maximum(int n)
{
    return n;
}

template <typename... Args>
int maximum(int n, Args... args)
{
    return std::max(n, maximum(args...)); // 其实std::max()已可接受任意个数之参数值
}
```

```cpp
// boost: util/printtuple.hpp
// helper: print element with index IDX of tuple
// with MAX elements
template<int IDX, int MAX, typename... Args>
struct PRINT_TUPLE {
    static void print(ostream& os, const tuple<Args...>& t) {
	os << get<IDX>(t) << (IDX+1==MAX?"":",");
	PRINT_TUPLE<IDX+1, MAX, Args...>::print(os,t);
    }
};

// partial specialization to end recursion
template<int MAX, typename... Args>
struct PRINT_TUPLE<MAX, MAX, Args...> {
    static void print (std::ostream& os, const tuple<Args...>& t) {
    }
};

// output operator for tuples
template<typename... Args>
ostream& operator<<(ostream& os, const tuple<Args...>& t) {
    os << "[";
    PRINT_TUPLE<0, sizeof...(Args), Args...>::print(os,t);
    return os << "]";
}

cout << make_tuple(7.5, string("hello"), bitset<16>(377), 42);
// provided there is a overloaded operator<<for bitset.
// [7.5,hello,0000000101111001,42]
```

```cpp
// 递归继承
template <typename... Values> class tuple;
template <> class tuple<> {};

template <typename Head, typename... Tail>
class tuple<Head, Tail...>
    : private tuple<Tail...>    
{
    typedef tuple<Tail...> inherited;
public:
    tuple() {}
    tuple(Head v, Tail... vtail)
	:m_head(v),inherited(vtail...) {} // 呼叫base ctor并予参数

    typename Head::type head() { return m_head; }
    inherited& tail() { return *this; } // return后转型为inherited
protected:
    Head m_head;
};
```

```cpp
// 组合继承
template <typename... Values> class tup;
template <> class tup<> {};

template <typename Head, typename... Tail>
class tup<Head, Tail...>
{
    typedef tup<Tail...> composited;
protected:
    composited m_tail;
    Head m_head;
public:
    tup() { }
    tup(Head v, Tail... vtail)
	: m_tail(vtail...), m_head(v) { }

    Head head() { return m_head; }
    composited& tail() { return m_tail; }
};
```

* `sizeof...(args)`：参数包内参数个数

2. C++11新增

* [C++ keyword](https://en.cppreference.com/w/cpp/keyword)

* Space in Template Expressions

```cpp
vector<list<int> >; // OK in each C++ version
vector<list<int>>;  // OK since C++11
```

* `nullptr`and`std::nullptr_t`

```cpp
void f(int);
void f(void*);
f(0);         // calls f(int)
f(NULL);      // calls f(int) if NULL is 0, ambiguous otherwise
f(nullptr);   // calls f(void*)
```

4.9.2\include\stddef.h
```cpp
#if defined(__cplusplus) && __cplusplus >= 201103L
#ifndef _GXX_NULLPTR_T
#define _GXX_NULLPTR_T
  typedef decltype(nullptr) nullptr_t;
```

* Automatic Type Deduction with auto

```cpp
auto l = [](int x)->bool {
...,
};
```

* Uniform Initialization（一致性初始化）

```cpp
int values[]{1, 2, 3};
vector<int> v{2, 3, 5, 7, 11, 13, 17};
vector<string> cities{"Berlin",       "New York", "London",
                      "Braunschweig", "Cairo",    "Cologne"};
complex<double> c{4.0, 3.0}; //equivalent to c(4.0,3.0)
```

* `constexpr`  
`const`并未区分出编译期常量和运行期常量  
`constexpr`限定在了编译期常量

3. Initializer Lists

```cpp
int i;    // i has undefined value
int j{};  // j is initialized by 0
int* p;   // p has undefined value
int* q{}; // q is initialized by nullptr

int x1(5.3);     // OK,but OUCH:x1 becomses 5
int x2 = 5.3;    // OK,but OUCH:x2 becomses 5
int x3(5.0);     // ERROR:narrowing
int x4 = {5.3};  // ERROR:narrowing
char c1{7};      // OK:even  though 7 is an int,this is not narrowing
char c2{99999};  // ERROR:narrowing(if 99999 doesn't fit into a char)
std::vector<int> v1 { 1, 2, 4, 5 }; // OK
std::vector<int> v2 { 1, 2.3, 4, 5.6 }; // ERROR:narrowing
// 以上的ERROR在GCC中是warning
```

* `initializer_list<>`

其背后是`array`

```cpp
void print(initializer_list<int> vals)
{
    for (auto p=vals.begin(); p!=vals.end(); ++p) { // a list of values
        cout << *p << "\n";
    }
}
print({12,3,5,7,11,13,17}); // pass a list of values to print()
```
```cpp
P p(77,5);    // P(int, int), a=77, b=5
P q{77,5};    // P(initializer_list<int>), values= 77 5
P r{77,5,42}; // P(initializer_list<int>), values= 77 5 42
P s={77,5};   // P(initializer_list<int>), values= 77 5
// 2和4在没有P(initializer_list<int>)时也可以调用P(int,int)

max({...});
min({...});
```
```cpp
template<class _E>
  class initializer_list
  {
  public:
    typedef _E 		value_type;
    typedef const _E& 	reference;
    typedef const _E& 	const_reference;
    typedef size_t 		size_type;
    typedef const _E* 	iterator;
    typedef const _E* 	const_iterator;

  private:
    iterator			_M_array;
    size_type			_M_len;

    // The compiler can call a private constructor.
    constexpr initializer_list(const_iterator __a, size_type __l)
    : _M_array(__a), _M_len(__l) { }

  public:
    constexpr initializer_list() noexcept
    : _M_array(0), _M_len(0) { }

    // Number of elements.
    constexpr size_type
    size() const noexcept { return _M_len; }

    // First element.
    constexpr const_iterator
    begin() const noexcept { return _M_array; }

    // One past the last element.
    constexpr const_iterator
    end() const noexcept { return begin() + size(); }
  };
```

* range-based for statement

```cpp
for ( decl : coll ) {
	statement
}
```

4. `array`

```cpp
template <typename _Tp, std::size_t _Nm>
  struct array
  {
      typedef _Tp          value_type;
      typedef _Tp*         pointer;
      typedef value_type*  iterator;
  
      // Support for zero-sized arrays mandatory.
      value_type _M_instance[_Nm ? _Nm : 1];
  
      iterator begin()
      { return iterator(&_M_instance[0]); }
  
      iterator end()
      { return iterator(&_M_instance[_Nm]); }
  
      ...
  };
```

5. `explicit`
* C++11之前：单参数构造函数（包括除第一参数以外参数有默认值的）加上`explicit`关键字可拒绝隐式构造
* C++11之后：可以是多参数构造函数

```cpp
class P
{
    public:
    P(int a, int b) {
	cout << "P(int a, int b) \n";
    }

    P(initializer_list<int>) {
	cout << "P(initializer_list<int>) \n";
    }

    explicit P(int a, int b, int c) {
	cout << "explicit P(int a, int b, int c) \n";
    }
};
void fp(const P&) { };

// explicit ctor with multi-arguments
P p1 (77, 5);             // P(int a, int b)
P p2 {77, 5};             // P(initializer_list<int>)
P p3 {77, 5, 42};         // P(initializer_list<int>)
P p4 = {77, 5};           // P(initializer_list<int>)
//! P p5 = {77, 5, 42};   // [Error] converting to 'P' from initializer list would use explicit constructor 'P::P(initializer_list<int>)'
P p6(77, 5, 42);          // explicit P(int a, int b, int c)

fp( {47, 11} );           // P(initializer_list<int>)
//! fp( {47, 11, 3} );    // [Error] converting to 'const P' from initializer list would use explicit constructor
fp( P{47, 11} );          // P(initializer_list<int>)
fp( P{47, 11, 3} );       // P(initializer_list<int>)

P p11 {77, 5, 42, 500};   // P(initializer_list<int>)
P p12 = {77, 5, 42, 500}; // P(initializer_list<int>)
P p13 {10};               // P(initializer_list<int>)
```

```cpp
class C
{
public:
    explicit C(const string& s); // explicit(!) type conversion from strings
    ...
};

vector<string> vs;
for (const C& elem : vs) { // ERROR,no conversion  from string to C defined
    cout << elem << endl;  //临时构造的右值对象只能用const引用接住
 }
```

6. `=default`，`=delete`
* 默认构造函数是空函数，但它会自动调用父类构造函数，这是作用之一
* 如果你自行定义了一个ctor，那么编译器就不会再给你一个default ctor
* 如果你强制加上`=default`，就可以重新获得并使用default ctor
* 构造函数、析构函数、拷贝/移动构造、拷贝/移动赋值的默认版本都是public且inline的
* 编译器产出的dtor是non-virtual，除非这个class的base class本身宣告有virtual dtor

```cpp
class Zoo
{
public:
    Zoo(int i1, int i2) : d1(i1), d2(i2) { }
    Zoo(const Zoo&) = delete;              // 拷贝构造
    Zoo(Zoo&&) = default;                  // 移动构造
    Zoo& operator=(const Zoo&) = default;  // 拷贝赋值
    Zoo& operator=(const Zoo&&) = delete;  // 移动赋值
    virtual ~Zoo() { }
	// 以上函数若未定义，编译器会加上默认版本
private:
    int d1,d2;
};
```

```cpp
class Foo
{
public:
    Foo(int i) : _i(i) { }
    Foo() = default; // 于是和上一个并存（ctor可以多个并存）

    Foo(const Foo& x) : _i(x._i) { }
    //! Foo(const Foo&) = default; // [Error] 'Foo::Foo(const Foo&)' cannot be overload
    //! Foo(const Foo&) = delete; // [Error] 'Foo::Foo(const Foo&)' cannot be overload

    Foo& operator=(const Foo& x) { _i = x._i; return *this; }
    //! Foo& operator=(const Foo& x) = default; // [Error] 'Foo& Foo::operator=(const Foo&)' cannot be overload
    //! Foo& operator=(const Foo& x) = delete; // [Error] 'Foo& Foo::operator=(const Foo&)' cannot be overload

    //! void func1() = default; //[Error] 'void Foo::func1()' cannot be defaulted
    void func2() = delete; // ok

    //! ~Foo() = delete; // 这会造成使用Foo object时出错=>[Error] use of deleted function 'Foo::~Foo()'
    ~Foo() = default;

private:
    int _i;
};

Foo f1(5);
Foo f2; // 如果没有写出=default版本=>[Error] no matching function for call to
        // 'Foo::Foo()'
Foo f3(f1); // 如果没有copy ctor = delete; [Error] use of deleted function
           // 'Foo::Foo(const Foo&)'
f3 = f2;   // 如果copy assign = delete; [Error] use of deleted function 'Foo& Foo::operator=(const Foo&)'
```

* `=default`：只可用于构造函数、析构函数、拷贝/移动构造、拷贝/移动赋值（六大函数编译器默认给出）
* `=delete`：可用于任何函数身上（=0只能用于virtual函数）

7. [当心编译器把变量声明当成函数声明！](https://blog.csdn.net/qfturauyls/article/details/108556707)

* 两条C++规则
  * C++中的普遍规律，即尽可能地解释为函数声明。
  * 把形式参数的声明用括号括起来是非法的，但给函数参数加上括号却是合法的

* 以下两种情况可能会发生变量声明被误认为函数声明
  * `A a();`
  * `A a(B(i));`（嵌套类型）
	
* 解决方法
  * 多增加一对括号来跳过编译器的分析机制！（将形参的声明用括号括起来是非法的，但给函数参数加上括号却是合法的）
  ```cpp
  A a((B(i)));
  ```
  * 先为变量取名
  ```cpp
  B b(i);
  A a(b);
  ```
  * 使用大括号`{}`初始化
  ```cpp
  A {B(i)};
  ```

* C++语法分析机制

下面这行代码声明了一个带`double`参数并返回`int`的函数：  
`int _Func( double _InDouble );`  
下面这行代码做了同样的事情，参数_InDouble两边的括号是多余的，会被忽略：  
`int _Func( double ( _InDouble ) );`  
下面这行代码声明了同样的函数，只是它省略了参数名称：  
`int _Func( double );`

再来看三个函数声明，第一个声明了一个函数`_Fuck()`，他的参数是一个不接收任何参数并返回一个`double`类型的函数的指针：  
`int _Fuck( double( *_ptrFunc ) () );`  
有另外一种方式可以表明同样的意思，唯一的区别是，`_ptrFunc`用非指针的形式来声明(这种形式在C语言和C++中都有效)：  
`int _Fuck( double _ptrFunc() );`  
跟通常一样，参数名称可以忽略，因此下面是`_Fuck()`的第三种声明，其中参数名`_ptrFunc`被省略了：  
`int _Fuck( double () );`

* Tips：可以将变量`return`并查看编译器报错信息来确定变量的类型

8. No-Copy and Private-Copy

* No-Copy

```cpp
struct NoCopy {
    NoCopy() = default;  // use the synthesized default constructor
    NoCopy(const NoCopy&) = delete;  // no copy
    NoCopy &operator=(const NoCopy&) = delete;  // no assignment
    ~NoCopy() = default;  // use the synthesized destructor
    // other members
};
```

* No-Dtor

```cpp
struct NoDtor {
    NoDtor() = default;  // use the synthesized default constructor
    ~NoDtor() = delete;  // we can't destroy object of type NoDtor
};
NoDtor nd; // error:NoDtor destructor is deleted
NoDtor *p = new NoDtor(); // ok:but we can't delete p
delete p;                 // error:NoDtor destructor is deleted
```
`=delete`告诉编译器不要定义它；必须出现在声明式；适用于任何成员函数；但若用于dtor后果自负

* Private-Copy

```cpp
class PrivateCopy {
private:
    // no access specifier;following members are private by default;
    // copy control is private and so is inaccessible to ordinary user code
    PrivateCopy(const PrivateCopy&);
    PrivateCopy &operator=(const PrivateCopy&);
    // other members
public:
    PrivateCopy() = default;  // use the synthesized default constructor
    ~PrivateCopy();  // users can define objects of this type but not copy them
};
```
此class不允许被ordinary user code copy，但仍可被friends和members copy；若欲完全禁止，不但必须把copy controls放到private内且不可定义之

例子：\boost\noncopyable.hpp
```cpp
namespace boost {
    // Private copy constructor and copy assignment ensure classes derived from
    // class noncopyable cannot be copied
    // Contributed by Dave Abrahams

namespace noncopyable_ { // protection from uninitended ADL
    class noncopyable
    {
    protected:
	noncopyable() { }
        ~noncopyable)_ { }
    private: // emphasize the following members are private
        noncopyable(const noncopyable&);
        const noncopyable& operator=(const noncopyable&);
    };
}

typedef noncopyable_::noncopyable noncopyable;
} // namespace boost
```
以上类的主要用途是被继承

9. Alias Template(template typedef)

```cpp
template <typename T>
using Vec = std::vector<T,MyAlloc<T>>;

Vec<int> coll;
```

* 宏和typedef无法达到同样效果
* 无法对alias做特化和偏特化，必须对原模版做

* alias template可帮助容器作为模板模板参数

```cpp
template <typename T>
using vec = vector<T, allocator<T>>;

template <template<typename> class Container, typename T>
void printcon(int n, T i) {
    Container<T> x(n,i);
    for ( auto a : x ) {
	cout << a << ' ';
    }
    cout << endl;
}

printcon<vec,string>(5, "Daisy");
```

10. Typedef Alias(similar to typedef)

```cpp
// type alias, identical to
// typedef void (*func)(int, int);
using func = void (*)(int,int);

// the name 'func' now denotes a pointer to function:
void example(int, int) {}
func fn = example;
```

```cpp
// type alias can introduce a members typedef name
template<typename T>
struct Container {
    using value = T; // typedef T value_type
};

// which can be used in generic programming
template<typename Cntr>
void fn2(const Cntr& c)
{
    typename Cntr::value_type n;
}
```

11. `using`
* using-directives for namespaces and using-declarations for namespace members
```cpp
using namespace std;
using std::cout;
```
* using-declarations for class members（确定类中用到某个类型时所属的命名空间）
```cpp
class bvector : public _Base
{
protected:
	using _Base::_M_allocate;
};
```
* type alias and alias template declaration(since C++11)

12. `noexcept`
* `noexcept`可以指定在什么条件下不抛异常

```cpp
void foo() noexcept; // void foo() noexcept(true);

void swap(Type& x, Type& y) noexcept(noexcept(x.swap(y)))
{
	x.swap(y);
}
```

* 异常如果没有被处理，会向调用者传递，如果最终没有被处理，则会调用`std::terminate()`，然后其默认调用`std::abort()`

* 只有当类的移动构造和移动赋值函数是`noexcept`的时，`std::vector`才会在2倍成长时调用移动构造函数来移动元素（移动构造函数和移动赋值函数一定不要抛异常也一定要写`noexcept`！！！）

13. `override`
* 标明了该成员函数重载了父类的成员函数，若实际没有重载，则报错

```cpp
struct Base {
    virtual void vfunc(float) { }
};

struct Derived1 : Base {
    virtual void vfunc(int) { }
    // accidentally create a new virtual function, when one intended to override a base class function.
    // This is a common problem, particularly when a user goes to modify the base class.
};

struct Derived2 : Base {
    virtual void vfunc(int) override { }
    // [Error] 'virtual void Derived2::vfunc(int)' marked override, but does not override

    // override means that the compiler will check the base class(es) to see if there is
    // a virtual function with this exact signature.
    // And if there is not, the compiler will indicate an error

    virtual void vfunc(float) override { }
};
```

14. `final`
* 限定类无法再被继承或者虚函数无法再被重载

```cpp
struct Base1 final {};

struct Derived1 : Base1 {};
   // [Error] cannot derive from 'final' base 'Base1' in derived type 'Derived1'

struct Base2 {
    virtual void f() final;
};

struct Derived2 : Base2 {
    void f();
    // [Error] overriding final function 'virtual void Base2::f()'
};
```

15. `decltype`

* 用来获取一个表达式（或对象）的类型
```cpp
map<string, float> coll;
decltype(coll)::value_type elem; // => map<string, float>::value_type elem;
```

* 用途
  * to declare return types
  ```cpp
  //template <typename T1, typename T2>
  //decltype(x+y) add(T1 x, T2 y);
  // x和y出现在其声明前，所以编译不通过
  ...
  template <typename T1, typename T2>
  auto add<T1 x, T2 y) -> decltype(x+y);
  // C++11新语法：auto func() -> int
  ```
  * metaprogramming
  ```cpp
  template <typename T>
  void test18_decltype(T obj)
  {
	  typedef typename decltype(obj)::iterator iType; // typedef typename T::iterator iType;
	  decltype(obj) anotherObj(obj);
  }
  ```
  * to pass the type of a lambda 
  ```cpp
  auto cmp = [](const Person &p1, const Person &p2) {
  	  return p1.lastname()<p2.lastname() ||
  		(p1.lastname()==p2.lastname()) &&
  		p1.firstname()<p2.firstname());
  };
  std::set<Person, decltype(cmp)> coll(cmp);
  ```
  
16. lambdas
* $[...]\;(...)\;mutable_{opt}\;throwSpec_{opt}\;->\;retType_{opt}\;\{...\}$
  * 三个选项只要有一个存在，就必须写小括号
  ```cpp
  // 最简形式
  [] { std::cout << "hello lambda" << std::endl; }
  ...
  [] {
      std::cout << "hello lambda" << std::endl;
  }
  (); // prints "hello lambda"
  ... 
  auto I = [] { std::cout << "hello lamda" << std::endl; };
  I();// prints "hello lambda"
  ```
  * `[...]`内为捕获的外部变量，默认为值传递且静态变量，值传递可指定默认值例如`[i=1]`，`&x`为传引用，`=`为捕获除引用的以外全部的外部变量并值传递；`[&]`可以捕获所有外部变量的引用（注意别使用局部变量，`=`捕获时也小心隐式`this`指针）
  * `mutable`：对于值传递，需要加上`mutable`才能更改
  ```cpp
  int id = 0;
  auto f = [id]() mutable {
    cout << "id:" << id << endl;
    ++id; // OK
  };
  id = 42;
  f();                           // id:0
  f();                           // id:1
  f();                           // id:2
  std::cout << id << std::endl;  // 42
  ```
  ```cpp
  int id = 0;
  auto f = [&id](int param) {
    cout << "id:" << id << endl;
    ++id;++param; // OK
  };
  id = 42;
  f();                           // id:42
  f();                           // id:43
  f();                           // id:44
  std::cout << id << std::endl;  // 45
  ```
  ```cpp
  int id = 0;
  auto f = [id]() {
    cout << "id:" << id << endl;
    ++id; // [Error] increment of read-only variable 'id'
  };
  id = 42;
  f();
  f();
  f();
  std::cout << id << std::endl;
  ```
  * lambda是inline的
  * lambda相当于一个匿名的function object（仿函数）
  ```cpp
  int id = 0;
  auto f = [id]() mutable {
    cout << "id:" << id << endl;
    ++id; // OK
  };
  // 以上代码相当于：
  class Functor {
    private:
      int id; // copy of outside id
    public:
      Functor(int outside_id) : id(outside_id) {}
      void operator() () {
  	std::cout << "id:" << id << std::endl;
  	++id; // OK
      }
  };
  Functor f(id);
  ```
  * lambda内也可以声明static变量，并且捕获的值传递变量也是static的
  * 容器的排序准则可以传入仿函数、函数指针或lambda对象 
  * 无法用lambda的类型来创建未初始化变量，会报错`error: use of deleted function`——lambda没有默认构造函数且无法被赋值

Tips：`remove_if()`只能将元素挪到末尾，因此往往搭配`erase()`使用：`vi.erase(remove_if(vi.begin(), vi.end() ,[x,y](int n){return x<n&&n<y);}), vi.end());`

17. Rvalue references

* 左值和右值
Lvalue：**可以**出现于`operator=`左侧者  
Rvalue：**只能**出现于`operator=`右侧者（a+b是右值）

```cpp
// 以int试验
int a = 9;
int b = 4;
a = b; // ok
b = a; // ok
a = a + b; // ok
a + b = 42; // [Error] lvalue required as left operand of assignment

// 以string试验
string s1("Hello");
string s2("World");
s1 + s2 = s2; // 竟然通过编译
cout << "s1:" << s1 << endl; // s1: Hello
cout << "s2:" << s2 << endl; // s2: World
string() = "World"; // 竟然可以对temp obj赋值

// 以complex试验
complex<int> c1(3, 8), c2(1, 0);
c1 + c2 = complex<int>(4, 9); // c1+c2可以当做Lvalue吗？
cout << "c1:" << c1 << endl;  // c1: (3,8)
cout << "c2:" << c2 << endl;  // c2: (1,0)
complex<int>() = complex<int>(4, 9); // 竟然可以对temp obj赋值
```

* Rvalue references（右值引用本质是把引用分为不同的类型来区分左值和右值，从而调用不同的函数）  
  * 右值当做实参时，右值引用版本的函数优先被调用；其次可以调用`const`的左值引用版本，而无`const`的左值引用版本不能以右值为实参
  * 左值当做实参时，可以使用`std::move()`强制调用右值引用版本（必须保证后续不再使用该变量）

18. perfect forwarding
* unperfect forwarding（连索调用时右值会被转为左值）

```cpp
void process(int &i) { cout << "process(int&):" << i << endl; }
void process(int &&i) { cout << "process(int&&):" << i << endl; }
void forward(int&& i) {
    cout << "forward(int&&):" << i << ",";
    process(i);
}

int a = 0;
process(a); // process(int&): 0
            // 变量被视为lvalue处理
process(1); // process(int&&): 1
            // temp object被视为Rvalue处理
process(move(a)); // process(int&&): 0
                  // 强制将a由Lvalue改为Rvalue
forward(2);       // forward(int&&): 2, process(int&):2
                  // Rvalue经由forward()传给另一函数却变成Lvalue
            // （原因是传递过程中它变成了一个named object）
forward(move(a)); // forward(int&&): 0, process(int&): 0
                  // Rvalue经由forward()传给另一函数却变成Lvalue
//! forward(a);   // [Error] cannot bind 'int' lvalue to 'int&&'
const int &b = 1;
//! process(b);   // [Error] no matching function for call to 'process(const int&)'
//! process(move(b)); // [Error] no matching function for call to
                      // 'process(std::remove_reference<const int&>::type)' //std::remove_reference=>type traits
//! int& x(5);        // [Error] invalid initialization of non-const reference of
	                  // type 'int&' from an rvalue of type 'int'
//! process(move(x)); // 上行失败，本行当然无庸置疑: "x" was not declared in this scope
```

* perfect forwarding

```cpp
template<typename T1, typename T2>
void functionA(T1&& t1, T2&& t2)
{
    functionB(std::forward<T1>(t1),
	      std::forward<T2>(t2));
}
```
```cpp
/**
 *  @brief  Forward an lvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type& __t) noexcept
  { return static_cast<_Tp&&>(__t); }

/**
 *  @brief  Forward an rvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
  {
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
	    " substituting _Tp is an lvalue reference type");
    return static_cast<_Tp&&>(__t);
  }

/**
 *  @brief  Convert a value to an rvalue.
 *  @param  __t  A thing of arbitrary type.
 *  @return The parameter cast to an rvalue-reference to allow moving it.
*/
template<typename _Tp>
  constexpr typename std::remove_reference<_Tp>::type&&
  move(_Tp&& __t) noexcept
  { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

19. move aware class

```cpp
class MyString {
public:
    static size_t DCtor; // 累计default-ctor调用次数
    static size_t Ctor;  // 累计ctor调用次数
    static size_t CCtor; // 累计copy-ctor调用次数
    static size_t CAsgn; // 累计copy-asgn调用次数
    static size_t MCtor; // 累计move-ctor调用次数
    static size_t MAsgn; // 累计move-asgn调用次数
    static size_t Dtor;  // 累计dtor调用次数
private:
    char* _data;
    size_t _len;
    void _init_data(const char *s) {
	_data = new char[_len+1];
	memcpy(_data, s, _len);
	_data[_len] = '\0';
    }
public:
    // default constructor
    MyString() : _data(NULL), _len(0) { ++DCtor; }

    // constructor
    MyString(const char* p) : _len(strlen(p)) {
	++Ctor;
	_init_data(p);
    }

    // copy constructor
    MyString(const MyString& str) : _len(str._len) {
	++CCtor;
	_init_data(str._data); // COPY
    }

    // move constructor, with "noexcept"
    MyString(MyString&& str) noexcept
	: _data(str._data), _len(str._len) {
	++MCtor;
	str._len = 0;
	str._data = NULL; // 重要
	// 不能删除str，让它的析构函数自己清理
    }

    // copy assignment
    MyString& operator=(const MyString& str) {
	++CAsgn;
	if (this != &str) {
	    if (_data) delete _data;
	    _len = str._len;
	    _init_data(str._data); //COPY!
	}
	else {
	}
	return *this;
    }

    // move assignment
    MyString& operator=(MyString&& str) noexcept {
	++MAsgn;
	if (this != &str) {
	    if (_data) delete _data;
	    _len = str._len;
	    _data = str._data; // MOVE!
	    str._len = 0;
	    str._data = NULL; // 重要
	}
	return *this;

	./a.
	      }

    // dtor
    virtual ~MyString() {
	++Dtor;
	if (_data) {
	    delete _data;
	}
    }
    bool operator<(const MyString &rhs) const // 为了set
    {
	return
	    string(this->_data) < string(rhs._data);
	// 借用现成事实：std::string能够比较大小
    }
    bool
    operator==(const MyString& rhs) const // 为了set
    {
	return
	    string(this->_data) == string(rhs._data);
	// 借用现成事实：string能够判断相等与否
    }

    char* get() const { return _data; }
};

size_t MyString::DCtor = 0;
size_t MyString::Ctor = 0;
size_t MyString::CCtor = 0;
size_t MyString::CAsgn = 0;
size_t MyString::MCtor = 0;
size_t MyString::MAsgn = 0;
size_t MyString::Dtor = 0;

namespace std // 必须放在std内
{    
	template<>
	struct hash<MyString> { // 为了unordered containers
	size_t
	operator() (const MyString& s) const noexcept
	{ return hash<string>() (string(s.get())); }
	// 借用现成的hash<string>
	// (...\4.9.2\include\c++\bits\basic_string.h)
	};
}
```
* 移动构造和拷贝构造函数必须是`noexcept`
* move aware对vector效率影响巨大，对deque的中间插入也有影响；对节点型容器影响不大

Tips：析构函数默认是`noexcept`

20. hash function

* 整型哈希  
`...\4.9.2\include\c++\bits\functional_hash.h`

```cpp
  template<typename _Result, typename _Arg>
    struct __hash_base
    {
      typedef _Result     result_type;
      typedef _Arg      argument_type;
    };

  /// Primary class template hash.
  template<typename _Tp>
    struct hash;

  /// Partial specializations for pointer types.
  template<typename _Tp>
    struct hash<_Tp*> : public __hash_base<size_t, _Tp*>
    {
      size_t
      operator()(_Tp* __p) const noexcept
      { return reinterpret_cast<size_t>(__p); }
    };

  // Explicit specializations for integer types.
#define _Cxx_hashtable_define_trivial_hash(_Tp) 	\
  template<>						\
    struct hash<_Tp> : public __hash_base<size_t, _Tp>  \
    {                                                   \
      size_t                                            \
      operator()(_Tp __val) const noexcept              \
      { return static_cast<size_t>(__val); }            \
    };

  /// Explicit specialization for bool.
  _Cxx_hashtable_define_trivial_hash(bool)

  /// Explicit specialization for char.
  _Cxx_hashtable_define_trivial_hash(char)

  /// Explicit specialization for signed char.
  _Cxx_hashtable_define_trivial_hash(signed char)

  /// Explicit specialization for unsigned char.
  _Cxx_hashtable_define_trivial_hash(unsigned char)

  /// Explicit specialization for wchar_t.
  _Cxx_hashtable_define_trivial_hash(wchar_t)

  /// Explicit specialization for char16_t.
  _Cxx_hashtable_define_trivial_hash(char16_t)

  /// Explicit specialization for char32_t.
  _Cxx_hashtable_define_trivial_hash(char32_t)

  /// Explicit specialization for short.
  _Cxx_hashtable_define_trivial_hash(short)

  /// Explicit specialization for int.
  _Cxx_hashtable_define_trivial_hash(int)

  /// Explicit specialization for long.
  _Cxx_hashtable_define_trivial_hash(long)

  /// Explicit specialization for long long.
  _Cxx_hashtable_define_trivial_hash(long long)

  /// Explicit specialization for unsigned short.
  _Cxx_hashtable_define_trivial_hash(unsigned short)

  /// Explicit specialization for unsigned int.
  _Cxx_hashtable_define_trivial_hash(unsigned int)

  /// Explicit specialization for unsigned long.
  _Cxx_hashtable_define_trivial_hash(unsigned long)

  /// Explicit specialization for unsigned long long.
  _Cxx_hashtable_define_trivial_hash(unsigned long long)

#undef _Cxx_hashtable_define_trivial_hash
```

* 浮点型哈希  
`...\4.9.2\include\c++\bits\functional_hash.h`

```cpp
struct _Hash_impl
{
  static size_t
  hash(const void* __ptr, size_t __clength,
 size_t __seed = static_cast<size_t>(0xc70f6907UL))
  { return _Hash_bytes(__ptr, __clength, __seed); }

  template<typename _Tp>
    static size_t
    hash(const _Tp& __val)
    { return hash(&__val, sizeof(__val)); }

  template<typename _Tp>
    static size_t
    __hash_combine(const _Tp& __val, size_t __hash)
    { return hash(&__val, sizeof(__val), __hash); }
};

struct _Fnv_hash_impl
{
  static size_t
  hash(const void* __ptr, size_t __clength,
 size_t __seed = static_cast<size_t>(2166136261UL))
  { return _Fnv_hash_bytes(__ptr, __clength, __seed); }

  template<typename _Tp>
    static size_t
    hash(const _Tp& __val)
    { return hash(&__val, sizeof(__val)); }

  template<typename _Tp>
    static size_t
    __hash_combine(const _Tp& __val, size_t __hash)
    { return hash(&__val, sizeof(__val), __hash); }
};

/// Specialization for float.
template<>
  struct hash<float> : public __hash_base<size_t, float>
  {
    size_t
    operator()(float __val) const noexcept
    {
// 0 and -0 both hash to zero.
return __val != 0.0f ? std::_Hash_impl::hash(__val) : 0;
    }
  };

/// Specialization for double.
template<>
  struct hash<double> : public __hash_base<size_t, double>
  {
    size_t
    operator()(double __val) const noexcept
    {
// 0 and -0 both hash to zero.
return __val != 0.0 ? std::_Hash_impl::hash(__val) : 0;
    }
  };

/// Specialization for long double.
template<>
  struct hash<long double>
  : public __hash_base<size_t, long double>
  {
    _GLIBCXX_PURE size_t
    operator()(long double __val) const noexcept;
  };

// @} group hashes

// Hint about performance of hash functor. If not fast the hash based
// containers will cache the hash code.
// Default behavior is to consider that hasher are fast unless specified
// otherwise.
template<typename _Hash>
  struct __is_fast_hash : public std::true_type
  { };

template<>
  struct __is_fast_hash<hash<long double>> : public std::false_type
  { };
```

* string哈希  
`...\4.9.2\include\c++\bits\basic_string.h`

```cpp
template<>
  struct hash<string>
  : public __hash_base<size_t, string>
  {
    size_t
    operator()(const string& __s) const noexcept
    { return std::_Hash_impl::hash(__s.data(), __s.length()); }
  };
```

21. `tuple`的旧版实现

* 《Modern C++ Design》中实现的`tuple`
```cpp
#define TYPELIST_1(T1) Typelist<T1, NullType>
#define TYPELIST_2(T1, T2) Typelist<T1, TYPELIST_1(T2)>
#define TYPELIST_3(T1, T2, T3) Typelist<T1, TYPELIST_2(T2, T3)>
#define TYPELIST_4(T1, T2, T3, T4) Typelist<T1, TYPELIST_3(T2, T3, T4)>
```

* boost中同样使用暴力枚举的方式实现了`tuple`

Tips：mete programming（元编程）是对类型进行操作而不像一般编程对变量进行操作

22. 智能指针

* 大多数时候使用`unique_ptr`，需要共享时再使用`shared_ptr`，循环引用时将其中一个改为`weak_ptr`
* 传参：
  * 函数内部只使用，不保存对象内容时
	```cpp
	void func(Widget*);
	void func(const shared_ptr<Widget>&)
	```
  * 函数中需要保存指针时
	```cpp
	void func(std::shared_ptr<Widget> ptr);
	```

23. inline
* inline在现代C++中的效果是声明一个函数为weak符号，和性能优化意义上的内联无关
* 优化意义上的内联指把函数体直接放到调用者那里去
