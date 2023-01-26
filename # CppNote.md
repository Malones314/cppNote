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

#### 1. 拷贝赋值的重载
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

#### 2. new / delete
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
#### 3. 继承(inheritance): A is-a B (A是一种B)
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

##### 3种继承方式
###### 1. public继承方式
```cpp
基类中所有 public 成员在派生类中为 public 属性
基类中所有 protected 成员在派生类中为 protected 属性
基类中所有 private 成员在派生类中不能使用
```
###### 2. protected继承方式
```cpp
基类中的所有 public 成员在派生类中为 protected 属性
基类中的所有 protected 成员在派生类中为 protected 属性
基类中的所有 private 成员在派生类中不能使用
```
###### 3. private继承方式
```cpp
基类中的所有 public 成员在派生类中均为 private 属性
基类中的所有 protected 成员在派生类中均为 private 属性
基类中的所有 private 成员在派生类中不能使用
```
#### 4. 复合(composition): A has-a B (A拥有B)

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
#### 5. 委托(delegation):	composition by reference 
	两者用指针相连, 两者生命不同步
```Cpp{.line-numbers}

class String{
	public:
		.....
	private:
		StringRep* rep;	//rep指向的东西可以随着需求的改变而改变
}
```
#### 6. Inheritance+Composition

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
	
#### 7. Delegation+Inheritance

```Cpp{.line-numbers}
class A{
	vector<B*> b;
}
class B{
	.....
}
//B类可被继承, A中存数据
```
#### 8. pointer-like classes: 智能指针
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

#### 9. function-like classes: 仿函数
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

#### 10. explicit
```cpp
用于接受一个以上实参的构造函数, 取消ctor的隐式调用
```
#### 11. =default  /  =delete
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
#### virtual pointer 和 virtual table
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
# 函数
```Cpp

函数参数要用reference传递	
函数返回值为函数中创造的本地变量、本地对象，则不能return by reference
non-const 且有数据共享的 function要考虑Copy On Write(COW)
```
#### 1.成员函数
```Cpp

每一个成员函数都默认带一个隐藏的this参数
(可能是第一个参数也可能是最后一个)
```
#### 2.conversion function: 转换函数

```Cpp

operator Type() const{	//没有返回类型，编译器默认返回类型为Type
	.....
}
一个类可以有多个转换函数, Type类型也不唯一,只要是之前出现过的Type就行
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
#### 1. 对于<<等特殊操作符只能重载为非成员函数
```Cpp{ .line-numbers}	

ostream& operator << ( ostream& os, const Type& t){
	return os << .....;
}
```
#### 2. 对于自增自减操作符重载
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
#### 1. partial specialization:
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
#### 2. template template parameter, 模板模板参数
```cpp{.line-numbers}

template<typename T, template <typename T> class Container>
class XCls{
	private:
		Container<T> c;
	public:
		.....
};

template<typename T>
using Lst = list<T, allocator<T>>;

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
#### 3. variadic templates(数量不定的模板参数):
	可以很方便的完成recursive function call(递归函数调用)
```cpp{.line-numbers}

void print(){ }
template< typename T, typename... Types>
void print( const T& firstArg, const Types&... args){	
//传入的参数至少是1个, 所以要重载一个参数为0的print
	cout << firstArg << endl;
	print( args...);
}
//可以通过sizeof...(args)可以知道包里面还有多少
```
#### 4.alias template
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
#### template template parameter
```cpp
alias template的意义：

Type function( mytype1 a, mytype2 b){
	mytype1< mytype2> c;	//error
	//mytype1 was not declared in this scope
	//mytype1 was not declared in this scope
	........
}

template< typename mytype1, typename T>
Type function( mytype1 a, mytype2 b){
	mytype1<mytype2> c;	//error
	//mytype1 is not a template
	.....
}

```
	
# 杂记
```Cpp
heap，或叫system heap, 是操作系统提供的一块global内存空间，程序可
动态分配从中获得若干blocks。从heap中获得的空间必须要手动的释放。

stack, 是存在于某作用域的一块内存空间，例如当调用函数, 
函数本身会形成一个stack用来存放放置它所接受的参数, 以及返回地址。

可以使用nullptr代替0或者NULL
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

#### uniform initialization
```cpp
可以用统一的{}初始化, 当编译器看见{ t1, t2, t3, ..., tn}时, 会做出一个
initializer_list<Type>，关联到一个array< Type, n>, 调用函数(eg:ctor)时
该array内的元素被编译器分解逐一传给函数。若函数参数是个initializer_list<Type>,
调用者不能给出数个Type参数然后以为他们会被自动转为一个initializer_list<Type>传入
```
##### initializer_list<>
```cpp
void function( std::initializer_list<Type> num){	//可接受任意个数的参数
	for( auto a = num.begin(); a != num.end(); a++)
	.....
}
function( {.....});
```