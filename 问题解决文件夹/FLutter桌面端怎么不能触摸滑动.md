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
到此还没有结束.
