---
title: C++面向对象程序设计
date: 2022-11-17 19:13:45
categories:
- C++
tags:
- 面向对象
---
# C++特性讲解

![image-20220510220258251](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220510220258251.png)

```c++
#include<iostream>
//尖括号一般指第三方库
#include "complex.h"
//双引号就是自定义头文件
```



![image-20220510220752738](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220510220752738.png)

防御式声明，为了让改头文件可以多次引用，相当于if判断，如果被引用过了就直接返回一个空。因为c++最后编译得到的就是一个大文件





如果class里面带指针那么拷贝构造一定不能使用编译器给你默认配置的

string在构建的时候需要自我赋值

![image-20220511151406599](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511151406599.png)

因为这里如果不写if判断的话，那么到时候两个对象指向的m_data都是内存中同一块区域中的数据，那么期中一个在清除对应数据后，另一个想要再去读取就会直接触发程序错误！



## new关键字讲解

![image-20220511153437895](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511153437895.png)

这里其实当我们每次都在使用Complex(1,2)在调用的时候（不管是在堆区还是栈区），第一位参数放置的默认都是this指针，但是实际上这个分配的对象首地址也就是指针指向的地址并不是我们去指定的，而且操作系统去分配一个可用的指针然后填进去。而this一般指的是调用者。

在这里使用**pc->构造函数**的方法则可以直接把我们创建的指针放置进去，从而指针的地址就正好为对象的首地址。

而在不同的编译器版本下处理逻辑也不同，在GUNC中是不允许主动调用构造函数，也就是pc->构造函数的方式是会报错的，也包括直接命名空间调用A::A()，都是报错行为。而在VC6中则是被允许的行为。

## 内存，栈管理

这里总共48个字节但是上面的cookie确是31=52字节，为什么呢？因为原因就是这里的一代表的是标志位，因为这里规定一块内存一定是16的倍数，所里不可能可以分配到1为也就是一个字节的八分之一。

所以这里的1代表标志位，表示正在使用分配的内存（尚未归还），如果归还了这里就会直接变成30.

![image-20220511155328148](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511155328148.png)

下图则是内存大小补全为16的倍数的图。

![image-20220511155555425](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511155555425.png)

只有function template才会进行类型推导

而class template则需要主动指定泛型

![image-20220511200413530](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511200413530.png)

## 单例结构

可以设置一个所有实例都共享的数据既可用static。

下图可以直接通过函数的方式使用，当函数被调用的时候才使用static关键字，没被调用就不使用。

![image-20220511200939528](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511200939528.png)

## 组合，委托

委托其实本质是就是**组合+引用**

## 构造函数私有化

为了就是让static得到的静态类可以直接调用私有定义的构造函数

而这里就是原型的体现，通过每继承得到的子类都会静态生成一个对象，然后会调用自己的私有化构造函数，进而向副本的prototypes中添加子类的实例对象。然后父类就可以通过prototypes来访问子类创建的对象并且调用其身上的方法，就是原型链继承的设计模型在里面，为了让父类知道子类的存在！

![image-20220512150154616](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220512150154616.png)



源码部分

![image-20220512150818337](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220512150818337.png)

其实这里需要在static之后在class外部再去定义一下，这样子才能真正给到内存



在这里clone会返回一个实例对象的copy，但是我们想要拷贝出来的这个样本会在隐式增加一下变量count，所以需要放在protected里面。因为不想要被其他人调用，只能由我设计者决定。然后在调用的时候用一个int 作区分但是不去使用。就可以返回一个_id被定义的clone出来的对象。所有实例对象实际上都是static出来对象那一份拷贝！类似于js的原型链，通过拷贝来实现属性传递。

![image-20220512151549570](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220512151549570.png)

## 转换函数

![image-20220516183514916](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220516183514916.png)

通过名称直接设定为double可以直接省略double的返回类型限定，当这个对象的实例触发强行转为double时，就睡自动调用这个运算符重载，然后直接返回分子和分母的除

```
Fraction a(3,5)
double d = 4 + a;//4.6
```

## 特化和泛化

### 特化

通过绑定特定类型的T当T为特定类型的时候就会直接走复合bool模板的vector类

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517170735030.png" alt="image-20220517170735030" style="zoom:33%;" />



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220524105323251.png" alt="image-20220524105323251" style="zoom: 33%;" />



