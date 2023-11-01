

# Chapter 1

#主要介绍一些面向对象编程思想以及程序编写流程...

# Chapter 2

1. 介绍CPP编写规范
2. 引入cin和cout用法
3. 引入using namespace std

# Chapter 3

1. cout显示不同格式的整形数据, 通过给cout传入hex等控制字;
```cpp
int var = 10;
cout << hex;
cout << var; //display 0x0a
/*
hex oct dec ...
*/
```
2. const的优点
* 能够明确指明类型;
* 能够限制作用域;
* 能够作用于复杂类型;
3. auto在c中和cpp中的区别: 在c中指代局部变量,cpp中指代自动指定类型;
# Chapter 4

1. 让编译器确定数组大小
```cpp
char array[] = {1, 2};
int length = sizeof(array) / sizeof(array[0]);
```
2. 字符串[末尾具有空字符`\0`才是字符串,这点至关重要]
3. cin.get支持使用拼接方式
```cpp
cin.get(a, num).get();//前一个cin.get返回一个cin对象,从而调用下一个get
```
4. string类的使用[必须包含头文件\<string\>], string类使得可以像操纵普通变量那样操纵字符串;
5. 匿名共用体没有指定名称,但其内部变量会占据同一片内存空间;
```cpp
struct S
{
	union
	{
		int a;
		float b;
	};
}
...
struct S val;
cout << val.a;
vout << val.b; //这两个变量指向同一片内存空间
```
6. new和delete的用法
```cpp
int ptr_int = new int;//申请单个int内存指针;
int ptr_ints = new int [10];//申请连续十个int内存指针;

delete ptr_int;
delete [] ptr_ints;
```
7. 简要介绍vector容器和array类的用法
```cpp
#include <vector>
using std::vector;
vector<int> var(10); //申明十个int类型内存变量
```
# Chapter 5

1. for, while, do while...
# Chapter 6

1. 字符函数库cctype
```cpp
#include <cctype>
//包含各种字符处理函数,如大小写转换之类的
```
2. break和continue的用法,主要注意continue的用法;
3. 文件输入输出处理,包含头文件fstream(可以练习一波);
# Chapter 7

1. 函数指针
* 作为参数传递给另一个函数;
* 形成转换表;
# Chapter 8

1. 内联函数inline,注意是否内联取决于编译器,如果要强制内联,则可以使用__attribute__((always_inline))[只适用于GCC编译器];
2. 引用
* 引用必须要初始化绑定;
3. 临时变量,引用与const
* 当传入函数的实参与[const修饰的引用]形参类型不匹配时,例如传入了一个表达式,则cpp会生成一个临时变量来让引用指向,注意该临时变量只在函数运行期间存在.
* 为什么只针对const修饰的引用呢?
	在早期,任何不匹配的实参传递,都会让函数生成一个临时变量,这会导致这样一个问题.假如函数的本质是通过引用修改实参,但由于引用指向了临时变量,因此这种修改并不会改变实参.
	而如果形参使用const修饰,则代表该变量无需修改,因此生成临时变量也不影响;
* 返回引用
	注意不要返回局部变量,可以返回引用参数或者动态变量;
4. 默认参数:注意从右到左设置默认参数,在函数声明时标注默认参数即可,定义时无需设置;
5. **函数重载**:关键在于参数数目和类型,以及排列顺序的差异,[仅仅返回值不同是不可以构成重载条件的];   本质是名称修饰,这也解释了在头文件中使用extern "C" {}修饰的原因.
6. **函数模板**泛型编程的一种
```cpp
template <typename T>
void fun(T var)
{
	T a;
}//template 和 typename是必须的, 其中template可以由class代替
```
7. 实例化和具体化
实例化可以分为隐式实例化和显式实例化,还有一种称为显式具体化,这些统称为具体化;[不是很懂,后面补充]
8. 

