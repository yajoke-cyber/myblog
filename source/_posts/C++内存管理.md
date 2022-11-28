---
title: C++内存管理
date: 2022-11-27 19:15:06
categories:
- C++
tags:
- 内存管理
---
# Primitives

## new关键字讲解

![image-20220511153437895](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511153437895.png)

这里其实当我们每次都在使用Complex(1,2)在调用的时候（不管是在堆区还是栈区），第一位参数放置的默认都是this指针，但是实际上这个分配的对象首地址也就是指针指向的地址并不是我们去指定的，而且操作系统去分配一个可用的指针然后填进去。而this一般指的是调用者。

在这里使用**pc->构造函数**的方法则可以直接把我们创建的指针放置进去，从而指针的地址就正好为对象的首地址。

而在不同的编译器版本下处理逻辑也不同，在GUNC中是不允许主动调用构造函数，也就是pc->构造函数的方式是会报错的，也包括直接命名空间调用A::A()，都是报错行为。而在VC6中则是被允许的行为。

### 重载opeartor new

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220911102856658.png" alt="image-20220911102856658" style="zoom:50%;" />

优先重载类内部的operator new，若无才会重载全局opeator new。但是也可以通过::new 的方式强制重载全局opeator new

### Array New

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901155225414.png" alt="image-20220901155225414" style="zoom:50%;" />

对应delete[]才会去删除一块连续的地址空间

![image-20221128192340780](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20221128192340780.png)

使用array new可以在内存前面添加一个数字表示需要delete的次数，如果直接delete的话在debug模式就会报错，是因为在delete的时候多包括了一个3的标记数字内存，进而使得程序无法解释这段内存从而出现断言报错，如果是delete[]的话则没有问题。

### placement new

总的来说作用相当于

以下为测试程序

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901160729025.png" alt="image-20220901160729025" style="zoom:50%;" />

通过placement new可以 使用new(p) ctor()的方式直接调用对应指针上类的构造函数，这才是所有版本都适应的主动调用构造函数的方式。

**本质**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901164340175.png" alt="image-20220901164340175" style="zoom:50%;" />

通过placement new 我们可以将一个指针传入，就可以在该指针指向的地址中创建对象，但是使用的指针区域可以是栈区也可以是堆区，所以要注意**placement new 并不分配内存**，而是将分配好的内存指针去调用对应的构造函数，只要一块内存调用了构造函数这块区域也会直接变成对应的对象存储起来，实现定地址的new。

### new依赖示意图

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901165350246.png" alt="image-20220901165350246" style="zoom:50%;" />

在调用expression的时候，第一行operator new其实就是单单调用了malloc，分配了内存但是并没有填充对象，使用下一行的placement new的时候才将分配出来的内存区域进行调用构造函数的操作来创建对象。

**class operator new**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901172643145.png" alt="image-20220901172643145" style="zoom:50%;" />

类内部new重载，因为当我们在使用new时，并没有对象会产生，因为重载往往是用于 a + b这种场景，所以这里必须使用static来使得没有对象也能调用重载函数，但是c++比较贴心，因为其应用场景的特殊性，因此程序员不加编译器也会去添加，所以不写static程序也是可以跑通的。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901184010764.png" alt="image-20220901184010764" style="zoom:50%;" />

在操作符前加::可以指定重载全局操作符而不会走优先的类内部重载 

### 重载placement new

注意所有new对应重载的第一个参数永远都是size_t类型的size，用来在malloc的时候获取需要申请的内存空间大小

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220901190321167.png" alt="image-20220901190321167" style="zoom:50%;" />



# Allocater

## 分配器allocator

上述操作在allocator之前出现，在allocator出现后就开始使用allocator容器来作为分配内存的方式

通过嵌入式指针的方式完成内存池

以上代码为free-lists节点的结构。我们知道内存池由链表连接在一起，如果每个节点都有额外指针，会导致资源的浪费。故使用union关键字，当**未分配时作obj视为指针，分配后视为对象**。

所以每一次获取到freestone都是上一次freestone指向的下一个可用区域

所谓内存池就是将原来同一个类下分散分布的对象统一放到一个数组中进行保存，此时所需要的cookie就是只有上下两个，如果没有用内存池进行管理的话那么new出来的对象数组每一个对象都会有自己的cookie，此时数组种对应的对象就是分散的，因此将对象进行集中管理统一发到一个池子中可以减少相当多的内存浪费，每一次都可以节省下两个上下cookie。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220903105127361.png" alt="image-20220903105127361" style="zoom:50%;" />

