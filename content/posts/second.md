---
title: "Second"
date: 2022-12-28T20:04:22+08:00
# weight: 1
tags: ["first", "second", "th"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# h1
## h2
### h3
[视频](https://www.bilibili.com/video/BV1v64y1q7JT?spm_id_from=333.337.search-card.all.click)

## 对象布局总览

影响类的实例化对象的大小因素：

- 非静态数据成员
- 虚成员函数
- 为了内存对齐所加上的padding



空对象为何要占用1字节？区分空类实例化出来的对象，使它们具有不同地址。

数据的声明顺序与内存布局？C++标准规定统一访问控制域内的声明顺序和存放顺序一致，不同访问控制域的数据成员的存放顺序没有强制规定，由编译器自行实现，大多数编译器还是保持声明顺序和存放顺序一致。

单继承对象布局（非多态、多态）？

- 非多态：继承体系从上到下叠叠乐

  为什么要保存基类对象部分的padding？（如果去掉padding，则可能将一个派生类对象赋值给基类对象时发生错误）

- 多态：改改被重写虚函数入口地址

> **C++语言保证，出现在派生类对象中的基类子对象有其完整原样性。**

多继承对象布局（非多态、多态）？

- 非多态：继承体系从上到下叠叠乐
- 多态：改改被重写虚函数入口地址

多重继承对象布局（非多态、多态）？

- 非多态：继承体系从上到下叠叠乐
- 多态：改改被重写虚函数入口地址









**gdb调试 ，好！**

x/20x  地址？

disassemble 地址？



> 虚继承？

https://blog.csdn.net/qq_36239569/article/details/116782920



## 构造函数语意学

编译器合成默认构造函数的情况，需满足以下情况：trival 与 non-trival（编译器为我们合成的默认构造函数一定是non-trival的）

1. 没有虚函数和虚基类
2. 任何非静态数据成员没有大括号或等号初始化
3. 直接继承的基类有non-trival的构造函数
4. 任何非静态数据成员有non-trival的构造函数



基类和数据成员的初始化时机：由内而外

- 基类构造函数被编译器默认放在构造函数的用户代码之前；
- 数据成员（类对象）的初始化时机按照数据成员的声明顺序一一初始化；

初始化顺序与成员初始化列表顺序无关，与声明顺序一致；**（试下）**



继承体系中构造函数调用关系：先构造基类再构造派生类；

## 析构函数语意学

编译器合成析构函数：?

如果class没有定义析构函数，那么只有class内含的数据成员、基类有析构函数的情况下，编译器才会自动合成一个析构函数（去调用数据成员、基类的析构函数）。

**验证代码：**



编译器扩充析构函数：

析构函数调用顺序：像剥开洋葱一样由外而内；

- 先析构派生类再析构基类；（派生类对象销毁时只调用了派生类析构函数，只不过编译器在里面加入了基类的析构函数）**验证代码：**

- 析构顺序与类数据成员的声明顺序相反，**验证代码：**

  **查看汇编代码；**









## Data语义学







## Function语义学

<font color="red">TODO</font>

类中的成员函数有*静态成员函数、虚成员函数、非静态非虚成员函数*，那么被调用时编译器作何处理？与调用非成员函数的性能有差异吗？如果有，差异在哪儿？

### 非成员函数



### 普通成员函数（非静态非虚函数）

为了让调用非静态成员函数能够有调用非成员函数一样的性能，编译器的实现是**将非静态成员函数转为非成员函数**：

1. 改写函数签名，参数列表中加入一个this指针（指向调用该成员函数的对象）
2. 将每一个对非静态数据成员的存取，修改为通过this指针存取
3. 将成员函数写成一个非成员函数，并进行name managling处理（提供在程序中独一无二的名称）

```cpp
// function.cc
class XXX {
  void Func1();
	void Func2() const;

 private:
  int a;
  int b;
};
void XXX::Func1() { cout << a << " " << b << endl; }
void XXX::Func2() { cout << a << " " << b << endl; }
// 编译器将Func函数处理为如下（伪码）：
void XXX::Func1(XXX* const this) { cout << this->a << " " << this->b << endl; }
void XXX::Func2(const XXX* const this) { cout << this->a << " " << this->b << endl; }
int main() { return 0; }
```

> <font color="red">TODO</font>
>
> 我使用macos下lldb小小地调试一波：
>
> ```shell
> g++ -o function function.cc -std=c++11 -g
> 
> nm function
> ......
> 0000000100003008 T __ZN3XXX4FuncEv
> 0000000100003150 T __ZNK3XXX4FuncEv
> ......
> # 可以看到XXX::Func1()、XXX::Func2()经由编译器name managling后的名称、以及地址
> lldb function
> ......
> ```
>
> 

### 虚成员函数

```cpp
class Base {
 public:
  Base() = default;
  ~Base() = default;
  void NormalFunc() { cout << "Base::NormalFunc" << endl; }
  virtual void VirtualFunc() { cout << "Base::VirtualFunc" << endl; }
  static void StaticFunc() { cout << "Base::StaticFunc" << endl; }

 private:
};
```

- 通过指针调用：走虚函数表，通过虚机制

  ```cpp
  int main() {
  	Base* ptr = new Base;
    ptr->VirtualFunc();  // 编译器处理为(*ptr->vptr[0])(ptr)
    return 0;
  }
  ```

- 通过对象调用：与普通成员函数同,压制虚拟机制

  > <font color="red">TODO</font>为什么？

  ```cpp
  int main() {
    Base b;
    b.VirtualFunc();  // 编译器处理为Base::VirtualFunc()
    return 0;
  }
  ```



#### 单一继承

#### 多重继承

#### 虚拟继承



### 静态成员函数

静态成员函数为类共享，独立于类的实例化对象，所以通常用于实现独立于类对象之外的存取操作。

```cpp
class Base {
 public:
  Base() = default;
  ~Base() = default;
  void NormalFunc() { cout << "Base::NormalFunc" << endl; }
  virtual void VirtualFunc() { cout << "Base::VirtualFunc" << endl; }
  static void StaticFunc() { cout << "Base::StaticFunc" << endl; }

 private:
};
```

静态成员函数将被转换为非成员函数调用：通过name mangling获得一个独一无二的名称。

```cpp
int main() {
	Base* ptr = new Base;
	ptr->StaticFunc();  // 调用方式一
  Base b;
  b.StaticFunc();  // 调用方式二
  Base::StaticFunc()  // 调用方式三
  delete ptr;
  return 0;
}
```

编译器不会为它加入一个this指针，所以它不能存取类中的非静态成员，也不能声明为const、volatile、virtual。





### 其他

**看汇编**；

- 全局函数

  调用函数流程：

  - 参数传递
  - 调用函数call
  - 处理返回值

- 成员函数：编译器把对象的首地址作为第一个参数，传递给函数。

  - 非静态非虚函数:

    ``t.sum(2);``等价于``Test::sum(&t, 2);``

    构造函数：

    ``Test t(2);``等价于``Test::Test(&t, 2);``

  - 虚函数:其调用借助虚函数表的二次跳转

    ```cpp
    Point2D *p = new Point3D;
    p->print();  // (*p->vptr[n](p))
    ```

> 除虚函数外，类的其他函数性能与全局函数基本一致。





***

[bilibili视频2](https://www.bilibili.com/video/BV1pY411j74X?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click)



0000000100003060