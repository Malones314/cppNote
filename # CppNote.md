# 类
1. 常用关键字和名称
1.1 ```ctor```: 构造函数
1.2 ```dtor```: 析构函数
1.3 ```explict```: 常用于构造函数前, 说明该构造函数只能显式调用, 让该构造函数不能隐式调用和复制初始化。
1.4 ```public```: 只有当前类能访问
1.5 ```protected```: 只有当前类和子类能访问
1.6 ```public```: 所有类都能访问
1.7 ```className()```: 临时对象

2. 类中能加```const```则要加```const```

3. 当成员函数的 ```const``` 版本和 ```non-const``` 版本同时存在时, ```const``` 对象只能调用 ```const``` 函数，```non-const``` 对象只能调用 ```non-const``` 函数

	```const``` 成员函数可以被 ```const``` 对象和 ```non-const``` 对象调用```non-const``` 成员函数只能被 ```non-const``` 对象调用

4. 类的构造函数要使用初始化列表

5. 如果类中有指针则必须要带有：
	```className( className& a)```: 拷贝构造函数
	```className& operator=( className& a)```:重载赋值操作符
	这两个函数和析构函数被称为Big Three

6. 如果类中有指针，默认的拷贝构造函数是**浅拷贝**，会造成内存泄露和指针指向同一位置
7. 
	```CPP
	classname c1;
	c1.memberFunction();	
		//等价于：classname::memberFunction( &c1);
		//c1相当于this指针, 如果函数为static函数则没有这一步
	```

8. ```static```成员函数没有```this```指针, 只能处理```static```数据

9. 调用```static```函数可以用object调用, 也可以通过classname调用

	```static```数据要在类外做一次定义(给不给初值都可以),类内只是声明

10. ```empty class``` 在经过编译器编译后就不是empty了, 有一个```copy ctor```、一个```copy assignment operator```、一个```dtor```, 所有这些function都是```public```且```inline```

11. ```default ctor``` 和 ```dtor``` 主要给编译器一个地方存放‘藏身幕后’的code, 
eg: 唤起```base classes```、```non-static members```的```ctors```和```dtors```

12. 编译器产生的```dtor```是```non-virtual```, 除非这个```class```的```base class```本身宣告有```virtual dtor```

13. 如果```class```中有```pointer```则要写big-three/big-Five, 如果没有则一般不需要写

14. 类中如果没有任何参数，占用的内存空间是0，但是```sizeof```显示的是1, 如果有虚函数```sizeof```是4(虚表指针)

## 拷贝赋值的重载

需要检测自我赋值, 好处：
	1.速度快, 效率高, 防止不必要的动作
	2.防止a = a的出错, 下例的高亮行如果没有自我赋值检测, 则a = a会出错

```cpp{.line-numbers highlight=5-7}		

classname& operator=( classname& a){	//因为可能连续赋值, 所以返回类型不能为void
	if( this == &a)
		return *this;

	delete[] this->data;				
	this->data = new Type[ a->size()];	
	Copy( this->data, a->data);			
	return *this;
}

 ```
## new / delete
```new``` 表示在堆上分配内存的请求，如果有足够的可用内存，```new``` 运算符会初始化内存，并将新分配和初始化的内存的地址返回给指针变量。
当使用```new```运算符来创造类对象时，对象的内存是使用堆中的 ```operator new``` 分配的。然后调用对应的```ctor```

```operator new``` 是一个分配原始内存的函数，```operator new``` **可以重载**，它不初始化内存，
即不调用构造函数。但是，在我们重载的 ```new``` 返回后，编译器也会在适用时自动调用构造函数。也可以全局或为特定类重载 ```operator new```。

```new``` 和 ```operator new``` 的主要区别在于：
1. ```new``` 是 C++ 中的**关键字**，```operator new``` 是**全局函数**。
2. ```new``` 运算符使用 ```operator new``` 函数分配内存，返回指向新内存块的指针，并调用构造函数初始化对象，而 ```operator new``` 函数只是分配内存并返回指针，不进行构造函数的初始化。
3. ```operator new``` 函数可以被重载以实现自定义的内存分配方式。

```new```时先分配memory, 再调用构造函数
```delete```时先调用析构函数, 再释放memory

如果```new```了带有指针的类的数组, ```delete```时没有[], 则只会唤起一次```dtor```，使用[]后编译器会知道要调用多少次```dtor```(在cookie后4字节会说明需要调用多少次)
但如果类中不带指针, 则```delete```时不使用[]也不会造成内存泄露(为了一致性, 建议也写[])

```new``` 和 ```delete``` 无法重载, ```new``` 和 ```delete``` 被分解成三个动作, 其中```operator new``` 和 ```operator delete``` 可以重载，还可以重载多种类型的 ```new()```、```delete()```, 但是不同版本要有独一无二的参数列表,
其中的第一参数必须为 ```size_t```
```cpp
new 被分解为：

void* mem = operator new ( sizeof( Type));
p = static_cast<Type*>( mem);
p->Type::Tyoe();

delete 被分解为：

p->~Type();
operator delete( p);
 ```
```cpp
operator new 重载：

void* myAlloc( size_t size){
	return malloc( size);
}
inline void* operator new ( size_t size){
	.....
	return myAlloc( size);
}
inline void* operator new[]( size_t size){
	.....
	return myAlloc( size);
}
 ```

```cpp
operator delete 重载:

void myFree( void* ptr){
	return free( ptr);
}
inline void operator delete( void* ptr){
	.....
	return myFree( ptr);
}
inline void operator delete( void* ptr){
	.....
	return myFree( ptr);
}

Type* a = ::new Type;	//强制调用global operator new
::delete a;		//强制调用global operator delete
 ```
## 多态

C++中的多态性有两种形式：

1. 静态多态性（Static Polymorphism）是通过C++的函数重载（Function Overloading）和运算符重载（Operator Overloading）实现的。在编译时，编译器会根据参数类型或运算符的操作数类型来决定使
用哪个函数或运算符重载。

2. 动态多态性（Dynamic Polymorphism）是通过C++的虚函数（Virtual Function）实现的。虚函数允许派生类重写基
类的函数，当通过基类指针或引用调用虚函数时，实际调用的是派生类的函数。

### 抽象类

当您尝试创建抽象类的实例时，会出现错误。抽象类是至少有一个纯虚函数的类，这意味着
它不能被直接实例化。要修复此错误，需要创建一个继承自抽象类的具体类的实例

### 继承(inheritance): A is-a B (A是一种B

1. 父类(base)数据可以被子类(derived)完全继承
函数的继承是继承调用权(子类能调用父类的函数)

2. 可以通过子类对象调用父类函数。

3. 子类若重写了父类的虚函数则通过子类调用的是子类重写的虚函数

4. Template Method:
	写好框架, 让子类具体实现函数

5. base的析构函数必须是```virtual```, 否则出现undefined behavior

6. 构造由内而外
	derived的ctor首先调用base的default ctor，然后再执行自己。
7. 析构由外而内
	derived的dtor首先执行自己，然后再调用base的dtor

8.
	```Cpp{.line-numbers}

	struct _List_node_base{
		_List_node_base* _M_next;
		_List_node_base* _M_prev;
	};
	template<typename _Tp>
	struct _List_node 
		: public _List_node_base{
			_Tp _M_data;
	};
	```

non-virtual function
	不希望derived class重新定义(override)它
virtual function
	希望derived class重新定义(override)它, 
	且你对它已经有默认定义
pure virtual function
	希望derived class一定要重新定义(override)
	它, 且你对它没有默认定义


### 3种继承方式
1. ```public```继承方式
	1.1 基类中所有 ```public``` 成员在派生类中为 ```public``` 属性
	1.2 基类中所有 ```protected``` 成员在派生类中为 ```protected``` 属性
	1.3 基类中所有 ```private``` 成员在派生类中不能使用

2. ```protected```继承方式
2.1 基类中的所有 ```public``` 成员在派生类中为 ```protected``` 属性
2.2 基类中的所有 ```protected``` 成员在派生类中为 ```protected``` 属性
2.3基类中的所有 ```private``` 成员在派生类中不能使用

3. ```private```继承方式
3.1 基类中的所有 ```public``` 成员在派生类中均为 ```private``` 属性
3.2 基类中的所有 ```protected``` 成员在派生类中均为 ```private``` 属性
3.3 基类中的所有 ```private``` 成员在派生类中不能使用

