# 显示动画
如果需要循环播放、暂停、多个动画协调时可以选择显示动画,显示动画需要用到`AnimationController`。Flutter现有控件以`...Teansition`结尾。

## AnimationController
`AnimationController`: 派生自`Animation<double>`,用于控制动画,使用完成后需要在`dispose()`方法中回收。它包含动画的启动`forward()`、停止`stop()` 、反向播放 `reverse()`等方法,生成数字的区间可以通过`lowerBound`和`upperBound`来指定。`_controller.value` 等价于 Tween 补间动画中的value。

### AnimationController简单的缩放示例🌰
```dart
class _HomePageWidgetState extends State<HomePageWidget> with SingleTickerProviderStateMixin {

late AnimationController _controller;

@override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(seconds: 1),
      lowerBound: 1, // 最小是1倍
      upperBound: 1.5, // 最大是1.5倍，这里的值等价于补间动画的begin和end
      vsync: this // 垂直同步:屏幕什么时候需要显示新的帧，因为不同的设置有不同的刷新频率，有的是60Hz，一秒刷新60次，16毫秒一次，有的是120Hz，8毫秒一次,Flutter提供了SingleTickerProviderStateMixin 获取同步数据，如果有多个控件需要监听屏幕刷新，使用 TickerProviderStateMixin
    )..repeat(reverse:true); //联级语法调用
  }

  @override
  void dispose() {
    _controller.dispose(); // 注意这里需要移除
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ScaleTransition(
        scale: _controller,
        child: Center(
          child: Container(
            width: 100,
            height: 100,
            color: Colors.blue,
            child: AnimatedContainer(duration: Duration(milliseconds: 100)),
          )
        ),
      ),
    );
  }
}
```

### `AnimationController`中的常用方法
```dart
_controller.forward(); // 运行一次
_controller.repeat(); //无限循环动画
_controller.repeat(reverse: true); // 动画完成后会慢慢还原到之前的状态再进行新的动画
_controller.reset(); //重置
_controller.stop(); //停止动画，控件会保持当前状态
```

### `AnimationController`和补间动画`Tween`的配合使用
```dart
class _HomeWidgetState extends State<HomeWidget> with SingleTickerProviderStateMixin {

late AnimationController _controller;

@override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(seconds: 1),
      vsync: this 
    )..repeat(reverse:true);
  }

  @override
  void dispose() {
    _controller.dispose(); // 注意这里需要移除
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SlideTransition(
        position: Tween(begin: Offset.zero,end: Offset(0,0.5)) // 添加补间动画
                .chain(CurveTween(curve: Curves.bounceOut)) // 添加曲线
                .animate(_controller),
        child: Center(
          child: Container(
            width: 100,
            height: 100,
            color: Colors.blue,
            child: AnimatedContainer(duration: Duration(milliseconds: 100)),
          )
        ),
      ),
    );
  }
}

// AnimationController 使用补间动画的另一种写法
_controller.drive(Tween(begin: Offset.zero,end: Offset(0,0.5)))
```

### 关于 Interval()
有时候一个整体动画时间是5秒，我们想在第3秒的时候开始动画，可以选择使用`Interval()`
```dart
Tween(begin: Offset.zero,end: Offset(0,0.5))
.chain(CurveTween(curve: Interval(0,0.5))) // 假设我们的动画时间是5秒，Interval(0,0.5)表示前面2.5秒完成动画之后，剩下2.5秒什么事情也不做
.animate(_controller),

// Interval(0,0.5)表示前面2.5秒完成动画之后，剩下2.5秒什么事情也不做
// Interval(0.5,0.8)表示前面2.5秒什么事情也不做，然后1.5秒完成动画，剩下的时间什么也不做，比如动画时间是5秒，5*0.5 = 2.5
```

## 交错动画示例
使用`Interval()`编写一个交错动画的示例，原理就是在动画的时间内，将每个时间段分配给单独的 Widget。`Tween()`可以组合多个效果进行使用。
```dart
class _DemoWidgetState extends State<DemoWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
        duration: Duration(seconds: 5),
        vsync:
            this // 垂直同步:屏幕什么时候需要显示新的帧，因为不同的设置有不同的刷新频率，有的是60Hz，一秒刷新60次，16毫秒一次，有的是120Hz，8毫秒一次,Flutter提供了SingleTickerProviderStateMixin 获取同步数据
        )
      ..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose(); // 注意这里需要移除
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          SlideBox( _controller,Colors.blue[100],Interval(0.0, 0.2),),
          SlideBox(_controller,Colors.blue[300],Interval(0.2, 0.4),),
          SlideBox(_controller,Colors.blue[500],Interval(0.4, 0.6),),
          SlideBox(_controller,Colors.blue[700],Interval(0.6, 0.8),),
          SlideBox(_controller,Colors.blue[900],Interval(0.8, 1.0),),
          ],
      ),
    ));
  }
}

class SlideBox extends StatelessWidget {
  SlideBox(this.controller,this.color,this.curve);
  final AnimationController controller;
  final Curve curve;
  final Color? color;

  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      position: Tween(begin: Offset.zero,end: Offset(0.1,0))
      .chain(CurveTween(curve: Curves.bounceOut)) // 添加回弹效果
      .chain(CurveTween(curve: curve)) // 在指定的时间完成移动效果
      .animate(controller),
      child: Container(width: 200, height: 50, color: color),);
  }
}
```



## 自定义动画控件 `AnimationBuilder`
自定义显示动画`AnimationBuilder`



