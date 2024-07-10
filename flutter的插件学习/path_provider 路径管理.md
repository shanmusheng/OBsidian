官网[path_provider |Flutter 封装 (pub.dev)](https://pub.dev/packages/path_provider)
# 官方介绍
一个 Flutter 插件，用于查找文件系统上常用的位置。 支持 Android、iOS、Linux、macOS 和 Windows。 并非所有平台都支持所有方法。
支持的平台和路径
![[path_provider支持的平台和路径.png]]
# 学习路径
## 常见API用法
### **获取临时目录**：

使用 `getTemporaryDirectory()` 方法可以获取临时目录路径。临时目录用于存储临时文件，当应用程序不再需要这些文件时，可以将其删除。系统可能会在应用程序不运行时清理这些文件。
### 获取文档目录
使用 `getApplicationDocumentsDirectory()` 方法可以获取应用程序的文档目录路径。这个目录适合存储用户生成的数据或需要长期保存的文件。文档目录中的文件在应用程序卸载前不会被系统清理。
### 获取外部存储目录
在 Android 上，可以使用 `getExternalStorageDirectory()` 获取外部存储目录路径。这个目录通常位于 SD 卡上，适合存储需要在应用之间共享的文件。需要注意的是，访问外部存储需要适当的权限。
### 获取应用程序支持目录
使用 `getApplicationSupportDirectory()` 方法可以获取应用程序支持目录路径。这个目录适合存储不需要展示给用户的文件，比如缓存文件或配置文件。