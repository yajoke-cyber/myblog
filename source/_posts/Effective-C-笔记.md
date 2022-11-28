---
title: Effective C++笔记
date: 2022-11-17 19:16:52
categories:
- C++
tags:
- Effective C++
---
# Effective C++

## explict

explict用来添加加构造函数的前面，表示该函数无法被隐式调用

```c++
class C
{
public:
    C(){
    };
    C(int) {
        cout << "haha" << endl;
    };
};

void doSth(C obj) {

}
int main() {
    C demo;
    doSth(123);
    
}
```

如下代码即使，当无explict的时候，此时doSth调用的123参数会自动调用class C中对应的重载构造函数然后得到一个对应的对象传入其中，致使本来无法调用doSth现在可以直接调用，因此除非真的有需求，不然建议每一个构造函数前方都加入explict。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915142847838.png" alt="image-20220915142847838" style="zoom:50%;" />





![image-20220915144351160](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915144351160.png)

## const

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915144553114.png" alt="image-20220915144553114" style="zoom: 80%;" />

![image-20220915144603719](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915144603719.png)

### 迭代器中的const

![image-20220915144910538](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915144910538.png)

原则和JavaScript一样，能用const就尽量用const，不仅能或者编译上的优化还可以提示代码中需要不经意的错误

![image-20220915151847754](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915151847754.png)

当const函数内部确实需要改对象的某些值时，此时需要使用mutable关键字，表面该变量在const函数中一样也可以修改。

## 拷贝构造函数

![image-20220915164247566](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915164247566.png)

使用private声明类中的拷贝构造，在外部调用不能通过**静态编译**，但是在成员函数中依然可以调用，只能约定来说不去调函数。

但是如果时采用继承base class的方式，让派生类调用构造函数时也会自动去调用父类的拷贝构造，但是其父类是private所以在这里会抛出异常，但是这种错误编译器察觉不到，只有把**两个类连接起来**时才能发生这个错误，所以此时禁用拷贝函数的错误捕获阶段从编译阶段延后到了连接阶段。

![image-20220915193102069](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915193102069.png)

![image-20220915193112416](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915193112416.png)

```c++
const Son& operator=(const Son& obj) {
        age = obj.age;
        Father::operator=(obj);
        return *this;
 }
```

在这里可以通过直接调用base class的构造函数来保证派生类在拷贝时也能拷贝到父类对应的私有成员变量。

## 公有继承（public)、私有继承(private)和保护继承(protected)

**公有继承（public)、私有继承(private)和保护继承(protected)三种继承方式，****可见即可以访问，不可见即不可以访问。**


\1.  **公有继承方式：**
　　**基类成员的可见性对派生类来说**，基类的公有成员和保护成员可见：基类的公有成员和保护成员作为派生类的成员时，它们都保持原有的状态；基类的私有成员不可见：基类的私有成员仍然是私有的，派生类不可访问基类中的私有成员。
　　**基类成员的可见性对派生类对象来说，基类的公有成员是可见的，其他成员是不可见。**


\2.  **私有继承方式：**
　　**基类成员的可见性对派生类来说**，基类的公有成员和保护成员是可见的：基类的公有成员和保护成员都作为派生类的私有成员，并且不能被这个派生类的子类所访问；基类的私有成员是不可见的：派生类不可访问基类中的私有成员。
　　**基类成员的可见性对派生类对象来说，基类的所有成员都是不可见的。**

\3.  **保护继承方式：**
　　**基类成员的可见性对派生类来说**，基类的公有成员和保护成员是可见的：基类的公有成员和保护成员都作为派生类的保护成员，并且派生类的子类可以访问；基类的私有成员是不可见的：派生类不可访问基类中的私有成员。
　　**基类成员的可见性对派生类对象来说，基类的所有成员都是不可见的。**

## C++隐式转换

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915205552153.png" alt="image-20220915205552153" style="zoom:67%;" />

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915205604248.png" alt="image-20220915205604248" style="zoom:50%;" />

因此当Font作为参数调用changeFontSize的时候，此时会执行Font中的隐式转换函数，将对应的RAII中组合的对象返回。

## 智能指针

![image-20220915224717141](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915224717141.png)

在同一行内执行的操作顺序可能会改变

所以可以先用独立语句将newed对象置入智能指针，然后再开始行内函数调用

![image-20220915224652275](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915224652275.png)

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220915231951252.png" alt="image-20220915231951252" style="zoom: 80%;" />

## 函数传参规则

![image-20220916092241653](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916092241653.png)

## operator *()

![image-20220916102015836](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916102015836.png)

将*对应的member改为 not member即可以使用 2 * Rational也可以得到实现。

![image-20220916101954265](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916101954265.png)





![image-20220916101800312](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916101800312.png)

## 泛型类如何编写特化swap

