#screen_brightness
官网[screen_brightness |Flutter 软件包 (pub.dev)](https://pub.dev/packages/screen_brightness)\
# 官方介绍
一个 Plugin，用于控制屏幕亮度，并实现了应用程序生命周期重置。

此插件仅更改应用程序亮度，而不更改系统亮度。所以这个插件不需要权限。
# 用法
这个插件控制应用程序亮度指的是只在这个应用里的亮度,不会控制手机系统的那个亮度
## 设置亮度
`ScreenBrightness().setScreenBrightness(value);`
不过这个是异步函数
```
  Future<void> _setBrightness(double value) async {
    await ScreenBrightness().setScreenBrightness(value);
  }
```
## 获取当前亮度
`ScreenBrightness().current;`
一样也是异步函数
```
Future<void> _getCurrentBrightness() async {  
  double brightness = await ScreenBrightness().current;   
}
```
# 例子
## 滑块控制屏幕亮度
首先定义一个常量来保存亮度数值
`double _brightness = 0.5;`
然后在应用init时,调取获取当前亮度的方法,把`_brightness`改为当前的亮度值
```
Future<void> _getCurrentBrightness() async {  
  double brightness = await ScreenBrightness().current;  
  setState(() {  
    _brightness = brightness;  
  });  
}

@override  
void initState() {  
  super.initState();  
  _getCurrentBrightness();  
}
```
然后再加上一个控制屏幕亮度的方法
```
Future<void> _setBrightness(double value) async {  
  await ScreenBrightness().setScreenBrightness(value);  
  setState(() {  
    _brightness = value;  //记得修改_brightness
  });  
}
```
滑块部分
```
Slider(  
  value: _brightness,  //亮度数值
  onChanged: _setBrightness,  //控制屏幕亮度的方法
  min: 0.0,  
  max: 1.0,  
),
```
到此为止实现了滑块控制屏幕亮度
### 完整代码
```
import 'package:flutter/material.dart';
import 'package:screen_brightness/screen_brightness.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Brightness Control',
      home: BrightnessControl(),
    );
  }
}

class BrightnessControl extends StatefulWidget {
  @override
  _BrightnessControlState createState() => _BrightnessControlState();
}

class _BrightnessControlState extends State<BrightnessControl> {
  double _brightness = 0.5;

  @override
  void initState() {
    super.initState();
    _getCurrentBrightness();
  }

  Future<void> _getCurrentBrightness() async {
    double brightness = await ScreenBrightness().current;
    setState(() {
      _brightness = brightness;
    });
  }

  Future<void> _setBrightness(double value) async {
    await ScreenBrightness().setScreenBrightness(value);
    setState(() {
      _brightness = value;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('控制屏幕亮度')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('当前亮度: ${(_brightness * 100).round()}%'),
            Slider(
              value: _brightness,
              onChanged: _setBrightness,
              min: 0.0,
              max: 1.0,
            ),
          ],
        ),
      ),
    );
  }
}

```