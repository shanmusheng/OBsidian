#provider 
官网 [provider | Flutter package (pub.dev)](https://pub.dev/packages/provider)
# 官方介绍
对 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html) 组件的上层封装，使其更易用，更易复用。

使用 `provider` 而非手动书写 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html)，有以下的优势:

- 简化的资源分配与处置
- 懒加载
- 创建新类时减少大量的模板代码
- 支持 DevTools
- 更通用的调用 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html) 的方式（参考 [Provider.of](https://pub.flutter-io.cn/documentation/provider/latest/provider/Provider/of.html)/[Consumer](https://pub.flutter-io.cn/documentation/provider/latest/provider/Consumer-class.html)/[Selector](https://pub.flutter-io.cn/documentation/provider/latest/provider/Selector-class.html)）
- 提升类的可扩展性，整体的监听架构时间复杂度以指数级增长（如 [ChangeNotifier](https://api.flutter-io.cn/flutter/foundation/ChangeNotifier-class.html)， 其复杂度为 O(N)）