## 复合(composition): A has-a B (A拥有B)

```Cpp{.line-numbers}
class queue{
	.....
	protected:
		deque<T> c;
	public:
		bool empty() const {
			return c.empty();
		}
		size_type size() const {
			return c.size();
		}
		.....
}
```

构造由内而外:
container的ctor先调用component的ctor，然后在执行自己的。

析构由外而内:
	container的dtor先执行自己，然后再调用component的dtor。

## 委托(delegation):	composition by reference 

两者用指针相连, 两者生命不同步
```Cpp{.line-numbers}

class String{
	public:
		.....
	private:
		StringRep* rep;	//rep指向的东西可以随着需求的改变而改变
}
```
## Inheritance+Composition

1.			
		base 
		|
		|
		derived-----composition

	构造函数调用次序：
		先base, 再composition, 最后derived
	析构函数调用次序：
		先derived, 再composition, 最后base
		
2. 
		base-----composition
		|
		|
		derived

	构造函数调用次序：
		先composition, 后base, 最后derived
	析构函数调用次序：
		先derived, 后base, 最后composition

## Delegation+Inheritance

```Cpp{.line-numbers}
class A{
	vector<B*> b;
}
class B{
	.....
}
//B类可被继承, A中存数据
```
## pointer-like classes: 指针类
```cpp{.line-numbers}
template< class T>
class shared_ptr{
public:
	T& operator*() const{
		return *px;
	}
	T* operator->() const{	//因为->操作符可以一直作用下去, 所以调用
				//返回后还是为px->xxx而非没有->
		return px;
	}
	shared_ptr(T* p): px(p) { }
private:
	T* px;
	long* pn;
.....
};
```

## function-like classes: 仿函数
```cpp{.line-numbers}
template < class T>
struct identity {
	const T& operator()(const T& x) const {
		return x;
	}
};
template< class Pair>
struct select1st {
	const typename Pair::first_type& operator()(const Pair& x)const{
		return x.first;
	}
};
template<class Pair>
struct select2nd {
	const typename Pair::second_type& operator() (const Pair& x) const{
		return x.second;
	}
};
```

## explicit

用于接受一个以上实参的构造函数, 取消ctor的隐式调用

## =default  /  =delete

如果自己定义了一个ctor/dtor那么编译器不会再给你一个default ctor/dtor，如果你强制加上```=default```, 就可以重新获得并使用default ctor/dtor(可以在继承等时候使用)

```=delete```表示显式删除一个函数或者类的特定函数。其作用是阻止编译器自动生成所写类中的某些函数（如默认构造函数、复制构造函数、移动构造函数、复制赋值运算符和移动赋值运算符），从而达到代码控制的目的。

拷贝构造函数不能被重载，如果自己写了拷贝构造函数则不能写=default

一个函数只能在最初定义的时候定义为delete
一般函数不能定义为default

default只能用于构造函数, 拷贝构造函数, 搬移构造函数, 搬移赋值函数, 析构函数

dtor定义=delete则无法destroy objects

```cpp
class A{
public:
	A( int a){
		.....
	}
	A() = default;	//如果没有则A a; 出错
};
```
```cpp
struct NoCopy{
	NoCopy() = default;
	NoCopy( const NoCopy&) = delete;	//无法拷贝
	NoCopy &operator=( const NoCopy&) = delete;	//无法赋值
	~NoCopy() = default;	//使用默认析构函数
	.....
};
struct NoDtor{
	NoDtor() = default;
	~NoDtor() = delete;	//无法destroy objects of type NoDtor
};
NoDtor nd;	//error
NoDtor *np = new NoDtor();	//ok, 但无法delete np
delete np;	//error

class PrivateCopy{	//此class只能被friends和members copy
	private:
		PrivateCopy( const PrivateCopy &);
		PrivateCopy &operator=( const PrivateCopy&);
		.....
	public:
		PrivateCopy() = default;
		~PrivateCopy();
};

class NoCopy {
public:
    NoCopy() {}
    NoCopy(const NoCopy&) = delete;               // 禁用复制构造函数
    NoCopy& operator=(const NoCopy&) = delete;    // 禁用赋值运算符
    // other member functions
};

int main() {
    NoCopy a;
    NoCopy b(a);	// 编译错误：use of deleted function 'NoCopy::NoCopy(const NoCopy&)'
    NoCopy c;
    c = a;	// 编译错误：use of deleted function 'NoCopy& NoCopy::operator=(const NoCopy&)'
    return 0;
}

 ```
# 对象模型(Object Model)
## virtual pointer 和 virtual table

类中如果有 virtual function, 则类大小为数据大小加一根指针的大小, 该指针为 virtual pointer

对象调用函数是静态调用, 指针调用函数是动态调用。
如果是对象调用虚函数则无需多想, 属于哪个Type则调用谁的函数
如果是指针调用虚函数，则new了哪个Type就调用那个的函数

virtual function调用不是静态绑定用call语句的调用, 是通过动态绑定，用 virtual pointer, 指向 virtual table, virtual table用来查 virtual function地址p指向类的对象, vptr为虚指针, n为虚表内函数序号(编译器在编译代码时会给, 从0开始)

```cpp
( *( p->vptr) [n])( p);
(* p->vptr [n])( p);	//也是对的, 因为*的优先级高于[]
```

```cpp
class A{
	virtual void function1(){ .....}
	virtual void function2(){ .....}
};
class B : public A{
	virtual void function1(){ .....}
};
class C : public B{
	virtual void function1(){ .....}
};

A可以调用：
	A::function1()
	A::function2()
B可以调用:
	A::function2()
	B::function1()
C可以调用：
	C::function1()
	A::function2()
```
## override

父类的虚函数子类如果要对其override则两者签名必须要完全相同

子类对父类的虚函数复写可在函数名后面加上overrid告知编译器是对其的复写,让编译器帮助检车是否有签名错误, 如果有则error
eg：
```void function ( int) {.....}```
```void function ( double ) {.....}```
两者签名不同, 不是override, 会把子类的当成一个新函数
如果子类写成：
```void function( double) override {.....}```
编译器报错

## final
finalClass不能被别的类继承, final表示这个类是继承关系下最底层的类，finalFunction不能被子类复写
```cpp
class finalClass final{
	.....
};
class A{
	virtual void finalFunction() final;
};
class B: A{
	void finalFunction();	//[Error] overriding final function
};
 ```
# 函数

函数参数要用reference传递	
函数返回值为函数中创造的本地变量、本地对象，则不能return by reference，non-const 且有数据共享的 function要考虑Copy On Write(COW)

## 成员函数
每一个成员函数都默认带一个隐藏的this参数(可能是第一个参数也可能是最后一个)
## conversion function: 转换函数
一个类可以有多个转换函数, Type类型也不唯一,只要是之前出现过的Type就行
```Cpp
operator Type() const{	//没有返回类型，编译器默认返回类型为Type
	.....
}
```

## lambda

一种inline function，可以被当作参数或本地变量, lambda的出现改变了
cpp标准库的使用方式
```[.....] (.....) mutableOpt throwSpecOpt -> retTypeOpt{.....}```
throwSpec：是否抛出异常
retType  ：返回类型
Opt      ：选用,可写可不写
如果三个都没有则(.....)也是可写可不写
(.....)放函数参数, 像一般函数一样

[.....]定义了lambda的捕获方式
1. []：没有使用任何函数对象参数。
2. [=]：函数体内可以使用Lambda所在作用范围内所有可见的局部变量(包括Lambda所在类的this)，并且是值传递方式。
3. [&]：同上不过是引用传递方式。
4. [this]：函数体内可以使用Lambda所在类中的成员变量。
5. [a]：将a按值进行传递。
6. [&a]：将a按引用进行传递。
7. [a,&b]：将a按值进行传递，b按引用进行传递。
8. [=, &a, &b]：除a和b按引用进行传递外，其他参数都按值进行传递。
9. [&, a, b]：除a和b按值进行传递外，其他参数都按引用进行传递。

```cpp
set使用lambda自定义自己的比较方式是, 在建立对象时要写出lambda对象名
eg：
auto f = [](.....){
	.....
}
std::set< typeName, decltype( f)> elem(f);
 ```

