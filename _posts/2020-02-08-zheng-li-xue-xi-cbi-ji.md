---
layout:       post
title:        "整理C++笔记"
author:       "GH"
header-style: text
catalog:      true
tags:
    - C++
---

### 学习C++ 的重要性不言而喻，春节在家没事就整理了一下C++的知识点。

<!-- more -->

#### 第一天

1. c++ 中是不能同时存在两个main函数。
2. getchar() 等待键盘输入。
3. 输出: cout << "xx"。
4. endl: 换行的意思。
5. 键盘赋值给： cin >> xx。
6. C语言不支持函数重载。【重载 -> 函数名相同，参数的个数，参数的类型，参数的顺序不同。】
7. extern "C"/ extern "C" { } -> 按照C语言的方式编译。
8. extern "C"/ extern "C" { } 使用在函数声明的地方。
9. c++ 支持默认参数，【默认参数只能从右边开始】
10. c++ 函数中使用默认参数，既有定义，也有实现，只能将默认参数使用在定义中 【可是是常量，也可以是全局变量】

#### 第二天

1. inline -> 1. 函数代码不多 2. 函数调用频率比较频繁【可以修饰函数声明、也可以修饰函数实现】
2. inline和宏的区别：inline还是有语法检测和函数检测
3. #pragma once：防止整个文件内容被重复引入。【#ifndef,#define,#endif 不受任何编译器限制，也可以用于部分代码不被重复引用】
4. 引用：1. 定义了一个引用,相当于变量的别名, 2. 对引用做计算，就相当于对引用的变量做计算 3. 定义引用的时候必须要赋值，不能修改指向 【对比指针更加安全】 4. 不存在引用的引用/引用的指针/引用数组

```c
int a = 10;
int b = 20;

int *p = &a;  // 相当于p的指针指向变量a
int *&rp = p; // 创建一个指针p的引用/别名rp
rp = &b;      // rp的指针指向变量b/相当于变量b的指针赋值给p

// 数组引用
int array[] = { 10, 20, 30 };
int (&rArray)[3] = array;
const: 1. 常量的意思，被修饰的变量不可修改。 2. 如果修饰的是类,结构体（的指针）其他的成员也不可以修改。3. 修饰的是其右边的内容 4. const 必须放在&左边才能算是常引用 5. 常引用可以用来修饰临时变量/表达式/函数返回值

int age = 10;
const int *p0 = &age; // *p0是常量  p0不是常量
int const *p1 = &age; // *p1是常量  p1不是常量
int * const p2 = &age; // *p2不是常量，p2是常量
const int * const p3 = &age; // *p3是常量，p3是常量

//
int age = 10;
int& const rage = age; // rage 的指向不能改 【不管这里加不加`const`, 引用都不能再次赋值】
int const& rage1 = age;// 不能通过引用修改所指向的内容 【因为rage的时候不需要添加&】
rage = 30;


int const *page = &age;  // 不能通过指针修改所指向的内容
int* const page1 = &age; // 不能修改指针的指向，但是可以通过指针修改指向的内容
// 函数参数用const修饰,入参可以是const， 也可以不是const.
// 函数参数不用const修饰, 入参不可以const
```

#### 第三天
> 汇编基础

