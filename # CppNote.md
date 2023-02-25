# 类
```Cpp

ctor: 构造函数
dtor: 析构函数

explict: 常用于构造函数前, 说明该构造函数只能
显式调用, 让该构造函数不能隐式调用和复制初始化。

public: 只有当前类能访问
protected: 只有当前类和子类能访问
public: 所有类都能访问

classname(): 临时对象

类中能加const则要加const

当成员函数的 const 版本和 non-const 版本同时存在时,
const 对象只能调用 const 函数
non-const 对象只能调用non-const 函数

const 成员函数可以被 const 对象和non-const 对象调用
non-const 成员函数只能被non-const 对象调用


类的构造函数要使用初始化列表

如果类中有指针则必须要带有：
classname( classname& a): 拷贝构造函数
classname& operator=( classname& a):重载赋值操作符
这两个函数和析构函数被称为Big Three

如果类中有指针，默认的拷贝构造函数是浅拷贝，
会造成内存泄露和指针指向同一位置

classname c1;
c1.memberFunction();	
	//等价于：classname::memberFunction( &c1);
	//c1相当于this指针, 如果函数为static函数则没有这一步

static成员函数没有this指针, 只能处理static数据

调用static函数可以用object调用, 
也可以通过classname调用

static数据要在类外做一次定义(给不给初值都可以),
类内只是声明

empty class 在经过编译器编译后就不是empty了, 有
一个copy ctor、一个copy assignment operator、一个dtor
所有这些function都是public且inline
default ctor 和 dtor 主要给编译器一个地方存放‘藏身幕后’的code, 
eg: 唤起base classes、non-static members的ctors和dtors

编译器产生的dtor是non-virtual, 除非这个class的base class本身宣告有virtual dtor

如果class中有pointer则要写big-three/big-Five, 如果没有则一般不需要写

```

## 1. 拷贝赋值的重载
```Cpp

需要检测自我赋值, 好处：
	1.速度快, 效率高, 防止不必要的动作
	2.防止a = a的出错, 下例的高亮行如果没有自我赋值检测, 则a = a会出错
```
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

## 2. new / delete
```cpp
new 表示在堆上分配内存的请求，如果有足够的可用内存，new 运算符会初始化内存
并将新分配和初始化的内存的地址返回给指针变量。
当使用new运算符来创造类对象时，对象的内存是使用堆中的 operator new 分配的。然后
调用对应的ctor
```
```cpp
operator new 是一个分配原始内存的函数，operator new 可以重载，它不初始化内存，
即不调用构造函数。但是，在我们重载的 new 返回后，编译器也会在适用时自动调用构造
函数。也可以全局或为特定类重载 operator new。

```
```Cpp
new时先分配memory, 再调用构造函数
delete时先调用析构函数, 再释放memory

如果new了带有指针的类的数组, delete时没有[], 则只会唤起一次dtor
使用[]后编译器会知道要调用多少次dtor(在cookie后4字节会说明需要调用多少次)
但如果类中不带指针, 则delete时不使用[]也不会造成内存泄露(为了一致性, 建议也写[])

new 和 delete 无法重载, new 和 delete 被分解成三个动作, 其中
operator new 和 operator delete 可以重载
还可以重载多种类型的 new()、delete(), 但是不同版本要有独一无二的参数列表,
其中的第一参数必须为 size_t
```
```Cpp
new 被分解为
void* mem = operator new ( sizeof( Type));
p = static_cast<Type*>( mem);
p->Type::Tyoe();
delete 被分解为
p->~Type();
operator delete( p);
```
```cpp
operator new 重载
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

operator delete 重载
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
## 3. 继承(inheritance): A is-a B (A是一种B)
```Cpp

父类(base)数据可以被子类(derived)完全继承
函数的继承是继承调用权(子类能调用父类的函数)

可以通过子类对象调用父类函数。

子类若重写了父类的虚函数则通过子类调用的是子类重写的虚函数

Template Method:
	写好框架, 让子类具体实现函数

base的析构函数必须是virtual, 否则出现
undefined behavior

构造由内而外
	derived的构造函数首先调用base的default
	构造函数，然后再执行自己。
析构由外而内
	derived的析构函数首先执行自己，然后再调用
	base的析构函数