```cpp
auto k = []{
	std::cout << "hello";
};	//;不能忘
k();	//输出hello

//直接调用：
[]{
	std::cout << "hello";
}();	//() ; 不能忘
```
```cpp
lambda相当于一种匿名的function object
auto f = [id]() mutable{	//无mutable则id不可更改
	cout << id;
	id++;
};
//相当于：
class Functor{
private:
	int id;
public:
	void operator(){
		cout << id;
		id++;
	}
};
Functor f;
```
```cpp
int id = 0;
auto f = [id]() mutable{
	cout << id;
	id++;
};
id = 1;
f();	//0
f();	//1
f();	//2
cout << id;	//1
```

# 操作符重载
带有=的重载考虑对象被连续赋值, 要reference
:: 、 . 、 .* 、 ? : 四个不能被重载

## 对于<<等特殊操作符只能重载为非成员函数
```Cpp{ .line-numbers}	
ostream& operator << ( ostream& os, const Type& t){
	return os << .....;
}
```
## 对于自增自减操作符重载
好的后置++的写法是内部调用前置++

成员函数的写法：
```cpp{.line-numbers}
//前置 ++
classname & classname::operator++(){
	n ++;
	return * this;
}
//后置 ++
classname classname::operator++( int k){ 
	classname tmp( *this);//记录修改前的对象
	n++;
	return tmp; //返回修改前的对象
}
//后置的++不能连续++两次, 即(i++)++错误
//前置的++可以连续++两次, 即++(++i)正确
```		
非成员函数写法：
```cpp{.line-numbers}
//前置--
classname& classname--( classname& d){
	d.n--;
	return d;    
}
//后置--
classname classname--( classname& d,int){
	classname tmp(d);
	d.n --;
	return tmp;
}
```

# template

class, function, member三大模板

编译器会对function template做实参推倒

class template要说明使用的template类型

泛化、specialization(特化)、partial specialization(偏特化)

## partial specialization:
1. 个数的偏特化
```cpp{.line-numbers}

template<typename T, typename Alloc=.....>
class vector{
	.....
};
template<typename Alloc=.....>
class vector<bool, Alloc> {
	.....
};
```
2. 范围的偏特化
```cpp{.line-numbers}

template< typename T>
class C{	//c1
	.....
};
template <typename T>
class C<T*>{	//c2
	.....
};
C<int> obj1;	//使用c1
C<int> obj2;	//使用c2
```
## variadic templates(数量不定的模板参数):
可以很方便的完成recursive function call(递归函数调用)
```cpp
//记住...出现位置
void print(){ }
template< typename T, typename... Types>
void print( const T& firstArg, const Types&... args){	
//传入的参数至少是1个, 所以要重载一个参数为0的print
	cout << firstArg << endl;
	print( args...);
}
//可以通过sizeof...(args)可以知道包里面还有多少
```
```cpp{.line-numbers}
void func(){}
template <typename T, typename... types>
void func( const T& firstElem, const types&... otherEle ){
    .....
    func( otherEle...);
}
template<typename... types>
void func(const types&... otherEle ){
    .....
    func( otherEle...);
}
//第三行的函数更加特化, 两者可以共存, 第八行的函数永远不会被调用
```
```cpp
//任意个数比大小得到最大的
int myMax( int n){
	return n;
}
template<typename... Args>
int myMax( int n, Args... args){
	return std::max( n, myMax( args...));	//通过不断调用max函数完成最大值的获取
		//其实max()已经可以接受任意个数的参数值
}
```
```cpp
//对于variadic template还有递归的创建类，递归的继承类
tenolate<typename... Value> class tuple;
template<>class tuple<>{};

template<typename Head, typename... Tail>
class tuple<Head, Tail...> : private tuple<Tail...>{
	typedef tuple<Tail...> inherited;
protected:
	Head m_head;
public:
	tuple(){}
	tuple(Head v, Tail... vtail):m_head(v), inherited(vtail...){}

	typename Head::type head(){	return m_head; }	//error, 如果Head
		//是int类型, 没有::运算符
	//应改为：
	auto head()->decltype( m_head){ return m_head;}
	//也可以直接写成：
	Head head() { return m_head;}

	inherited& tail(){
		return *this;	//return 的是inherited是除去了第一个元素的包
	}
};
```
```cpp
//对于variadic template还有递归的复合类
tenolate<typename... Value> class tuple;
template<>class tuple<>{};

template<typename Head, typename... Tail>
class tuple<Head, Tail...> : private tuple<Tail...>{
	typedef tuple<Tail...> composited;
protected:
	composited m_tail;
	Head m_head;
public:
	tuple(){}
	tuple(Head v, Tail... vtail):m_head(v), m_tail(vtail...){}
	Head head() { return m_head;}

	composited& tail(){ return m_tail; }
};
```
## alias template
```cpp
template< typename T>
using vec = std::vector<T, myAlloc<T>>;
vec<int> myVec;
不能对alias template做任何特化
```

```#define ```和``` typedef ```都无法有上述效果
 ```cpp
#define vec<T> template <tempname T> std::vector< T, myAlloc<T>>;
vec<int> myVec; 
|
 ------> template <tempname int> std::vector< int, myAlloc<int>>; myVec;
```
```typedef``` 不接受参数, 最多能写成
```typedef std::vector< int, myAlloc<int>> vec; ```

### template template parameter 模板模板参数
```cpp
Type function( mytype1 a, mytype2 b){
	mytype1< mytype2> c;	//error
	//mytype1 was not declared in this scope
	//mytype1 was not declared in this scope
	........
}
```
```cpp
template< typename mytype1, typename T>
Type function( mytype1 a, mytype2 b){
	mytype1<mytype2> c;	//error
	//mytype1 is not a template
	.....
}
```
```cpp
template< typename mytype1, 
	template <typename mytype1/*此处的mytype1可以不写*/> class Container>
class XCls{
	private:
		Container<T> c;
	public:
		.....
};

template<typename T>
using Lst = list<T, allocator<T>>;	//alias template的意义

XCls<string, list> mylst1;	//error
XCls<string, Lst> mylst2;
```
而下面的就不是template template parameter:
```cpp{.line-numbers}

template <class T, class Sequence = deque<T>>
class stack{
	.....
	protected:
		Sequence c;	//底层容器
};
stack<int>s1;
stack<int, list<int>> s2;
```
# smart pointers
```cpp
#include<memory>	//使用的头文件
```
smart pointers对于 RAII（RAII:Object lifetime and resource management资源获取即初始化） 或资源获取即初始化编程至关重要。
```cpp
使用步骤：
	1.将smart point声明为局部变量，不要在智能指针身上使用new malloc
	2.在type参数中指定封装指针的指向类型
	3.将原始指针传递给智能指针构造函数中的新对象
	4.使用重载的 -> 和 * 运算符访问对象
	5.让smart point删除对象
```

优点：
用于帮助确保程序没有内存和资源泄漏，并且是异常安全的。确保资源获取在对象初始化的同时发生，以便在一行代码中创建并准备好对象的所有资源。

智能指针在生命结束时自动delete，智能指针是在堆栈上声明的类模板，并使用指向堆分配对象的原始指针进行初始化。智能指针负责删除原始指针指定的内存。智能指针析构函数包含对 delete 的调用，并且因为智能指针是在堆栈上声明的，所以当智能指针超出范围时会调用其析构函数，即使在堆栈更远的地方抛出异常也是如此。

智能指针类重载 -> 和 * 运算符以返回封装的原始指针，智能指针有自己的成员函数，可使用.来访问。

不支持指针算法，只支持move赋值（禁用复制赋值）:

```cpp
std::unique_ptr<int> ptr1(new int(10));
std::unique_ptr<int> ptr2;

// 普通赋值操作，不会产生任何输出
ptr2 = ptr1;

// 使用 move 赋值，将 ptr1 所指向的资源所有权转移给 ptr2
ptr2 = std::move(ptr1);

// 将 ptr2 转移给 ptr3，ptr2 的指向变为 NULL
std::unique_ptr<int> ptr3 = std::move(ptr2);

```
```cpp
std::shared_ptr<int> ptr1(new int(10));
std::shared_ptr<int> ptr2;

// 普通赋值操作
ptr2 = ptr1;

// move赋值操作
ptr2 = std::move(ptr1);

// 转移所有权
std::shared_ptr<int> ptr3 = std::move(ptr2);

```
## 一个deleter的设计
```cpp
class state_deleter {  // a deleter class with state
  int count_;
public:
  state_deleter() : count_(0) {}
  template <class T>
  void operator()(T* p) {
    std::cout << "[deleted #" << ++count_ << "]\n";
    delete p;
  }
};
```
## 常用成员函数
```cpp
unique_ptr<type> ut1;	//使用默认的deleter
unique_ptr<type> ut2;	//使用默认的deleter
unique_ptr<type, deleter> ut3;	//使用自己设计的deleter
type* t
```
返回原始指针 
```cpp
pointer get() const noexcept;
t = ut1.get();
```
1. 返回是否为空	
```cpp
explicit operator bool() const noexcept;
if(ut1)	..... 	//判断ut1是否是空
```
2. 返回其值并将其替换为空指针来释放其存储指针的所有权，此调用不会销毁原
始指针，=左边的对象还需要释放内存  
```cpp
t = ut1.release() ut1中的原始指针变为空指针，值返回赋值给t，t还需要重新delete
```
3. 销毁当前由 unique_ptr（如果有）管理的对象并取得 p 的所有权
```cpp
void reset (pointer p = pointer()) noexcept;
ut1.reset( new type (value));	//ut1的原始指针为value
ut1.reset();	//销毁ut1的原始指针
```
4. 返回存储的deleter
```cpp
deleter_type& get_deleter() noexcept;
const deleter_type& get_deleter() const noexcept;
ut3.get_deleter();	//得到ut3的deleter
```
5. 重载<<操作符，让smart pointer能像普通指针一样输入输出
```cpp
template <class charT, class traits, class T>  
basic_ostream<charT,traits>& operator<< ( 
	basic_ostream<charT,traits>& os, const shared_ptr<T>& x );
cout << *ut1 << endl;
```
6. 等同于use_count() == 0，dtor可能还没被调用，但对该对象的析构即将发生。若原始指针已经被delete则返回true，否则false
```cpp
bool expired() const noexcept
```
7. 获取共享资源所有权的 shared_ptr，如果原始指针已过期，则成员函数返回一个空的 shared_ptr 对象；否则它返回一个拥有原始指针指向的资源的 ```shared_ptr<T>``` 对象。
```cpp
shared_ptr<T> lock() const noexcept
 ```

## shared_ptr

除了存储的原始指针以外，还有一个指针指向控制块，控制块有一个引用计数器，用于存储指向同一内存的共享指针的数量，在最后一个引用被释放时,指向的内存才释放。

在创造包含很多指针的```vector```时考虑用```unique_ptr```

creat、delete、reset一个shared_ptr时,更新计数器，检测是否是第一个/最后一个指针

注:
1. 不要用同一个原始指针初始化多个 ```shared_ptr```，会造成二次销毁。
2. 禁止使用指向 ```shared_ptr``` 的指针,使用 ```shared_ptr``` 的指针指向一 ```shared_ptr```时，引用计数并不会加一，操作 ```shared_ptr``` 的指针很容易就发生野指针异常。
3. 避免循环引用造成内存泄漏

对于可能大量复制赋值的对象，因为```shared_ptr```让同一块内存被多个引用，而非复制内存
到另一块内存。

```shared_ptr```引用计数本身是安全无锁的，但是对象的读写不是， 因为shared_ptr有两个数据成员，读写操作不能简单的原子化。

使用```shared_ptr```可能因为遗漏了一个拷贝，造成意外延长对象的生命周期。

```cpp
std::shared_ptr的示例

#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> p1 = std::make_shared<int>(10);	//C++14及以后
    std::shared_ptr<int> p2 = p1; // 创建一个新的 shared_ptr 对象 p2，与 p1 共享所管理的对象

    std::cout << "p1.use_count() = " << p1.use_count() << std::endl; // 输出 p1 对象当前的引用计数
    std::cout << "p2.use_count() = " << p2.use_count() << std::endl; // 输出 p2 对象当前的引用计数

    p1.reset(); // p1 释放其所管理的对象，引用计数减 1
    std::cout << "p1.use_count() = " << p1.use_count() << std::endl; // 输出 p1 对象当前的引用计数
    std::cout << "p2.use_count() = " << p2.use_count() << std::endl; // 输出 p2 对象当前的引用计数

    p2.reset(); // p2 释放其所管理的对象，这时引用计数为 0，对象被销毁
    std::cout << "p2.use_count() = " << p2.use_count() << std::endl; // 输出 p2 对象当前的引用计数

    return 0;
}

```
循环引用造成的内存泄漏：
假设有两个类 A 和 B，它们分别拥有一个指向对方的 ```shared_ptr```。这种情况下，当 A和 B 的实例都不再被使用时，它们仍然无法被释放，因为它们互相持有对方的```shared_ptr```，导致引用计数无法降为零。
```cpp

struct A;
struct B{
    shared_ptr<A> a_;
};
struct A{
    shared_ptr<B> b_;
};

int main(){
    auto b = make_shared<B>();
    auto a = make_shared<A>();

    b->a_ = a;
    a->b_ = b;
		//此时a，b的引用计数都是2
    return 0;
} //此时a，b的引用计数各减1，都为1，这两个对象都不会被销毁，从而发生内存泄露。
```

## weak_ptr

为避免循环引用导致的内存泄露，就需要使用 ```weak_ptr```, ```weak_ptr``` 不持有对象，所以不能通过 ```weak_ptr``` 去访问对象的成员，可以通过```lock()```方法将```weak_ptr```转换为```shared_ptr```，从而访问对象的成员。



不能单独使用weak_ptr，要和share_ptr搭配使用，```weak_ptr```是一种弱引用，它不会增加所管理对象的引用计数，也不会影响对象的生命周期，因此不会导致循环引用。

**重点理解如何避免循环引用**：
1. 什么是循环引用：
循环引用指的是两个或多个对象之间相互引用，且每个对象都持有对方的指针，导致这些对象无法被释放。使用 ```shared_ptr``` 时，每个对象都会持有一个计数器，记录有多少个 ```shared_ptr``` 指向该对象。如果两个对象相互持有 ```shared_ptr``` 指针，它们的计数器将无法降为零，从而导致内存泄漏。

2. 如何通过```weak_ptr```来解决循环引用：
```weak_ptr``` 通过不持有对象的强引用，而是持有一个“弱引用”来避免循环引用。当我们需要访问所指向对象时，我们可以通过 ```weak_ptr``` 获得一个 ```shared_ptr``` 对象，然后使用该对象进行操作。如果所指向的对象已被释放，则该操作将不起作用，因为 ```weak_ptr``` 指向的对象已不存在。

```lock()``` 成员函数的含义是将 ```weak_ptr``` 对象转换为一个 ```shared_ptr``` 对象，如果 ```weak_ptr```所指向的对象已经被释放了，则返回一个空的 ```shared_ptr``` 对象。可也用于判断一个```weak_ptr```是否有效。

```expired()```成员函数的含义是判断 ```weak_ptr```是否有效，如果返回true，则说明与```weak_ptr```关联的对象已经被析构，否则说明对象仍存在。在 ```weak_ptr```使用前，必须判断其是否有效。


注：
1. ```weak_ptr```需要在```shared_ptr```已经创建的情况下使用，否则会抛出异常。这是因为```weak_ptr```只是```shared_ptr```的一种观察者，它不能自己独立管理对象的生命周期。
2. 在使用```weak_ptr```时，我们需要检查它所引用的对象是否存在。
3. 在使用```weak_ptr时```，我们需要注意避免使用过期的对象。当我们通过```lock()```方法获取```shared_ptr```时，有可能已经存在其他线程对该对象进行了释放，所以我们需要在获取```shared_ptr```后立即检查它是否为空，以避免使用过期的对象。

```cpp
// 创建一个shared_ptr指向动态分配的int类型内存
std::shared_ptr<int> ptr(new int(10));

// 创建一个weak_ptr指向与ptr共享的内存
std::weak_ptr<int> weak_ptr(ptr);

// 判断weak_ptr是否过期
if (weak_ptr.expired()) {
		std::cout << "weak_ptr已过期" << std::endl;
} else {
		std::cout << "weak_ptr未过期" << std::endl;
}

// 通过weak_ptr创建一个shared_ptr对象
std::shared_ptr<int> ptr2 = weak_ptr.lock();

// 判断weak_ptr是否过期
if (weak_ptr.expired()) {
		std::cout << "weak_ptr已过期" << std::endl;
} else {
		std::cout << "weak_ptr未过期" << std::endl;
}
 ```