```c
// Intel / AT&T汇编
// 寄存器是在CPU里面的
// x86(32bit)  eax/ebx/ecx/edx/ebp...
// x86(64bit) |63-56|55-48|47-40|39-32|31-24|23-16|15-8|7-0|
//								                  |AH..|AL.|
//		                                          |AX......|
//									  |EAX.................|
//		      |RAX.........................................|

/*
R开头的寄存器是64位的，占8个字节/ E开头的寄存器是64位的，占4个字节
[地址值]
word = 2个字节 dword = 4个字节(double) qword = 8个字节(quad)
call调用函数
lea dest [地址值]

move eax, [0x12313..] // 将从地址 0x12313..取四个字节的值赋值给eax
lea dest [0x12313..]  // 将地址 0x12313..  赋值给 dest
ret函数返回
xor异或
inc自增/ dec自减
j开头的一般都是跳转，大多数的带条件的跳转，一般跟test/cmp等指令配合使用

jmp 0x12129.. // 无条件的跳转
// 有条件的跳转
cmp eax, ebx
jz 0x12129..
mov eax, 1
mov ebx, 2
开始开始调用前，栈空间已经分配好了，栈内存的特点：1. 由CPU自动创建和回收的

从内存地址开始往高地址走，取几个字节，就是取到的值
一个变量的地址值是它所有字节中最小的地址值

引用的本质就是地址
引用不能修改如何实现？ 【C++的规则】

int age = 10;
int &rage = age; // 相当于 int * const rage = &age 作用一样
当引用指向了不同类型的数据时，会产生临时变量，即引用指向的并不是初始化的那个变量
表达式可以被赋值 => (a = b) = 4;
*/
```

#### 第四天
1. C++中可以使用struct/class定义类.
2. struct默认属性都是public
3. class默认属性都是private

```c
struct Person {
	// 成员变量
	int m_id;
	int m_age;
	int m_height;

	// 成员函数
	void run() {
		// this指向Person对象的指针
		// this里面存储的就是person对象的地址值
		this->m_id = 11;
		this->m_age = 22;
		this->m_height = 33;
	};
};

int main() {

	// 在栈空间分配了内存给person对象
	// 这个person的对象的内存会自动回收，不需要开发人员去管理
	// person对象占4个字节
	Person person;
	person.m_id = 1;
	person.m_age = 2;
	person.m_height = 3;
	person.run(); // 函数代码不占用空间【在编译的完成的时候就已经决定了函数的地址，调用函数直接就是call xxx】

	cout << sizeof(person) << endl; // 12个字节
	/*
	* 属性值在对象内的分布是连续的，分别占4个字节
	*/
	cout << &person << endl;
	cout << &person.m_id << endl;
	cout << &person.m_age << endl;
	cout << &person.m_height << endl;

	/*
	* 1.  取出person 的对象地址，存入到pPerson
	* 2.  取出pPerson也就是m_id的地址进行赋值
	* 3/4 取出pPerson的地址偏移4/8个字节，进行赋值
	*/
	Person* pPerson = &person;
	pPerson->m_id = 4;
	pPerson->m_age = 5;
	pPerson->m_height = 6;

	return 0;
}
```

#### 第五天

