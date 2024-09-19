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