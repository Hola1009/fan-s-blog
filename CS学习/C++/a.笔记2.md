算了,  看书学太慢了, 书中的知识太细了, 难以抓住我想要的重点, 又怕漏掉什么, 还是直接看视频学吧, 入门快, 这篇笔记也是一样只记录我不了解或是忘记而且会用到的知识,

## 浮点型
小数字面量默认是 `double` 类型, 所以声明 `float` 需要这样
```cpp
float f = 3.14159265f;
```

另外 默认情况下 cout 只打印小数后 6 位
```cpp
double d = 3.14159265
cout << d << endl // ==> 3.141592
```

科学计数法
```cpp
float e = 3e-2;
```

## 引用
`&` 就是引用符号了, 它作用在函数传参的过程中, 进行值传递
下面是一个简单的例子, 在函数中对变量的修改会影响到调用方的变量, 因为是一个函数中的b和调用方的a是一个引用
```cpp
#include <iostream>
#include <vector>
using namespace std;
void fuc(int & a) {
	a = 10;
}
int main() {
	int b = 0;
	fuc(b);
	cout << b; // ==> 10
}
```
注意事项: 引用必须进行初始化, 而且一旦初始化了就不能进行更改了

本质: 引用的本质是一个指针常量

### 引用常量
作用: 引用传参防止误修改的, 这样传入的引用将是只读的
```cpp
void showNum(const int & val) {
	cout << val;
}
```

## 函数
### 函数的默认参数
```cpp
int func(int b = 10) {
	cout << b;
}
```

### 函数重载
更Java一样, 但是不用写注解, 不介绍了
引用传参 和 指针传参 不会起冲突
```cpp
void func(int * a) {
}
void func(int & a) { 
}
```

注意事项:
- 引用常量, 普通引用 作为参数重载, 默认走的是普通引用作为参数
- 引用参量类型 和 对应的普通类型 作为参数重载, 会发生冲突, 但是编译能过
- 普通类型 碰到默认参数, 也会冲突, 但是编译能过

## 面向对象
### struct 和 class 的区别
- 默认访问权限不同, struct 默认是 public, class 默认是 private

### 对象的初始化和清理
构造函数和析构函数没写编译器会有默认的空实现

构造函数语法: `类名(){}` :

解析函数语法: `~类名(){}` : 不可有参数, 对象销毁前会自动调用

#### 构造函数的分类和调用
分类:
- 无参
- 有参
- 拷贝构造函数: 如 `Person( const Person & person)`

调用:
- 括号法: `Person p1` , `Person p1(10)`
- 显示法: `Person p1 = Person(10)`
- 隐式转换法: `Person p1 = 10` 编译器会自动转成 `Person p1 = Person(10)` 
  `Person p2 = p1` => `Pseron p2 = Person(p1)`

#### 拷贝构造函数的调用时机
- 使用已经创建完毕的对象来初始化新对象
- 值传递的方式给函数参数传值
- 值方式返回局部对象

#### 构造函数的调用规则
1. 创建一个类 c++ 默认实现了3个函数(如果没有自定义)
	- 无参构造函数
	- 拷贝函数 (浅拷贝)
	- 解析函数
2. 写了有参/拷贝 就不提供默认的了

#### 初始化列表
就像下面这样在 参数表()后面 以变量名(参数)的形式为属性赋值, 不同的赋值之间用逗号分隔, 最后不要忘记加{}
```cpp
class Person {
public:
	Person():a(10),b(20),c(30){}
	Person(int a, int b, int c):a(a),b(b),c(c){}
	int getA() {
		return a;
	}
	
private:
	int a;
	int b;
	int c;
};
```

#### 类对象作为成员
类对象属性在对象初始化之前进行初始化, 在对象解析之后进行解析

#### 静态成员
静态变量的初始化不能在局部代码内
```cpp
# include <iostream>
using namespace std;
class Person {	
public:
	static int number;
};
int Person::number = 100;
int main() {
	cout << Person::number << endl;
	return 0;
}
```
与Java中的区别: 类内声明, 类外进行初始化

### C++ 对象模型和 this 指针
空对象默认占一个字节, 只有非静态成员变量是属于对象的, 静态成员变量, 非静态函数, 静态函数 都不属于对象

#### this 指针
用途:
- 解决名称冲突 `this->age = age`
- 返回对象本身: 返回类型应该是一个引用类型(否则返回的是一个拷贝), 用来实现链式编程, 例如`cout` 

#### 空指针访问成员函数
如果成员变量中没有使用本地成员变量, 那么成员函数是可以正常访问的(这点和java不同), 但是如果使用了本地成员变量就会报错 

#### const 修饰成员函数
常函数
- 常函数中不可以修改成员属性的值
- 成员属性被 `mutable` 关键字修饰后, 常函数可以修改

常对象
- 被声明时被 `const` 修饰的对象
- 常对象只能修饰常函数

### 友元
#### 全局函数做友元

全局函数可以访问对象的私有成员

```cpp
# include <iostream>
# include <string>
using namespace std;
class Person {	
	// primary code
	friend void func(Person & p);

private:
	string value;
public:
	Person(string value) {
		this->value = value;
	}
};

void func(Person & p) {
	cout << p.value;
}

int main() {
	Person p("zhangsan");
	func(p);
	return 0;
}
```

#### 类做友元
和上面类似, 在类中声明下友
```cpp
friend class GoodGay;
```

#### 成员函数做友元
```cpp
friend void GoogGay::visit();
```

