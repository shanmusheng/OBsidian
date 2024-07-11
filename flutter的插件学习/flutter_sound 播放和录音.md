#flutter_sound 
官网[flutter_sound | Flutter package (pub.dev)](https://pub.dev/packages/flutter_sound)
[Flutter Sound | The τ Project documentation. (canardoux.xyz)](https://flutter-sound.canardoux.xyz/readme.html)
# 官方介绍
Flutter Sound 是一个 Flutter 软件包，允许您播放和录制以下音频：

- 人造人
- iOS系统
- Flutter Web

Flutter Sound 为以下用途提供了高级 API 和小部件：

- 播放音频
- 录制音频

Flutter Sound 可用于播放从资产一直到实现完整媒体播放器的哔哔声。

该库在 Mozilla 公共许可证 MPL2.0 下发布。

如果您的 App 将受 GPL 许可证的约束， 您可能要考虑使用 GPL [Tau Sound Project 9.0](https://pub.dev/packages/tau_sound)：与 Flutter Sound 8.3 相比，Tau Sound 9.0 提供了多项增强功能。
# 学习路径
## 播放
###  1. FlutterSoundPlayer 实例化
要播放某些内容，您必须实例化播放器。大多数时候，你只需要一个玩家，你可以把这个实例放在你的类的变量初始化中：
```
 FlutterSoundPlayer _myPlayer = FlutterSoundPlayer();
```
### 2. 打开和关闭音频会话
在调用之前，您必须打开会话。`startPlayer()`

完成后，**必须**关闭会话。放置这些动词的好地方是 过程 和 .`initState()``dispose()`
```
@override
  void initState() {
    super.initState();
    _myPlayer.openAudioSession().then((value) {
      setState(() {
        _mPlayerIsInited = true;
      });
    });
  }



  @override
  void dispose() {
    // Be careful : you must `close` the audio session when you have finished with it.
    _myPlayer.closeAudioSession();
    _myPlayer = null;
    super.dispose();
  }
```
### 3.播放你的声音
要播放您调用的声音，请 .停止呼叫的声音`startPlayer()``stopPlayer()`
```
void play() async {
    await _myPlayer.startPlayer(
      fromURI: _exampleAudioFilePathMP3,
      codec: Codec.mp3,
      whenFinished: (){setState((){});}
    );
    setState(() {});
  }

  Future<void> stopPlayer() async {
    if (_myPlayer != null) {
      await _myPlayer.stopPlayer();
    }
  
```
## 录音
### 1. FlutterSoundRecorder 实例化

要播放某些内容，您必须实例化录音机。大多数时候，你只需要一个记录器，你可以把这个实例放在你的类的变量初始化中：

```
  FlutterSoundRecorder _myRecorder = FlutterSoundRecorder();
```

### 2. 打开和关闭音频会话

在调用之前，您必须打开会话。`startRecorder()`

完成后，**必须**关闭会话。放置这些动词的上帝位置是在程序中。`initState()``dispose()`

```
@override
  void initState() {
    super.initState();
    // Be careful : openRecorder return a Future.
    // Do not access your FlutterSoundPlayer or FlutterSoundRecorder before the completion of the Future
    _myRecorder.openRecorder().then((value) {
      setState(() {
        _mRecorderIsInited = true;
      });
    });
  }



  @override
  void dispose() {
    // Be careful : you must `close` the audio session when you have finished with it.
    _myRecorder.closeRecorder();
    _myRecorder = null;
    super.dispose();
  }
```

### 3.记录一些东西

要录制您称为 .停止您调用的录音机`startRecorder()``stopRecorder()`

```
  Future<void> record() async {
    await _myRecorder.startRecorder(
      toFile: _mPath,
      codec: Codec.aacADTS,
    );
  }


  Future<void> stopRecorder() async {
    await _myRecorder.stopRecorder();
  }
```