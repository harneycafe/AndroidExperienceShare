# <Flutter技术入门与实践> 读书笔记
#flutter

## 第1章：开启Flutter之旅
1. 跨平台，丝滑的交互体验，响应式编程框架，60fps。
2. 集成StatefulWidget的Widget，他们的可变状态存储在State的子类中。
3. State是一个组件的UI数据模型，是组件渲染时的数据依据。Flutter程序的运行可以认为是一个巨大的状态机，用户的操作，请求API和系统事件的触发都是推动状态机运行的触发点，触发点通过调用setState方法推动状态机进行相应。
4. ```flutter doctor```
该命令会检查环境并在终端显示报告。
5. Flutter需要安装在 Android 4.1(API level 16) 或更高版本。
6. flutter devices 验证连接的Android设备。
7. flutter run 启动程序。

## 第2章，第3章：Flutter基础知识，Dart语言简述
1. ```void main => runApp(Widget app);```
Main函数是Flutter框架的入口，调用runApp启动Flutter程序。
2. 全局主题有许多参数可以设置，来定义应用程序内部的颜色和字体样式，在程序中调用。
```
color:Theme.of(context).accentColor,
```
3. Dart 是AOT编译，也支持JIT编译。
4. Dart 是弱数据类型语言。
5. Dart 没有pbulic,private的概念。私有特性通过变量或者函数前加下划线来表示。
6. 定义的变量不会变化，可以使用final或const来指明。const是编译时常量，可以通过计算来获取值。
7. Dart是强bool类型检查的，有的语言中0是false,非0为true,Dart中不是。
8. Dart中函数可以像参数一样传递给其他函数，便于做回调。
9. Dart中所有函数都会有返回值，默认为null。
10. as 操作符用于类型转换，is 和 is! 用于类型检查。
11. ```
expr1 ??= expr2;  //如果expr1为空，则将expr2计算的值分配给expr1,否则expr1 不变，三目运算符
```
12. ^ 异或运算符：相同异或为0，不相同异或为1。
13. .. 级联运算符，便于链式调用。
14. 不关心操作数的当前下标，forEach方法是简便的。
15. break跳出整个循环，continue跳出当前循环。
16. 基于mixin的继承方式是指：一个类可以继承多个父类，相当于其他语言的多继承，用with关键字来实现。
17. 每一个类的实例，系统都会隐式的包含get和set方法。
18. 每一个枚举类型都有index的getter,用来标记元素的位置。
19. 当引用的库有冲突的名字的时候，可以用as指定前缀。
	```import ‘package:lib2’ as lib2```
20. 除了as,引用包还可以使用show(只引用)和hide(除此之外全部引用)关键字。

## 第4章，第5章，第6章：常用组件，MD组件，IOS组件
1. Container 的宽高：double.indinity 可以强制撑满，不设置则随子布局的大小。
2. Icon 组件不可交互，交互需要使用IconButton，IconButton的onPress如果为空，则按钮不可用。
3. RaisedButton 凸起按钮，FlatButton 扁平按钮组件。SimpleDialog通常呵呵SimpleDialogOption组件一起使用。
4. 声明测试数据：
 ```new List<String>.generate(500,(i)=>’item $I’)；```
5. GridView.count 通过单行展示哥舒翰创建GridView,GridView.exent通过最大宽度创建。
6. 表单组件Form组件做表单提交试用，需要和TextFormField(类似于EditText)配合和使用。
7. MaterialApp有许多属性来配置MD布局。
8. Scaffold 的resizeToAvoidBottomPadding属性控制底部重绘，避免遮盖键盘的问题。
9. SliverAppBar是可以跟随内容滚动的AppBar。

## 第7章：页面布局
1. 百分比布局：FractionallySizedBox。控住是否显示布局：Offstage。溢出父容器显示：OverflewBox。宽高比布局：AspectRatio。
2. Expanded布局：防止内容溢出;内容不足时补足。
3. FittedBox布局：通过BoxFit属性控制子控件的缩放，类似于图片。
4. IndexedStack 布局：层叠但只显示最外一层布局，有index属性索引。
5. SizeBox 布局：有特定的大小，必须指定宽高。
6. Wrap: 按宽高自动换行，可用作流标签布局。

## 第8章：手势
1. Flutter 手势系统分两层，第一层为触摸原事件，第二层就是我们可以检测到的手势包括点击，拖动和缩放等。
2. 触摸原事件包括：
	1. PointerDownEvent：用户与屏幕产生了联系。
	2. PointerMoveEvent：手指从屏幕上的一个位置移动到了另一个位置。
	3. PointerUpEvent：手指离开了屏幕。
	4. PointerCancelEvent：此指针的输入不再指向此应用程序。

## 第9章：资源和图片
1. 加载JSON文件
 ```
import ‘package:flutter/sevices.dart’ show rootBunndle;
Future<String> loadAsset() async {
Return await rootBundle.loadString(‘assets/config.json’);
} 
 ```
2. 加载依赖包中的图像，必须给AssetImage提供Package参数。

## 第10章：路由及导航
1. 页面跳转返回数据：使用 async+await 来进行Navigator进行push，页面返回的时候Navigator的pop方法中加入返回的参数。
2. 声明测试数据的方法：
 ``` new List.generate(20,(i)=>new Product(’第${I}条数据’));```

## 第11章：组件装饰和视觉效果
1. 自定义绘制：Canvas是画板对象，提供各种自定义绘制形状的方法；Paint是画笔，提供设置绘画的各种参数。使用时，在CustomPainet的Widget中使用。
2. Opacity : 设置透明的的Widget。
3. DecroatedBox：可以从多方面进行装饰处理的Widget,如颜色，形状，阴影，渐变及背景图片等。
4. RoteatedBox：旋转组件，控制child发生旋转。
5. Clip：Flutter提供许多组件来完成裁剪功能，如：ClipOval,ClipRRect,ClipRect,ClipPath等。

## 第12章：动画
1. AnimatedOpacity：实现透明度调整的动画Widget。
2. Hero：MD的页面联动跳转动画。

## 第13章，第14章，第15章，第16章：Flutter插件开发，开发工具和使用技巧，测试与发布应用，综合案例
1. 加载页面的停顿要放在initState函数处理，原因是必须等页面渲染完了才行，否则加载的页面就看不到了。
2. 延时执行：Future.delayed(Duration(second:3),(){//TODO})。
3. Flutter 的 Widget Inspector ，是AS中很强的开发者工具。
4. Observatory 性能调试工具，在Debug下点击秒表可以生成报告。
5. 在AS中，点击工程右键，可以选择打开Android项目和IOS项目。
6. Shift+热重载按钮 = 热重启。