```c
struct Person {
private:
	int m_age;

public:
	void setAge(int age) {
		// 过滤的操作
		if (age < 0) return;
		// 不能通过.来获取值，例如this.xx 这是不对的
		this->m_age = age;
	}

	int getAge() {
		return this->m_age;
	}
};

class Person {// struct 只有成员访问权限的区别，其他没有区别
public:
	int m_age;
	void run() {
		cout << "run() " << endl;
	};
};

/* 内存空间的分布
1. 栈空间 【自动回收和分配】
2. 堆空间 【需要我们主动申请和释放】
3. 代码区
4. 全局区 【全局变量，特点：整个程序运行过程中都存在】
*/
在c++的堆空间如何创建内存？

/*
1. malloc -> free
2. new -> delete
3. new [] -> delete []
*/

void test() {
	// 申请4个字节的堆空间内存
	int* p = (int*)malloc(4);
	*p = 10;
	cout << *p << endl;
	free(p);
}

void test2() {
	int* p = new int;
	delete p;
};

void test3() {
	// 申请10个连续的int类型
	int* p = (int*)malloc(sizeof(int) * 10);
	free(p);

	int* p1 = new int[10];
	delete[] p1;
};

// 下列代码会出现内存泄漏
void test4() {
	int* p = new int; // 第一个int *的指针会出现内存泄漏
	p = new int;

	delete p;
};

void test5() {
	int size = sizeof(int);
	int* p = (int*)malloc(size); // 申请完内存之后，*p的值是脏数据
	// memory set => [从p地址开始的4个字节, 每个字节都存放0]
	memset(p, 0, size);

	cout << *p << endl;

	free(p);
};

void test6() {
	int* p1 = new int; // 没有初始化
	int* p2 = new int(); // 初始化为0
	int* p3 = new int(5); // 初始化为5
	int* p4 = new int[3]; // 没有初始化
	int* p5 = new int[3](); // 初始化为0
	int* p6 = new int[3]{}; // 初始化为0
	int* p7 = new int[3]{ 5 }; // 初始化为 5，0，0
};

struct Person {
	int m_age;
};

// 全局区
Person g_person;

int main() {
	// 栈空间
	Person person;

	// 堆空间
	Person* p = new Person();
	p->m_age = 20;
	// 释放空间
	delete p;

	Person* p1 = (Person*)malloc(sizeof(Person));
	// 释放空间
	free(p1);

	return 0;
}
构造函数的注意事项：

#include <iostream>
using namespace std;

struct Person {
	int m_age;

	Person() {
		cout << "Person() " << endl;
		this->m_age = 0;
		// 格式化Person中的所有值为0：memset(this, 0, sizeof(Person));
	};

	// 构造函数重载 overload
	// 一旦自定义了构造函数，一定要用自定义的构造函数初始化对象
	Person(int age) {
		this->m_age = age;
		cout << "Person(int age)" << endl;
	};
};

Person g_person(); // 这是一个函数声明

int main() {
	Person person3(); // 这是一个函数声明，无参，返回值类型为Person

	// 创建的person对象在栈空间
	Person person1;
	Person person(30);
	person.m_age = 20;
	cout << "age is " << person.m_age << endl;

	// 创建的person对象在堆空间
	Person* person2 = new Person();
	delete person2;
	Person* person3 = new Person(100);
	delete person3;

	// 通过malloc创建对象内存的时候不会调用构造函数
	Person* person4 = (Person*)malloc(sizeof(Person));
	delete person4;

	// 在某些特定的情况下，编译器会为类生成空的无参的构造函数

	return 0;
};
成员变量的初始化问题

#include <iostream>
using namespace std;

struct Person {
	int m_age;
};

// 全局区（成员变量的初始值为0）【不管有没有实现构造函数，都会默认初始化成员变量】
Person g_person;

/*
！！如果自定义了构造函数，除了全局区，其他内存空间的成员变量默认都不会初始化，需要开发人员手动初始化
*/

//!! 1个字节  => 0000 0000 (8个bit)

int main() {
	// 栈空间（成员变量没有初始化）【不管有没有实现构造函数，都不会初始化成员变量】
	Person person;
	// 堆空间
	Person* p1 = new Person; // 成员变量没有初始化 【不管有没有实现构造函数，都不会初始化成员变量】
	Person* p2 = new Person(); // 成员变量有初始化 【如果没有定义构造函数则会初始化成员变量，如果定义了构造函数则不会初始化成员变量】

	cout << g_person.m_age << endl;
	cout << p1->m_age << endl;
	cout << p2->m_age << endl;

	return 0;
};
```

#### 第六天