不懂可以看课程中Static allocater

allocater中本质上是维护了一个链表，其中每一块区域的引用都是之前对应的chunk的地址，其next对应chunk的下一个地址。因此我们可以通过allocater的使用来查看其是否chunk使用情况。freestore对应每一块内存区的

回收内存区就是把原先的

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220903125732322.png" alt="image-20220903125732322" style="zoom:50%;" />



### 内存池归还

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220903174515138.png" alt="image-20220903174515138" style="zoom:50%;" />

先将回收区域前四个字节的设置嵌入式指针为freestone的next的next，再将freestone的next设置为要归还的内存地址，就可以在链表中重新插入空闲的内存，并且按照顺序会继续形成一个串。

## 内存，栈管理

这里总共48个字节但是上面的cookie确是31=52字节，为什么呢？因为原因就是这里的一代表的是标志位，因为这里规定一块内存一定是16的倍数，所里不可能可以分配到1为也就是一个字节的1/8。

所以这里的1代表标志位，表示正在使用分配的内存（尚未归还），如果归还了这里就会直接变成30.

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511155328148.png" alt="image-20220511155328148" style="zoom:50%;" />

下图则是内存大小补全为16的倍数的图。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511155555425.png" alt="image-20220511155555425" style="zoom:50%;" />

只有function template才会进行类型推导

而class template则需要主动指定泛型

![image-20220511200413530](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220511200413530.png)

## 指定编译器预处理

![image-20220903153027507](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220903153027507.png)

通过设置new_handler可以处理当申请内存但是内存不足时需要触发的回调函数，在这里可以选择清除其他获得memory或者直接

终止程序abort(),exit();

通过 = default 或者 = delete可以指定当前编译器是否添加默认函数

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220903150442288.png" alt="image-20220903150442288" style="zoom:50%;" />

# G4.9Free_list全局内存分配

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904150032238.png" alt="image-20220904150032238" style="zoom:50%;" />

序号0对应字节大小为8的内存存放，而序号3则对应类大小为32字节的内存存放，并且第一次会调用malloc申请40个，另外20个作为战备池pool，可以被其他区域使用。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904151928771.png" alt="image-20220904151928771" style="zoom:50%;" />

当申请单位为64字节的内存时，会将先前的pool数据进行重新切割为10个，并且将其指向改为pool的地址。此时因为要求划分的区域为1~20，所以pool会被全部用完此时pool处理后大小为0；

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904152149244.png" alt="image-20220904152149244" style="zoom:50%;" />

因为pool大小为0因此会重新申请pool，大小为2000，其中2000 = 原先总申请量1280 /16后经过一系列操作剩余的值也就是2000.

但是当出现碎片化情况时，处理逻辑就会改变

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904153742354.png" alt="image-20220904153742354" style="zoom:50%;" />

当pool大小为80时，此时申请104大小的内存，其单个碎片都会大于pool整体大小，因此这里将pool直接链接但数组处理，也就是9号，直接当为碎片使用。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904160354596.png" alt="image-20220904160354596" style="zoom:50%;" />

当申请内存不足当前8号使用时，会向最近的9号以及后续索取内存，如果9号索取完了就索取10号，并且每次索取后会有剩余空间，比如索取10号一块会剩余16字节无用，就将16字节作为pool

总结：当存在pool时候，直接使用pool作为链表指向然后存储，如果pool小于需要的要求，则会借其他数组中的数据来填充pool，否则需要重新开辟20个选择单元大小的数组，并且再用算法计算出pool需要的数量，如果剩余的空间不够20个开辟，则会直接触发new_handler也就是此时出现堆区空间不够的问题。

问题就出现了，还剩余312个字节，也可以够120字节去存放，所以这里可以对120需要申请的一串数组不断除2，知道得到的值小于312字节时，即是我们需要开辟的内存空间并分成对应块挂载120字节（14号位置）上，但是c++没做，源码说为了预防多线程下的灾难性问题？不过这里是G2.9的原始内存池代码，是对以往版本的讲解，后续会有更好的优化。

### 源码解析

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904182205586.png" alt="image-20220904182205586" style="zoom:50%;" />