```
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
```Cpp
non-virtual function
	不希望derived class重新定义(override)它
virtual function
	希望derived class重新定义(override)它, 
	且你对它已经有默认定义
pure virtual function
	希望derived class一定要重新定义(override)
	它, 且你对它没有默认定义
```

### 3种继承方式
#### 1. public继承方式
```cpp
基类中所有 public 成员在派生类中为 public 属性
基类中所有 protected 成员在派生类中为 protected 属性
基类中所有 private 成员在派生类中不能使用
```
#### 2. protected继承方式
```cpp
基类中的所有 public 成员在派生类中为 protected 属性
基类中的所有 protected 成员在派生类中为 protected 属性
基类中的所有 private 成员在派生类中不能使用
```
#### 3. private继承方式
```cpp
基类中的所有 public 成员在派生类中均为 private 属性
基类中的所有 protected 成员在派生类中均为 private 属性
基类中的所有 private 成员在派生类中不能使用
```
## 4. 复合(composition): A has-a B (A拥有B)

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
```Cpp

构造由内而外
	container的构造函数先调用component的
	default构造函数，然后在执行自己的。
析构由外而内
	container的析构函数先执行自己，然后再
	调用component的析构函数。
```
## 5. 委托(delegation):	composition by reference 
	两者用指针相连, 两者生命不同步
```Cpp{.line-numbers}

class String{
	public:
		.....
	private:
		StringRep* rep;	//rep指向的东西可以随着需求的改变而改变
}
```
## 6. Inheritance+Composition

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
	
## 7. Delegation+Inheritance

```Cpp{.line-numbers}
class A{
	vector<B*> b;
}
class B{
	.....
}
//B类可被继承, A中存数据
```
## 8. pointer-like classes: 智能指针
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

## 9. function-like classes: 仿函数
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

## 10. explicit
```cpp
用于接受一个以上实参的构造函数, 取消ctor的隐式调用
```
## 11. =default  /  =delete
```cpp
如果自己定义了一个ctor/dtor那么编译器不会再给你一个default ctor/dtor
如果你强制加上=default, 就可以重新获得并使用default ctor/dtor(可以在继承等时候使用)

=delete告诉编译器不要定义该函数, 必须出现在声明式中, 还用与任何成员函数, 如果
用于dtor则后果自负。

拷贝构造函数不能被重载，如果自己写了拷贝构造函数则不能写=default
一个函数只能在最初定义的时候定义为delete
一般函数不能定义为default
default只能用于构造函数, 拷贝构造函数, 搬移构造函数, 搬移赋值函数, 析构函数

dtor定义=delete则无法destroy objects

```
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
```
# 对象模型(Object Model)
## virtual pointer 和 virtual table
```Cpp
类中如果有 virtual function, 则类大小为数据大小加一根指针的大小, 
该指针为 virtual pointer

对象调用函数是静态调用, 指针调用函数是动态调用
如果是对象调用虚函数则无需多想, 属于哪个Type则调用谁的函数
如果是指针调用虚函数，则new了哪个Type就调用那个的函数

virtual function调用不是静态绑定用call语句的调用, 是通过动态绑定，
用 virtual pointer, 指向 virtual table, 
virtual table用来查 virtual function地址
p指向类的对象, vptr为虚指针, n为虚表内函数序号(编译器在编译代码时会给, 从0开始)

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
```cpp
父类的虚函数子类如果要对其override则两者签名必须要完全相同
子类对父类的虚函数复写可在函数名后面加上overrid告知编译器是对其的复写,
让编译器帮助检车是否有签名错误, 如果有则error
eg：
void function ( int) {.....}
void function ( double ) {.....}
两者签名不同, 不是override, 会把子类的当成一个新函数
如果子类写成：
void function( double) override {.....}
编译器报错
```
## final
```cpp
class finalClass final{
	.....
};
finalClass不能被别的类继承, final表示这个类是继承关系下最底层的类
class A{
	virtual void finalFunction() final;
};
class B: A{
	void finalFunction();	//[Error] overriding final function
};
finalFunction不能被子类复写
```
# 函数
```Cpp