```c
析构函数的特点：

struct Person {
	int m_age;
	// 对象创建完毕的时候调用
	Person() {
		cout << "Person()" << endl;
		this->m_age = 0;
	};
	// 对象销毁的时候调用
	// 析构函数特点： 1. 不能重载 2. 使用malloc创建的内存，调用free, 不能触发析构函数 3. 构建函数/析构函数必要public
	~Person() {
		cout << "~Person()" << endl;
	};
};
对象内存的管理

#include <iostream>
using namespace std;

// 如果这个对象里面没有任何成员变量，也会占1个字节
struct Car {
	Car() {
		cout << "Car()" << endl;
	};

	~Car() {
		cout << "~Car()" << endl;
	};
};

struct Person {
	int m_age;  // 4个字节
	Car *m_car; // 一个指针，占4个字节，起初不会立即创建一个Car对象

	// 初始化工作
	Person() {
		cout << "Person created" << endl;
		this->m_car = new Car();
	};

	// 内存回收，清理工作（回收Person对象内部的申请的空间）
	~Person() {
		delete this->m_car;
		cout << "Person destoryed" << endl;
	};
};

int main() {
	Person* p = new Person(); // p指针在栈空间  Person/Car都在堆空间

	cout << sizeof(Car) << endl; // 啥属性都没有，也会占有1个字节

	delete p;

	return 0;
};
声明和实现分离的操作
Person.h文件 定义声明

#pragma once

// 声明
class Person {
	int m_age;
public:
	Person();
	~Person();
	void setAge(int age);
	int getAge();
};
Person.cpp文件 实现内容

#include "Person.h"
#include <iostream>

using namespace std;

// :: 域运算符号 【跟在函数名的前面】
// 实现
Person::Person() {
	cout << "Person()" << endl;
};
Person::~Person() {
	cout << "~Person()" << endl;
};
void Person::setAge(int age) {
	this->m_age = age;
};
int Person::getAge() {
	return this->m_age;
};
命名空间的概念

#include <iostream>
#include "Person.h"
using namespace G2;
using namespace std;


// 命名空间不影响内存布局
namespace GX {
	int g_no;
	class Person {
	public:
		int m_age;
	};
};


namespace GH {
	int no;
	class Person {
	public:
		int m_height;
	};

	void test() {

	};
};

void test1() {
	GX::Person person;
	person.m_age = 10;
	GX::g_no = 1;

	GH::Person person2;
	person2.m_height = 10;
	GH::Person* person3 = new GH::Person();
	person3->m_height = 11;
	GH::test();
};

// 命名空间可以嵌套
namespace GG {
	namespace HH {
		int m_age;
	};
};

void test11() {};

namespace G1 {
	int m_age;
};

namespace G1 {
	int m_height;
};

// => 合并命名空间, 和上面是等效的
namespace G1 {
	int m_age;
	int m_height;
};

int main() {
	GG::HH::m_age = 10;

	using namespace GG::HH;
	m_age = 11;

	::test11(); // 空的，就是最大的命名空间

	return 0;
};
C++ 关于继承

// C++ 里面没有想Java/Object-C里面一样有基类
int main() {
	
	GoodStudent gs;
	gs.m_age = 20;
	gs.m_no = 1;
	gs.m_money = 2000;

	cout << sizeof(Person) << endl;
	cout << sizeof(Student) << endl;
	cout << sizeof(GoodStudent) << endl;

	// 内存排布
	// 父类的成员变量在前面，子类的成员变量在后面
	cout << &gs << endl;
	cout << &gs.m_age << endl;
	cout << &gs.m_no << endl;
	cout << &gs.m_money << endl;

	return 0;
};
成员访问权限的问题：

/*
子类内部访问父类成员的权限，是取以下2个权限最小的那个一个
1. 成员本身的访问权限
2. 上一级父类的继承方式
*/
// 开发中的继承都是要用public关键字，这样可以保证父类的权限特性

struct Person {
protected: // 子类内部/当前类内部可以访问
	int m_age;
};

struct Student: public Person {
private:// 只有当前类内部可以访问（class默认）
	int m_score;
};

struct GoodsStudent: public Student {
public: // 任何地方都可以访问 （struct默认）
	int m_money;
};
初始化列表

struct Person {
	int m_age;
	int m_height;

	Person() {
		// ！！直接调用构造函数，会创建一个临时对象，传入一个临时的地址给this指针
		// ！！构造函数的互相调用必须要在初始化列表中调用
		Person(0, 0);
		/*this->m_age = 0;
		this->m_height = 0;*/
	}

	// 初始化列表 : m_age(age), m_height(height) // 语法糖
	// 1. 只能用在构造函数中
	// 2. 初始化列表中的顺序只和 成员变量声明的顺序有关【跟内存的排布】
	Person(int age, int height) : m_age(age), m_height(height) {}
	/*Person(int age, int height) {
		this->m_age = age;
		this->m_height = height;
	}*/

	void display() {
		cout << "age is " << this->m_age << endl;
		cout << "height is " << this->m_height << endl;
	}
};
```

