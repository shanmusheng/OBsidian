这个原因是在Flutter桌面端，`PageView` 默认并不支持触摸滑动，主要是因为桌面环境通常依赖鼠标事件。
额,那要是想实现触摸滑动呢
欸?自定义一个就好了
```
class DesktopScrollBehavior extends MaterialScrollBehavior {
  @override
  Set<PointerDeviceKind> get dragDevices => {
        PointerDeviceKind.touch,
        PointerDeviceKind.mouse,
      };
}
```
上面这个就是自定义了`ScrollBehavior`,可以允许鼠标和触摸设备都参与滑动.
然后再使用这个自定义的`MaterialScrollBehavior`,使用地点为`MaterialApp`
```
MaterialApp(
        scrollBehavior: DesktopScrollBehavior(),//在这里使用
        title: 'Flutter Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
          useMaterial3: true,
        ),
        home: GalleryHome(),
        // home: RecordingScreenProvider(),
      )
```
这样就实现鼠标也可以触摸滑动了.
到此还没有结束.同理还可以自定义滚动效果

```
  
//自定义滚动效果  
class CustomScrollPhysics extends ScrollPhysics {  
  CustomScrollPhysics({ScrollPhysics? parent}) : super(parent: parent);  
  @override  
  CustomScrollPhysics applyTo(ScrollPhysics? ancestor) {  
    // TODO: implement applyTo  
    return CustomScrollPhysics(parent: buildParent(ancestor));  
  }  
  
  @override  
  double applyPhysicsToUserOffset(ScrollMetrics position, double offset) {  
    // TODO: implement applyPhysicsToUserOffset  
    // 自定义用户滑动的偏移量处理  
    return offset * 0.5;  
  }  
  
  @override  
  // TODO: implement dragStartDistanceMotionThreshold  
  //滑动距离多少时,开始滑动  
  double? get dragStartDistanceMotionThreshold => 10;  
  
  @override  
  bool shouldAcceptUserOffset(ScrollMetrics position) {  
    // return true; // 接受所有用户滑动  
  
    // 在某些情况下不接受用户输入，比如滚动已经到达边界  
    if (position.atEdge) {  
      return false; // 不接受用户输入  
    }  
    return true; // 其他情况接受用户输入  
    }  
}
```
`applyPhysicsToUserOffset`是自定义用户滑动的偏移量处理,比如我滑动100px,实际上滑动50px
`dragStartDistanceMotionThreshold`是滑动距离多少时,开始滑动,当手势滑动超过10px时才开始算作滑动
`shouldAcceptUserOffset`在某些情况下可以滑动,比如这个是到了边界时,不能滑动