只要把上文中AB其中一个shared_ptr改成weak_ptr就能避免内存泄漏
```cpp
struct A;
struct B{
    shared_ptr<A> a_;
};
struct A{
    weak_ptr<B> b_;
};

int main(){
    auto b = make_shared<B>();
    auto a = make_shared<A>();

    b->a_ = a;
    a->b_ = b;
		//此时a的引用计数是2, b的引用计数器为1
    return 0;
} //此时a, b的引用计数各减1, b的引用计数器为0, 让A析构
```
```cpp
struct A{
    int a = 0;
};

auto ms = make_shared<A>();
weak_ptr<A> wp(ms);
cout << wp->a << endl;   // error
```
## unique_ptr

own their pointer uniquely独占式的智能指针
可以安全替代原始指针，只会在对象析构的时候释放一次

当std::unique_ptr被销毁时，它所管理的对象也会被自动释放。

unique_ptr 中唯一的数据成员是原始指针, 这让unique_ptr和原始指针一样大, 使用重载 ```*``` 和 ```->```运算符的智能指针访问封装指针并不比直接访问原始指针慢很多。

不能使用=来转移对象所有权，要使用```move```

在以下两个情况下```unique_ptr```指向的对象消亡：
1. ```unique_ptr``` 对象被销毁( 生命周期结束)
2. ```unique_ptr``` 对象通过 operator= 或 reset() 分配另一个指针


由于```std::unique_ptr```是独占式的智能指针，不能直接将一个```std::unique_ptr```对象赋值给另一个```std::unique_ptr```对象，需要使用```std::move```函数进行转移所有权

注：
  1. 创建```unique_ptr```对象时，需要使用```new运算符```动态分配内存，并将其作为```unique_ptr```的ctor。

  2. ```unique_ptr```对象可以通过移动构造函数或移动赋值运算符进行转移，转移后原来的```unique_ptr```对象将失去所有权，新的```unique_ptr```对象将独占资源所有权。

  3. ```unique_ptr```对象不支持拷贝构造函数或拷贝赋值运算符，因为这会导致两个对象共享同一块内存资源，从而破坏了独占式所有权的特性。

  4. 当需要自定义资源释放函数时，可以通过传递一个删除器（deleter）函数对象或函数指针来指定资源
的释放方式。

  5. 当多个```unique_ptr```对象相互引用时，也可能会出现循环引用的问题。为了避免这个问题，可以使用裸指针或weak_ptr来打破循环引用。

```cpp
std::unique_ptr<int> foo;
std::unique_ptr<int> bar;
foo = std::unique_ptr<int>(new int (101));  // rvalue
bar = std::move(foo); // using std::move, foo已经失效，以后不能使用foo
 //bar = foo; //error unique_ptr不允许赋值操作不能放在等号的左边(函数的参数和返回值例外)
```
```cpp
#include <iostream>
#include <memory>

int main() {
	// 创建一个 unique_ptr 来管理动态分配的整数数组
	std::unique_ptr<int[]> arrPtr(new int[5]);

	// 初始化数组元素
	for (int i = 0; i < 5; ++i) {
			arrPtr[i] = i * 2;
	}

	// 使用智能指针访问和修改数组元素
	for (int i = 0; i < 5; ++i) {
			std::cout << "arrPtr[" << i << "] = " << arrPtr[i] << std::endl;
	}

	// 不需要手动释放内存，当 unique_ptr 超出作用域时，它会自动释放所管理的内存

	return 0;
}
```
```cpp
auto spi = make_unique<int>(10);
auto upi = make_unique<int> ( 20);
std::cout << "upi:" << upi.get() << std::endl;	//upi:0x1d1668
std::cout << "spi:" << spi.get() << std::endl;	//spi:0x1d1658
swap( upi, spi);
std::cout << "upi:" << upi.get() << std::endl;	//upi:0x1d1658
std::cout << "spi:" << spi.get() << std::endl;	//spi:0x1d1668
```
创建unique_ptr的方式有两种：
1. ```std::unique_ptr<int> upi(new int);```
2. ```auto upi = std::make_unique<int>();```

这两种方式都可以创建一个指向某个类型的```unique_ptr```，但是使用```std::make_unique```函数可以更加方便和安全，因为它可以自动为```unique_ptr```分配内存空间，并且可以避免手动分配内存空间时可能出现的内存泄漏等问题 
```make_unique``` 是在 C++14 中引入的。
# 杂记

1. heap，或叫system heap, 是操作系统提供的一块global内存空间，程序可
动态分配从中获得若干blocks。从heap中获得的空间必须要手动的释放。

2. stack, 是存在于某作用域的一块内存空间，例如当调用函数, 函数本身会形成一个stack用来存放放置它所接受的参数, 以及返回地址。

3. 可以使用nullptr代替0或者NULL

4. 浅拷贝时要想到是否需要自我赋值检测

5. ```break``` 要和 loop or switch 一起使用，用于跳出单层的loop or switch

6. ```void*```: 没有关联数据类型的指针。```void``` 指针可以保存任何类型的地址，并且可以类型转换为任何类型。


```cpp{.line-numbers}
Type c4;	//c4为global object，其生命在整个程序结束时才结束, 作用域是整个程序

Type function( Type& c){
	Type c1;	//c1被称为auto object, 其生命在作用域结束时结束。
			//自动调用析构函数。
	Type* c2 = new Type;
		//不能忘记delete, 否则造成内存泄露, 当作用域结束时c2
		//所指向的heap object仍然存在, 但指针c2的生命却结束了, 
		//作用域之外再也看不到c2, 也就无法delete c2
	static Type c3;	
		//c3被称为static object,其生命在作用域结束后仍存在，
		//直到整个程序结束。		
	.....
}
 ```
```cpp{.line-numbers}
struct A{
	typedef int ii;
};
A a;
sizeof(a); //理论是0, 结果是1 
```
```cpp{.line-numbers}
for( decl : coll){	//ranged-base for, 尽量传引用
	.....
}
 ```
 
## inline 和 define 的区别

```#define```是一个预处理指令，它在编译之前将文本替换为指定的值。```#define```的主要目的是为了方便地定义常量或函数宏，从而避免在程序中多次出现相同的值或代码。


```inline```关键字用于在编译时将函数代码插入到函数调用点处。当函数被声明为```inline```时，编译器会尝试将函数调用转换为直接在调用点处执行函数的代码。这样可以减少函数调用的开销，并提高程序的执行效率。


```#define```和```inline```都可以用于代码优化，但它们的实现方式和语法不同。```#define```是一个预处理指令，只能用于常量或函数宏的定义，而```inline```是一个函数关键字，用于将函数代码插入到函数调用点处。
另外，```inline```通常应该用于较小的函数，而较大的函数应该使用普通函数调用，因为将大函数声明为```inline```可能会导致代码膨胀和性能下降。
# 杂项
## uniform initialization

可以用统一的{}初始化, 当编译器看见```{ t1, t2, t3, ..., tn}```时, 会做出一个```initializer_list<Type>```，关联到一个```array< Type, n>```, 调用函数(eg:ctor)时，该```array```内的元素被编译器分解逐一传给函数。若函数参数是个```initializer_list<Type>```,调用者不能给出数个```Type参数```然后以为他们会被自动转为一个```initializer_list<Type>```传入

### initializer_list<>
```cpp
void function( std::initializer_list<Type> num){	//可接受任意个数的参数
	for( auto a = num.begin(); a != num.end(); a++)
	.....
}
function( {.....});
```

## Type Alias(类似typedef)
```cpp
//两者等价
typedef void ( *func)();
using func = void(*) ();
void function(){
	......
}
func fun = function;	//函数名称就是一个函数指针
//以下函数指针写法是错误的：
void (*function)()	//这是一个变量不能像函数一样定义
{
	.....
}
```
## using的几种用法

1. ```using namespace std``` ```using std::cout``` 等using namespace 和 namespace 的member
2. ```using ClassName::MemberName```：使用ClassName中的MemberName
3. type alias 和 alias template


## char 数组和 char 指针

常量字符串通常在单独的空间里，几个指针如果指向相同的字符串时，实际上会指向
同一个内存地址，而```char```数组则不会

```cpp
char str1[] = "hello";
char str2[] = "hello";
char* str3 = "hello";
char* str4 = "hello";
printf( "str1: 0x%x\n",str1);   //str1: 0x6dfef2
printf( "str2: 0x%x\n",str2);   //str2: 0x6dfeec
printf( "str3: 0x%x\n",str3);   //str3: 0x4b902e
printf( "str4: 0x%x\n",str4);   //str4: 0x4b902e
```
str1和str2是两个字符串数组，会为它们分别分配两个长度为6个字节的空间，并把"hello"的内容分别复制到数组里，这是两个初始地址不同的数组，因此str1和str2的值也不同。