#### 第七天

```c
构造函数列表的功能

class Person {
	int m_age;
	int m_height;
public:
	//Person(int age = 0, int height = 0) :m_age(age), m_height(height) {}
	// 定义，默认参数只能写在函数的声明中
	Person(int age = 0, int height = 0);
};

// 实现分离
// 构造函数的初始化列表只能写在实现中
Person::Person(int age, int height) :m_age(age), m_height(height) {}

int main() {
	Person person;
	Person person1(10);
	Person person2(10, 180);

	return 0;
};
父类的构造函数
子类默认会调用父类的无参构造函数
子类默认会调用父类的无参构造函数
子类的构造函数显式调用父类的有参构造函数，就不会区调用父类的无参构造函数
如果父类里面没有无参的构造函数，子类必须调用有参的构造函数

#include <iostream>
using namespace std;

// 子类默认会调用父类的无参构造函数
// 子类的构造函数显式调用父类的有参构造函数，就不会区调用父类的无参构造函数
// 如果父类里面没有无参的构造函数，子类必须调用有参的构造函数
class Person {
	int m_age;
public:
	//Person() {
	//	cout << "Person()" << endl;
	//};

	Person(int age) : m_age(age) {
		cout << "Person(int age)" << endl;
	}
};

class Student :public Person {
	int m_score;
public:
	Student(): Person(0) {
		cout << "Student()" << endl;
	}

	//Student(int age, int score): m_score(score), Person(age) {
	//	cout << "Student(int age, int score)" << endl;
	//}
};
父类指针的问题

// 父类指针可以指向子类对象， 这个是安全的，开发中经常用到
Person* stu = new Student();
// 子类指针指向父类对象，这个是不安全的
Student* person = (Student *)new Person();
如果子类继承父类的方式不是public, 那么就不能用父类指针指向子类对象
多态的要素：1. 子类重写父类成员函数 2. 父类指针指向子类对象 3. 利用父类指针调用重写的成员函数【必须是虚函数】

虚表的内存排布

#include <iostream>
using namespace std;

class Animal {
public:
	int m_age;
	virtual void speak() {
		cout << "Animal::Speak()" << endl;
	}
	virtual void run() {
		cout << "Animal::run()" << endl;
	}
};

class Cat : public Animal {
public:
	int m_life;
	void speak() {
		cout << "Cat::Speak()" << endl;
	}
	void run() {
		cout << "Cat::run()" << endl;
	}
};

int main() {

	Animal* cat = new Cat();
	cat->m_age = 20;
	cat->speak();
	cat->run();
	/*
	// 取出cat指针变量里面的存储的地址值
	// eax里面存放的是Cat对象的地址值
	mov         eax,dword ptr [cat]  

	// 取出cat对象的前面4个字节内容给edx
	// edx里面存储的是虚表的地址
	mov         edx,dword ptr [eax] 

	mov         esi,esp  
	mov         ecx,dword ptr [cat]  

	// 取出虚表地址（再偏移4个字节）取前4个字节的内容给eax
	// eax里面存储的是Cat::run的函数地址
	mov         eax,dword ptr [edx+4]  

	call        eax  
	*/

	/*
	同一个类的虚表的都是同一的
	*/
	Animal* a1 = new Animal(); // 虚表地址：84 9d f9 00 -> 这个是小端(从右往左读)  0xf99d84
	Animal* a2 = new Animal(); // 虚表地址：84 9d f9 00
	Animal* a3 = new Animal(); // 虚表地址：84 9d f9 00

	return 0;
};
```

#### 第八天

