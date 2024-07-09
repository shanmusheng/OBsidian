官网[permission_handler |Flutter 封装 (pub.dev)](https://pub.dev/packages/permission_handler)
# 官方介绍
在大多数操作系统上，权限不仅在安装时授予应用。 相反，开发人员必须在应用程序运行时请求用户的许可。

该插件提供了一个跨平台（iOS、Android）API 来请求权限并检查其状态。 您还可以打开设备的应用设置，以便用户授予权限。  
在 Android 上，您可以显示请求权限的理由。
# 学习路径
## 用途
该插件是用来管理设备权限的控制,比如允许使用照相机,录音等
## 用法(针对安卓)
### 首先是配置安卓权限

  在 `AndroidManifest.xml` 中添加需要请求的权限。例如:
```
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

### 然后在代码里调用`permission_handler`提供的API:

#### 请求单个权限
```
// 请求单个权限
Future<void> requestCameraPermission() async {
  if (await Permission.camera.request().isGranted) {
    // 权限已授予
  } else {
    // 权限被拒绝
  }
}
```

#### 请求多个权限
```
Future<void> requestMultiplePermissions() async {
  Map<Permission, PermissionStatus> statuses = await [
    Permission.camera,
    Permission.location,
    Permission.storage,
  ].request();

  // 处理每个权限的状态
  if (statuses[Permission.camera].isGranted) {
    // 相机权限已授予
  }
  if (statuses[Permission.location].isGranted) {
    // 位置权限已授予
  }
  if (statuses[Permission.storage].isGranted) {
    // 存储权限已授予
  }
}
```

#### 检查权限状态
```
Future<void> checkPermissions() async {
  if (await Permission.camera.isGranted) {
    // 相机权限已授予
  } else if (await Permission.camera.isDenied) {
    // 相机权限被拒绝
  } else if (await Permission.camera.isPermanentlyDenied) {
    // 相机权限被永久拒绝，需要打开应用设置手动授予
    openAppSettings();
  }
}

```
## 实例


```
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: PermissionExample(),
    );
  }
}

class PermissionExample extends StatefulWidget {
  @override
  _PermissionExampleState createState() => _PermissionExampleState();
}

class _PermissionExampleState extends State<PermissionExample> {
  Future<void> _checkAndRequestCameraPermission() async {
    if (await Permission.camera.isGranted) {
      // 相机权限已授予
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Camera permission granted')),
      );
    } else if (await Permission.camera.isDenied) {
      // 请求相机权限
      PermissionStatus status = await Permission.camera.request();
      if (status.isGranted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Camera permission granted')),
        );
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Camera permission denied')),
        );
      }
    } else if (await Permission.camera.isPermanentlyDenied) {
      // 相机权限被永久拒绝，引导用户打开设置
      bool opened = await openAppSettings();
      if (opened) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Opened app settings')),
        );
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Failed to open app settings')),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Permission Handler Example'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: _checkAndRequestCameraPermission,
          child: Text('Check Camera Permission'),
        ),
      ),
    );
  }
}

```