开辟内存大于要求时直接使用malloc，不适用内存池来管理。

其中删除内存直接通过链表指向指向0的方式删除，没用使用free，是因为这里呈现的涉及思想就是固定大小的内存池，因此这里程序依然可以使用内存池中的存储，因为开辟的池子大小一直是固定的，而固定的malloc才会使得性能比较高，可以尽可能少去调用，牺牲了空间但是获取了时间上的高效

**Refill**

为实现数组中建立链表和实现内存切割分块的函数。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220904183733081.png" alt="image-20220904183733081" style="zoom:50%;" />

for从 1开始是因为第一个需要返回给用户，因此直接从第二个开始做链接，再把free_link_list指向第二个即可

这里每一次把指针都转型为char* 是为了方便对指针做运算，做了转型后 + size 则指针可以直接跨越size个字节，在跨域以后再把指针转向为union的obj* 从而继续设置其next的值。

**测试结果**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220905110801041.png" alt="image-20220905110801041" style="zoom:50%;" />

相比于右边的1百万次，左边使用内存池只malloc了122次，相当于节省了(一百万-122 )*8 的字节量，可以说节省了大量内存。

# VC6 malloc内存分配

vc在进入c++ main主程序时会进行一些初始化操作，这里的指向为栈的执行顺序，所以为倒序执行。以下讲解c++程序执行中第一块内存是如何申请的过程如下（第一块由隐藏的系统初始化申请，以下为流程讲述）

## 源码讲解

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220905134042638.png" alt="image-20220905134042638" style="zoom:50%;" />

vc10直接将对应的选择函数包装到了HeapAlloc内部，因此这里直接使用操作系统开辟内存的HeapAlloc方法，里面已经涉及到了windowsHeap管理，也就是让操作系统去管理。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220905135615420.png" alt="image-20220905135615420" style="zoom:50%;" />

在调用heap_init函数的时候，主要负责的就是讲**HeapCreate**得到的内存初始是4096个字节作为开辟好的内存区域，然后讲16个header作为连续数组填入开辟的连续空间中来方便管理，大致结构如下，作为代码中需要的内存开辟时使用。

然后调用**HeapAlloc**则是向直接创建的内存池中要内存，此时第三个参数表示大小就会返回对的的连续内存的地址，这里用__shb_pHeaderList做接收，Header结构如下。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220905135928841.png" alt="image-20220905135928841" style="zoom:50%;" />

**_ioinit()**

此时进入_ioinit函数用来处理io的内存管理初始化

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220906171315289.png" alt="image-20220906171315289" style="zoom:50%;" />

右图第4行是对应函数行号，也就是进入函数的代码地址

右图第5行结构体中的nDataSize是开辟内存数据大小

在调用_heap_alloc_dbg的时候会进入函数，分配出右边内存大小的struct，用于dbg也就是调试使用，其中的gap用来检查是否越界读取。用原先的nSize = 100h也就是256 + struct + 无人区大小既可得到blockSize，然后走_heap_alloc_base函数

代码继续向下走（在heap_alloc_dbg函数中）

**heap_alloc_dbg**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220906171636409.png" alt="image-20220906171636409" style="zoom: 50%;" />

其中在io初始化的过程中malloc的字段本身全部被用struct包装起来，然后用链表的方式链接起来作为SBH进行管理，只有在调试模式下面才会调用对应的_dbg为结尾的函数，才能时刻精确的查看内存对应的值，才可以时时刻刻追踪内存对应变化。

**heap_alloc_base**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220905133701172.png" alt="image-20220905133701172" style="zoom:50%;" />



这里通过_heap_alloc_base来获得所申请的内存大小，当小于门槛1016字节（1024 - 8，8为cookie大小，因为这里还没添加cookie所以需要减去cookie的大小）的时候采用SBH heap函数处理，否则采用HeapAlloc函数处理，类似于之前new 申请的内存是否走内存池还是直接走malloc。

**_sbh_alloc_block()**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220906190922390.png" alt="image-20220906190922390" style="zoom:50%;" />

进入sbh_alloc函数时，在调试模式假设申请三个字节下实际拿到的内存是3+struct大小 + 2 * cookie再补齐到16的倍数的字节大小

图示需要130个内存大小作为存储，因为补齐为16的倍数，所以这里为16进制最右位一定是0，但这里使用131原因是为了标志**内存正在使用还没有被销毁**，当内存被归还时cookie的值就会从131改为130。然后继续调用_sbh_alloc_new_region()