函数参数要用reference传递	
函数返回值为函数中创造的本地变量、本地对象，则不能return by reference
non-const 且有数据共享的 function要考虑Copy On Write(COW)
```
## 1.成员函数
```Cpp

每一个成员函数都默认带一个隐藏的this参数
(可能是第一个参数也可能是最后一个)
```
## 2.conversion function: 转换函数

```Cpp

operator Type() const{	//没有返回类型，编译器默认返回类型为Type
	.....
}
一个类可以有多个转换函数, Type类型也不唯一,只要是之前出现过的Type就行
```

## 3.lambda
```cpp
一种inline function，可以被当作参数或本地变量, lambda的出现改变了
cpp标准库的使用方式

[.....] (.....) mutableOpt throwSpecOpt -> retTypeOpt{.....}
throwSpec是否抛出异常
retType返回类型
Opt:选用,可写可不写
如果三个都没有则(.....)也是可写可不写
(.....)放函数参数, 像一般函数一样

[.....]定义了lambda的捕获方式
1.[]:没有使用任何函数对象参数。
2.[=]:函数体内可以使用Lambda所在作用范围内所有可见的局部变量(包括Lambda所在类
的this)，并且是值传递方式。
3.[&]:同上不过是引用传递方式。
4.[this]:函数体内可以使用Lambda所在类中的成员变量。
5.[a]:将a按值进行传递。
6.[&a]:将a按引用进行传递。
7.[a,&b]:将a按值进行传递，b按引用进行传递。
8.[=, &a, &b]:除a和b按引用进行传递外，其他参数都按值进行传递。
9.[&, a, b]:除a和b按值进行传递外，其他参数都按引用进行传递。
```
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

# reference(实质上就是指针)
```Cpp

使用reference传递参数，传递者无需知道接收者是以reference形式接受
很少直接声明一个变量类型是reference, 常用于参数类型和返回类型的描述

//两者无法并存, 被视为same signature(签名)
Type function(const Type a){	..... }
Type function(const Type& a){ ..... }
//但是如果一个函数在) 加上了const则可以并存, const也被视为签名的一部分

reference设初始值之后再也不能改变代表别的变量

Type& r = x;	//r代表了x, r的大小和x的大小一样
Type xx = a;
r = a;		//把a赋值给r, 现在r和x都等于a

注：
	1.x的地址在哪r的地址就在哪
	2.sizeof(X) == sizeof(r)

Java里所有变量都是reference
```


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
```Cpp

两个例子不报错的原因都是因为: 传递者(使用者)无需关注
参数是否以reference的形式实现传递
```
# 操作符重载
```Cpp

带有=的重载考虑对象被连续赋值, 要reference
:: 、 . 、 .* 、 ? : 四个不能被重载
```
## 1. 对于<<等特殊操作符只能重载为非成员函数
```Cpp{ .line-numbers}	

ostream& operator << ( ostream& os, const Type& t){
	return os << .....;
}
```
## 2. 对于自增自减操作符重载
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
```Cpp

class, function, member三大模板

编译器会对function template做实参推倒

class template要说明使用的template类型

泛化、specialization(特化)、partial specialization(偏特化)
```
## 1. partial specialization:
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
## 2. variadic templates(数量不定的模板参数):
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
## 3.alias template
```cpp
template< typename T>
using vec = std::vector<T, myAlloc<T>>;
vec<int> myVec;
不能对alias template做任何特化
```
```cpp
#define 和 typedef 都无法有上述效果

#define vec<T> template <tempname T> std::vector< T, myAlloc<T>>;
vec<int> myVec; 
|
 ------> template <tempname int> std::vector< int, myAlloc<int>>; myVec;

typedef 不接受参数, 最多能写成
typedef std::vector< int, myAlloc<int>> vec; 
```
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
# 杂记
```Cpp
heap，或叫system heap, 是操作系统提供的一块global内存空间，程序可
动态分配从中获得若干blocks。从heap中获得的空间必须要手动的释放。

stack, 是存在于某作用域的一块内存空间，例如当调用函数, 
函数本身会形成一个stack用来存放放置它所接受的参数, 以及返回地址。

可以使用nullptr代替0或者NULL

浅拷贝时要想到是否需要自我赋值检测

break 要和 loop or switch 一起使用，用于跳出单层的loop or switch

