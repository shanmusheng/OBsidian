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
如果原本的宽度
设置为100
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
## 演示
```
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';

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
      designSize: Size(360, 690),
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

class ScreenView extends StatefulWidget {
  const ScreenView({super.key});

  @override
  State<ScreenView> createState() => _ScreenViewState();
}

class _ScreenViewState extends State<ScreenView> {
  @override
  Widget build(BuildContext context) {
    Size size = MediaQuery.of(context).size;
    return Scaffold(
      appBar: AppBar(
        title: Text('$size'),
      ),
      body: SingleChildScrollView(
        child: Column(
          children: [
            Container(
              color: Colors.blue,
              width: 100.w,
              height: 50.h,
              child: Text('100w50h'),
            ),
            Container(
              color: Colors.blue,
              width: 100,
              height: 100,
              child: Text('100 100'),
            ),
            Container(
              color: Colors.blue,
              width: 100.w,
              height: 100.h,
              child: Text('100w100h'),
            ),
            Container(
              color: Colors.blue,
              width: 300,
              height: 200,
              child: Text('300 200'),
            ),
          ],
        ),
      ),
    );
  }
}
```
![[PixPin_2024-09-19_21-26-09.gif]]
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
   当用`ScreenUtilInit` 初始化时,会指定一个设计稿的尺寸(通常是 `designSize` 参数),`ScreenUtil` 会根据当前设备的实际宽度和高度计算出一个比例因子。
```
ScreenUtilInit(
  designSize: const Size(375, 812), // 设计稿尺寸
  builder: (context, child) {
    return MyApp();
  },
);
```
在这里，`375` 和 `812` 是设计稿中的宽度和高度。而当前设备的实际宽度和高度可以通过 `MediaQuery` 获取：
```
double deviceWidth = MediaQuery.of(context).size.width;
double deviceHeight = MediaQuery.of(context).size.height;
```
通过比较设计稿和设备的宽度、高度，`ScreenUtil` 计算出一个比例因子：
```
double scaleWidth = deviceWidth / 375; // 相对于设计稿宽度的缩放因子
double scaleHeight = deviceHeight / 812; // 相对于设计稿高度的缩放因子
```
有了这个初始化比例系数就可以去动态调整尺寸
### 动态调整尺寸

有了比例因子后，`ScreenUtil` 可以通过 `.w`、`.h`、`.sp` 等方法，根据比例因子对 UI 元素的宽度、高度、字体大小等进行缩放。

- `.w` 是基于宽度的缩放，例如 `100.w` 表示设计稿上宽度为 100 的元素会根据比例调整。
- `.h` 是基于高度的缩放。
- `.sp` 用于调整字体大小。



具体实现上，这些方法是通过扩展方法实现的。例如，`.w` 是 `double` 的一个扩展方法：
![[Pasted image 20240919215050.png]]
```
  ///[ScreenUtil.setWidth]
  double get w => ScreenUtil().setWidth(this);

  ///[ScreenUtil.setHeight]
  double get h => ScreenUtil().setHeight(this);

  ///[ScreenUtil.radius]
  double get r => ScreenUtil().radius(this);
```
`ScreenUtil` 内部有对应的 `setWidth()` 和 `setHeight()` 方法，它们通过比例因子来调整数值：
```
double setWidth(num width) => width * scaleWidth;
double setHeight(num height) => height * scaleHeight;
```
而这些也在官方给出的源码里有解释
![[Pasted image 20240919215226.png]]
# 总结
`ScreenUtil` 的核心是通过比例系数来对宽度、高度、字体等进行动态缩放，从而实现屏幕适配。它通过 `MediaQuery` 获取设备尺寸，并结合设计稿的尺寸，计算比例因子，然后通过扩展方法实现适配的功能。
## 优势

- **简化适配过程：** 通过 `.w`、`.h`、`.sp` 等扩展方法，开发者不需要自己手动计算比例，极大地简化了屏幕适配的工作。
- **高效：** `ScreenUtil` 基于设计稿的尺寸进行统一管理，可以在不同设备上保持一致的显示效果。
- **灵活性高：** 可以根据项目需求动态调整适配的设计尺寸，适用于各种不同分辨率的设备。
## 劣势
### 1. **固定设计稿尺寸的限制**