str3和str4是两个指针，无需分配内存用来存储字符串内容，只要把他们指向"hello"在内存的地址就可以，由于"hello"是常量字符串，在内存中只有一个拷贝，因此指向同一个地址，str3和str4相同

## noexcept
```cpp
Type function() noexcept;	//相当于Type function() noexcept(true);
Type function() noexcept(条件a);	//在条件a为真下不会出现异常
```
异常一定要被处理, 如果不处理则一直向上抛出异常, 如果一直没被处理则调用```std::terminate()```,```std::ternimate()```调用```std::abort()```, ```abort()```会结束整个程序

## decltype
```cpp
decltype关键字可以让编译器找出表达式的type
eg：
map< string, double> c;
.....
decltype(c)::value_type e;	//相当于 map<string, double>::value_type e;
```
用法1：声明一个return type
```cpp
template< class type1, class type2>
decltype ( x + y) add( type1 x, type2 y);	
	//编译无法通过, 因为在使用decltype (x+y)时编译器不知道x、y是什么
```
```cpp
//正确写法：
auto add( type1 x, type2 y) -> decltype ( x + y);
```
用法2：可用于template metaprogramming(模板元编程)
```cpp
template< typename T>
void funtion( T obj){
	map< string, double>::value_type elem1;
	map< string, double> coll;
	decltype(coll)::value_type elem2;
	typedef typename decltype(obj)::iterator iType;	//typename告诉编译器后面
		//的是一个类型名而非变量名, 有::正规的编译器都要求要写typename
	iType elem3;
}
```
用法3：传递lambda的类型
面对lambda, 我们手上往往只有对象, 类型常用```auto```来让编译器推倒, 想要获得其type就需要借助decltype

## mutable 

用来修饰类的数据成员, 如果类的成员变量被声明为```mutable```，则可以在任何情况下更改该变量的值，即使在常量对象上也是如此。

```cpp
class MyClass {
public:
    void setValue(int value) const {
        m_value = value; // 在常量对象中修改mutable变量
    }

    int getValue() const {
        return m_value;
    }

private:
    mutable int m_value; // mutable成员变量
};

int main() {
    const MyClass obj;
    obj.setValue(42); // 仍然可以修改m_value的值
    std::cout << obj.getValue(); // 输出：42
    return 0;
}
```
```cpp

用于lambda表达式, 可以修改按值捕获的方式中的变量
auto function = [=]() {	// error
	x = 1;
};
auto function = [=]() mutable { 
	x = 1;
};
```
## volatile
修饰变量声明，表示该变量的值可能会在程序的控制范围之外发生变化，因此编译器不会对该变量进行优化。因为编译器无法确定该变量何时会被修改。
通常用于访问硬件寄存器或多线程共享变量。

```cpp
volatile int count = 0; // 声明一个volatile变量

void increment() {
    ++count; // 对volatile变量进行操作
}

int main() {
    while (count < 10) {
        increment();
    }
    std::cout << "Count: " << count << std::endl;
    return 0;
}

```
## move

可以解决unnecessary copying问题 和 做出perfect forwardding

临时对象是一种Rvalue, Rvalue不能放在 = 左边
编译器面对临时对象一定会当成Rvalue reference

被move之后原来的东西就不能用了(因为是浅拷贝, 移动了指针, 而非重新分配内存)

```cpp
type c;
type c1(c);	
type c2( std::move(c));	//这条语句之后, 都不能使用c了
function( move( a));	//调用Rvalue版本的function
```
对于一个类应该有对应的copy assignment和move assignment

move ctor对以结点方式存储的容器影响不大

```cpp
std::forward(arg)
```
如果arg不是Lvalue reference则返回arg的Rvalue reference，
如果arg是Lvalue reference则不做任何改变
```cpp
template <class T> T&& forward (typename remove_reference<T>::type& arg) noexcept;
template <class T> T&& forward (typename remove_reference<T>::type&& arg) noexcept;

两者都返回static_cast<decltype(arg)&&>(arg)
```

这是一个辅助函数，允许将作为Rvalue reference的参数完美转发到推导类型，保留任何
可能涉及的```move```语义。需要此函数是因为所有命名值（例如函数参数）始终评估为左值（即使是那些声明为右值引用的值）, 这给将参数转发给其他函数的模板函数保留潜在的
```move```语义带来了困难。

```cpp
class MyString{
private:
	char* _data;
	size_t _len;
	void _init_data( const char* s){
		_data = new char[ _len+1];
		memcpy( _data, s, _len);
		_data[ _len] = '\0';
	}
public:
	MyString() : _data( NULL), len(0) { ++DCtor;}
	MyString( const char* p) : _len( strlen( p)){
		_init_data( p);
	}
	MyString( const MyString& str) : _len( str._len) {
		_init_data( str._data);	//copy
	}
	myStirng( MyString&& str) noexcept : _data( str._data), _len( str._len){
		str._len = 0;
		str._data = NULL;	//重要！必须写, 不然发生错误
	}
	MyString& operator=( const MyString& str){
		if( this != &str){
			if( _data) 
				delete _data;
			_len = str._len;
			_init_data( str._data);	//copy
		}else{		}
		return *this;
	}
	MyString& operator=( const MyString&& str) noexcept{
		if( this != &str){
			if( _data) 
				delete _data;
			_len = str._len;
			_data = str._data;	//move
			str._len = 0;
			str._data = NULL;	//重要！必须写, 不然发生错误
		}
		return *this;
	}
	virtual ~MyString(){
		if( _data) 	//上面写的str._data = NULL;的作用, 如果不写则会发生内存泄漏
			delete _data;
	}
};
 ```
## tuple
```cpp
tuple的做法是递归的继承

tuple<type1, type2, type3> t1;
auto t2 = make_tuple( element1, element2, element3);
get<n>t1;
typedef tuple<type1, type2, type3> TypleType;
tuple_size<TypleType>::value	//tuple内个数, 3
tuple_element<1, TypleType>::tupe element1 = 3;
	//相当于：type1 element = 3;

```
## 预处理
```cpp
#if 整型常量表达式1
    程序段1
#elif 整型常量表达式2
    程序段2
#elif 整型常量表达式3
    程序段3
#else
    程序段4
#endif
```
```cpp
#ifdef  宏名
    程序段1
#else
    程序段2
#endif
//如果宏名被定义过，则执行程序段1
```
```cpp
#ifndef 宏名
    程序段1 
#else 
    程序段2 
#endif
//如果宏名没有被定义过，则执行程序段1
``` 
## 一些type_traits中的模板
### remove_all_extents
如果 T 是某种类型 X 的多维数组，则提供 X 的成员 的类型，否则类型为 T。
不能对remove_all_extents进行特化
```cpp
template< class T >
struct remove_all_extents;
```
```cpp
template<class T>
struct remove_all_extents { typedef T type; };
 
template<class T>
struct remove_all_extents<T[]> {
    typedef typename remove_all_extents<T>::type type;
};
 
template<class T, std::size_t N>
struct remove_all_extents<T[N]> {
    typedef typename remove_all_extents<T>::type type;
};
```
example
```cpp
template<class A>
void info(const A&) {
    typedef typename std::remove_all_extents<A>::type Type;
    std::cout <<  typeid(Type).name() << endl;
}
class className2 { int m; };
int main()
{
    float a0;
    float a1[1][2][3];
    float a2[1][1][1][1][2];
    float* a3;
    int a4[3][2];
    double a5[2][3];
    struct X { int m; } x0[3][3];
    class className1 { int m; } x1[3][3];
    class className2 x2[3][3];
    cout << "a0  "; info(a0);	//a0  f
    cout << "a1  "; info(a1);	//a1  f
    cout << "a2  "; info(a2);	//a2  f
    cout << "a3  "; info(a3);	//a3  Pf
    cout << "a4  "; info(a4);	//a4  i
    cout << "a5  "; info(a5);	//a5  d
    cout << "x0  "; info(x0);	//x0  Z4mainE1X
    cout << "x1  "; info(x1);	//x1  Z4mainE10className1
    cout << "x2  "; info(x2);	//x2  10className2
    return 0;
}
```
### is_destructible
```cpp
template< class T >
struct is_destructible;
```
如果 T 是reference type，则提供等于```true```的成员常量值。
若 T 是```const```或```volatile```(包含```void```)或函数类型,则提供等于```false```的成员常量值。
若 T 是对象类型，令```U=std::remove_all_extents<T>::type```，若表达式 ```std::declval<std::remove_all_extents<T>::type&>().~U()```