void*: 没有关联数据类型的指针。void 指针可以保存任何类型的地址，
并且可以类型转换为任何类型。

```

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

## 1.uniform initialization
```cpp
可以用统一的{}初始化, 当编译器看见{ t1, t2, t3, ..., tn}时, 会做出一个
initializer_list<Type>，关联到一个array< Type, n>, 调用函数(eg:ctor)时
该array内的元素被编译器分解逐一传给函数。若函数参数是个initializer_list<Type>,
调用者不能给出数个Type参数然后以为他们会被自动转为一个initializer_list<Type>传入
```
### initializer_list<>
```cpp
void function( std::initializer_list<Type> num){	//可接受任意个数的参数
	for( auto a = num.begin(); a != num.end(); a++)
	.....
}
function( {.....});
```

## 2.Type Alias(类似typedef)
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
## 3.using的几种用法
```cpp
1. using namespace std;	using std::cout; 等using namespace 和 namespace 的member
2. using ClassName::MemberName;	//使用ClassName中的MemberName
3. type alias 和 alias template
```

## 4.noexcept
```cpp
Type function() noexcept;	//相当于Type function() noexcept(true);
Type function() noexcept(条件a);	//在条件a为真下不会出现异常

异常一定要被处理, 如果不处理则一直向上抛出异常, 如果一直没被处理则调用std::terminate(),
std::ternimate()调用std::abort(), abort()会结束整个程序
```
## 5.decltype
```cpp
decltype关键字可以让编译器找出表达式的type
eg：
map< string, double> c;
.....
decltype(c)::value_type e;	//相当于 map<string, double>::value_type e;
```
### 用法1：声明一个return type
```cpp
template< class type1, class type2>
decltype ( x + y) add( type1 x, type2 y);	
	//编译无法通过, 因为在使用decltype (x+y)时编译器不知道x、y是什么

//正确写法：
auto add( type1 x, type2 y) -> decltype ( x + y);
```
### 用法2：可用于template metaprogramming(模板元编程)
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
### 用法3：传递lambda的类型
```cpp
面对lambda, 我们手上往往只有对象, 类型常用auto又编译器推倒, 想要获得
其type就需要借助decltype
```
## 6.mutable 
```cpp
用来修饰类的数据成员, 而被 mutable 修饰的数据成员, 可以在 const 成员函数中修改

用于lambda表达式, 可以修改按值捕获的方式中的变量
auto function = [=]() {	// error
	x = 1;
};
auto function = [=]() mutable { 
	x = 1;
};

```
## 7.move
```cpp
可以解决unnecessary copying问题 和 做出perfect forwardding
临时对象是一种Rvalue, Rvalue不能放在 = 左边
编译器面对临时对象一定会当成Rvalue reference

被move之后原来的东西就不能用了(因为是浅拷贝, 移动了指针, 而非重新分配内存)
eg:	
	type c;
	type c1(c);	
	type c2( std::move(c));	//这条语句之后, 都不能使用c了

function( move( a));	//调用Rvalue版本的function
对于一个类应该有对应的copy assignment和move assignment

move ctor对以结点方式存储的容器影响不大
```
```cpp
std::forward(arg)
如果arg不是Lvalue reference则返回arg的Rvalue reference，
如果arg是Lvalue reference则不做任何改变
template <class T> T&& forward (typename remove_reference<T>::type& arg) noexcept;
template <class T> T&& forward (typename remove_reference<T>::type&& arg) noexcept;
两者都返回static_cast<decltype(arg)&&>(arg)
```
```cpp
这是一个辅助函数，允许将作为Rvalue reference的参数完美转发到推导类型，保留任何
可能涉及的move语义。需要此函数是因为所有命名值（例如函数参数）始终评估为左值
（即使是那些声明为右值引用的值）, 这给将参数转发给其他函数的模板函数保留潜在的
move语义带来了困难。
```
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
## 8. tuple
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
## 9. 预处理
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
## 10.一些type_traits中的模板
### remove_all_extents
如果 T 是某种类型 X 的多维数组，则提供 X 的成员 的类型，否则类型为 T。
不能对remove_all_extents进行特化
```
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

