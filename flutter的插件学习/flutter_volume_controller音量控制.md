#flutter_volume_controller 
官网[flutter_volume_controller | Flutter package (pub.dev)](https://pub.dev/packages/flutter_volume_controller)
# 官方介绍
一个 Flutter 插件，用于控制系统音量并监听不同平台上的音量变化。
- 控制系统和媒体卷。
- 侦听音量变化。
A Flutter plugin to control system volume and listen for volume changes on different platforms.
目前基于flutter_volume_controller: ^1.3.2,是可以在Android,iOS,macOS,Windows,Linux上支持使用
![[Pasted image 20240923180239.png]]
# 学习路径
## 使用
### 获取音量
```
    double? volume = await FlutterVolumeController.getVolume();
```
### 设置音量
```
FlutterVolumeController.setVolume(volume);
```