**_sbh_alloc_new_region()**

![image-20221128192454058](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20221128192454058.png)

调用new_region的时候会new 一个tagRegion的对象，这个对象内部有一个黄色框的存储，里面有32组和下面的32个Group，其中每一个Group里面是长度为64的链表数组，所以new一个对象大概消耗了16KB的内存为了控制接下来要说的虚拟地址空间，下一步才要真正处理向操作系统申请空间了。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220906195413085.png" alt="image-20220906195413085" style="zoom:50%;" />

右边的虚拟内存每一块是32k，然后32k又被细分为8个page，所以每一个page就是4KB，其中Group0就指向了第一块虚拟内存，由64个链表的存储部分去做存储，所以这里sbh就有64个链表来管理8个page，相当于挖一大块然后分割成64个小块然后串起来。只有需要的时候才会开始挖，这里只用了第一块所以只需要挖第一块，当第一块用完也就是链表指向最后一个都不为空的时候，此时再挖第二块。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220906214613319.png" alt="image-20220906214613319" style="zoom:50%;" />

使用4080是因为一个page是4088不是16的倍数所以需要删去一点作为补齐,在这里GROUP使用的是tagEntry的指向，默认指向本身的元素，但是这里的listHead本身只有两个字节，本着Next指针指向自己的前一个int的地址，而Prev则指向寻找前两个int的地址，所以两个指针默认都会指向上一个ListHead的第二块内存，就出现了GRUOP所有listHead都是向上指本身前一个的现象。

**page具体切割过程<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220907121832630.png" alt="image-20220907121832630" style="zoom:50%;" />

每一个page对应4088也就是ff0 - 130会剩下ec0大小为剩余尚未使用的内存空间。

在这里会将debug模式下开辟的内存块写入，其中返回的指针注意是004d0ed0（红色文字），和上面131之间相差不是连续的字节，说明其实返回的地址就是内存块中nsize区域的地址，做了减运算以后得到了内存的真实地址然后返回。其中内存块每一项对应，

0042ee08对应代码文件名,00081对应代码行数, 100对应nsize大小，2对应开辟内存块为CRT_BLOCK种类，而我们自己开辟的则属于NORMAL_BLOCK对应数字为1。

然后开辟的内存空间大小为130，此时用130h / 16 = 19，19 - 1= 18即为得到存储空间对应的链表指向，此时链表中index为18的指针则会指向这一块内存，原理与G4.9全局内存管理部分几乎一致。

## 图示申请概览

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220907130616602.png" alt="image-20220907130616602" style="zoom:50%;" />

1. 首先在ioinit的是hi偶会调用HeapAlloc函数，意思就是开辟堆区，会开辟16个header，在这里只会把第一个header丢出作为使用

2. 第一个header在开辟完成后对应VirtualAlloc函数，会在操作系统上**预留**一块内存用于申请，但是此时还没有走申请操作

3. 等到继续走HeapAlloc函数的时候，此时传入参数表示要开辟**REGION**，也就是上图黄色区域，其中对应header指针则会指向黄色部分的地址，黄色区域会开辟出32个GROUP，每一个GROUP都是64对链表，并且在GROUP中，会把对应申请的内存大小和INDEX相对应，比如申请16字节大小那么对应INDEX就是0，

   **REGION解析**

   REGION 黄色上面的区域32个64位二进制数组构成，第一行中最后一个bit是1表示GROUP 0中最后一个链表是有指向的。当为0时表示当前INDEX的链表指向为默认值，在其上没有挂载空间可以存放内存。

4. 在初始化以后就会走VirtualAlloc，参数中的commit表示是确认开辟，因此会真的从内存中开辟对应1mb并且分为32块

5. 从分出来的32块中取出一块并且再划分为8个page，每一个page对应大小为4088字节，为了凑到16的倍数把8字节作为保留，其余则直接用于存储，然后8个page用双链表的结构串起来，并且GROUP对应的最后一个链表对中的Next指向Page1，preV指向Page8。