在unevaluated context中合法，则value等于```true```，否则value为```false```。

```declval<T>```:将任何类型 T 转换为引用类型，从而可以在```decltype```中使用成员函数，而无需通过构造函数。

```cpp
用法：
template <template _TP>
std::is_destructible<_TP>{}	//true或false
 ```
### is_trivially_destructible
```cpp
template< class T >
struct is_trivially_destructible;
```
和```is_destructible```一样，另加了```std::remove_all_extents<T>::type```是non-class类型或带有trivial dtor的class类型(隐式定义的析构函数)

```cpp
用法：
template <template _TP>
std::is_trivially_destructible<_TP>{}	//true或false
```
#### trivial destructor

trivial dtor是不执行任何操作的dtor，有trival dtor的对象不需要delete-expression，可以通过简单地解除分配它们的存储来处理。所有c兼容的数据类型(POD)都可以简单delete。

POD:Plain Old Data 一种 C++ 类型，它在 C 中具有等效项，并且使用与 C 相同的规则来进行初始化、复制、布局和寻址。没有ctor，dtor，virtual member function
eg:struct Fred x; 不初始化 Fred 变量 x 的成员。

满足以下几个条件：
1. 析构函数不是用户提供的(要么被隐式声明，要么在其第一个声明中被显
式定义为默认值)
2. 析构函数不是虚函数(即基类析构函数不是虚函数)
3. 所有direct base class 都有 trivial destructor direct base class 在其派生类的声明中直接作为基说明符出现的基类
4. 所有class type的non-static member或array都有trivial destructors

### is_nothrow_destructible

和is_destructible一样，不过dtor是noexcept


## 强制类型转换

### static_cast
```Cpp
static_cast<Type>(value);
```
用途：
1. 基本数据类型之间的转换，如int换成char
2. 把任何类型的表达式转换成void类型

注：
1. 没有运行时检查来保证转换的安全性
2. 把派生类转换成基类(上行转换)是安全的
3. 把基类转换成派生类(下行转换)是不安全的，因为没有动态类型检查

不安全的情况：
1. 当将一个指向基类的指针或引用转换为指向派生类的指针或引用时，如果实际对象的类型不是所转换的类型。
2. 当将浮点数转换为整数时，如果浮点数的值大于目标类型的最大值或小于最小值。
3. 当将指针或引用转换为其他类型时，如果指针或引用指向的对象被销毁或被释放。

### dynamic_cast
```Cpp
dynamic_cast<baseClass>(pointer_to_)
```
在进行下行转换时```dynamic_cast```具有类型检查的功能，比```static_cast```安全，转换后必须是类的```pointer```、```reference```或```void*```，基类要有虚函数，可以交叉转换只能用于存在虚函数的父子关系的强制类型转换，对于指针，转换失败返回```nullptr```，对于```reference```，转换失败抛出与 ```std::bad_cast``` 类型的处理程序相匹配的异常


### reinterpret_cast
将一个pointer或reference转换成另一种类型的指针或者引用，且转换之后指针指向的是同一块内存区域，但是被解释为了另一种类型，**平台移植性比较差**。

这种转换方式仅仅进行二进制位的拷贝，没有进行类型的检查和转换。这意味着转换前后的类型可以是完全不同的，甚至是无关的类型。这种转换方式非常危险，可能导致未定义的行为，例如访问非法的内存区域或者将一种类型的指针错误地解释成另一种类型的指针。

在使用 reinterpret_cast 时，应该非常小心并且谨慎，一般情况下应该尽量避免使用。如果必须使用它，应该首先检查转换前后的类型是否可以进行此类转换，以及转换后指针是否指向有效的内存区域。

```cpp
#include <iostream>

int main() {
	int a = 10;
	char* p = reinterpret_cast<char*>(&a);
	std::cout << "a = " << a << std::endl;  //a = 10
	std::cout << "p[0] = " << static_cast<int>(p[0]) << std::endl;  //p[0] = 10
	std::cout << "p[1] = " << static_cast<int>(p[1]) << std::endl;  //p[1] = 0
	std::cout << "p[2] = " << static_cast<int>(p[2]) << std::endl;  //p[2] = 0
	std::cout << "p[3] = " << static_cast<int>(p[3]) << std::endl;  //p[3] = 0
    return 0;
}
```
### const_cast

常量指针转换成非常量指针，并且仍然指向原来的对象，常量引用被转换成非常量引用，并且仍然指向原来的对象，去掉类型的const或volatile

## 赋值表达式中的各种类型

### lvalue

出现在赋值表达式的左侧,包括变量名、const 变量、数组元素、返回左值引用的函数调用、bit-field、union和class member。可以 取地址的，有名字的，非临时的 就是左值

左值引用要求右边的值必须能够取地址，如果无法取地址，可以用常引用

### xvalue

生命接近结束的值，资源可能被移动
一个 xvalue 表达式有一个地址，涉及右值引用的表达式的结果，你的程序不再可以访问它，但可以用来初始化一个右值引用，它提供了对表达式的访问。包括返回右值引用的函数调用，以及数组下标、成员和指向成员表达式的指针，其中数组或对象是右值引用。

### gvalue
lvalue或xvalue.

### rvalue

出现在赋值表达式右侧，一个临时对象或其子对象，或者一个与对象无关的值。不能取地址的，没有名字的，临时的就是右值

### prvalue 

纯右值，不是 xvalue 的右值，没有程序可以访问的地址，包括文字、返回非引用类型的函数调用，以及在表达式求值期间创建但只能由编译器访问的临时对象。

## reference

使用reference传递参数，传递者无需知道接收者是以reference形式接受
很少直接声明一个变量类型是reference, 常用于参数类型和返回类型的描述
```Cpp
//两者无法并存, 被视为same signature(签名)
Type function(const Type a){	..... }
Type function(const Type& a){ ..... }
//但是如果一个函数在) 加上了const则可以并存, const也被视为签名的一部分
```
reference设初始值之后再也不能改变代表别的变量
```cpp
Type& r = x;	//r代表了x, r的大小和x的大小一样
Type xx = a;
r = a;		//把a赋值给r, 现在r和x都等于a
```
注：
1. x的地址在哪r的地址就在哪
2. sizeof(X) == sizeof(r)

Java里所有变量都是reference

```Cpp{.class1 .class .line-numbers}
//eg1:
Type& function( Type* t ){
	.....
	return *t;	//返回指针指向的内容，不报错
}
```
```Cpp{.class2 .class .line-numbers}
//eg2:
void function( Type&t ){
	.....
}
Type t;
function( t);	//使用value作为参数传递，不报错
```

两个例子不报错的原因都是因为: 传递者(使用者)无需关注参数是否以```reference```的形式实现传递

类的成员函数必须为引用返回的情况：

1. 返回值是类的成员变量：如果成员变量需要在函数外部被访问和修改，可以将其作为引用返回。这样可以避免复制成员变量的值，提高程序效率。

2. 返回值是类的对象本身：如果函数需要对类的对象进行修改，并且希望这些修改对调用者可见，可以将对象本身作为引用返回。

3. 返回值是类的数组或向量：如果类定义了数组或向量类型，可以将其作为引用返回，以提高程序效率。

需要注意的是，在返回引用时，必须确保引用指向的对象在函数退出后仍然有效。如果引用指向的对象是函数内部创建的局部变量，那么在函数退出后该引用将变为未定义行为。

总之，在类的成员函数中，需要根据实际情况选择适当的返回值类型。如果需要返回类的成员变量或对象本身，可以考虑使用引用返回。

## static

1. 静态变量：```static``` 可以用于声明静态变量，静态变量的生命周期与程序的生命周期相同，其作用域仅限于定义它的函数或类，因此它只能在定义它的函数或类的作用域内访问。静态变量的初始值为 0。

2. 静态函数：```static``` 可以用于声明静态函数，静态函数与类的实例化对象无关，因此可以通过类名直接调用静态函数，无需先实例化类。静态函数只能访问类的静态成员，不能访问类的非静态成员。