### 运算符重载
#### 加号运算符重载
对于内置的数据类型, 编译器知道如何运算的, 那类对象呢? 该怎么做呢
使用 `operator+` 来实现 `+` 的逻辑, 可以用全局来实现, 成员方法也可以, 全局变量传入两个参数, 成员变量传入一个参数

#### 左移运算符重载
相当于 Java 中的 toString 重载 `operator<<(ostream &cout)` 方法,  -> 简化版本 `p << cout` 这是不对的, 所以一般也不用成员函数来重载左移运算符 也就是 `opertor(ostream & cout, Person & p)` 这种, 为了符合链式调用需要返回 cout 的引用

#### 函数调用运算符重载
仿函数, 像这样定义好仿函数`void operator()(string name)` 通过对象名()调用 `p("abcd")`;

### 继承
语法 `class 子类 : 继承方式 父类`
```cpp
class Father {
	void show() {
		cout << "父类"
	}
}

class Son : public Father {
	
}
```

#### 继承方式
- 公共继承 (public): 继承到的属性范围修饰符范围不变
- 保护继承 (protected): 将继承到的属性的访问修饰符范围拉低到 protected
- 私有继承 (private): 将继承到的属性的访问修饰符范围拉低到 private

#### 同名成员处理方式
同名属性
- 访问子类同名成员, 直接访问即可
- 访问父类同名成员, 需要加作用域: `s.Fatner::val`

同名函数
- 访问子类同名函数, 直接调用
- 访问父类同名函数, 也是加上父类作用域

注意点, 如果子类中出现了同名函数, 那么子类中的同名函数会隐藏父类中所有的同名函数, 所以父类中重载的函数无法直接被调用

#### 继承中同名静态成员处理方式
处理方式和非静态是一样的加上父类作用域


#### 菱形继承
菱形继承导继承到两份同名数据, 导致资源浪费, 解决方法, 使用虚拟继承 `:virtual`


### 多态
动态多态: 派生类和虚函数实现运行时多态

在父类函数前面加上 `virtual` 实现玩绑定, 达到动态多态的目的

#### 纯虚函数和抽象类
纯虚函数就是Java中的抽象函数, 需要被子类实现的函数
语法: `virtual 返回值类型 函数名 (参数列表) = 0`

抽象类特点:
- 无法实例化
- 子类必须重写抽象类中的纯虚函数

抽象类无需特意声明, 只要有一个纯虚函数存在, 这个类就是抽象类

#### 虚析构和纯虚析构



## 模板
### 模板的概念
就是Java中的泛型编程

#### 函数模板语法
函数模板作用: 
建立一个通用函数, 其函数返回值类型和形参类型可以不是具体指定的, 用一个虚拟的类型来表示

```cpp
template<typename T>
函数声明或定义
```

解释:
template --- 声明创建模板
typename --- 用其来指代一种类型

下面是一个简单的使用
```cpp
template<typename T>
void mySwap(T & a, T & b) {
	T temp = a;
	a = b;
	b = temp;
}

int main() {
	 int a = 0;
	 int b = 1;
	 // 自动类型推导
	 mySwap(a, b);
	 cout << "a = " << a << endl;
	 cout << "b = " << b << endl;
	return 0;
}
```

上面用到了自动类型推导, 还可以显示指定类型, 像下面这样
```cpp
mySwap<int>(a, b) 
```

如果模板函数是使用class来定义的, 那么是调用时是必须显示指定类型的
```cpp
template<class T>
```

#### 普通函数和函数模板的区别
- 普通函数调用时可以发生隐式类型转换
- 函数模板使用自动类型推导, 不会发生隐式类型转换
- 模板函数使用显示指定类型, 会发生隐式类型转换

#### 普通函数和函数模板的调用规则
1. 如果函数模板和普通函数都可以调用, 优先调用普通函数
2. 可以通过空模板参数列表来强制调用函数模板
3. 函数模板也可以发生重载
4. 如果函数模板可以产生更好的匹配, 优先调用函数模板


#### 模板函数的局限性
如果模板函数中的赋值操作, 而传入的参数又是数组..., 所以模板并不是万能的

### 类模板
类模板的定义和函数模板类似
```cpp
template<class K, class V>
class MyMap {
	K key;
	V value;
}
```


#### 类模板与函数模板的区别
1. 类模板没有自动类型推导
2. 类模板在模板参数列表中可以设置默认参数

```cpp
template<class K = string, class V = int>
class MyMap {
	K key;
	V value;
}
```

#### 类模板中成员函数的创建时机
类模板中成员函数在调用时才回去创建

#### 类模板对象作函数参数
类模板实例化出的对象, 向函数传参的方式

一共有3种传入方式
1. 指定传入类型 ---直接显示对象的数据类型
```cpp
void show(Map<string, int> &map) {

}
```
2. 参数模板化 ---将对象中的参数变为模板进行传递
```cpp

```
3. 整个类模板化 ---将这个对象类型 模板化进行传递

#### 类模板与继承
当类模板碰到继承时, 需要注意一下几点
- 当子类继承的父类是一个类模板时, 子类在声明的时候, 要指定出父类中T的类型
```cpp
class Son:plulic Base<int>
```
- 如果不指定, 编译器无法给子类分配内存
- 如果想灵活指定出父类中T的类型, 子类也需变为类模板
```cpp
template<clasws T1, class T2>
class Son:plulic Base<T2>
```


#### 类模板成员函数的类外实现
```cpp
template<class T1, clss T2>
Person<T1, T2>::showPerson()
```

#### 类模板与友元
friend 声明一下
```cpp
// 类中声明
// 要加 <> 让其知道 
friend void printPerson<>(Person<T1, T2> p)
```
