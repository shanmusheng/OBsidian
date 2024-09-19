#flutter_screenutil
官网[flutter_screenutil | Flutter package (flutter-io.cn)](https://pub-web.flutter-io.cn/packages/flutter_screenutil)
中文文档[flutter_screenutil/README_CN.md at master · OpenFlutter/flutter_screenutil (github.com)](https://github.com/OpenFlutter/flutter_screenutil/blob/master/README_CN.md)
## 官方介绍
**flutter 屏幕适配方案，用于调整屏幕和字体大小的flutter插件，让你的UI在不同尺寸的屏幕上都能显示合理的布局!**

_注意_：此插件仍处于开发阶段，某些API可能尚不可用。
## 使用(这里仅介绍使用方法一)
要在每个使用的地方导入包
`import 'package:flutter_screenutil/flutter_screenutil.dart';`
### 属性

| 属性              | 类型                | 默认值            | 描述                                 |
| :-------------- | :---------------- | :------------- | :--------------------------------- |
| designSize      | Size              | Size(360, 690) | 设计稿中设备的尺寸(单位随意,建议dp,但在使用过程中必须保持一致) |
| deviceSize      | Size              | null           | 物理设备的大小                            |
| builder         | Widget Function() | Container()    | 一般返回一个MaterialApp类型的Function()     |
| orientation     | Orientation       | portrait       | 屏幕方向                               |
| splitScreenMode | bool              | false          | 支持分屏尺寸                             |
| minTextAdapt    | bool              | false          | 是否根据宽度/高度中的最小值适配文字                 |
| context         | BuildContext      | null           | 传入context会更灵敏的根据屏幕变化而改变            |
| child           | Widget            | null           | builder的一部分，其依赖项属性不使用该库            |
| rebuildFactor   | Function          | _default_      | 返回屏幕指标更改时是否重建。                     |
**初始化并设置配置尺寸及字体大小是否根据系统的字体大小辅助选项来进行缩放**
在使用前请设置好设计稿的宽度和高度,传入设计稿的宽度和高度(单位随意,但在使用过程中必须保持一致)一定要进行初始化(只需设置一次),以保证在每次使用之前都配置好了适配尺寸
## 初始化方法
在最外层包裹上`ScreenUtilInit`
```
void main() {
  runApp(
    const MyApp(),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return ScreenUtilInit(
      designSize: Size(360, 690),//设计稿为360w690h
      minTextAdapt: true,
      splitScreenMode: true,
      child: MaterialApp(
        title: 'Flutter Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
          useMaterial3: true,
        ),
        home: ScreenView(),
        // home: RecordingScreenProvider(),
      ),
    );
  }
}
```
## 设置后的使用方法
如果原本的宽度设置为100
```
            Container(
              color: Colors.blue,
              width: 100,
              height: 100,
              child: Text('100 100'),
            ),
```
使用flutter_screenutil后
```
            Container(
              color: Colors.blue,
              width: 100.w,
              height: 100.h,
              child: Text('100w100h'),
            ),
```
此时的100.w就相当于设计稿里的100,或者更直观的说,由于这个案例的设计稿w是360,所以当width为360.w时,不论如何,都会占满宽度
![[Pasted image 20240919213456.png]]
(这里暂时没有提及关于字体和其他扩展方法,将会在后面统一介绍)
## 原理
`ScreenUtil` 是通过计算屏幕的比例系数，基于设计稿的尺寸和当前设备的实际尺寸进行比例换算，从而实现屏幕适配的。在不同设备上，`ScreenUtil` 会根据设计稿尺寸（通常是 UI 设计给定的标准分辨率，如 375x812 的 iPhone X 尺寸）和当前设备的实际分辨率，动态调整布局元素的尺寸、字体大小等，使得不同设备上的显示效果尽量保持一致.
## 核心原理
`ScreenUtil` 主要通过两个关键步骤实现适配：
1. 计算比例系数
   设计稿的宽高和当前设备的实际宽高进行比较,得到比例系数.
   使用比例系数对各个UI的宽度,高度,字体等进行缩放.
2. 动态适配:
   使用 `.w`、`.h`、`.sp` 等扩展方法，根据比例系数自动调整元素的尺寸.
## 具体实现步骤
### 初始化比例系数
   当用`ScreenUtilInit` 初始化时,会指定一个设计稿的尺寸(通常是)