```c
如果子类中没有覆写父类的的虚函数，那么它存放的就是父类的虚函数地址。
如果子类中没有任何覆写的函数，那么它还是存在虚表地址，子类和父类的虚表地址不一样，但是内容是一样的。
如何调用父类中的成员函数。
如果使用了多态，析构函数也要使用虚函数。这样才能保证正常销毁对象。
如果子类继承了抽象类，没有完全覆盖虚函数，则子类也属抽象类

父类::xxx
#include <iostream>
using namespace std;

// 含有纯虚函数的类，是抽象类 【不能直接实例化】
class Animal {
public:
	// 纯虚函数 【没有函数体的函数，且初始化为0, 定义接口规范】
	virtual void speak() = 0;
	virtual void run() = 0;
};

// 如果子类继承了抽象类，没有完全覆盖虚函数，则子类也属抽象类
class Dog : public Animal {
public:
	void speak() {
		cout << "Dog speak()" << endl;
	}

	void run() {
		cout << "Dog run()" << endl;
	}
};
多继承
如果子类继承的多个父类都有虚函数，那么子类对象就会产生对应的多张虚表
如果是同名函数，想要调用指定函数的话，就在指定函数名前面加 对象:: [如果自己没有函数的话，就会报错，因为二意性]
虚基类

/*
1. 虚表地址  --------------------------->  1. 0: 虚表指针与本类起始的偏移量 （一般为0）
2. m_score                                2. 8：虚基类成员与本类的偏移量
3. m_age    // 虚基类的属性值放在最后
*/
```


#### 第九天

```c
static相关知识
静态成员：

存储在数据段 （全局区，类似全局变量） 2. 整个程序运行过程中只有有一份内存
与下面的全局静态变量相比，它可以设置访问权限（private,protected,public）,达到局部共享的目的
静态成员必须初始化 2. 必须在类外面初始化 3. 初始化的时候不能带static
如果类的定义和实现都分开的话，需要将static定义的属性值初始化的是步骤放在.cpp中
1.直接通过类来获取 2.直接通过对象来获取 3.直接通过指针来获取
静态成员函数：
静态函数内部不能使用this（this指针只能用在非静态成员函数内部）
不能用虚函数
内部不能访问非静态成员变量，只能访问静态成员变量/函数
非静态成员函数可以访问 静态成员函数和静态成员变量
构造函数和析构函数不能是静态函数
声明和实现分离的时候，实现中不能出现static

class Person1 {
	static int getAge();
};

int Person1::getAge() {
	return 1;
}
/*
	静态成员变量 
	1. 存储在数据段 （全局区，类似全局变量）
	2. 整个程序运行过程中只有有一份内存
*/

// 与下面的全局静态变量相比，它可以设置访问权限（private,protected,public）,达到局部共享的目的

/*
	1. 静态成员必须初始化
	2. 必须在类外面初始化
	3. 初始化的时候不能带static
*/

// 如果类的定义和实现都分开的话，需要将static定义的属性值初始化的是步骤放在.cpp中

int main() {
	// 1.直接通过类来获取
	Car::m_price = 30;
	// 2.直接通过对象来获取
	Car car3;
	cout << car3.m_price << endl;
	// 3.直接通过指针来获取
	Car* p = new Car();
	cout << p->m_price << endl;
}
单例模式

#include <iostream>
using namespace std;

/*
	我们希望某些类的实例对象永远只有一个
	单例模式
	条件：

	1. 构造函数私有化
	2. 定义一个私有的静态成员变量指针，用于指向单例对象
	3. 提供一个公共的返回单例对象的静态成员函数
*/

#include <iostream>
using namespace std;

/*
	我们希望某些类的实例对象永远只有一个
	单例模式
	条件：

	1. 构造函数私有化
	2. 定义一个私有的静态成员变量指针，用于指向单例对象
	3. 提供一个公共的返回单例对象的静态成员函数
*/

class Rocket {
public:
	static Rocket* sharedRocket() {
		// p_thread
		if (ms_rocket == NULL) {
			ms_rocket = new Rocket();
		}
		return ms_rocket;
	}
	static void freeRocket() {
		if (ms_rocket == NULL) return;
		delete ms_rocket;
		ms_rocket = NULL; // 回收内存、抹掉数据
	}
private:
	static Rocket *ms_rocket;
	Rocket() {
		cout << "Rocket()" << endl;
	}
};

Rocket* Rocket::ms_rocket = NULL;

如果子类父类中存在同名的静态成员变量，是可以的，并且不存在继承功能

class Person {
public:
	static int m_age;
};

int Person::m_age = 1;

class Student : public Person {
public:
	static int m_age;
};

int Student::m_age = 2;

int main() {
	Person* p1 = new Person();
	Student* s1 = new Student();
	
	/*
	Person location is 0094C034
   Student location is 0094C038
	*/
	cout << "Person location is " << &p1->m_age << endl;
	cout << "Student location is " << &s1->m_age << endl;

	return 0;
}
```