6. 此时ioinit需要申请的内存也就是130h， 带有dbg struct的内存块申请，并且将其中对应真实存储数据的内存地址返回，130h / 16 = 19，所以此时GROUP中INDEX =18.但是查看REGION**发现数组第18位位0**，因此并没有挂载内存块在上面，所以就会向右边查询到最后一位发现是1，找到链表中最后的链表指向的page1，并把内存块存入page1中。page1其余的空间为ff0 - 130 = ec0，作为剩余空间的大小（每一次申请内存时链表的第一位enites就会+1，释放时-1，知道enties为0时表示无内存申请，此时整一块内存空间就会被直接释放，也就是下述过程）。

   **补充：**

   在page1中每一块内存都是一个内嵌式指针，当这块区域未被申请时，指针就会指向下一个区域，而在申请区域中的指针就不管了，任由使用者覆盖，但是在收回后就要对其重新赋值为嵌入式指针，嵌入式指针使用原理与G4.9部分几乎一样。

## 图示释放概览

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220907153917982.png" alt="image-20220907153917982" style="zoom:50%;" />

当内存释放的时候做的事情就简单多了，首先是Entites- 1，

其次把对应管理区的241h改为了240h，

并将其前八个字节改为两个嵌入式指针，

然后根据其大小把其指定到对应链表管理，例如240h / 16 = 36 所以应该有index为35去管理，此时将index为35的指针拉过去，然后把对应在Entry数组上对应的位数+1，表示该节点下面有可以存储的内存块数量，**只有当回收的时候才会做拉取内存块链表的操作**。

链表最上面的节点意思表示正在管理的内存块数量，因此哪里对于的GROUP有一块内存块可以申请出去也不会算在管理的内存块数量上，而在REGION标志的数组中的0 和1则直接表示该节点下是否有未分配的空闲内存以便于使用。

### 链表的使用

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220914172912298.png" alt="image-20220914172912298" style="zoom:67%;" />

当分配b0大小时，也就是index 11 此时向数组中寻找发现对应位数为0，因此去向右查询数字大于1，找到240对应index 35数字为1，此时选择占用其中的一部分并且将剩余部分也就是240 - b0 = 190得到一块新的内存大小，算出index  = 24，此时将数组24的index 的值改为1，因为为分配操作所以Entites+1，将剩余的控件190把对应对应GROUP index也就是24的链表拉过去，然后判断内存35剩余中能否继续分配来觉得是否要把数组对应标志是0还是1。因为原先index的35中的链表中对应index链表中取走了一块，已经没有240的可用内存块，所以直接将REGION数组中35index的值由1改为0，通过该方式就可以保证对应数组index下的可用内存数量和值相对应，不足便向高位取。

当旧的GROUP全部用完时，此时就会开辟一个新的GROUP1，并将对应上图的红色大0改为1，对应当前使用的GROUP，然后重新开辟虚拟内存.... 重复之前的流程。

### 区块合并

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220907163127553.png" alt="image-20220907163127553" style="zoom:50%;" />

利用当前块的上cookie和前一个块的下cookie做检测，当两个最后一位都为0时，则直接将两块进行合并，不断重复直至无法合并为止，然后将对应块挂到指定链表上。

这里的黄色区域意思为限制，当某一块想要读取下一块内存发现初始值是-1的时候，此时表面下面的内存不可用，进而会走另外一个流程。

## 回收指针p

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908143850990.png" alt="image-20220908143850990" style="zoom:50%;" />

1. 通过指针p的值就可以回收到对应的内存区域地址

2. 通过不断的遍历HeaderList，通过headerList的两个指针可以获取右侧虚拟内存的头尾指针来判断是否在对应header中，不在就再遍历下一个节点

3. 当找到时，通过p可以知道具体处于第几个Group管理，知道对应的Group index = p - &GroupHeader/ 右侧虚拟内存单位大小（32kb

   32 kb = 1mb / 32)

4. 再通过其指针本来是指向对应的内存，通过向上偏移固定的大小就可以得到cookie的值，也就是是其内存大小，可以得出对应的freeList index，然后对对应的内存做处理（如上述释放）

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908154100590.png" alt="image-20220908154100590" style="zoom:50%;" />

实际上再分配的时候还需要处理defer指针，当真的记录开辟区数量为0的时候，此时并不急着回收开辟的区域，而是用指针保留起来作为预留，等到下一个全回收再次触发的时候再对应开辟的一个group全部回收掉。这也可以解释为什么VC内存分配要做的这么繁琐的原因，因为当切成对应的小块时，每一次回收就是小块的回收，可以保持内存能够维持一个稳定而不用大幅度变动。

