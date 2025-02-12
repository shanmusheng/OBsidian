#package_info_plus
[package_info_plus | Flutter package](https://pub.dev/packages/package_info_plus)

```
import 'package:package_info_plus/package_info_plus.dart';

Future<void> getAppVersion() async {
  // 获取应用的版本信息
  PackageInfo packageInfo = await PackageInfo.fromPlatform();

  String version = packageInfo.version;  // 版本号
  String buildNumber = packageInfo.buildNumber;  // 构建号
  String appName = packageInfo.appName; //app名称
  String packageName = packageInfo.packageName;/包名
  print('App Version: $version');
  print('Build Number: $buildNumber');
}

```
![[Pasted image 20250212113700.png]]