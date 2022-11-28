---
title: 学习Android开发
date: 2022-11-28 20:25:04
categories:
- Dart
tags:
- Flutter
---
# React Native学习笔记

## ScrollView

`ScrollView`适合用来显示数量不多的滚动元素。放置在`ScrollView`中的所有组件都会被渲染，哪怕有些组件因为内容太长被挤出了屏幕外。如果你需要显示较长的滚动列表，那么应该使用功能差不多但性能更好的`FlatList`组件。下面我们来看看[如何使用长列表](https://reactnative.cn/docs/using-a-listview)。

### 长列表使用

`FlatList`组件用于显示一个垂直的滚动列表，其中的元素之间结构近似而仅数据不同。

`FlatList`更适于长列表数据，且元素个数可以增删。和[`ScrollView`](https://reactnative.cn/docs/using-a-scrollview)不同的是，`FlatList`并不立即渲染所有元素，而是优先渲染屏幕上可见的元素。

`FlatList`组件必须的两个属性是`data`和`renderItem`。`data`是列表的数据源，而`renderItem`则从数据源中逐个解析数据，然后返回一个设定好格式的组件来渲染。

如果要渲染的是一组需要分组的数据，也许还带有分组标签的，那么`SectionList`将是个不错的选择

### 样式列表



在Native中样式采用的是层叠样式，子盒子字体的样式会自动继承父盒子字体的样式，因此这里需要将使用的样式直接包装成一个组件

我们相信这种看起来不太舒服的给文本添加样式的方法反而会帮助我们生产更好的 App：

是一种继承式的使用

```html
包装成组件，然后组件应用样式
<View>
  <MyAppText>
    这个组件包含了一个默认的字体样式，用于整个应用的文本
  </MyAppText>
  <MyAppHeaderText>这个组件包含了用于标题的样式</MyAppHeaderText>
</View>


class MyAppHeaderText extends Component {
  render() {
    return (
      <MyAppText>
        <Text style={{ fontSize: 20 }}>
          {this.props.children}
        </Text>
      </MyAppText>
    );
  }
}

如果需要使用自定义的样式，可以直接向里面放一个有自己样式的Test节点


```

### 事件

使用onPress来触发点击事件

## VirtualizedList

[`FlatList`](https://www.react-native.cn/docs/flatlist)和[`SectionList`](https://www.react-native.cn/docs/sectionlist)的底层实现。FlatList 和 SectionList 使用起来更方便，同时也有相对更详细的文档。一般来说，仅当想获得比 FlatList 更高的灵活性（比如说在使用 immutable data 而不是 普通数组）的时候，你才应该考虑使用 VirtualizedList。

Vritualization 通过维护一个有限的渲染窗口（其中包含可见的元素），并将渲染窗口之外的元素全部用合适的定长空白空间代替的方式，极大的改善了内存消耗以及在有大量数据情况下的使用性能。这个渲染窗口能响应滚动行为。当一个元素离可视区太远时，它就有一个较低优先级；否则就获得一个较高的优先级。渲染窗口通过这种方式逐步渲染其中的元素（在进行了任何交互之后），以尽量减少出现空白区域的可能性。

Vritualization 可以使用函数的返回值来作为使用，因此返回的值的类型可以具有多样性，非常之灵活

# Flutter学习笔记

Stack用来层叠两个图层

为什么Flutter在创建stateWidget的时候build要放在state的地方呢

1.在build的时候需要依赖State中的变量

2.在Flutter的运行过程中，Widget是不断销毁和创建的，但是stateWidget中的state必须要保持

然后引入一个文件的时候会导入这个文件所有的dart文件，但是如果名称前面有_为前缀则为私有变量，是不会被外界变量所引入

![image-20220420095108773](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220420095108773.png)

icon的本质就是具有特殊形状的字体



Flutter为我们提供了一种相对布局的组件：Align，Align组件允许我们通过修改它的属性来调整子组件的位置，并且可以根据子组件的宽高来确定Align自身的的宽高。

- **alignment**

首先我们来看一下Align组件的alignment属性，我们可以通过它修改子组件的相对位置。废话不多说，上代码：

```text
Container(
  width: 300,
  height: 300,
  color: Colors.red,
  child: Align(
    alignment: Alignment.bottomRight,
    child: Text("12345678")
  ),
);
```

下为效果图

![img](https://pic1.zhimg.com/80/v2-d0da003762bd0985e98b26797b01ee90_720w.jpg)



## Align

`Align`组件可以调整子组件的位置，并且可以根据子组件的宽高来确定自身的宽高，定义如下：

```
Align({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor,
  this.heightFactor,
  Widget child,
})
复制代码
```

- `alignment`：需要一个`AlignmentGeometry`类型的值，表示子组件在父组件中的起始位置。`AlignmentGeometry`是一个抽象类，它有两个常用的子类：`Alignment`和`FractionalOffset`，我们将在下面的示例中详细介绍。
- `widthFactor`和`heightFactor`是用于确定`Align`组件本身宽高的属性；它们是两个缩放因子，会分别乘以子元素的宽、高，最终的结果就是`Align`组件的宽高。如果值为`null`，则`Align`组件的宽高将会占用尽可能多的空间。

Container中的Alignment源码讲解

这里使用Alignment（0，0）让子元素居中

如果父Container存在Alignment属性，则会让其子组件被包裹一个Align和wrapper，让Align继承父亲的宽高，并且对里面包裹的子元素做居中处理

如果父Container不存在Alignment属性，则会让Text宽高延申等于父亲的宽高但是Text本身的文字排版顺序并改变，还是左上角，因此这里的文本还是左上角处理

![image-20220420111107918](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220420111107918.png)

**注意点**如果Container里面直接包一个Text类的话，会将Text的宽高拉满

![image-20220420112010928](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220420112010928.png)

这里的Width意思是孩子宽度的多少倍，可以用来列表指定数量渲染

## Flex

![image-20220420134404900](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220420134404900.png)

![image-20220420134415408](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220420134415408.png)

## ListWidget

默认是全部加载ListView中的每一个item

ListView和GridView 都拥有一个Builder方法，然后其builder返回的child只有在进入视窗时才会被加载，性能较高，可根据应用场景使用

使用View 每一个container中的宽高都不能自己设置，一定要通过父亲来设置子组件在容器内存储的最大宽度或者高度来定宽高，然后通过设置宽高比例来获得高度

![image-20220423105826251](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423105826251.png)

GridView 也可以调整主轴方向改为横轴偏移！

调用builder构造函数的时候只有item显示才会被加载

build是默认生成函数，会被父亲调用来生成组件

而`class.builder()`是构造函数，传入不同的参数然后使其在调用build的时候表现不同

当调用builder时候就不会调用class默认的构造函数马，因此可以产生不同key值的实例对象

`Sliversafearea` 可以让sliver默认在安全区域显示，同时上滑的时候可以进入安全区域



![image-20220423121821770](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423121821770.png)

`SliverPadding`可以实现一开始默认有padding，同时向上滚动的时候padding会消失

其中CustomScrollView就可以实现存放sliver数组来实现多个sliver同时存在

![image-20220423123050895](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423123050895.png)

在其中使用SliverAppBar的属性，可以在List中加入AppBar的同时，使用pin：true可以让其固定，就是向上拉之后会逐渐缩小，但是标题依然会存在，只是放置的背景会消失，然后标题固定。

否则也可以向上拉至标题也会消失，然后列表直接充满，并且flexibleSpace会提供一个拉入逐渐缩小的动画效果

![image-20220423123057586](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423123057586.png)

ListTile

可以实现像列表联系人的效果

## 监听列表

### 监听方式Crotroller

所有的列表属性都有一个controller的属性，用来监听列表滑动 

![image-20220423125835425](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423125835425.png)

可以修改默认偏移量为300

在initState钩子阶段一般都是添加一些监听（类比componentDidMount）

脚手架上的`floatingButton`适合来做点击回到最顶部的功能

![image-20220423124802143](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423124802143.png)

![image-20220423130440456](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423130440456.png)

添加controller，不要忘记了

![image-20220423130124514](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423130124514.png)

用完记得销毁

### 监听方式NotifcationLisstener

使用Widget来包裹子组件

可以使用查看总滚动，可以监听开始滚动和结束滚动

![image-20220423125553940](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220423125553940.png)

当有多个列表嵌套时候，return true会默认取消底层的冒泡，return false则会开启冒泡

## Isolate

可以开启一个新线程，来同时处理两件事，看**`Coderwhy`**公众号来查看

双线程通信可以使用Computed

compute接受的回调函数必须是全局的

## 实战经验

### 父Row根据子内容变换宽度

![image-20220424091551419](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220424091551419.png)

通过设置mainAxisSize.min可以让得到的row直接包裹子元素，而不是占满一行

### 图片圆角

裁剪图片，使其出现圆角，图片本身会根据比例自适应，因此只需定宽或者高就可以

![image-20220424144517091](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220424144517091.png)



### 图片和文字在同一行上

行内文字分开可以使用`textspan`来包括三个`WidgetSpan`，然后通过设置`WidgetSpan`独自的`alignment`属性就可以控制图片和文字在一行上，同时还可以当其中的span放不下时可以自动换行

![image-20220424162534806](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220424162534806.png)

但是这里Widget又有一个劣势就是当出现换行时，所有Widget都会换行，从水平主轴改为垂直主轴

记住这里map返回的是一个Iterable，需要调用toList()方法才可以转为List数组

因此就是可以讲标题中的每一个文字都作为一个widgetspan，然后

![image-20220424163149421](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220424163149421.png)





​	

![image-20220424165838939](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220424165838939.png)

### 原理

其实原理很简单，widget分为两种，一种是statewidget，还有一种是普通widget，当使用statewidget时，状态state得以保存的关键就是通过**diff算法**

即Widgettree生成的Elementtree和之前生成的旧的ElementTree的同级节点做对比，如果同级节点的key或者widget type和之前的一样，则会保持state的复用，例如节点下对应key的node还在，[1,2,3]和[1,2]中总能找到key相等的node节点，则state可以得到复用，

或者是下面只有一个ListItem 则可以保证每次Listitem对应的旧elementtree都为其本身，因此不用指定key也可以实现state的复用

就像是[1]和[1]，不管怎么找只要类型不变，则都是默认同一个，故state会做保留

如果改变的话，直接生成新的element，其state也会重新生成，同时用新的element tree对比旧的来部分生成render tree的节点，然后部分替代最后的render tree，因此就可以高效率的介绍render tree的重新生成！

### key

可以通过传递key的方式来传递具体的widget对象（有/无状态的都可以），只需要生成一个localkey或者全局key然后给需要使用的widget绑定对应的key既可以，然后通过传递key就可以实现引用传递，类似于React中的refer

### 状态管理

#### inheritedWidget

类比于React中的context语境，可以让Element沿着自己的父母去寻找widget进而找到对应的contextElement上面的state属性



![image-20220425164220850](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425164220850.png)

包裹的context会用这个dependOnWidgetof这个方法返回context本身的widget节点





然后在子widget就可以展示对应的数据，并且也可以使用setstate去进行修改

![image-20220425164038199](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425164038199.png)

变化

![image-20220425164742683](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425164742683.png)

也可以使用构造函数的方法来传入counter，进而可以把counter的方法在在myapp中保存，然后传给context，然后使用floatingbutton来修改state中的counter的方法，进而改变每一次context传下去的值

Myapp-context

​				---widget01

​				---widget02

​			-floatingbutton

**updateshouldnotify**



![image-20220425165504555](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425165504555.png)

返回的Boolean，当为true，则会把子widget中的didChangeDependenices触发，但是无论为true还是false，didChangeDependenices都会在初始化时调用一次

![image-20220425165628655](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425165628655.png)

#### Provider

![image-20220425174639848](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425174639848.png)

类比于redux

需要添加依赖使用	

![image-20220425175013734](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220425175013734.png)

直接在最外层包裹使用

![image-20220426120626325](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426120626325.png)

首先创建ViewModel类用来做store，使用notifyListeners来更新所有依赖这个数据的widget，类似于发布订阅

![image-20220426120544147](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426120544147.png)

在其他位置**使用共享数据**

![image-20220426120825811](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426120825811.png)

![image-20220426122147271](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426122147271.png)

或者使用consumer来读取数据

两种方式的区别是什么

![image-20220426122430169](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426122430169.png)

##### 优化方法

但是当我们每一次更新state的值的时候，对应的builder都会重新构建一次，如果其中有很多其他的widget就会非常的浪费性能

方法一：

![image-20220426123414977](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426123414977.png)



因此这里可以直接把需要传入的child作为builder的参数传入，然后就可以通过传参数的方式，从而不会因此传入的child的重构

方法二：

**修改数据**

使用consumer来修改provider中的值

![image-20220426122041961](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426122041961.png)

方法三：

Selector优化

![image-20220426124558048](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426124558048.png)

#### MutiProvider

直接是用数组可以一次使用多个Provider

![image-20220426125530221](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426125530221.png)

同时consumer也可以进行复合使用，即传入两个泛型，来生产不同对应的值

![image-20220426125843166](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426125843166.png)

### modelView store发送网络请求

![image-20220426130035021](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426130035021.png)

在构造器里面写入网络请求，然后在得到结果后调用notifyListeners即可

## 屏幕适配

![image-20220426152659516](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426152659516.png)

**方法一 使用自适应组件Fittedbox**

使用Fittedbox，当一个container里面的内容超过其本身的宽度的时候，Fittedbox可以将其内容压缩至和container的宽度保持一致。

类似于图片的backgounde：cover属性，自适应布局

**方法二 使用屏幕尺寸适配单位**

## Flutter事件监听

## 路由

需要跳转的路由界面记得用**MaterialPageRoute**来包裹，从而可以跳转可以成功

当使用路由表的时候可以使用这个方式来获取传递过来的参数

![image-20220426192755287](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426192755287.png)

然后可以使用传递参数的方式来实现路由传参

![image-20220426192844504](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426192844504.png)

**动态路由**

![image-20220426193114891](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426193114891.png)

当HYDetailPage构造函数要求传入参数的时候，我们就不能使用命名路由的方式来实现路由跳转了

![image-20220426193846184](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426193846184.png)

可以通过添加GenerateRoute的方式，来动态生成路由，并且通过settings可以传递跳转路由的方式，传递规则如上

![image-20220426193915579](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426193915579.png)

记得设置unknown错误页面，用来捕获错误路由跳转

## 动画

创建的类记得要混入这个类![image-20220426203652628](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426203652628.png)

![image-20220426203644929](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220426203644929.png)

## 路由跳转动画

## Hero

移动端开发会经常遇到类似这样的需求：

- 点击一个头像，显示头像的大图，并且从原来图像的Rect到大图的Rect
- 点击一个商品的图片，可以展示商品的大图，并且从原来图像的Rect到大图的Rect

这种跨页面共享的动画被称之为享元动画（Shared Element Transition）

在Flutter中，有一个专门的Widget可以来实现这种动画效果：Hero

实现Hero动画，需要如下步骤：

- 1.在第一个Page1中，定义一个起始的Hero Widget，被称之为source hero，并且绑定一个tag；
- 2.在第二个Page2中，定义一个终点的Hero Widget，被称之为 destination hero，并且绑定相同的tag；
- 3.可以通过Navigator来实现第一个页面Page1到第二个页面Page2的跳转过程；

## 引用传递问题

![image-20220428203933128](https://yajokepic.oss-cn-chengdu.aliyuncs.com/image-20220428203933128.png)

https://blog.csdn.net/weixin_36213143/article/details/112286730

参考该博客

其实本质可以说是，因为子类是父类的拓展，因此本身创建的对象内存肯定是会大于父类的

假如父类的对象有2M，那么子类创建出来的对象就是2+1M，但是当使用父类的引用去访问对应堆区中的引用的时候，父类引用只申请了2M的内容，但是又想访问子类所创建的3M内容，那么java处于内存安全考虑，是不会让一个指针访问其没有申请的区域，此时就会报错（但是C不会，这里可以从反汇编的角度分析）

但是父类引用子类创建的对象时，却可以，因为父类可以访问子类创建出的3M中的前2M，所以是没有任何问题的。不会出现超出读取！