![image-20220524105541855](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220524105541855.png)

若出现template<>表明就是需要特化的意思

通过相同的class名但是指定不同的类型来实现特化

### 偏特化

通过相同的class名称，但是对应不同的约束，比如T和T*对应的约束，当传入指针的时候就会走T\*的class模板。

是特化中的特化也就是偏特化

可以用来缩小范围来使用

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517170553971.png" alt="image-20220517170553971" style="zoom: 50%;" />

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220524115526490.png" alt="image-20220524115526490" style="zoom: 50%;" />

偏特化就是当类型差不多的时候，用具体的细节来选择对应模板

比如左边就是bool和Alloc数量来决定class，而右边则是当相同类型时采用指针和const 来偏特化

## 模板模板参数

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517181141291.png" alt="image-20220517181141291" style="zoom:50%;" />

可以通过类似于函数传参数的方式，传入一个未定义具体类型的模糊模板类。

然后通过Container内部指定模板就可以确定模板类型，就是private里面的代码。

但是实例是错误的，原因是传入的list作为模板实际上需要传两个模板参数，但是在class具体实现里面只传入了一个故就会报错

typename和class在一层使用的时候是可以相互嵌套的

但是当出现嵌套使用的时候，此时typename的优势就会体现而出

[c++中typename和class的区别介绍](https://blog.csdn.net/qq_25252775/article/details/84586006)

### 区别场景

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517181903874.png" alt="image-20220517181903874" style="zoom:50%;" />

当我们使用默认模板的时候，此时传入的模板必须是清晰的，不模糊的，这就是和上述模板模板参数区别的场景，如果传一个模糊的模板编译器则会直接报错



### 类型推导auto

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517190542955.png" alt="image-20220517190542955" style="zoom:50%;" />

使得变量的类型直接通过推导而出

就是foreach差不多的意思

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220517191248127.png" alt="image-20220517191248127" style="zoom:50%;" />

## 引用reference

![image-20220518115404164](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220518115404164.png)

当使用reference的时候取地址也是和原来的变量相同，这个实际上是编译器营造的假象，本质上其实还是一个不同地址的指针，不过默认帮你做了取*的操作

## 重载关键字new

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220519201219077.png" alt="image-20220519201219077" style="zoom:50%;" />

当我们重载new的时候，第一个必须传入size_t类型的参数，这个就是需要new出来实例对象所需要分配的空间，因此会被默认传入，所以这里也必须要接受，不然会报错



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220519201507359.png" alt="image-20220519201507359" style="zoom:50%;" />

在这里第一个参数传入void*，表示函数重载。此函数只有当构造函数执行但是抛出异常的时候使用，因为构造函数执行时已经分配好内存空间了，所以在出现异常时应当将内存空间返回来避免内存泄漏

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220519203905272.png" style="zoom:50%;" />

这里就是通过staic声明一个函数，当调用的时候就会去寻找这个函数的定义，然后通过new的方式来进行内存大小的区分来开辟扩充区域。这里在真正开辟空间的时候就开辟了一个s的空间+扩容的string空间。

## C++函数模板

**方法一**

通过官方特性使用函数模板

```
template <typename T>//先声明模板参数 T
  typename T add(const T &num1, const T &num2)//定义模板函数，注意参数的类型
  {
      return num1 + num2;
  }
  
  int main() 
  {
      cout << add(1, 3) << endl;//模板的实例化：int add(const int &num1, const int &num2)
     cout << add(3.0, 9.9) << endl; //实例化：double add(const double &num1, const double &num2)
 return 0; 12 }
```

**方法二Functor实现**

通过操作符()重载实现函数模板

![image-20220531213910472](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220531213910472.png)

上面的identity struct就是通过继承对应的function从而获得了参数约束，即传入第一个参数和返回参数必须是同一个。

使用struct传递函数的好处在于 当使用template<class T,class Func>的模板时，可以直接把struct当作类型传入进去，调用就是直接调用Func类型的（）重载，可以直接使用Func();

# C++标准库

动态模板

```
void haha() {

};
//动态模板
template<typename T,typename... Types>
void haha(const T& arg,const Types&...args) {
    cout << arg <<endl;
    //sizeof(args)可以获取参数包里面参数的数量
    haha(args...);
    //通过递归来不断截取参数，最后一个参数得到的是一个空，故需要添加一个参数为空的函数作为重载！
};
haha(123,"nmd",22,22);
```

在这里我们看到的脚本语言的影子，通过模板函数就可以实现不对参数类型做限制并且可以使用...运算符来对不定数量的参数类型做收集

## STL六大部件

![image-20220519210031556](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220519210031556.png)

### Container



#### deque

![image-20220530163037743](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220530163037743.png)

通过连续数组存放buffer空间，通过迭代器造成连续空间存储假象

![image-20220530163109597](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220530163109597.png)

![image-20220530171019544](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220530171019544.png)

通过重载+运算符，当+的值小于buffersize时，此时就不移动iterator，如果+的值超过当前buffersize，则取差值然后直接跳转第offset个buffer，进而实现线性模拟。

这里的this指iterator类，并且[]的+号已经被之前重载过

#### Map

![image-20220603154714206](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220603154714206.png)

以下为Map[]部分源码，相比于insert，中括号做了判断，当Map里如果有则直接返回迭代器，如果没有就在最合适的位置插入以后返回插入位置的迭代器。最后再起对应迭代器的data部分的value给return出去。

##### HashTable

对取得hashcode对HashTable的长度取模运算，最后得到的余数即为value需要插入在hasharray中的位置。

碰到hash冲突的时候，会采用链表的方式解决。

下面为hash bucket的数组，每一次需要扩容都会让数组的下一位赋值给HashTable的长度M。

但是以下是2.9的远古版本，现在的bucket数量已经换了一种新方式

![image-20220604144130430](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220604144130430.png)

当插入的节点元素数量大于等于HashTable的vector长度时，此时HashTable就会扩容，此时一开始的HashTableM = 53就会扩容为

53*2 = 106 然后获取最接近104的素数 97 ，所以97即为新长度。

因此，原先hash中所有节点都会重新排列，例如hashcode为54即插入在数组1号位的元素，此时会改到为第54 + 1位，所以可以知道当hash在发送扩容的时候所造成的性能损耗是非常大。



### Hash



#### 自定义Hash函数

通过动态模板的方式不断剪切value，然后对共享的seed做多次处理，就可以得到一个非常乱的seed，即为hashcode。就可以实现一个hashfunction，本质上也是调用了hash的基本类型偏特化版本来实现。

万用hashfunciton的求和如下

![image-20220610113203400](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610113203400.png)

![image-20220610113221262](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610113221262.png)



![image-20220610113235120](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610113235120.png)

实现顺序为1>2->3->4，通过进入hash_combine不断对seed进行修改，通过将对应对象的基本属性，例如name为string，或者age为int放入，调用std:hash<string>修改seed和std:hash<int>继续修改seed，就可以得到最终的seed即hashcode一个不规则的值。

![image-20220610113101455](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610113101455.png)

如果传入的是string或者int。

就会走hash的string和int偏特化版本，会返回一个特定的hashcode。

### iterator

其中 a++ 和++a必须做明显区分的就是，a++返回的是一个copy值而++a返回的是reference。

意思就是int a = 1；

a++得到值返回的一个 1，而++a返回的则是a这个变量。

而在立即数上使用运算符是不被允许的，1++会报错；所以a++++会被编译器报错，而++++a则可以正常运行。

self operator++（int）；

表示后置++重载

self& operator++（）；

表示前置++运算符重载

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220524223610205.png" alt="image-20220524223610205" style="zoom:50%;" />



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220524223631632.png" alt="image-20220524223631632" style="zoom:50%;" />

这里就是 self tmp = *this并没有调用起运算符重载\*也就是因为首先被判断的是拷贝构造函数，因此\*this在这里就是成为了拷贝构造函数的参数使用，也就失去了运算符重载的环境，所以不会调用其运算符重载的函数。



#### iterator_traits

![image-20220525104410393](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220525104410393.png)

![image-20220525104750540](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220525104750540.png)

因为迭代器往往可能不是一个智能指针，而是一个原始指针，所以这里需要对智能指针直接使用而原始指针则需要给他添加特性。而c++是如何判断这个指针是智能指针还是原始指针呢，可以通过偏特化来实现。

这里就是通过把传进来的type传入iterator_traits来进行筛选，通过当传入的值指针就会进入<T*>的类型，进而直接把指针定义为valuetype。而如果传入的是除了指针以外的则会直接走默认，也就是class指针中的value_type类型来定义为新的value_type



### Functor（Function Object）

编写的functor如果没有继承binary_function就没有融入STL。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606162851992.png" alt="image-20220606162851992" style="zoom:50%;" />

继承的binary_function代码如下，实际size理论上为0，主要用来回答adapter的提问

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606163032977.png" alt="image-20220606163032977" style="zoom:50%;" />

以下就是

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606163910457.png" alt="image-20220606163910457" style="zoom:67%;" />

Functor虽然好用，但是class自带的template不支持泛型推导功能，因此如果需要实现自动推导泛型的功能，必须得额外封装一层模板函数层来自动推导其class得模板。

![image-20220607185007531](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220607185007531.png)

binder2nd作用是把第二个参数绑定到函数的第二个参数位置。

原理主要是，首先调用函数时会直接调用not1()，而会先调用not1中的参数函数，就会得到一个需要传入参数的functor，然后调用not1的时候，相当于直接调用这个functor(x)，就可以把实际参数传入。注意，括号中的函数首先会第一步执行！

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220607185039770.png" alt="image-20220607185039770" style="zoom: 67%;" />

下面时not1源码

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220607200500972.png" alt="image-20220607200500972" style="zoom:67%;" />

### Adapter

### Algorithms

#### Copy

用来copy两个迭代器之间的数据给目标变量。

以下为Copy具体的解析过程

![image-20220606113537736](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606113537736.png)

首先根据泛化，若得到的是指针，则指针利用指针的特性直接内存拷贝。

进入分支的特化部分则是打算判断迭代器是不是特殊的指针，比如array的迭代器本质上就是指针，因此此时就会走特化的，在特化中丢入Type Traits 类似于Iterator Traits返回对应类型来判断对应数据的构造函数是不是简单构造，意思是在构造函数除了初始化是否有其他额外操作，没有则为简单构造可以直接进行内存拷贝，如果有的话为复杂构造，则需要一个一个调用构造函数并拷贝到对应新位置。

如果是特殊的迭代器则进入下一个分支，然后根据特殊的迭代器中如果是RandomAcessIterator则走差值loop，即线性的存储结果可以直接依靠差值来获取迭代次数.

如果不是的话就会走上面的，每次循环都判断以下first == last来判断是否copy是否结束。

#### Count和find

![image-20220606151733831](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606151733831.png)

计算迭代器之间对应的等于value的数量，如果求两个迭代器的距离可以直接用distance()

![image-20220606151758689](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606151758689.png)

使用find不带的成员可以使用全局函数简单查找，而带find的都是复杂的数据结构，不能用简单的++或者是独特的数据结构下有自己的优化算法，因此不需要用全局的。

![image-20220606152019169](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220606152019169.png)

## istream_iterator

![image-20220609181622702](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220609181622702.png)

通过实例对象，每一次得到输入的value然后在++以后被赋值为其取值新的value，本身++重载会返回原来的迭代器对象，然后继续被++。

为了适配下述的copy，因此其源码需要重载++部分。

这里eos没有参数，因此为iostream的读取末端，

iit在初始化时会先读一次，为的就是适配copy在使用的时候会先读取迭代器的值。	

![image-20220609181919660](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220609181919660.png)

![image-20220609181825735](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220609181825735.png)

源码部分

![image-20220609182119818](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220609182119818.png)

## Bind参数绑定

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220608193000540.png" alt="image-20220608193000540" style="zoom:67%;" />

用来提前填好一个参数，等第二个来就可以直接调用剩余参数部分。

## const 模板

![image-20220603151802811](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220603151802811.png)

给与一个const类型的int，就可以实现对应类型的int值不能被修改，可以参考typescript的const。

## 函数形参简写

当我们所使用的这个参数仅仅用为重载而不需要使用到这个形参的时候，这时可以只传type而不用传递形参。

![image-20220605152856035](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220605152856035.png)

将得到的haha实例放入doSth中确实可以调用doSth函数，其实本质就是重载函数的简写。

![image-20220605154210363](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220605154210363.png)

## typename和typedef区别



![image-20220525103747470](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220525103747470.png)

在这里使用typename为了就是确认I中所对应的value_type应该对应的是模板类型，而不是静态成员，不然编译器在编译这个部分的时候就会产生二义性，不知道对应取出来的value_type是类型还是是变量，所以编译器就会等到具体执行才会确定类型，但这个举动就会导致后续确认这个value_type的类型的检查出问题。因此这里需要一个typename来强行指定这里的其实就是一个模板类型

[深入理解typedef和typename关键字](https://blog.csdn.net/yuan_hs_hf/article/details/8808708)

![image-20220605155601921](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220605155601921.png)

利用typeid的C++官方库，可以直接拿到对应的classname，打印的是在编译过程中产生的typename

## Tuple

源码讲解

![image-20220610152730806](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610152730806.png)

通过动态模板，通过递归来实现自下而上的继承链条，拿到第一个参数在声明以后将剩余的模板重新丢入然后继续切割，最后特化继承一个空泛型使得继承终止。

tail通过传入子类的本身然后转型为继承的父类，相当于子类对象类型收缩为父类，此时收缩后的head函数返回的也指的是父类的head。

## type traits（类型特性）

下述的type traits用来表示其是不是重要类（成员变量具有指针指向的类），如果是不重要类则在走算法的copy方式的时候，则直接通过memcpy的方式进行拷贝，如果为重要类的话，则每一个拷贝都是按流程创建对应的类，然后把对应的构造函数参数传入来拷贝类。

int类就为不重要类，因此编写一个特化版本指定其typdef为true，等使用拷贝算法的时候就会直接走memcpy。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611131118137.png" alt="image-20220611131118137" style="zoom: 50%;" />

其作用不仅仅是以上

有is_void类来判断是否是空

![image-20220611155450884](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611155450884.png)

当传入is_void时，如果传入的类型是void，其中会先通过remove_cv来去除类型的constant和validate属性，然后用处理后的类型如果是void则会走void的偏特化，就会继承true_type，此时该类下面就会存在命名空间true_type，使用并不会报错。

## cout

cout源码如下

![image-20220611163617023](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611163617023.png)

extern类似于c++的export，可以将变量到处给外部

给自定义的类的实例对象可以被cout使用时，需要添加全局函数。

例如以下的complex，直接添加不定义在类中的全局operator<<重载函数来解决自定义类的cout输出函数，此时第一个参数是a <<b 的a，第二参数为b

![image-20220611164236181](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611164236181.png)

## moveable节点



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611174147677.png" alt="image-20220611174147677" style="zoom: 67%;" />

总共插入三百万个元素，构造函数却调用了七百多万次，原因就是在vector在扩容元素拷贝的过程中又重新调用了构造函数，这也就造成了构造函数的次数远比插入节点数更大。

在元素拷贝时，当存在指针时，指针的指向节点的创建实际上会造成一定上的性能损耗，例如图下。

![image-20220611180717842](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611180717842.png)

但是实际上，拷贝后原先指向的节点是直接删除的，因此设计一种方式当拷贝发生时可以重新利用之前的指向，而不用创建新节点。

而 moveable节点是在扩容发生节点拷贝的情况下，其优势才会体现出来。例如链表的长度并不受限制，因此其长度本质上可以无限增长。

![image-20220611175309883](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611175309883.png)

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611175830196.png" alt="image-20220611175830196" style="zoom: 80%;" />

Mctor原理，首先其传入的参数是&&为引用的引用，通过传入其本身然后去的调用拷贝构造函数，将旧元素的指针赋值给生成的新元素的指针，然后再把旧节点的指向删除，因为做完拷贝动作以后会删除旧节点的指针，所以在析构函数中应该做判断，若指向为空就不调用delete。

使用案例如下

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611181118870.png" alt="image-20220611181118870" style="zoom: 67%;" />

因为movable会修改原来元素的指针指向，因此在调用move函数以后，原来的元素应该不能再被使用了，所以这里最后的方式是直接传入一个临时对象，即class()的方式来产生，这样使用完之后对象就会被立刻销毁。

move函数的本质并不是修改原先类型的拷贝构造函数，而是为参数一个特殊的&&的标识，当调用对应container的拷贝构造时，就会根据是&&还是&来选择走浅拷贝Mctor还是深拷贝ctor。

比如vector中的Mctor本质上就是拷贝了原先的三个指针，start，cur和end，因此走Mctor的拷贝效率可以近似于0s，而如果走传统拷贝构造进行拷贝，则需要一个memcpy，存在一定的时间消耗。