```c++
namespcae Cname{
template<typename T>
class C
{
public:
    C() {
    };
    C(T a) {
        val = a;
    };
    void swap(C<T>& rhs) {
    	T temp = this->val;
    	this->val = rhs.val;
    	rhs.val = temp;
	}
private:
    T val;
};
template<typename T>
void swap(C<T>& lhs, C<T>& rhs) {
    lhs.swap(rhs);
    cout << "special" << endl;
}
}
//调用方式为 Cname::swap(c1,c2);
```

使用public的member func方式实现可以访问私有变量，然后通过non-member的方式来实现函数模板如以下。

在这里记住需要使用函数模板+重载的方式来实现swap，这样当C类的函数调用swap的时候就采用函数重载的版本，注意这里这里不能使用模板偏特化。

![image-20220916105129962](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916105129962.png)

然后swap为了避免用户的不知觉调用，可以把它和class放置在同一namespace中，这样子用户调用就需要调用swap就更容易进入到namespcae对应的swap函数中，降低代码风险。

## 转型

![image-20220916113908539](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916113908539.png)

![image-20220916115015610](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916115015610.png)

当调用base class的成员函数的时候，不用通过指针转型来调用，因为指针转型会让编译器生成一个临时的基于base class的副本，然后调用成员函数一切修改都是基于base class的副本进行修改，因此这里不用转型，直接调用base class 的成员func即可

## inline

![image-20220916140927786](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916140927786.png)

## 接口继承和实现继承

![image-20220916155127912](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916155127912.png)

想继承其中得一个函数又不想继承其重载版本，如果使用using 函数名会把重载版本也继承过来

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916155719181.png" alt="image-20220916155719181" style="zoom: 67%;" />

同时当private但是又想暴露一两个base class得方法得时候，可以使用转交函数，即自己重写名称然后直接使用Base::mf1()得形式来调用。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220916155825024.png" alt="image-20220916155825024" style="zoom:67%;" />

使用using Base::mf1也会让mf1对应得重载函数可出现（在名字被遮掩的情况下）

## 重点条款

34  37

## 缺省函数不应被重载

![image-20220917105843073](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220917105843073.png)



![image-20220917134851123](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220917134851123.png)

当一个derived class 继承base class的时候，一定要注意就是调用派生类和基类都有缺省虚函数virtual，但缺省函数的调用是基于静态绑定的，这就是为了在编译阶段就确定其应当执行的函数，提高运行阶段的效率。所以如果去调用derived->draw()，实际上调用的还是base->draw()，因为其对应函数指针已经在编译阶段确定好了。 

**只有缺省函数才会被静态绑定**因为当你没传参数的时候其函数内容已经注定了，所以无参调用会直接走静态绑定的缺省函数

## 继承泛型类应当记住的点

```c++
class Tem1 {
public:
    void doSth() {
        cout << 123;
    }
};
template<typename T>
class Tem2 :public Tem1<T> {
public:
    void haha() {
        int a = 10;
        doSth();
    }
};
```

这里在调用doSth的时候会产生二义性，因为这里的doSth可能是Tem1全特化的一个版本中的函数，这里编译器无法知道调用的doSth是哪个版本中的Tem1，而且Tem1在特化的时候可能没有这个函数，所以编译器在这里不会让该程序通过。

改良方法：

1. 将doSth 改为 Tem1<T>::doSth;
2. 使用this->doSth直接通过函数指针来调用

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918095405171.png" alt="image-20220918095405171" style="zoom:67%;" />

通过直接将doSth加上一般化特性，这样可以保证以后全特化Tem1都必须加上这个一般性的特性，不然会获得一个错误。

![image-20220918095631070](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918095631070.png)

## template膨胀

![image-20220918102503024](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918102503024.png)

![image-20220918102516383](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918102516383.png)

这样当使用Tem2<int , 10>,Tem2<int , 5>会在编译器生成两段一致的函数代码，这样会导致代码膨胀也就是安装包体积较大

## 模板拷贝构造函数

![image-20220918111040746](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918111040746.png)

所以这里需要普通的拷贝赋值调用泛化拷贝赋值

## 编写自己的iterator_traits

```c++
class Iter {
public:
    typedef bidirectional_iterator_tag iterator_category;
};
template<typename T>
struct iterator_traits<Iter<T>>{
    typedef typename Iter<T>::iterator_category iterator_category;
};
int main() {
    list<int> a;
   listInt(iterator_traits<Iter<int>>::iterator_category());
}
```

这里trait通过获取迭代器模板中的iterator_category来获得类型，并且各个类型对应的基类呈继承关系。因为这里可以直接使用对应的trait得到的迭代器类型通过重载来在编译期就可以判断需要执行的函数。

这里通过全特化向traits中添加对应的迭代器种类。

```c++
void listInt( forward_iterator_tag) {
    cout << "forward";
};
void listInt( bidirectional_iterator_tag) {
    cout << "bid";
};
void listInt(input_iterator_tag) {
    cout << "intput";
};
```

![image-20220918172620997](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220918172620997.png)