如果 T 是reference type，则提供等于true的成员常量值。
若 T 是const或volatile(包含void)或函数类型,则提供等于false的成员常量值。
若 T 是对象类型，令U=std::remove_all_extents<T>::type，
若表达式 std::declval<std::remove_all_extents<T>::type&>().~U()
在unevaluated context中合法，则value等于true，否则value为false 。
	declval<T>:将任何类型 T 转换为引用类型，从而可以在decltype 
	中使用成员函数，而无需通过构造函数。
```
```cpp
用法：
template <template _TP>
std::is_destructible<_TP>{}	//true或false
```
### is_trivially_destructible
```cpp
template< class T >
struct is_trivially_destructible;
和is_destructible一样，另加了std::remove_all_extents<T>::type
是non-class类型或带有trivial dtor的class类型
```
```cpp
用法：
template <template _TP>
std::is_trivially_destructible<_TP>{}	//true或false
```
#### trivial destructor
```cpp
trivial dtor是不执行任何操作的dtor，有trival dtor的对象不需要
delete-expression，可以通过简单地解除分配它们的存储来处理。所有c
兼容的数据类型(POD)都可以简单delete
	POD:Plain Old Data 一种 C++ 类型，它在 C 中具有等效项，并且
	使用与 C 相同的规则来进行初始化、复制、布局和寻址。没有ctor，
	dtor，virtual member function
	eg:struct Fred x; 不初始化 Fred 变量 x 的成员。
满足以下几个条件：
1.析构函数不是用户提供的(要么被隐式声明，要么在其第一个声明中被显
式定义为默认值)
2.析构函数不是虚函数(即基类析构函数不是虚函数)
3.所有direct base class 都有 trivial destructor
	direct base class 在其派生类的声明中直接作为基说明符出现的基类
4.所有class type的non-static member或array都有trivial destructors
```
### is_nothrow_destructible
```cpp
和is_destructible一样，不过dtor是noexcept
```

## 11.强制类型转换

### static_cast
```Cpp
static_cast<Type>(value);
用途：
	1. 基本数据类型之间的转换，如int换成char
	2. 把任何类型的表达式转换成void类型
注：
	没有运行时检查来保证转换的安全性
	把派生类转换成基类(上行转换)是安全的
	把基类转换成派生类(下行转换)是不安全的，因为没有动态类型检查
```
### dynamic_cast
```Cpp
dynamic_cast<baseClass>(pointer_to_)
在进行下行转换时dynamic_cast具有类型检查的功能，比static_cast安全，转换后必须
是类的pointer、reference或void*，基类要有虚函数，可以交叉转换
只能用于存在虚函数的父子关系的强制类型转换，对于指针，转换失败返回nullptr，对于
reference，转换失败抛出与 std::bad_cast 类型的处理程序相匹配的异常
```

### reinterpret_cast
```cpp
可以将整型转换为指针，也可以把指针转换为数组；可以在指针和引⽤⾥进⾏肆⽆忌惮的转
换，平台移植性比较差。
```

### const_cast
```cpp
常量指针转换成非常量指针，并且仍然指向原来的对象，常量引用被转换成非常量引用，
并且仍然指向原来的对象，去掉类型的const或volatile
```

## 12.赋值表达式中的各种类型

### lvalue
```cpp
出现在赋值表达式的左侧,包括变量名、const 变量、数组元素、返回左值引用的函
数调用、bit-field、union和class member。
```

### xvalue
```cpp
生命接近结束的值，资源可能被移动
一个 xvalue 表达式有一个地址，涉及右值引用的表达式的结果，你的程序不再可以访问
它，但可以用来初始化一个右值引用，它提供了对表达式的访问。包括返回右值引用的函数
调用，以及数组下标、成员和指向成员表达式的指针，其中数组或对象是右值引用。

```
### gvalue
```cpp
lvalue或xvalue.
```
### rvalue
```cpp
出现在赋值表达式右侧，一个临时对象或其子对象，或者一个与对象无关的值。
```
### prvalue 
```cpp
纯右值，不是 xvalue 的右值，没有程序可以访问的地址，包括文字、返回非引用类型的
函数调用，以及在表达式求值期间创建但只能由编译器访问的临时对象。
```

## 内存泄漏