#### 第十天

* const 成员
> 被const修饰的成员变量、非静态成员变量

* const成员变量
> 必须初始化（类内部初始化），可以在声明的时候直接初始化赋值/也可以初始化列表赋值

* const成员函数（非静态）
> 函数的声明和实现都必须要带const
> 不能在函数内部修改当前对象的成员变量（非static）
> 内部只能调用const成员函数,static成员函数
> 非const成员函数可以调用const成员函数
> 函数通过添加const和非const，可以构成重载
> 非const对象（指针）优先调用非const成员函数、const对象（指针）只能调用const成员函数

* 拷贝构造函数

* 如果函数的入参是字符常量

> Car(int price, char *name = NULL) :m_price(price) {}

* 使用对象类型作为函数的参数或者返回值，可能会产生一些不必要的中间对象

#### 第十一天

```c
匿名对象
// 匿名函数
Person().display();
参数传入对象时：
Person person; // 构造一次
test1(person); // 作为参数传去进去的时候，会再创建一个Person对象（所以这里会有一个拷贝构造）

test1(Person()); // 调用匿名构造函数，只会调用一次构造函数，不会调用拷贝构造
隐式构造
/*
	Person(int age), age = 10,this = 00EFFCE0 - 创建一个新的对象1
	Person(int age), age = 20,this = 00EFFC14 - 隐式构造创建一个临时对象2，并且把它的属性值赋值给对象1的属性值
	~Person(), age = 20,this = 00EFFC14       - 析构对象2
	~Person(), age = 20,this = 00EFFCE0       - 析构对象1
*/
Person person(10);  //!! 如果没有单参数构造函数，则会报错
person = 20; // 隐式构造：Person(20)

// 禁止隐式构造
explicit Person(int age) : m_age(age) {}
编译器自动生成的构造函数
C++的编译器会在默些特定的情况下，为类自动生成无参的构造函数，例如：
成员变量在声明的同时进行了初始化。【需要进行赋值】
有定义虚函数。 【需要创建虚表】
虚继承其他类。 【需要创建虚表】
包含了对象类型的成员（如果是指针类型则不需要调用构造函数），且这个成员有构造函数 【需要调用子类的构造函数】
父类有构造函数。【需要先创建父类】
对象创建后，需要做一些额外操作时（比如内存操作，函数调用），编译器一般都会为其生成构造函数。

友元 [编译器行为]
如果将函数A（非成员函数）声明为类C的友元函数，那么函数A就能直接访问类C对象的所有成员。

class Point {
	friend Point add(const Point&, const Point&); // 定义友元函数 【全局的】
	friend class Math; // 定义友元类
private:
	int m_x;
	int m_y;
public:
	Point(int x, int y) :m_x(x), m_y(y) {}
	int getX() const { return this->m_x; }
	int getY() const { return this->m_y; }
};

Point add(const Point& point1, const Point& point2) {
	return Point(point1.m_x + point2.m_x, point1.m_y + point2.m_y);
}

class Math {
public:
	Point add(const Point& point1, const Point& point2) {
		return Point(point1.m_x + point2.m_x, point1.m_y + point2.m_y);
	}
};
内部类/局部类【定义在函数内部】

运算符重载

#include <iostream>
using namespace std;

class Point {
	friend Point operator+(const Point&, const Point&);
private:
	int m_x;
	int m_y;
public:
	Point(int x, int y) : m_x(x), m_y(y) {}
	static void display(const Point &point) {
		cout << "(x: " << point.m_x << ", y: " << point.m_y << ")" << endl;
	}
};

// 操作符重载
Point operator+(const Point &p1, const Point &p2) {
	return Point(p1.m_x + p2.m_x, p1.m_y + p2.m_y);
}

int main() {
	
	Point::display(Point(1, 2) + Point(2, 3)); // 等价 =>  display(operator(Point(1, 2), Point(2, 3)));
	Point::display(operator+(Point(1, 2), Point(2, 3)));


	return 0;
}
通过指针this获取对象 => '*this'
const对象只能调用const的成员函数
```

