```
import 'dart:async';
import 'dart:ffi';
import 'dart:math';
import 'dart:ui';

import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';
import 'package:flutter_blue_plus/flutter_blue_plus.dart';
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart' as fbs;
import 'package:flutter_screenutil/flutter_screenutil.dart';
import 'package:flutter_volume_controller/flutter_volume_controller.dart';
import 'package:provider/provider.dart';

import '../../app_theme.dart';
import '../../generated/l10n.dart';
import '../../models/public/wifi_model.dart';
import '../../routes/monar_routes.dart';
import '../../utils/ble_provider.dart';
import '../../utils/blue_classic_provider.dart';
import '../../utils/overlay_manage.dart';
import '../../utils/wifi_blue_combined.dart';
import '../../widgets/image_animation.dart';

class V6ProductsDetailsView extends StatefulWidget {
  final DeviceIdentifier address;
  const V6ProductsDetailsView({super.key, required this.address});

  @override
  State<V6ProductsDetailsView> createState() => _V6ProductsDetailsViewState();
}

class _V6ProductsDetailsViewState extends State<V6ProductsDetailsView>
    with TickerProviderStateMixin {
  ScrollController _scrollController =
      ScrollController(initialScrollOffset: 200.h); //监听展开的值
  double _expandedHeight = 400.h; // 设置初始展开高度
  double currentExtent = 0;
  double currentExtentInW = 217.1.h;
  late BleProvider bleProvider;
  late BleInformation bleInfo;
  BluetoothDevice? bleConnection; //当前连接的设备
  ///
  late CombinedProvider combinedProvider;
  late PageController controller;
  late PageController controllerWifi;
  late AnimationController _controller; //动画旋转
  int currentPage = 0;
  double currentLight = 100.0;
  double currentVolume = 0.2;
  Timer? _timer; // 定时器
  bool isAuto = false; // 自动亮度
  bool isLoading = false; // 标志wifi是否在加载中
  bool setWifi = false; // 标志wifi正在输入密码
  bool _isRotating = false; //刷新wifi
  bool _isDebounced = false; // 刷新wifi防抖状态
  bool deleteWifi = false; // 刷新wifi防抖状态
  String currentWifi = ''; //删除的wifi
  ///关机模式
  double _dx = (315 - 153).w; // 记录 X 轴偏移量
  double _maxDx = (315 - 153).w; // 最大右滑偏移量
  bool _isHolding = false; // 是否按住 3 秒
  bool _restartTriggered = false; // 防止重启和关机同时触发
  bool _isDisabled = false; // 是否禁用滑动
  Timer? _holdTimer; // 计时器
  bool right = false;
  late AnimationController _controllerShut;
  late Animation<double> _animation;
  late final TextEditingController textEditingController;
  String _charCount = ''; //字数显示
  final FocusNode textFieldFocusNode = FocusNode();
  bool _isListening = false; // 添加标志位
  bool isTextFieldEnabled = false; // 变量来控制是否允许焦点
  ///亮度模式
  int lightMode = 0;
  List<LightMode> lightModes = [
    LightMode(
      mode: 0,
      img: 'assets/final_icons/Dark Night.png',
      str: 'splash_home_device_details_Speaker_brightness_display_daytime',
      light: 32.0,
    ),
    LightMode(
      mode: 1,
      img: 'assets/final_icons/Soft Dusk.png',
      str: 'splash_home_device_details_Speaker_brightness_display_cloudy',
      light: 128.0,
    ),
    LightMode(
      mode: 2,
      img: 'assets/final_icons/day_hight.png',
      str: 'splash_home_device_details_Speaker_brightness_display_At_night',
      light: 255.0,
    ),
    // LightMode(
    //   mode: 3,
    //   img: 'assets/icons/png/pngX4/Group_1921 (2).png',
    //   str:
    //       'splash_home_device_details_Speaker_brightness_display_Turn_off_the_screen',
    //   light: 0,
    // ),
  ];
  @override
  void initState() {
    super.initState();
    _initVolume();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500), // 动画持续时间
    );
    // 监听滚动变化
    _scrollController.addListener(_scrollListener);
    controller = PageController();
    // 获取蓝牙状态管理
    controllerWifi = PageController();
    bleProvider = Provider.of<BleProvider>(context, listen: false);
    combinedProvider = Provider.of<CombinedProvider>(context, listen: false);
    currentLight = combinedProvider.allStatus.luminosity?.toDouble() ?? 0.0;
    print('亮度为${currentLight}');
    isAuto = combinedProvider.allStatus.isAutoBrightOn;

    ///获取蓝牙信息
    bleInfo = bleProvider.bleInfoList.firstWhere(
        (bleInfo) => bleInfo.deviceId == widget.address,
        orElse: () => BleInformation(deviceId: const DeviceIdentifier('')));

    ///获取蓝牙连接设备
    bleConnection =
        bleProvider.v6GetConnection(widget.address.toString() ?? '');

    // 检查连接状态并获取Wi-Fi信息
    if (bleConnection?.isConnected ?? false) {
      WidgetsBinding.instance.addPostFrameCallback((_) {
        _initProcess();
      });
    }
    startListening();

    _controllerShut = AnimationController(
      vsync: this,
      duration: Duration(milliseconds: 300), // 回弹动画时间
    );
    _animation = Tween<double>(begin: 0, end: 0).animate(_controllerShut)
      ..addListener(() {
        setState(() {
          _dx = _animation.value;
        });
      });
    textEditingController = TextEditingController()..addListener(_onTextChange);
    textEditingController.text = bleInfo.deviceName ?? "";
    _charCount = textEditingController.text;
  }

  Future<void> _initProcess() async {
    if (_isDebounced) return; // 如果已经在处理，直接返回

    print('getWifiNetworksAndSetIP开始');
    _isDebounced = true;
    setState(() {}); // 更新UI

    // 清空当前网络数据
    bleProvider.networksData?.clear();

    // 启动旋转动画
    _controller.repeat();

    // 执行异步操作
    await bleProvider.getWifiNetworksAndSetIP();

    print('getWifiNetworksAndSetIP结束');

    // 停止旋转动画
    _controller.stop();
    _controller.reset();

    // 更新UI
    setState(() {});

    // 解除防抖操作
    _isDebounced = false;
  }

  //文字编辑
  void _onTextChange() {
    // bleInfo.renameImageGroup(textEditingController.text);
    bleInfo.deviceName = textEditingController.text;
    bleProvider.saveBlueInfoList(bleInfo);
    setState(() {});
    if (textEditingController.text == '') {
      // bleInfo.renameImageGroup(_charCount);
      bleInfo.deviceName = _charCount;
      // textEditingController.text = _charCount;
    }
  }

  //关机动画右
  void _animateRightBack() {
    _animation = Tween<double>(begin: _dx, end: 0).animate(
      CurvedAnimation(parent: _controllerShut, curve: Curves.easeOut),
    );
    _controllerShut.forward(from: 0);
  }

  //关机动画左
  void _animateLiftBack() {
    _animation = Tween<double>(begin: _dx, end: _maxDx).animate(
      CurvedAnimation(parent: _controllerShut, curve: Curves.easeOut),
    );
    _controllerShut.forward(from: 0);
  }

  /// 开始 3 秒倒计时（检测是否触发重启）
  void _startHoldTimer() {
    _isHolding = true;
    _restartTriggered = false; // 先默认未触发重启
    _holdTimer = Timer(Duration(seconds: 3), () {
      if (_isHolding) {
        print('🔄 触发重启');
        bleProvider.reboot(); // 触发重启
        setState(() {
          _isDisabled = true; // 禁用滑动
        });
        _restartTriggered = true; // 记录重启已触发
      }
    });
  }

  void _cancelHoldTimer() {
    _isHolding = false;
    _holdTimer?.cancel();
  }

  /// 触发关机
  void _triggerShutdown() {
    if (!_restartTriggered) {
      print('⏻ 触发关机');
      setState(() {
        _isDisabled = true; // 禁用滑动
      });
      bleProvider.shutdown(); // 触发关机
    }
  }

  @override
  void dispose() {
    _scrollController.removeListener(_scrollListener);
    _scrollController.dispose();
    _controller.dispose();
    _controllerShut.dispose();
    controller.dispose();

    textEditingController.dispose();
    textFieldFocusNode.dispose();
    stopListening();
    super.dispose();
  }

// 重新创建可用的 FocusNode
  void _enableAndRequestFocus() {
    setState(() {
      isTextFieldEnabled = true; // 允许输入框获取焦点
    });
    Future.delayed(Duration(milliseconds: 100), () {
      FocusScope.of(context).requestFocus(textFieldFocusNode); // 触发焦点
    });
  }

// 取消焦点，隐藏键盘
  void _unfocusTextField() {
    setState(() {
      isTextFieldEnabled = false; // 允许输入框获取焦点
    });
    FocusScope.of(context).unfocus(); // 取消焦点，隐藏键盘
  }

// 滚动监听器
  void _scrollListener() {
    currentExtent = _scrollController.position.extentBefore;
    // 获取设备的实际宽度和设计稿的宽度
    double screenHight = ScreenUtil().screenHeight;
    double designHight = 844; // 假设设计稿宽度为375

    // 计算适配比例
    double ratio = screenHight / designHight;
    print(ratio);
    // 将像素值转换为适配单位（h）
    currentExtentInW = currentExtent / ratio;
    print('currentExtent${currentExtent}'); //x
    print('currentExtentInW${currentExtentInW}'); //x.h
    setState(() {
      _expandedHeight = 600.h - currentExtentInW; // 计算当前展开高度
    });
    print('_expandedHeight${_expandedHeight / ratio}');
  }

  // 初始化音量
  Future<void> _initVolume() async {
    currentVolume = await FlutterVolumeController.getVolume() ??
        0.1; // _currentVolume = (await FlutterVolumeController.getVolume())!;
  }

  /// 停止监听
  void stopListening() {
    _isListening = false; // 停止监听

    _timer?.cancel();
    _timer = null;
    print(' 停止监听');
  }

  /// 调用 sendControlMusicInfo 方法并更新状态
  Future<void> _fetchMusicInfo() async {
    print('获取音乐信息开始:_fetchMusicInfo ');
    try {
      await combinedProvider.sendControlMusicInfo();
    } catch (e) {
      print('获取音乐信息失败2222: $e');
      // stopListening();
    }
  }

  /// 启动定时器，每 10 秒调用一次 sendControlMusicInfo
  /// 启动监听，确保每次完成后才等待 3 秒
  void startListening() {
    if (_isListening) return; // 防止重复启动
    _isListening = true;

    Future<void> _delayedFetchMusicInfo() async {
      while (_isListening) {
        // 只有在监听状态下才执行
        await _fetchMusicInfo(); // 等待执行完成
        if (_isListening) {
          await Future.delayed(const Duration(seconds: 3)); // 延迟 3 秒
        }
      }
    }

    _delayedFetchMusicInfo(); // 启动异步任务
  }

  @override
  Widget build(BuildContext context) {
    final bleProvider = Provider.of<BleProvider>(context);
    // final blueClassicProvider = Provider.of<BlueClassicProvider>(context);
    final combinedProvider = Provider.of<CombinedProvider>(context);

    /// 处理返回按钮
    Future<bool> onWillPop() async {
      if (bleProvider.blueOverlayVisible) {
        return false; // 阻止默认的返回操作
      }
      if (OverlayManager.loadOverlayVisible) {
        return false; // 阻止默认的返回操作
      }
      if (OverlayManager.isOverlayVisible) {
        OverlayManager.removeOverlay();
        return false; // 阻止默认的返回操作
      }
      return true; // 允许默认的返回操作
    }

    return WillPopScope(
      onWillPop: onWillPop, // 捕捉返回按钮事件
      child: GestureDetector(
        onTap: _unfocusTextField, // 点击空白处取消焦点
        child: Scaffold(
          backgroundColor: AppTheme.v6ThemeBlank,
          body: Stack(
            children: [
              CustomScrollView(
                physics: ClampingScrollPhysics(), // 这会禁用反弹效果，像 Android

                controller: _scrollController,
                slivers: [
                  SliverAppBar(
                    floating: true,
                    pinned: false, // 固定住标题，避免滚动时被遮住
                    expandedHeight: 600.h,
                    backgroundColor: AppTheme.v6ThemeBlank,
                    centerTitle: true, stretch: true, // 启用拉伸效果
                    toolbarHeight: 50.h,
                    title: Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        GestureDetector(
                          onTap: () {
                            if (isTextFieldEnabled) {
                              textFieldFocusNode.requestFocus(); // 只有允许时才请求焦点
                            }
                          },
                          child: Container(
                            constraints: BoxConstraints(
                              maxWidth: 180.w,
                            ),
                            child: TextField(
                              enableInteractiveSelection: isTextFieldEnabled,
                              showCursor: isTextFieldEnabled, // 只有在允许输入时显示光标
                              readOnly: !isTextFieldEnabled, // 控制是否允许直接点击获取焦点
                              focusNode: textFieldFocusNode, // 绑定 FocusNode
                              controller: textEditingController,
                              textAlign: TextAlign.center,
                              decoration: InputDecoration(
                                hintStyle: TextStyle(
                                    fontSize: 20.sp,
                                    color: Colors.white,
                                    fontWeight: FontWeight.bold),
                                hintText: bleInfo.deviceName ?? "",
                                border: InputBorder.none,
                              ),
                              style: TextStyle(
                                  fontSize: 20.sp,
                                  color: Colors.white,
                                  fontWeight: FontWeight.bold),
                            ),
                          ),
                        ),
                        SizedBox(
                          width: 10.w,
                        ),
                        GestureDetector(
                          onTap: _enableAndRequestFocus,
                          child: Image.asset(
                            'assets/final_icons/edit.png',
                            scale: 4,
                          ),
                        ),
                        SizedBox(
                          width: 30.w,
                        )
                      ],
                    ),
                    // title: Text(
                    //   bleInfo.deviceName ?? "",
                    //   style: TextStyle(
                    //     fontWeight: FontWeight.bold,
                    //     fontSize: 20.sp,
                    //     color: Colors.white,
                    //   ),
                    // ),
                    leading: GestureDetector(
                      onTap: () {
                        Navigator.pop(context);
                      },
                      child: Image.asset(
                        'assets/final_icons/retrun.png',
                        scale: 4,
                      ),
                    ),
                    flexibleSpace: FlexibleSpaceBar(
                      // expandedTitleScale: 2,

                      titlePadding: EdgeInsets.only(),
                      background: Stack(
                        // fit: StackFit.loose,
                        children: [
                          ImageFiltered(
                            imageFilter: ImageFilter.blur(
                                sigmaX: 180,
                                sigmaY: 180,
                                tileMode: TileMode.decal),
                            child: Image.asset(
                              'assets/final_icons/image 297.png',
                              fit: BoxFit.cover,
                              height: 620.h,
                              width: 390.w,
                            ),
                          ),
                          Positioned(
                            top: _expandedHeight / 6, // / 1.h
                            right: 0, left: 0,
                            child: AnimatedBuilder(
                              animation: _scrollController,
                              builder: (context, child) {
                                double scale = 1.0; // 默认的缩放倍数
                                scale = 1.0 -
                                    currentExtentInW /
                                        200.h *
                                        0.4; // 假设最小缩放倍数为 0.5
                                return Transform.scale(
                                  scale: scale, // 设置缩放倍数
                                  child: Container(
                                    width: 427.w,
                                    height: 427.w,
                                    decoration: BoxDecoration(
                                        color: Colors.transparent,
                                        shape: BoxShape.circle,
                                        boxShadow: [
                                          BoxShadow(
                                            // color: Color(0xffdff111)
                                            //     .withOpacity(1),
                                            color: Color(0xffA22124)
                                                .withOpacity(0.6),
                                            blurRadius: 60,
                                          ),
                                        ]),
                                  ), // 你的标题内容
                                );
                              },
                            ),
                          ),
                          Positioned(
                            top: _expandedHeight / 1.11 + 10.h, // / 1.h
                            child: IgnorePointer(
                              ignoring: right,
                              child: SizedBox(
                                  width: 390.w, child: builditemMode()),
                            ),
                          ),
                        ],
                      ),
                      // centerTitle: true,
                      title: Container(
                        margin: EdgeInsets.only(
                          // top: 260.h - (currentExtent * 200 / 248), //这里是60-260
                          top: max(100.h, 265.h - currentExtentInW), //这里是60-260
                          bottom: 0.h,
                        ),
                        child: Stack(
                          children: [
                            Align(
                              alignment: Alignment.topCenter,
                              child: AnimatedBuilder(
                                animation: _scrollController,
                                builder: (context, child) {
                                  double scale = 1.0; // 默认的缩放倍数
                                  scale = 1.0 -
                                      currentExtentInW /
                                          200.h *
                                          0.5; // 假设最小缩放倍数为 0.5
                                  return Transform.scale(
                                    scale: scale, // 设置缩放倍数
                                    child: Image.asset(
                                      'assets/design/splash/珍珠耳环_20.png',
                                      scale: 4,
                                    ), // 你的标题内容
                                  );
                                },
                              ),
                            ),
                          ],
                        ),
                      ),
                      stretchModes: [StretchMode.zoomBackground], // 拉伸背景并放大
                    ),
                  ),
                  SliverToBoxAdapter(
                    child: Container(
                      color: Color(0xff461C16).withOpacity(0.4),
                      child: ClipRRect(
                        borderRadius: BorderRadius.only(
                          topLeft: Radius.circular(30.0.r),
                          topRight: Radius.circular(30.0.r),
                        ),
                        child: Container(
                          width: 360.w,
                          height: 417.h,
                          color: Colors.transparent,
                          child: PageView(
                            physics: NeverScrollableScrollPhysics(),
                            controller: controller,
                            children: [
                              wifiCard(context, bleProvider),
                              lightContainer(context),
                              musicCard(context, combinedProvider),
                              shutCrad(bleProvider),

                              // ShutdownCard(bleProvider: bleProvider),
                            ],
                          ),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
              IgnorePointer(
                child: ClipRRect(
                  child: Opacity(
                    opacity: (1 - _dx / _maxDx).clamp(0.0, 1.0),
                    child: BackdropFilter(
                      filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
                      child: Container(
                        width: 390.w,
                        height: _expandedHeight / 0.9 - 35.h, // / 1.h
                        decoration:
                            BoxDecoration(color: Colors.black.withOpacity(0.3)),
                      ),
                    ),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Container shutCrad(BleProvider bleProvider) {
    return Container(
        height: 360.h,
        width: 390.w,
        padding:
            EdgeInsets.symmetric(vertical: 15.h, horizontal: (390 - 345).w / 2),
        decoration: BoxDecoration(
          color: AppTheme.white,
          borderRadius: BorderRadius.circular(15.r),
        ),
        child: Column(
          children: [
            SizedBox(
              height: 20.h,
            ),
            GestureDetector(
              onHorizontalDragUpdate: _isDisabled
                  ? null
                  : (details) {
                      setState(() {
                        _dx += details.delta.dx; // 拖动更新 X 轴
                        print(_dx);
                        if (_dx <= 10) {
                          _dx = 0; // 限制最小值
                          if (!_isHolding) {
                            print('🟢 进入 OFF 持续检测');
                            _startHoldTimer(); // 开始倒计时
                          }
                        } else {
                          print('🟢 退出 OFF 持续检测');
                          _cancelHoldTimer(); // 取消倒计时
                        }
                      });
                    },
              onHorizontalDragEnd: _isDisabled
                  ? null
                  : (details) {
                      _cancelHoldTimer(); // 松手时取消计时
                      if (_dx <= 10) {
                        _triggerShutdown(); // 触发关机（若未触发重启）
                        right = true; // 记录当前状态为 OFF
                      }
                      if (right) {
                        _animateRightBack(); // 右滑超过一定值后，松手会自动回到初始位置
                      } else {
                        _animateLiftBack();
                      }
                    },
              child: Container(
                width: 345.w,
                height: 64.h,
                padding: EdgeInsets.all(10.w),
                decoration: BoxDecoration(
                    color: Color(0xffF3F4F9),
                    borderRadius: BorderRadius.circular(12.r)),
                child: Stack(
                  children: [
                    SizedBox(
                      width: 315.w,
                      height: 46.h,
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Container(
                            width: 50.w,
                            height: 46.h,
                            decoration: BoxDecoration(
                              color: Color(0xffE1E1E1),
                              borderRadius: BorderRadius.circular(23.r),
                            ),
                            child: Center(
                              child: Text(
                                'OFF',
                                style: TextStyle(
                                    fontSize: 20.sp,
                                    color: Colors.white,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                          ),
                          Container(
                            width: 50.w,
                            height: 46.h,
                            decoration: BoxDecoration(
                              color: Color(0xffE1E1E1),
                              borderRadius: BorderRadius.circular(23.r),
                            ),
                            child: Center(
                              child: Text(
                                'ON',
                                style: TextStyle(
                                    fontSize: 20.sp,
                                    color: Colors.white,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                          ),
                        ],
                      ),
                    ),
                    Transform.translate(
                      offset: Offset(_dx, 0), // 应用水平偏移
                      child: Container(
                        width: 153.w,
                        height: 46.h,
                        decoration: BoxDecoration(
                          color: AppTheme.v6Red2,
                          borderRadius: BorderRadius.circular(23.r),
                        ),
                        alignment: Alignment.center,
                        child: Text(
                          right ? "OFF" : "ON",
                          style: TextStyle(
                              fontSize: 20.sp,
                              color: Colors.white,
                              fontWeight: FontWeight.bold),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ));
  }

  ///音乐模块
  Container musicCard(BuildContext context, CombinedProvider combinedProvider) {
    return Container(
      height: 360.h,
      width: 390.w,
      padding:
          EdgeInsets.symmetric(vertical: 15.h, horizontal: (390 - 346).w / 2),
      decoration: BoxDecoration(
        color: AppTheme.white,
        borderRadius: BorderRadius.circular(15.r),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            height: 20,
          ),
          Text(
            S.of(context).splash_home_device_details_enjoy_the_music,
            style: AppTheme.v6blank16bold,
          ),
          Container(
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),
            height: 163.h,
            width: 346.w,
            padding: EdgeInsets.symmetric(vertical: 10.w, horizontal: 20.h),
            decoration: BoxDecoration(
                color: AppTheme.v6ThemeColors,
                borderRadius: BorderRadius.circular(12.r)),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                SizedBox(
                  width: 10.w,
                ),
                Row(
                  children: [
                    bleConnection?.isConnected ?? false
                        ? combinedProvider.infoResult.title != ''
                            ? ClipRRect(
                                borderRadius: BorderRadius.circular(8.r),
                                child: Image.asset(
                                  'assets/design/dynamic/album.png',
                                  width: 55.w,
                                  height: 55.w,
                                  fit: BoxFit.contain,
                                ),
                              )
                            : ClipRRect(
                                borderRadius: BorderRadius.circular(8.r),
                                child: Container(
                                  width: 55.w,
                                  height: 55.w,
                                  decoration: BoxDecoration(
                                    color: Color(0xffE7E7E7),
                                    borderRadius: BorderRadius.circular(8.r),
                                  ),
                                  child: Center(
                                    child: Image.asset(
                                      'assets/final_icons/Music.png',
                                      width: 30.w,
                                      height: 30.w,
                                      scale: 4,
                                      fit: BoxFit.contain,
                                    ),
                                  ),
                                ),
                              )
                        : ClipRRect(
                            borderRadius: BorderRadius.circular(8.r),
                            child: Container(
                              width: 55.w,
                              height: 55.w,
                              decoration: BoxDecoration(
                                color: Color(0xffE7E7E7),
                                borderRadius: BorderRadius.circular(8.r),
                              ),
                              child: Center(
                                child: Image.asset(
                                  'assets/final_icons/Music.png',
                                  width: 30.w,
                                  height: 30.w,
                                  scale: 4,
                                  fit: BoxFit.contain,
                                ),
                              ),
                            ),
                          ),
                    SizedBox(
                      width: 40.w,
                    ),
                    bleConnection?.isConnected ?? false
                        ? combinedProvider.infoResult.title != ''
                            ? Column(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                mainAxisAlignment: MainAxisAlignment.start,
                                children: [
                                  SizedBox(
                                    width: 180.w,
                                    child: Text(
                                      combinedProvider.infoResult.title == ''
                                          ? "Die with a smile"
                                          : combinedProvider.infoResult.title ??
                                              "Die with a smile",
                                      style: TextStyle(
                                          fontSize: 16.sp,
                                          overflow: TextOverflow.ellipsis,
                                          fontWeight: FontWeight.bold,
                                          color: Colors.black),
                                    ),
                                  ),
                                  SizedBox(
                                    width: 180.w,
                                    child: Text(
                                      combinedProvider.infoResult.artist == ''
                                          ? "Lady gaga & Bruno Mars"
                                          : combinedProvider
                                                  .infoResult.artist ??
                                              "Lady gaga & Bruno Mars",
                                      overflow: TextOverflow.ellipsis,
                                      style: TextStyle(
                                        fontSize: 13.sp,
                                        color: AppTheme.v6AF,
                                      ),
                                    ),
                                  ),
                                ],
                              )
                            : Text(
                                'No music playing',
                                style: TextStyle(
                                    fontSize: 16.sp,
                                    fontWeight: FontWeight.bold,
                                    color: Color(0xffAFAFAF)),
                              )
                        : Text(
                            S.of(context).splash_home_device_no_music,
                            style: TextStyle(
                                fontSize: 16.sp,
                                fontWeight: FontWeight.bold,
                                color: Color(0xffAFAFAF)),
                          ),
                  ],
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    InkWell(
                      child: Container(
                          width: 30.w,
                          height: 30.h,
                          margin: EdgeInsets.symmetric(horizontal: 10.w),
                          child: Image.asset(
                            'assets/icons/png/pngX4/上一首_1.png',
                            fit: BoxFit.contain,
                          )),
                      onTap: () async {
                        setState(() {});
                        // await skipPrevious();
                        await combinedProvider.sendControlMusic(3);
                      },
                    ),
                    // combinedProvider.infoResult.status == "playing"
                    combinedProvider.play
                        ? InkWell(
                            onTap: () async {
                              combinedProvider.play = false;

                              setState(() {});
                              try {
                                await combinedProvider.sendControlMusic(1);
                                combinedProvider.play = false;

                                setState(() {});
                              } catch (e) {
                                combinedProvider.play = true;

                                setState(() {});
                              }

                              // await pause();
                              // isplay = false;
                            },
                            child: Container(
                              width: 30.w,
                              height: 30.h,
                              margin: EdgeInsets.symmetric(horizontal: 10.w),
                              child: Image.asset(
                                  'assets/icons/png/播放器-暂停_44_1.png'),
                            ))
                        : InkWell(
                            onTap: () async {
                              combinedProvider.play = true;
                              setState(() {});
                              try {
                                await combinedProvider.sendControlMusic(0);
                                combinedProvider.play = true;
                                setState(() {});
                              } catch (e) {
                                combinedProvider.play = false;
                                setState(() {});
                              }

                              // await resume();
                              // isplay = true;
                            },
                            child: Container(
                              width: 30.w,
                              height: 30.h,
                              margin: EdgeInsets.symmetric(horizontal: 10.w),
                              child: Image.asset(
                                'assets/icons/png/播放_(6)_1.png',
                                fit: BoxFit.contain,
                              ),
                            )),
                    InkWell(
                        onTap: () async {
                          setState(() {});
                          // await skipNext();
                          await combinedProvider.sendControlMusic(2);
                        },
                        child: Container(
                            width: 30.w,
                            height: 30.h,
                            margin: EdgeInsets.symmetric(horizontal: 10.w),
                            child: Image.asset(
                              'assets/icons/png/pngX4/上一首_2.png',
                              fit: BoxFit.contain,
                            ))),
                  ],
                ),
                Row(
                  children: [
                    Image.asset(
                      'assets/final_icons/sound.png',
                      scale: 4,
                      width: 31.w,
                      height: 23.h,
                      fit: BoxFit.contain,
                    ),
                    Expanded(
                      child: Slider(
                        min: 0,
                        max: 1,
                        value: currentVolume,
                        activeColor: AppTheme.v6Red2,
                        onChanged: (value) {
                          setState(() {});
                          currentVolume = value;
                          print(currentVolume);
                          FlutterVolumeController.setVolume(currentVolume);
                        },
                        onChangeEnd: (value) {
                          setState(() {});
                          print(currentVolume);
                          // FlutterVolumeController.setVolume(
                          //     currentVolume);
                          // combinedProvider.sendLuminosity(
                          //     currentLight.toInt());
                          // sendLuminosity
                        },
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
          Container(
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),
            height: 95.h,
            width: 346.w,
            // padding: EdgeInsets.symmetric(vertical: 12.w, horizontal: 15.h),
            padding: EdgeInsets.only(top: 12.w, left: 15.h, bottom: 12.w),
            decoration: BoxDecoration(
                color: AppTheme.v6ThemeColors,
                borderRadius: BorderRadius.circular(12.r)),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Row(
                  children: [
                    Text(
                      S.of(context).splash_home_device_details_sound_mode,
                      style: AppTheme.v6blank16bold,
                    ),
                    // Image.asset(
                    //   'assets/icons/png/pngX4/均衡器_(1)_9.png',
                    //   scale: 4,
                    // ),
                  ],
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                ),
                SingleChildScrollView(
                  scrollDirection: Axis.horizontal,
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Container(
                        height: 33.h,
                        constraints: BoxConstraints(minWidth: 95.w),
                        // width: 117.w,
                        margin: EdgeInsets.only(right: 10.w),
                        padding: EdgeInsets.symmetric(horizontal: 10.w),
                        decoration: BoxDecoration(
                            color: AppTheme.white,
                            borderRadius: BorderRadius.circular(24.r)),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Image.asset(
                              'assets/icons/png/pngX4/音乐_10.png',
                              width: 18.w,
                              height: 18.w,
                              fit: BoxFit.contain,
                              scale: 4,
                            ),
                            SizedBox(
                              width: 3.w,
                            ),
                            Text(
                              S
                                  .of(context)
                                  .splash_home_device_details_sound_mode_STANDARD,
                              style: AppTheme.v6blank14bold,
                            ),
                          ],
                        ),
                      ),

                      ///
                      Container(
                        height: 33.h,
                        constraints: BoxConstraints(minWidth: 95.w),
                        margin: EdgeInsets.only(right: 10.w),
                        padding: EdgeInsets.symmetric(horizontal: 10.w),
                        decoration: BoxDecoration(
                            color: AppTheme.white,
                            borderRadius: BorderRadius.circular(24.r)),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Image.asset(
                              'assets/icons/png/pngX4/语音_9 (1).png',
                              scale: 4,
                            ),
                            SizedBox(
                              width: 3.w,
                            ),
                            Text(
                              S
                                  .of(context)
                                  .splash_home_device_details_sound_mode_VOCAL,
                              style: AppTheme.v6blank14bold,
                            ),
                          ],
                        ),
                      ),
                      Container(
                        height: 33.h,
                        constraints: BoxConstraints(minWidth: 95.w),
                        margin: EdgeInsets.only(right: 10.w),
                        padding: EdgeInsets.symmetric(horizontal: 10.w),
                        decoration: BoxDecoration(
                            color: AppTheme.white,
                            borderRadius: BorderRadius.circular(24.r)),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Image.asset(
                              'assets/icons/png/pngX4/跳舞_10.png',
                              scale: 4,
                            ),
                            SizedBox(
                              width: 5.w,
                            ),
                            Text(
                              S
                                  .of(context)
                                  .splash_home_device_details_sound_mode_PARTY,
                              style: AppTheme.v6blank14bold,
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Row builditemMode() {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: [
        deviceStatusIcon(
          isConnected: false,
          page: 0,
          connectedImage: 'assets/icons/png/pngX4/wifi.png',
          disconnectedImage: 'assets/icons/png/pngX4/wifi_select.png',
        ),
        deviceStatusIcon(
          isConnected: false,
          page: 1,
          connectedImage: 'assets/icons/png/pngX4/light.png',
          disconnectedImage: 'assets/icons/png/pngX4/light_select.png',
        ),
        deviceStatusIcon(
          isConnected: false,
          page: 2,
          connectedImage: 'assets/icons/png/pngX4/volume.png',
          disconnectedImage: 'assets/icons/png/pngX4/volume_select.png',
        ),
        deviceStatusIcon(
          isConnected: false,
          page: 3,
          connectedImage: 'assets/icons/png/pngX4/power.png',
          disconnectedImage: 'assets/icons/png/pngX4/power_select.png',
        ),
      ],
    );
  }

  ///Wifi模块
  Widget wifiCard(BuildContext context, BleProvider bleProvider) {
    List<String> filteredSsids =
        List.from(combinedProvider.allStatus.currentSsids)
          ..remove(bleProvider.currentWifiName);
    return PageView(
      controller: controllerWifi,
      physics: NeverScrollableScrollPhysics(),
      children: [
        SingleChildScrollView(
          physics: ClampingScrollPhysics(),
          child: Container(
            width: 360.w,
            height: (480 +
                    ((combinedProvider.allStatus.currentSsids.length) * 60)
                        .clamp(0, 170))
                .h,
            padding: EdgeInsets.symmetric(
                vertical: 15.w, horizontal: (390 - 346).w / 2),
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(15.r),
              color: AppTheme.v6White,
            ),
            margin: EdgeInsets.only(bottom: 10.h),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              mainAxisAlignment: MainAxisAlignment.start,
              children: [
                SizedBox(
                  height: 20.h,
                ),

                /// 设备当前连接wifi列表
                Visibility(
                  visible: bleProvider.currentWifiName != '',
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        S
                            .of(context)
                            .splash_home_device_details_net_connect_current,
                        style: TextStyle(
                            fontSize: 16.sp,
                            color: AppTheme.v6Blank,
                            fontWeight: FontWeight.bold),
                      ),
                      GestureDetector(
                        onTap: () async {
                          currentWifi = bleProvider.currentWifiName;
                          deleteWifi = true;
                          setState(() {});
                          controllerWifi.jumpToPage(1);
                        },
                        child: Container(
                          alignment: Alignment.centerLeft,
                          padding: EdgeInsets.symmetric(
                              horizontal: 20.w, vertical: 1.h),
                          margin: EdgeInsets.symmetric(vertical: 15.h),
                          height: 57.h,
                          width: 346.w,
                          decoration: BoxDecoration(
                            color: Color(0xff9D2D2B),
                            borderRadius: BorderRadius.circular(12.r),
                          ),
                          child: Row(
                            // mainAxisAlignment: MainAxisAlignment.spaceBetween,
                            children: [
                              Image.asset(
                                'assets/final_icons/WIfi_4.png',
                                scale: 4,
                                width: 20.w,
                                height: 20.w,
                              ),
                              SizedBox(
                                width: 10.w,
                              ),
                              SizedBox(
                                width: 200.w,
                                child: Text(
                                  bleProvider.currentWifiName,
                                  softWrap: false,
                                  overflow: TextOverflow.ellipsis,
                                  style: TextStyle(
                                      color: AppTheme.white,
                                      fontSize: 16.sp,
                                      fontWeight: FontWeight.bold),
                                ),
                              ),
                              Expanded(child: SizedBox()),
                              Container(
                                // color: Colors.white,
                                width: 50.w,
                                height: 52.h,
                                alignment: Alignment.centerRight,
                                child: Row(
                                  mainAxisAlignment: MainAxisAlignment.end,
                                  children: [
                                    Image.asset(
                                      'assets/final_icons/锁 5.png',
                                      scale: 4,
                                      width: 16.w,
                                      height: 16.w,
                                      fit: BoxFit.cover,
                                    ),
                                    SizedBox(
                                      width: 15.w,
                                    ),
                                    Image.asset(
                                      'assets/final_icons/wifi_to.png',
                                      width: 7.w,
                                      // height: 16.w,
                                      fit: BoxFit.contain,
                                      scale: 4,
                                    ),
                                  ],
                                ),
                              ),
                            ],
                          ),
                        ),
                      ),
                    ],
                  ),
                ),

                /// 设备保存wifi列表
                Visibility(
                  visible: filteredSsids.isNotEmpty,
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            S.of(context).splash_home_device_details_device,
                            style: TextStyle(
                                fontSize: 16.sp,
                                color: AppTheme.v6Blank,
                                fontWeight: FontWeight.bold),
                          ),
                          SizedBox(),
                        ],
                      ),
                      Container(
                        width: 360.w,
                        height: (filteredSsids.length) * 52.h,
                        constraints: BoxConstraints(maxHeight: 170.h),
                        // padding:
                        //     EdgeInsets.symmetric(horizontal: 20.w, vertical: 1.h),
                        padding:
                            EdgeInsets.only(left: 20.w, top: 1.h, bottom: 1.h),
                        margin: EdgeInsets.symmetric(vertical: 15.h),
                        decoration: BoxDecoration(
                          color: Color(0xffF3F4F9),
                          borderRadius: BorderRadius.circular(8.r),
                        ),
                        child: ListView.separated(
                          padding: EdgeInsets.zero, // 去掉默认的 padding
                          itemCount: filteredSsids.length,
                          separatorBuilder: (BuildContext context, int index) {
                            return Divider(
                              height: 0,
                            );
                          },
                          itemBuilder: (context, index) {
                            String? network = filteredSsids[index];
                            return Container(
                              alignment: Alignment.centerLeft,
                              height: 52.h,
                              decoration: BoxDecoration(
                                color: Color(0xffF3F4F9),
                                borderRadius: BorderRadius.circular(8.r),
                              ),
                              child: Row(
                                // mainAxisAlignment:
                                //     MainAxisAlignment.spaceBetween,
                                children: [
                                  Image.asset(
                                    'assets/final_icons/wifi-4-no.png',
                                    scale: 4,
                                    width: 20.w,
                                    height: 20.w,
                                  ),
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  GestureDetector(
                                    onTap: () async {
                                      try {
                                        bleProvider.ssidController.text =
                                            network;
                                        await bleProvider.sendWifiNoPassword();
                                        bleProvider.ssidController.text = '';
                                        bleProvider.currentWifiName = network;
                                        // combinedProvider.allStatus.currentSsids
                                        //     .remove(network);
                                      } catch (e) {
                                        bleProvider.ssidController.text = '';
                                      }
                                    },
                                    child: SizedBox(
                                      width: 200.w,
                                      child: Text(
                                        network,
                                        softWrap: false,
                                        overflow: TextOverflow.ellipsis,
                                        style: TextStyle(
                                            color: AppTheme.v6Blank,
                                            fontSize: 16.sp,
                                            fontWeight: FontWeight.w600),
                                      ),
                                    ),
                                  ),
                                  Expanded(child: SizedBox()),

                                  ///跳转
                                  GestureDetector(
                                    onTap: () async {
                                      currentWifi = network;
                                      deleteWifi = true;
                                      setState(() {});
                                      controllerWifi.jumpToPage(1);
                                      // Navigator.pushNamed(
                                      //   context,
                                      //   MonarRoutes.v6Wifi,
                                      //   arguments: network,
                                      // );
                                    },
                                    child: Container(
                                      // color: Colors.white,
                                      width: 70.w,
                                      height: 52.h,
                                      alignment: Alignment.centerRight,
                                      child: Row(
                                        mainAxisAlignment:
                                            MainAxisAlignment.end,
                                        children: [
                                          Image.asset(
                                            'assets/final_icons/锁 8.png',
                                            scale: 4,
                                            width: 16.w,
                                            height: 16.w,
                                            fit: BoxFit.cover,
                                          ),
                                          SizedBox(
                                            width: 15.w,
                                          ),
                                          Image.asset(
                                            'assets/final_icons/wiif_to_no.png',
                                            width: 7.w,
                                            // height: 16.w,
                                            fit: BoxFit.contain,
                                            scale: 4,
                                          ),
                                          SizedBox(
                                            width: 20.w,
                                          ),
                                        ],
                                      ),
                                    ),
                                  ),
                                ],
                              ),
                            );
                          },
                        ),
                      ),
                    ],
                  ),
                ),

                /// wifi列表
                SizedBox(
                  width: 344.w,
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        S.of(context).splash_home_device_details_other_net,
                        style: TextStyle(
                            fontSize: 16.sp,
                            color: AppTheme.v6Blank,
                            fontWeight: FontWeight.bold),
                      ),
                      GestureDetector(
                        onTap: () async {
                          if (_isDebounced) return; // 如果已经在处理，直接返回
                          print('getWifiNetworksAndSetIP开始');
                          _isDebounced = true;
                          setState(() {});
                          bleProvider.networksData?.clear();
                          _controller.repeat();
                          await bleProvider.getWifiNetworksAndSetIP(); // 停止旋转动画
                          // combinedProvider.allStatus.currentSsids
                          //     .remove(bleProvider.currentWifiName);
                          print('getWifiNetworksAndSetIP结束');
                          _controller.stop();
                          _controller.reset();
                          setState(() {});
                          _isDebounced = false;
                        },
                        child: AnimatedBuilder(
                          animation: _controller,
                          builder: (BuildContext context, Widget? child) {
                            return Transform.rotate(
                              angle: -_controller.value * 2 * pi, // 旋转角度
                              child: child,
                            );
                          },
                          child: Image.asset(
                            width: 23.w,
                            height: 19.h,
                            'assets/final_icons/Vector.png',
                            scale: 4,
                            fit: BoxFit.contain,
                          ),
                        ),
                      )
                    ],
                  ),
                ),
                Container(
                  width: 344.w,
                  height:
                      ((bleProvider.networksData?.length ?? 0) * 50.h + 190.h),
                  constraints: BoxConstraints(maxHeight: 190.h),
                  // padding:
                  //     EdgeInsets.symmetric(horizontal: 20.w, vertical: 1.h),
                  padding: EdgeInsets.only(left: 20.w, top: 1.h, bottom: 1.h),
                  margin: EdgeInsets.symmetric(vertical: 15.h),
                  decoration: BoxDecoration(
                    color: !_isDebounced ? Color(0xffF3F4F9) : Colors.white,
                    borderRadius: BorderRadius.circular(12.r),
                  ),
                  child: !_isDebounced
                      ? ListView.separated(
                          padding: EdgeInsets.zero, // 去掉默认的 padding
                          itemCount: bleProvider.networksData?.length ?? 0,
                          separatorBuilder: (BuildContext context, int index) {
                            return const Divider(
                              color: Color(0xffDCDCDC),
                              height: 0,
                            );
                          },
                          itemBuilder: (context, index) {
                            String? network = bleProvider.networksData?[index];
                            return GestureDetector(
                              onTap: () {
                                bleProvider.ssidController.text = network!;
                                final WifiModel info =
                                    bleProvider.wifiModelData.firstWhere(
                                  (wifiModel) => wifiModel.wifiName == network,
                                  orElse: () => WifiModel(wifiName: ''),
                                );
                                bleProvider.passwordController.text =
                                    info.wifiPassword ?? '';
                                OverlayManager.showWifiOverlay(
                                    bleProvider: bleProvider,
                                    combinedProvider: combinedProvider);
                              },
                              child: Container(
                                alignment: Alignment.centerLeft,
                                height: 50.h,
                                decoration: BoxDecoration(
                                  color: Color(0xffF3F4F9),
                                  borderRadius: BorderRadius.circular(8.r),
                                ),
                                child: Row(
                                  children: [
                                    Image.asset(
                                      'assets/final_icons/wifi-4-no.png',
                                      scale: 4,
                                      width: 20.w,
                                      height: 20.w,
                                    ),
                                    SizedBox(
                                      width: 10.w,
                                    ),
                                    SizedBox(
                                      width: 200.w,
                                      child: Text(
                                        network ??
                                            S.of(context).wifi_Unknown_network,
                                        softWrap: false,
                                        overflow: TextOverflow.ellipsis,
                                        style: TextStyle(
                                            color: AppTheme.v6Blank,
                                            fontSize: 16.sp,
                                            fontWeight: FontWeight.w600),
                                      ),
                                    ),
                                    Expanded(child: SizedBox()),
                                    GestureDetector(
                                      onTap: () async {
                                        currentWifi = network ??
                                            S.of(context).wifi_Unknown_network;
                                        deleteWifi = false;
                                        setState(() {});
                                        controllerWifi.jumpToPage(1);
                                        // Navigator.pushNamed(
                                        //   context,
                                        //   MonarRoutes.v6Wifi,
                                        //   arguments: network,
                                        // );
                                      },
                                      child: Container(
                                        width: 70.w,
                                        height: 52.h,
                                        alignment: Alignment.centerRight,
                                        // color: Colors.white,
                                        child: Row(
                                          mainAxisAlignment:
                                              MainAxisAlignment.end,
                                          children: [
                                            Image.asset(
                                              'assets/final_icons/锁 8.png',
                                              scale: 4,
                                              width: 16.w,
                                              height: 16.w,
                                              fit: BoxFit.cover,
                                            ),
                                            SizedBox(
                                              width: 15.w,
                                            ),
                                            Image.asset(
                                              'assets/final_icons/wiif_to_no.png',
                                              width: 7.w,
                                              // height: 16.w,
                                              fit: BoxFit.contain,
                                              scale: 4,
                                            ),
                                            SizedBox(
                                              width: 20.w,
                                            ),
                                          ],
                                        ),
                                      ),
                                    ),
                                  ],
                                ),
                              ),
                            );
                          },
                        )
                      : Center(
                          child: Text(
                            S
                                .of(context)
                                .splash_home_device_details_other_net_refreshing,
                            style: TextStyle(
                                color: Color(0xff858585), fontSize: 16.sp),
                          ),
                        ),
                ),
              ],
            ),
          ),
        ),
        SingleChildScrollView(
          child: Container(
            width: 360.w,
            height:
                (480 + (combinedProvider.allStatus.currentSsids.length) * 52).h,
            padding: EdgeInsets.symmetric(
                vertical: 15.w, horizontal: (390 - 346).w / 2),
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(15.r),
              color: AppTheme.v6White,
            ),
            margin: EdgeInsets.only(bottom: 10.h),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.center,
              mainAxisAlignment: MainAxisAlignment.start,
              children: [
                SizedBox(
                  height: 20.h,
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      S.of(context).splash_home_device_details_device_details,
                      style: TextStyle(
                          fontSize: 16.sp,
                          color: AppTheme.v6Blank,
                          fontWeight: FontWeight.bold),
                    ),
                    SizedBox(),
                  ],
                ),

                /// WIFI信息
                Container(
                  width: 360.w,
                  height: 108.h,
                  constraints: BoxConstraints(maxHeight: 170.h),
                  padding:
                      EdgeInsets.symmetric(horizontal: 20.w, vertical: 1.h),
                  margin: EdgeInsets.symmetric(vertical: 15.h),
                  decoration: BoxDecoration(
                    color: Color(0xffF3F4F9),
                    borderRadius: BorderRadius.circular(8.r),
                  ),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Container(
                        alignment: Alignment.centerLeft,
                        height: 50.h,
                        decoration: BoxDecoration(
                          color: Color(0xffF3F4F9),
                          borderRadius: BorderRadius.circular(8.r),
                        ),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            SizedBox(
                              width: 80.w,
                              child: Text(
                                'WI-FI',
                                softWrap: false,
                                overflow: TextOverflow.ellipsis,
                                style: TextStyle(
                                    color: AppTheme.v6Blank,
                                    fontSize: 16.sp,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                            SizedBox(
                              width: 140.w,
                              child: Text(
                                currentWifi,
                                softWrap: false,
                                overflow: TextOverflow.ellipsis,
                                textAlign: TextAlign.end,
                                style: TextStyle(
                                    color: Color(0xff828282),
                                    fontSize: 14.sp,
                                    fontWeight: FontWeight.w600),
                              ),
                            ),
                          ],
                        ),
                      ),
                      Divider(
                        height: 1,
                        color: Color(0xffDCDCDC),
                      ),
                      Container(
                        alignment: Alignment.centerLeft,
                        height: 50.h,
                        decoration: BoxDecoration(
                          color: Color(0xffF3F4F9),
                          borderRadius: BorderRadius.circular(8.r),
                        ),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            SizedBox(
                              width: 150.w,
                              child: Text(
                                'WI-FI strength',
                                softWrap: false,
                                overflow: TextOverflow.ellipsis,
                                style: TextStyle(
                                    color: AppTheme.v6Blank,
                                    fontSize: 16.sp,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                            SizedBox(
                              width: 100.w,
                              child: Text(
                                '100%',
                                softWrap: false,
                                overflow: TextOverflow.ellipsis,
                                textAlign: TextAlign.end,
                                style: TextStyle(
                                    color: Color(0xff828282),
                                    fontSize: 14.sp,
                                    fontWeight: FontWeight.w600),
                              ),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),

                /// 删除网络
                Visibility(
                  visible: deleteWifi,
                  child: GestureDetector(
                    onTap: () async {
                      // currentWifi
                      // try {
                      bleProvider.ssidController.text = currentWifi;
                      // await bleProvider.sendWifiForget();
                      // if (combinedProvider.allStatus.currentSsids
                      //     .contains(currentWifi)) {
                      //   combinedProvider.allStatus.currentSsids
                      //       .remove(currentWifi);
                      // }
                      // currentWifi = '';
                      // bleProvider.ssidController.text = '';
                      // controllerWifi.jumpToPage(0);
                      // } catch (e) {}
                      OverlayManager.showWifiDelectOverlay(
                          bleProvider: bleProvider,
                          combinedProvider: combinedProvider,
                          onTap: () {
                            controllerWifi.jumpToPage(0);
                          });
                    },
                    child: Container(
                      width: 155.w,
                      alignment: Alignment.center,
                      height: 37.h,
                      decoration: BoxDecoration(
                        color: Color(0xff9D2D2B),
                        borderRadius: BorderRadius.circular(20.r),
                      ),
                      child: Text(
                        S.of(context).splash_home_device_details_net_delete,
                        softWrap: false,
                        overflow: TextOverflow.ellipsis,
                        style: TextStyle(
                            color: AppTheme.white,
                            fontSize: 16.sp,
                            fontWeight: FontWeight.bold),
                      ),
                    ),
                  ),
                ),
                SizedBox(
                  height: 10.h,
                ),

                /// 返回
                GestureDetector(
                  onTap: () {
                    controllerWifi.jumpToPage(0);
                  },
                  child: Container(
                    width: 155.w,
                    alignment: Alignment.center,
                    height: 37.h,
                    decoration: BoxDecoration(
                      color: Color(0xff9D2D2B),
                      borderRadius: BorderRadius.circular(20.r),
                    ),
                    child: Text(
                      S.of(context).splash_home_device_details_net_no_delete,
                      softWrap: false,
                      overflow: TextOverflow.ellipsis,
                      style: TextStyle(
                          color: AppTheme.white,
                          fontSize: 16.sp,
                          fontWeight: FontWeight.bold),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }

  ///亮度模块
  Widget lightContainer(BuildContext context) {
    return Container(
      // margin: EdgeInsets.only(top: 10.h),
      height: 360.h,
      width: 390.w,
      padding:
          EdgeInsets.symmetric(vertical: 15.h, horizontal: (390 - 346).w / 2),
      decoration: BoxDecoration(
        color: AppTheme.white,
        borderRadius: BorderRadius.circular(15.r),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            height: 20,
          ),
          Text(
            S.of(context).splash_home_device_details_light,
            style: AppTheme.v6blank16bold,
          ),
          Container(
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),
            height: 43.h,
            width: 346.w,
            padding: EdgeInsets.symmetric(vertical: 1.w, horizontal: 20.h),
            decoration: BoxDecoration(
                color: AppTheme.v6ThemeColors,
                borderRadius: BorderRadius.circular(12.r)),
            child: Row(
              children: [
                Image.asset(
                  'assets/final_icons/light.png',
                  scale: 4,
                  width: 26.w,
                  fit: BoxFit.contain,
                ),
                Expanded(
                  child: Slider(
                    min: 0,
                    max: 255,
                    value: currentLight,
                    activeColor: Color(0xff9D2D2B),
                    onChanged: (value) {
                      setState(() {});
                      currentLight = value;
                      isAuto = false;
                    },
                    onChangeEnd: (value) {
                      setState(() {});
                      print(currentLight.toInt());
                      combinedProvider.sendLuminosity(currentLight.toInt());
                      // sendLuminosity
                    },
                  ),
                ),
              ],
            ),
          ),
          Text(
            S.of(context).splash_home_device_details_Speaker_brightness_display,
            style: AppTheme.v6blank16bold,
          ),
          Container(
            height: 133.h,
            width: 346.w,
            margin: EdgeInsets.only(top: 10.h),
            padding: EdgeInsets.symmetric(vertical: 5.w, horizontal: 10.h),
            decoration: BoxDecoration(
                color: AppTheme.v6ThemeColors,
                borderRadius: BorderRadius.circular(12.r)),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: List.generate(lightModes.length, (index) {
                return buildSpeakDisplayItem(
                    context: context, mode: lightModes[index]);
              }),
            ),
          ),
          Container(
            height: 42.h,
            width: 346.w,
            margin: EdgeInsets.only(top: 20.h),
            padding: EdgeInsets.symmetric(vertical: 2.w, horizontal: 10.h),
            decoration: BoxDecoration(
                color: AppTheme.v6ThemeColors,
                borderRadius: BorderRadius.circular(12.r)),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  S.of(context).splash_home_device_details_automatic,
                  style: AppTheme.v6blank16bold,
                ),
                CupertinoSwitch(
                  value: isAuto,
                  onChanged: (bool value) {
                    if (value) {
                      print('1');
                      combinedProvider.sendLuminosity(-1);
                      isAuto = value;
                      setState(() {});
                    } else {
                      combinedProvider.sendLuminosity(100);
                      isAuto = value;
                      setState(() {});
                    }
                  },
                )
              ],
            ),
          ),
        ],
      ),
    );
  }

  ///亮度模块模式选择
  Widget buildSpeakDisplayItem({
    required BuildContext context,
    required LightMode mode,
  }) {
    return GestureDetector(
      onTap: () {
        print(lightMode);
        print(mode.light);
        isAuto = false;
        lightMode = mode.mode;
        combinedProvider.sendLuminosity(mode.light.toInt());

        currentLight = mode.light;
        setState(() {});
      },
      child: Column(
        children: [
          Image.asset(
            mode.img,
            width: 80.w,
            height: 80.h,
            fit: BoxFit.fitHeight,
            scale: 4,
          ),
          SizedBox(
            width: 80.w,
            height: 20.h,
            child: FittedBox(
              fit: BoxFit.scaleDown,
              child: Text(
                AppTheme.getV6LightModeTitle(context, mode.str),
                textAlign: TextAlign.center,
              ),
            ),
          ),
          Container(
            width: 17.w,
            height: 17.w,
            decoration: BoxDecoration(
              border: Border.all(
                  color:
                      lightMode == mode.mode ? AppTheme.v6Red2 : Colors.black),
              color:
                  lightMode == mode.mode ? AppTheme.v6Red2 : Colors.transparent,
              shape: BoxShape.circle,
            ),
            child: Center(
                child: Image.asset(
              'assets/final_icons/对 2.png',
              scale: 4,
            )),
          ),
        ],
      ),
    );
  }

  /// 构建统一状态图标
  Widget deviceStatusIcon(
      {required bool isConnected,
      required int page,
      required String connectedImage,
      required String disconnectedImage}) {
    return GestureDetector(
      onTap: () {
        controller.jumpToPage(page);
        currentPage = page;
        setState(() {});
      },
      child: Container(
        width: 73.w,
        height: 73.w,
        decoration: BoxDecoration(
          color: Colors.transparent,
          border: currentPage == page
              ? Border.all(color: Colors.white, width: 1.w)
              : Border.all(color: Colors.transparent),
          borderRadius: BorderRadius.circular(18.r),
        ),
        child: Center(
          child: Container(
            width: 62.w,
            height: 62.w,
            decoration: BoxDecoration(
              color: currentPage == page ? AppTheme.v6E7 : Color(0xffDCDCDC),
              borderRadius: BorderRadius.circular(16.r),
            ),
            child: Image.asset(
              disconnectedImage,
              scale: 4,
            ),
          ),
        ),
      ),
    );
  }
}

class ShutdownCard extends StatefulWidget {
  const ShutdownCard({
    super.key,
    required this.bleProvider,
  });

  final BleProvider bleProvider;

  @override
  State<ShutdownCard> createState() => _ShutdownCardState();
}

class _ShutdownCardState extends State<ShutdownCard>
    with SingleTickerProviderStateMixin {
  double _dx = (315 - 153).w; // 记录 X 轴偏移量
  double _maxDx = (315 - 153).w; // 最大右滑偏移量
  bool right = false;
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: Duration(milliseconds: 300), // 回弹动画时间
    );
    _animation = Tween<double>(begin: 0, end: 0).animate(_controller)
      ..addListener(() {
        setState(() {
          _dx = _animation.value;
        });
      });
  }

  void _animateRightBack() {
    _animation = Tween<double>(begin: _dx, end: 0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    );
    _controller.forward(from: 0);
  }

  void _animateLiftBack() {
    _animation = Tween<double>(begin: _dx, end: _maxDx).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    );
    _controller.forward(from: 0);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container(
        height: 360.h,
        width: 390.w,
        padding:
            EdgeInsets.symmetric(vertical: 15.h, horizontal: (390 - 345).w / 2),
        decoration: BoxDecoration(
          color: AppTheme.white,
          borderRadius: BorderRadius.circular(15.r),
        ),
        child: Column(
          children: [
            SizedBox(
              height: 20.h,
            ),
            GestureDetector(
              onHorizontalDragUpdate: (details) async {
                setState(() {
                  _dx += details.delta.dx; // 拖动时更新 X 轴偏移量
                  // if (_dx >= _maxDx) {
                  //   right = false;
                  //   _dx = _maxDx; // 限制左滑最大值
                  //   print('右滑');
                  // }
                  if (_dx <= 0) {
                    right = true;
                    _dx = 0; // 限制右滑最大值
                    print('右滑');
                    widget.bleProvider.shutdown();

                    ///触发关机
                  }
                  print(_dx);
                });
              },
              onHorizontalDragEnd: (details) {
                // _animateBack(); // 松手后回弹
                if (right) {
                  _animateRightBack(); // 右滑超过一定值后，松手会自动回到初始位置
                } else {
                  _animateLiftBack();
                }
              },
              child: Container(
                width: 345.w,
                height: 64.h,
                padding: EdgeInsets.all(10.w),
                decoration: BoxDecoration(
                    color: Color(0xffF3F4F9),
                    borderRadius: BorderRadius.circular(12.r)),
                child: Stack(
                  children: [
                    SizedBox(
                      width: 315.w,
                      height: 46.h,
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Container(
                            width: 50.w,
                            height: 46.h,
                            decoration: BoxDecoration(
                              color: Color(0xffE1E1E1),
                              borderRadius: BorderRadius.circular(23.r),
                            ),
                            child: Center(
                              child: Text(
                                'OFF',
                                style: TextStyle(
                                    fontSize: 20.sp,
                                    color: Colors.white,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                          ),
                          Container(
                            width: 50.w,
                            height: 46.h,
                            decoration: BoxDecoration(
                              color: Color(0xffE1E1E1),
                              borderRadius: BorderRadius.circular(23.r),
                            ),
                            child: Center(
                              child: Text(
                                'ON',
                                style: TextStyle(
                                    fontSize: 20.sp,
                                    color: Colors.white,
                                    fontWeight: FontWeight.bold),
                              ),
                            ),
                          ),
                        ],
                      ),
                    ),
                    Transform.translate(
                      offset: Offset(_dx, 0), // 应用水平偏移
                      child: Container(
                        width: 153.w,
                        height: 46.h,
                        decoration: BoxDecoration(
                          color: AppTheme.v6Red2,
                          borderRadius: BorderRadius.circular(23.r),
                        ),
                        alignment: Alignment.center,
                        child: Text(
                          right ? "OFF" : "NO",
                          style: TextStyle(
                              fontSize: 20.sp,
                              color: Colors.white,
                              fontWeight: FontWeight.bold),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ));
  }
}

// Row(
// mainAxisAlignment: MainAxisAlignment.spaceEvenly,
// children: [
// ShutButton(
// onTap: widget.bleProvider.shutdown,
// icon: 'assets/icons/png/pngX4/电源_12.png',
// text: S.of(context).splash_home_device_details_shutdown,
// title: 'dialog_inquire_shutdown',
// content: 'dialog_inquire_shutdown_why',
// yes: 'dialog_inquire_shutdown',
// no: 'dialog_inquire_no',
// ),
// ShutButton(
// onTap: widget.bleProvider.reboot,
// icon: 'assets/icons/png/pngX4/刷新_1.png',
// text: S.of(context).splash_home_device_details_reboot,
// title: 'dialog_inquire_reboot',
// content: 'dialog_inquire_reboot_why',
// yes: 'dialog_inquire_reboot',
// no: 'dialog_inquire_no',
// ),
// ],
// ),

class ShutButton extends StatefulWidget {
  const ShutButton({
    super.key,
    required this.onTap,
    required this.icon,
    required this.text,
    required this.title,
    required this.content,
    required this.yes,
    required this.no,
  });
  final Function onTap;
  final String icon;
  final String text;
  final String title;
  final String content;
  final String yes;
  final String no;
  @override
  State<ShutButton> createState() => _ShutButtonState();
}

class _ShutButtonState extends State<ShutButton> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        OverlayManager.showDialogOverlay(
            customCallback: () {
              widget.onTap();
            },
            title: widget.title,
            no: widget.no,
            yes: widget.yes,
            content: widget.content);
      },
      child: SizedBox(
        width: 87.w,
        height: 124.h,
        child: Column(
          children: [
            Container(
              width: 87.w,
              height: 84.h,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(14.r),
                color: AppTheme.white,
              ),
              child: Center(
                child: Image.asset(
                  widget.icon,
                  width: 42.w,
                  height: 42.h,
                  fit: BoxFit.contain,
                ),
              ),
            ),
            SizedBox(
              height: 10.h,
            ),
            SizedBox(
              width: 87.w,
              child: FittedBox(
                fit: BoxFit.scaleDown,
                child: Text(
                  widget.text,
                  style: TextStyle(fontSize: 20.sp),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class LightMode {
  final int mode;
  final String img;
  final String str;
  final double light;
  LightMode({
    required this.mode,
    required this.img,
    required this.str,
    required this.light,
  });
}

// /// 构建蓝牙状态图标
// Widget buildDeviceStatusIcon(
//     bool isConnected, String connectedImage, String disconnectedImage) {
//   return Container(
//     width: 60.w,
//     height: 60.h,
//     decoration: BoxDecoration(
//       color: AppTheme.white,
//       borderRadius: BorderRadius.circular(16.r),
//     ),
//     child: isConnected
//         ? Image.asset(
//       connectedImage,
//       scale: 4,
//     )
//         : Image.asset(
//       disconnectedImage,
//       scale: 4,
//     ),
//   );
// }
//
// /// 构建蓝牙连接控制按钮
// Widget buildConnectionControl(String address) {
//   return Container(
//     width: 94.w,
//     height: 78.h,
//     decoration: BoxDecoration(
//       color: AppTheme.white,
//       borderRadius: BorderRadius.circular(16.r),
//     ),
//     child: Column(
//       children: [
//         FeedbackImage(
//           onTap: () async {
//             final isConnected =
//                 blueClassicProvider.v6Connections[address]?.isConnected ??
//                     false;
//             if (isConnected) {
//               await blueClassicProvider.v6disconnectFromDevice(address);
//             } else {
//               await blueClassicProvider.v6ConnectImageFinalToDevice(
//                   blueInfo, context);
//               await blueClassicProvider
//                   .getWifiNetworksAndSetIP(widget.address);
//             }
//           },
//           img:
//           blueClassicProvider.v6Connections[address]?.isConnected ?? false
//               ? 'assets/design/Subtract_connect.png'
//               : 'assets/icons/png/Frame_40.png',
//         ),
//       ],
//     ),
//   );
// }
//
// /// 构建设备信息部分
// Widget buildDeviceInfoSection() {
//   return Container(
//     width: 1.sw,
//     height: 203.h,
//     margin: EdgeInsets.only(top: 10.h),
//     padding: EdgeInsets.all(8.w),
//     decoration: BoxDecoration(
//       color: AppTheme.white,
//       borderRadius: BorderRadius.circular(16.r),
//     ),
//     child: Column(
//       children: [
//         // 可以在这里添加更多的设备信息
//       ],
//     ),
//   );
// }

```