`ScreenUtil` 依赖于设计稿的尺寸（通过 `designSize` 参数设置），如果设计稿的尺寸选择不当或者团队在设计时没有统一规范，可能会导致在某些设备上适配效果不佳。特别是在处理极端的屏幕尺寸（如平板或特别小的手机）时，设计稿和实际屏幕尺寸的差异可能会放大，导致 UI 显示效果不理想。

- **劣势体现**：如果设计稿过于局限于某一类设备（如某一款特定手机），而忽略了其他设备的屏幕尺寸，`ScreenUtil` 的适配会表现得不够灵活。

### 2. **比例适配可能导致布局不一致**

`ScreenUtil` 基于屏幕的宽度和高度进行比例缩放，但不同设备之间的屏幕比例并不完全相同。例如，宽高比例为 16:9 的设备和宽高比例为 19.5:9 的设备显示出来的效果可能不同。如果只按照宽度或高度进行缩放，可能会导致组件在不同设备上的显示比例失调，某些界面元素可能在某些设备上显得过大或过小。

- **劣势体现**：同样的组件可能在不同设备上占用不同的空间，导致显示不一致，特别是在宽高比例差异较大的设备上。

### 3. **字体适配的不完美**

虽然 `ScreenUtil` 提供了 `sp` 方法来适配字体大小，但字体大小的适配比组件的宽度和高度更为复杂，因为不同设备的分辨率、像素密度（DPI）不一样，某些设备的文字可能显得过大或过小，尤其在大屏设备上，适配的字体可能显得不符合用户的阅读习惯。

- **劣势体现**：在不同的设备上，字体可能显得过大或过小，特别是在高 DPI 的设备上，字体大小需要特别调整以获得更好的可读性。

### 4. **忽略实际的屏幕使用环境**

`ScreenUtil` 只关注设计稿尺寸与实际设备屏幕尺寸之间的比例关系，但忽略了设备的使用环境。例如，有些设备上会有状态栏、虚拟导航栏或其他占用屏幕空间的组件，`ScreenUtil` 并没有处理这些情况，可能会导致布局上的误差。

- **劣势体现**：在有虚拟按键的设备上，底部或顶部的空间可能被系统组件（如状态栏或虚拟按键）占用，导致实际的有效可用空间与 `ScreenUtil` 计算的结果不一致。

### 5. **在大型复杂项目中的局限性**

对于一些 UI 复杂且动态变化的应用，单纯依赖 `ScreenUtil` 进行适配可能会不够灵活。例如，某些复杂的动态 UI 需要根据设备尺寸实时调整布局，而 `ScreenUtil` 的适配方式是基于初始化时的设计稿尺寸，不能动态调整布局。

- **劣势体现**：复杂的交互场景和动态布局可能需要结合 `MediaQuery`、`LayoutBuilder` 等 Flutter 内置的布局工具进行更细粒度的控制，而 `ScreenUtil` 本身更适合静态布局的适配。

### 6. **对开发者的依赖性强**

`ScreenUtil` 的使用需要开发者在每个 UI 组件的宽度、高度、字体等方面显式地调用适配方法（如 `.w`、`.h`、`.sp`），如果开发者在某些地方忘记使用这些适配方法，可能会导致这些元素在不同设备上没有进行适配，显示效果不统一。

- **劣势体现**：开发者需要时刻保持警觉，确保所有的 UI 元素都使用了 `ScreenUtil` 提供的适配方法，否则很容易在某些屏幕上出现显示不一致的问题。

### 7. **不适用于所有类型的 UI**

`ScreenUtil` 对于常规的、基于固定设计尺寸的应用适配非常有效，但如果应用需要大量的流式布局、相对布局，或者需要根据父组件的尺寸自动调整大小，`ScreenUtil` 可能不是最佳选择。例如，在处理流式布局或响应式设计时，Flutter 提供的 `Flexible`、`Expanded`、`MediaQuery`、`LayoutBuilder` 等工具可能更加合适。

- **劣势体现**：对于需要自适应父容器大小、动态布局变化的场景，`ScreenUtil` 的比例适配方式可能不够灵活。

### 8. **维护成本**

如果项目初期没有正确使用 `ScreenUtil` 进行适配，后期可能需要花费大量时间去修改和调整代码。尤其是在团队协作的项目中，如果开发者没有统一适配策略，维护这些适配代码可能会变得繁琐。

- **劣势体现**：需要对代码进行严格的管理和审核，确保每个开发者都遵循一致的适配规则，否则后期维护会增加代码复杂度。