3. 静态成员变量：```static``` 可以用于声明静态成员变量，静态成员变量属于类的所有实例共享，也就是说它们在内存中只有一份拷贝，无论类的实例化对象有多少个，它们都指向同一个内存地址。静态成员变量必须在类外定义和初始化，并且不属于任何一个类的实例化对象。

4. 静态断言：```static_assert``` 是 C++11 新增的一个关键字，用于在编译期间进行静态断言。它的作用类似于运行时断言，用于检查某个条件是否满足，如果条件不满足则在编译期间发出错误信息。

5. 静态局部变量：在函数内部使用 ```static``` 可以声明静态局部变量，静态局部变量的生命周期与程序的生命周期相同，但是其作用域仅限于定义它的函数，只有第一次调用函数时才会初始化，之后每次调用函数都会使用上一次保存下来的值。

```cpp
//static_assert:
constexpr int arrSize = 10;
static_assert(arrSize > 0, "数组大小必须大于 0");
	// error: static assertion failed: 数组大小必须大于 0
```

## const
```cpp
const int* p; // p 是一个指向常量的指针，不能通过 p 修改指向的对象
int* const p; // p 是一个常量指针，指向的对象可以修改，但不能指向其他对象
const int* const p; // p 是一个指向常量的常量指针，不能修改指向的对象，也不能指向其他对象
```

注意事项：
1. const 变量必须在声明时进行初始化，否则会出现编译错误。
2. const 变量的值不能被修改。任何试图修改 const 变量的操作都将导致编译错误。
3. const 变量可以用于函数参数和函数返回类型。当函数参数被声明为 const 类型时，函数
不能修改该参数的值，从而确保函数的安全性和可靠性。
4. const 指针可以指向可修改的对象，但是不能通过该指针修改对象的值。
5. const 成员函数是类中的一种特殊函数，它不会修改对象的状态。在 const 成员函数中，不能修改类中的非 const 成员变量，也不能调用非 const 成员函数。

```cpp
int x = 10;
const int* p;	//p 指向一个常量，不能通过p修改指向的对象
p = &x;	
*p = 20;	//error:表达式必须是一个可以修改的左值
//但是可以通过别的指针修改
int* q = const_cast<int*> p;	//将p转变成可以修改的指针
*q = 20;
```
### constexpr 
C++11 引入的一个关键字，用于声明一个常量表达式或一个能在编译期求值的函数。它告诉编译器，在编译时期对表达式进行计算，而不是在运行时计算。

使用 constexpr 声明的常量表达式可以在编译时期被求值，如果尝试在运行时期修改它们的值会导致编译器报错。这对于需要预先计算值并在程序运行时使用的常量非常有用，因为它可以提供更好的性能和安全性。

使用 constexpr 声明的函数必须满足以下要求：
1. 函数体必须包含唯一的 ```return``` 语句或唯一的 ```return``` 表达式。
2. 函数的参数和返回值必须是字面类型或字面类型的引用。
3. 函数体中只能包含能在编译时期完成计算的语句。
4. 使用 ```constexpr``` 声明的函数可以在编译期被调用，如果输入的参数是常量表达式，函数也将通过编译期求值来计算返回值。如果输入的参数不是常量表达式，那么函数将在运行期计算返回值。
实例：
```cpp
constexpr int square(int x) {
    return x * x;
}

constexpr int a = 5;
constexpr int b = square(a); // 等价于 constexpr int b = a * a;

const int c = 10;
constexpr int d = square(c); // 因为 c 不是常量表达式，所以在运行期计算

```
a 是一个常量表达式，因此在编译期调用了 square 函数，并将 25 赋值给 b。而 c 不是常量表达式，因此在运行期将调用 square 函数来计算 100 并赋值给 d。

### const_cast
const_cast 是C++中的一个类型转换操作符，用于将const修饰的对象转换为非const类型。在C++中，const关键字用于声明对象为只读，即不允许修改其值。但有时候我们可能需要对一个const对象进行修改，这时就可以使用const_cast来去除const属性。
常用于指针和引用。
```cpp
#include <iostream>
int main() {
	const int number = 10;
	const int* constPtr = &number;  // 指向常量的指针

	// 使用 const_cast 移除 const 属性
	int* nonConstPtr = const_cast<int*>(constPtr);

	// 修改对象的值
	*nonConstPtr = 20;

	// 打印修改后的值
	std::cout << "Modified number: " << number << std::endl;

	return 0;
}
```
## enum

1. 常规枚举
```cpp
enum Color {
    RED,
    GREEN,
    BLUE
};
```
2. 作用域枚举（枚举类）
```cpp
enum class Color {
    RED,
    GREEN,
    BLUE
};
```
与传统的常规枚举相比，作用域枚举提供了更强的作用域限定和类型安全性。
在传统的常规枚举中，枚举值的名称在枚举所在的作用域中是全局可见的，这可能会引起命名冲突。而作用域枚举引入了一个新的作用域，枚举值的名称在该作用域中被限定，不会与其他作用域中的名称冲突。
3. 枚举类的底层类型指定
```cpp
enum class Color : int {
    RED,
    GREEN,
    BLUE
};
```
4. 枚举成员初始化
```cpp
enum Color {
    RED = 1,
    GREEN = 2,
    BLUE = 3
};
```
5. 前向声明的枚举
```cpp
enum Color : int;

void printColor(Color color);  // 可以使用前向声明的枚举类型
```
## union
```union```是一种特殊的数据类型，允许在相同的内存空间中存储不同类型的数据。union的成员共享同一块内存，但每次只能存储其中一个成员的值。union的大小取决于其最大成员的大小。
与枚举相比，```union```有以下几个区别：
1. 数据存储方式：枚举存储的是一组离散的常量值，而```union```存储的是不同类型的数据，共享同一块内存空间。

2. 数据类型限制：枚举的成员只能是整数类型，而```union```的成员可以是任意类型，包括基本类型、自定义类型和指针类型等。

3. 内存占用：枚举类型的大小由底层整数类型决定，而```union```的大小由其最大成员的大小决定。

4. 类型安全性：枚举提供了类型安全的常量值，而```union```需要手动确保访问正确的成员类型，否则可能会导致未定义的行为。
## 内存

Text section: 包含要由处理器执行的二进制指令的部分

Data section: 包含非零初始化静态数据

BSS (Block Started by Symbol):包含零初始化的静态数据。程序中未初始化的静态数据初始化为 0 并放在此处

Heap: 包含动态分配的数据

Stack: 包含您的自动变量、函数参数等 

heap从低地址往高地址增长，stack从高地址往低地址增长，两者相对

有时Data section、bss和heap统称为“数据段”， 它的末尾由一个名为 program break 或 brk 的指针划定。 brk pointer指向heap的末尾。

Linux中可通过```sbrk()```系统调用来操作程序中段
调用 ```sbrk(0)``` 给出程序中断的当前地址。
使用正值调用 ```sbrk(x)``` 会将 brk 递增 x 字节，从而分配内存。
使用负值调用 ```sbrk(-x)``` 会使 brk 减少 x 字节，从而释放内存。
失败时，```sbrk()``` 返回 ```(void*) -1```

```sbrk()``` 并不是真正的线程安全。它只能按照后进先出的顺序增长或收缩。这与heap的增长收缩方式相同，glibc依旧使用```sbrk()```分配不太大的内存。

### 内存泄漏

程序未能释放掉不再使⽤的内存的情况,失去了对该段内存的控制,因⽽造成了内存的浪费。进程突出前，其他程序无法使用泄漏的内存。

#### 内存泄漏分类
##### heap leak
```malloc```、```new```等从堆中分配的一块内存，用完后没有```free```、```delete```，导致堆中的这部分内存不能再被使用，导致heap leak

##### resource leak

主要指程序使⽤系统分配的资源⽐如 Bitmap,handle ,SOCKET 等没有使⽤相应的函数释
放掉，导致系统资源的浪费，可导致系统效能降低，系统运⾏不稳定


##### 没有将基类的dtor定义为虚函数

当基类指针指向⼦类对象时，如果基类的析构函数不是 ```virtual```，那么⼦类的析构函数将不会被调⽤，⼦类的资源没有正确是释放，因此造成内存泄露


指针的指向改变，分配的内存没有释放，都可能造成内存泄漏。

通过将内存的分配封装再类中，ctor分配内存，dtor释放内存；使用智能指针，可以避免内存泄漏。