## VC6总结

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908155826205.png" alt="image-20220908155826205" style="zoom:50%;" />

由上而下的理解，此时要把allocater和malloc结合考虑，gc 和vc实际上都是由类似的一套内存管理系统来管理，所以分配过程都是类似的。这样一套叠一套并不是一个很好的选择，所以在VC10中就删去了malloc对内存的处理部分。

# Loki Allocator

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908171532572.png" alt="image-20220908171532572" style="zoom:50%;" />

Chunk类是FixedAllocator的内置类，是仅为FixedAllocator提供服务的类

## Chunk::Allocate

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908170714349.png" alt="image-20220908170714349" style="zoom:50%;" />

在这里对应pData_指向一块连续内存，数字表明当前最先序号，64表示可用区块数量。其中在对应最先序号的指针分别向两边扩散，走势相反，为了方便当最高优先级被开辟时可以直接通过拉Pheader的指向到最先序号位置的下一个，并将链表指向的下一个作为下一个最先选择元素，从而使得链表继续连通，如上图示。

## Chunk::Dellocate

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220908171223167.png" alt="image-20220908171223167" style="zoom:50%;" />

回收元素可以根据指针的位置找到对应的chunk，然后求出块序号，再重新插入进链表中（头插法），并且将最高优先块序号改为插入的内存序号。

## FixedAllocator::Allocate

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220909150838087.png" alt="image-20220909150838087" style="zoom:50%;" />

在这里其实原理很简单，通过不断的遍历chunk struct 数组，得到其对应值查看是否有无空闲区块可用，当没有的时候就会向数组里面重新推入一个元素，但是注意这里是采用vector容器，所以在push_back的时候可能会发生**vector的拷贝操作**，因此需要把原来的指向重新设置，也就是deallocChunk被设置的原因，不能让他指向vector转移之前也就是被收回的内存区域。

在这里分配的时候没有带new，就是可以直接进行内存拷贝而不会占用cookie。

## FixedAllocator::Deallocate

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220909151638210.png" alt="image-20220909151638210" style="zoom:50%;" />

这里需要对deallocChunk进行一个检测，以防它指向的值仍然是上拷贝之前的内存区域，检测完毕后再进行拷贝操作。在这里p实际上是chunk指向数组的内存地址，而deallocChunk则是需要回收的chunk，需要知道当前执行deallocChunk的指向是哪一个，以便于执行上述的操作。deallocChunk是成员变量，是静态成员成员，是被共享的。

VicinityFind本质上其实就是迭代查找，不过是一次for循环有两个指针分别从两边开始查找。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220909154731952.png" alt="image-20220909154731952" style="zoom:50%;" />

每一次全回收都把chunk放到尾部（对调），当尾部已经出现全回收的时候此时则直接将尾部free，然后自己和尾部对调，保持延迟回收的特性。

## 最上层的allocator

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220909155857908.png" alt="image-20220909155857908" style="zoom:50%;" />

本质上就是将对应的Chunk排成数组并分段，然后方便去分组查找对应的chunk数据。其实现细节和内存管理无关。

## Loki_allocator总结

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220909155551683.png" alt="image-20220909155551683" style="zoom:50%;" />

所以在allocator这么底层的容器中就不要使用vector了，自己写一个和vector类似的数组就行，并且如果不替代分配全局也是没有问题的，因为vector默认使用的是标准分配器。

# Other Issues

## bitmap_allocator

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220911094542632.png" alt="image-20220911094542632" style="zoom:50%;" />

用bitmap来标志内存的使用情况，用来处理同一类实例对象的分配，同一种对象只会被分配到同一块内存空间中，并且每次开辟新的blocks都是以指数级的方式来开辟。

# const

![image-20220911101715599](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220911101715599.png)

const只能加到成员函数上

# 大总结

调用new class的时候调用的是new表达式，而**new表达式会被拆分为两步**

1. 调用::operator new重载函数，其内部调用了malloc 申请内存并且将申请的内存地址返回。
2. 使用指针接受内存地址的值，然后通过指针来调用构造函数，把类中的数据填入对于内存区域。

类似的**operator表达式会被拆分为两步**

1. 通过填入对象存储的指针，通过指针调用类中的析构函数
2. 调用::operator delete，其内部又调用了free，将内存返还操作系统