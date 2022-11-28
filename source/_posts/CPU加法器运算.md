---
title: CPU加法器运算
date: 2022-11-17 19:18:45

categories:
- 计算机基础
tags:
- CPU
---
## 浮点数

关于偏置编码的使用，为啥是127而不是128

以及用移码来表示更小的数字（都是为了简化两个数字的比较）

便于浮点数比大小。

如果阶码（指数）也用补码来表示，就会使得一个浮点数中出现两个符号位：浮点数自身的和浮点数指数部分的。这样的结果是，在比较两个浮点数大小时，无法像比较整数时一样使用简单的无逻辑的二进制比较。

故而浮点数的指数部分采用了移码（无符号整数）来表示。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610203132513.png" alt="image-20220610203132513" style="zoom: 50%;" />

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220610203322004.png" alt="image-20220610203322004" style="zoom: 50%;" />

字和字长

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220611104256015.png" alt="image-20220611104256015" style="zoom:50%;" />

### 浮点数加减运算

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614142528131.png" alt="image-20220614142528131" style="zoom: 33%;" />



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614143829838.png" alt="image-20220614143829838" style="zoom: 33%;" />

### 加减运算要点

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614144753384.png" alt="image-20220614144753384" style="zoom: 33%;" />

### 浮点数运算异常状况

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614142824524.png" alt="image-20220614142824524" style="zoom: 50%;" />

所以以上的结论就可以用来解决以下问题

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614143039476.png" alt="image-20220614143039476" style="zoom:33%;" />

当浮点数除0的时候，可以使用阶码全1表示无穷大

而int除0时，并没有任何值可以表示无穷大，因此会直接抛出异常处理

### 浮点数精度提升

需要添加一个保护位和一个舍入位来保证浮点数的精度

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614145756311.png" alt="image-20220614145756311" style="zoom:50%;" />

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614145732394.png" alt="image-20220614145732394" style="zoom:50%;" />

结果为偶数意思就是让黑色数字的最后一位必须为0。

所以当一个特别大的浮点数加一个很小的数时，就会把被加数加入舍入位然后舍去为0，因此大小还是等于原来的数，这也就是我们平时所说的大数吃小数。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614153717514.png" alt="image-20220614153717514" style="zoom: 50%;" />

蓝色部分正好等于0.1，可以把蓝色位置移动到.的位置，正好前移20位，移动后的值就是0.1，因此这里就等于0.1 * 2的-20次方。

### 浮点数转换

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220614150457253.png" alt="image-20220614150457253" style="zoom:33%;" />

float只能精确表示最大为24位有效整数，而int为32位

## 补码运算规则

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612182042297.png" alt="image-20220612182042297" style="zoom:50%;" />



<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612182219205.png" alt="image-20220612182219205" style="zoom: 50%;" />

所以说假设最大值为16的话，此时-6的补码就是10，因为方便做加法才这样设定的，实际上为16+ -6 + 8 mod 16就等于2

此时-6和8的和就可以通过求其补码再相加的方式求出，而其补码也就是16 + -6实际上本质就是求-6的对立面也就是10,我们知道-6的二进制为10，为1010因此就为-6的补码，为了方便和正数做运算才遮掩这个i转换的。

而实际上注意 6+10也就等于16，此时就不需要再在意符号，直接用正数6来算出其对应补码10，通过0110 + x = 10000，故这里的x也就等于6取个反再+1，所以补码本质上就是其整数取反再+1。

![image-20220612182051963](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612182051963.png)

通过取其整数再取反+1，此时其-B的补码和B的取反+1和

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612183720922.png" alt="image-20220612183720922" style="zoom:67%;" />

**说白了计算机只存正数，如果是负数也要帮你转换方便做加法运算的方式存进去**，你存3对应0010可以补成1110，然后在实现8-3时，也就是8 + -3的补码，就可以让8-1110，进而会超出进而求模得到结果5，也就是通过对称的方式求补码。

## ALU

ALU则是连续n未加法器，通过OF来看是否发生进位进而丢给下一个的单位加法器的cin处理，cin为输入分别于对应需要+的两个指亦或然后做或处理

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612190456161.png" alt="image-20220612190456161" style="zoom: 50%;" />

用以上的图片来判断是否发生进位，即Cout是否是1，如果其中任意一个条件满足则Cout就是1

记住取反一定是异或一个FF操作，因为异或你给1异或1结果为0，你给0亦或1结果为1，如果选择00异或，则0异或0为0，一异或1为1，结果就不变了。

截断操作可以通过左移右移来实现

## 移位运算

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612193256819.png" alt="image-20220612193256819" style="zoom: 50%;" />

**拓展运算**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612193627397.png" alt="image-20220612193627397" style="zoom:67%;" />

**截断运算**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612193739784.png" alt="image-20220612193739784" style="zoom:50%;" />

### 加法标志位

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612211848323.png" alt="image-20220612211848323" style="zoom: 50%;" />

**判断溢出条件**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220612212132234.png" alt="image-20220612212132234" style="zoom: 50%;" />

无符号只看CF位，带符号只看带符号位

同时两个异号数相加一定不会发生溢出，只有同号相加才会发生溢出	

## 乘积运算

### 乘积计算

![image-20220613122020350](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613122020350.png)

本质上就是对应十进制乘法运算，每一位0101都乘以0101然后再相加

发生了溢出现象，对应起始就是各位的乘积

### 乘积溢出

**程序员判断溢出**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613121638300.png" alt="image-20220613121638300" style="zoom:50%;" />

通过做除法就可以判断得到的值是否是逾期

**编译器判断溢出**

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613121708702.png" alt="image-20220613121708702" style="zoom:50%;" />

通过判断高n位是否为全0或者全一就可知道是否溢出，但是还需要判断得到0或者1是否等于低n位的最高位。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613122632003.png" alt="image-20220613122632003" style="zoom:50%;" />

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613123130517.png" alt="image-20220613123130517" style="zoom: 50%;" />

### 无溢出标志

没有标志位置可以像OF和CF一样来判断溢出，因为每一次溢出所得到的数都不是像加分一样固定就进一位，而是一次可以进很多位，，因此需要通过2n中的高n位来判断是否溢出，判断方法如上。

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613123321055.png" alt="image-20220613123321055" style="zoom: 50%;" />

### 乘法缓冲区攻击

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613123757219.png" alt="image-20220613123757219" style="zoom: 50%;" />

通过溢出算法来使得分配的内存较小，但是可以改写未被分配的内存，因此可以植入自己编写的代码

### 编译器性能优化乘法

<img src="https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220613123925168.png" alt="image-20220613123925168" style="zoom: 50%;" />



