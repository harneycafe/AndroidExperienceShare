# Flutter SZ笔记
#flutter

### 功能性组件
1. WillPopScope 返回键拦截的 Widget。
2. InheritedWidget 组件之间数据共享的组件：didChangeDependencies()
可以再依赖数据发生变化时做一些昂贵的操作之后执行build,而避免每次build都执行昂贵的操作，如网络请求。
3. InheritedWidget 在widget 树中传递方向为自上而下。

### 数件处理与通知 

1. 原始指针事件 Pointer Event 可以被监听，使用 Listener Widget,
Pointer 包含 指针移动距离，指针位置等许多信息。

### 时间机制
1. 在Widget树中，每一个节点都可以分发通知，通知会沿着当前节点（context）向上传递，所有父节点都可以通过NotificationListener来监听通知，Flutter中称这种通知由子向父的传递为“通知冒泡”（Notification Bubbling），这个和用户触摸事件冒泡是相似的，但有一点不同：Notification可以中止，但用户触摸事件不行。
2. 很多Widget 比如ScrollNotification都用到了Notification。
3. EventBus:
[全局事件总线 · 《Flutter实战》](https://book.flutterchina.club/chapter8/eventbus.html)

### 动画
**Animation**
1. Animation ：Animation对象本身和UI渲染没有任何关系。它用于保存动画的插值和状态。
2. Animation 对象是一个在一段时间内依次生成一个区间（由Tween负责）之间值的类。Animation对象的输出值可以是线性的、曲线的、一个步进函数或者任何其他曲线函数（由Curve负责）。 
Curve类 ：控制动画的曲线，可以继承 Curve，重写其 transform 来实现自己的曲线，0到1。
3. Animation 可以添加监听器：
	1. addListener 在每一帧都会被调用。帧监听器中最常见的行为是改变状态后调用setState()来触发UI重建。
	2. addStatusListener 动画开始、结束、正向或反向（见AnimationStatus定义）时会调用StatusListener。

**AnimationController**
1. AnimationController用于控制动画，它包含动画的启动forward()、停止stop()、反向播放reverse()等方法。
2. AnimationController派生自Animation，AnimationController生成数字的区间可以通过：lowerBound，upperBound## 来指定，如：
```
final AnimationController controller = new AnimationController( 
 duration: const Duration(milliseconds: 2000), 
 lowerBound: 10.0,
 upperBound: 20.0,
 vsync: this
);
```

3. AnimationController 需要 vsync 参数，他接收 TickerProvider 对象来提供Ticker。
4. Flutter 应用启动时会绑定一个SchedulerBinding，Tick 通过SchedulerBinding 来添加屏幕刷新回调，即自己的TickerCallBack。
通过将 SingleTickerProviderStateMixin 添加到State的定义中，然后将State对象作为 vsync 的值。
5. 路由销毁时需要释放动画控制器(controler)。
6. AnimationBuild 可以 不用显式的添加帧监听器，减小构建范围，封装常用的过渡动画。

## 页面切换动画
1. Material库中提供 MaterialPageRoute，它可以使用和平台风格一致的路由切换动画，如在iOS上会左右滑动切换，而在Android上会上下滑动切换。如果在Android上也想使用左右切换风格，可以直接使用CupertinoPageRoute。
2. 如果想自定义路由切换动画，可以使用PageRouteBuilder，我们也可以直接继承PageRoute类来实现自定义路由。
3. 无论是MaterialPageRoute、CupertinoPageRoute，还是PageRouteBuilder，它们都继承自PageRoute类，而PageRouteBuilder其实只是PageRoute的一个包装。
4. 正常推荐PageRouteBuilder，特殊情况例如在应用过渡动画时我们需要读取当前路由的一些属性，这时就只能通过继承PageRoute的方式了。

## 自定义Widget
1. 自定义widget三种方法：组合，自绘，实现 RenderObject。
自绘方式是调用 CustomPaint 和 Canvas 绘制的。
RenderObject 系列类内部也是调用 Canvas绘制的，所以本质和自绘一样。
2. 自定义Widget需要注意：必要参数要用@required# 标注。为了保证代码健壮性，我们需要在用户错误地使用Widget时能够兼容或报错提示（使用assert断言函数）。
3. 自绘的方式：CustomPatient 中有 RepaintBoundary 属性，他会隔离子节点和 CustomPaint 的绘制边界，返回 ture 的话，外部状态改变，组件也会重绘，返回 false 则不需要重绘。
4. 绘制尽可能多的分层，配合每层 RepaintBoundary 来避免不必要的绘制。

## Flutter UI
1. Flutter框架将对 Widget 树的操作映射到了Element树上，真正绘制操作的是与 Element 树相连的 Render 树。
2. 从创建到渲染的大体流程是：根据 Widget  生成 Element，然后创建相应的RenderObject 并关联到 Element.renderObject 属性上，最后再通过RenderObject来完成布局排列和绘制。
3. BuildContext就是Widget对应的Element，所以我们可以通过context在StatelessWidget和StatefulWidget的build方法中直接访问Element对象。
4. 如果要从头到尾实现一个 RenderObject 是比较麻烦的，我们必须去实现layout、绘制和命中测试逻辑，但是大多数时候我们可以直接在Widget层通过组合或者CustomPaint完成自定义UI。如果遇到只能定义一个新RenderObject的场景时（如要实现一个新的layout算法的布局容器），可以直接继承自RenderBox，这样可以帮我们减少一部分工作。

## Dart 线程模型及异常捕获
1. Java和OC都是多线程模型的编程语言，任意一个线程触发异常且没被捕获时，整个进程就退出了,意味着程序终止。但Dart和JavaScript不会，它们都是单线程模型。
2. 在Dart中，所有的外部事件任务都在事件队列中，如IO、计时器、点击、以及绘制事件等。微任务队列优先级高，如果微任务队列执行耗时任务，将影响GUI。
3. 在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是**当前任务**的后续代码就不会被执行了，也就是说一个任务中的异常是不会影响其它任务执行的。
4. onError 是 FlutterError 的一个静态属性，，如果我们想自己上报异常，只需要提供一个自定义的错误处理回调即可，如：
```
void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    reportError(details);
  };
 …
}
```

5. 其他无法捕捉异步的异常：空指针异常，Future中的异常 。
Dart中有一个runZoned(…)方法，可以给执行对象指定一个Zone。Zone表示一个代码执行的环境范围，为了方便理解，读者可以将Zone类比为一个代码执行沙箱，我们可以在其中捕获日志，调度微任务，定时任务等，代码：
```
void collectLog(String line){
    ... //收集日志
}
void reportErrorAndLog(FlutterErrorDetails details){
    ... //上报错误和日志逻辑
}

FlutterErrorDetails makeDetails(Object obj, StackTrace stack){
    ...// 构建错误信息
}

void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    reportErrorAndLog(details);
  };
  runZoned(
    () => runApp(MyApp()),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
        collectLog(line); // 收集日志
      },
    ),
    onError: (Object obj, StackTrace stack) {
      var details = makeDetails(obj, stack);
      reportErrorAndLog(details);
    },
  );
}

```