#### 第十二天

```c
操作符重载
// ++x
void operator++() {
	this->m_x++;
	this->m_y++;
}

// x++
void operator++(int) {
	this->m_x++;
	this->m_y++;
}
a++与++a运算符的汇编解析
a++ 会返回a以前的值进行运算，运算完毕之后才会让a的值加1。【不能进行赋值】
++a 会直接让a的值加1，并且作为最新的a进行运算。【可以进行赋值】

int b = a++ + 8;
// eax = a
mov         eax,dword ptr [a]  
// eax += 8
add         eax,8  
// b = eax [此时a的值没有发生改变]
mov         dword ptr [b],eax  

// ecx = a
mov         ecx,dword ptr [a]  
// ecx += 1
add         ecx,1  
// a = ecx [a++] 
mov         dword ptr [a],ecx  


int c = ++a + 8;
// 先将a的值加1，然后覆盖a的值
// eax = a
mov         eax,dword ptr [a]  
// eax += 1
add         eax,1  
// a = eax [a++] 
mov         dword ptr [a],eax  

// ecx = a [a = 9]
mov         ecx,dword ptr [a]  
// ecx += 8
add         ecx,8  
// c = exc
mov         dword ptr [c],ecx 
重载
// 操作符重载
Point operator+(const Point &p1, const Point &p2) {
	return Point(p1.m_x + p2.m_x, p1.m_y + p2.m_y);
}

// 重载 cout << Point;
ostream& operator<<(ostream& cout, const Point& point) {
	return cout << "(x: " << point.m_x << ", y: " << point.m_y << ")";
}
字符串重载实战
重写String类
// String.h
#pragma once
#include <iostream>
using namespace std;

class String {
	friend ostream& operator<< (ostream&, const String&);
public:
	String& operator=(const char* cstring);
	String& operator=(const String &string);
	String(const String &string);
	String(const char* cstring);
	~String();
private:
	char* m_cstring;
	String& assign(const char* cstring);
};

// String.cpp
#include "String.h"

String::String(const char* cstring) {
	assign(cstring);
}

String::String(const String& string) {
	assign(string.m_cstring);
}

String::~String() {
	assign(NULL);
}

String& String::operator=(const String& string) {
	return assign(string.m_cstring);
}

String& String::operator=(const char* cstring) {
	return assign(cstring);
}

String& String::assign(const char* cstring) {
	// 如果指向一样的堆空间
	if (this->m_cstring == cstring) return *this;

	// 如果传入进来的cstring == NULL的话就会达到释放指针的效果
	// 释放旧的字符串
	if (this->m_cstring != NULL) {
		cout << "operator=(const char* cstring) - delete[] String: " << this->m_cstring << endl;
		delete[] this->m_cstring;
		this->m_cstring = NULL;
	}
	if (cstring != NULL) {
		// 创建字符串空间
		this->m_cstring = new char[strlen(cstring) + 1]{};
		// copy赋值
		strcpy(this->m_cstring, cstring);

		cout << "operator=(const char* cstring) - new - String: " << cstring << endl;
	}

	return *this;
}

ostream& operator<< (ostream & cout, const String & string) {
	if (string.m_cstring == NULL) { return cout; }
	return cout << string.m_cstring;
}
```