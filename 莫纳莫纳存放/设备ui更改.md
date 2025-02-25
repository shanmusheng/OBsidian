```
import 'dart:async';
import 'dart:ui';

import 'package:animated_text_kit/animated_text_kit.dart';
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';

import 'package:provider/provider.dart';
import 'package:url_launcher/url_launcher.dart';

import '../../app_theme.dart';
import '../../generated/l10n.dart';
import '../../routes/monar_routes.dart';
import '../../utils/blue_classic_provider.dart';
import '../../utils/navigator_provider.dart';
import '../../utils/overlay_manage.dart';
import '../../utils/wifi_blue_combined.dart';
import '../device_connect_module/views/devices_connect_page.dart';

class V6ProductsHome extends StatefulWidget {
  const V6ProductsHome({super.key, required this.onTap});
  final Function onTap;

  @override
  State<V6ProductsHome> createState() => _V6ProductsHomeState();
}

class _V6ProductsHomeState extends State<V6ProductsHome> {
  Timer? _timer; // 定时器
  late CombinedProvider combinedProvider;
  late final BlueClassicProvider blueClassicProvider;
  late final ScrollController controller;
  BluetoothState? bluetoothState;
  bool blueState = false;
  bool blueScan = false; //当前正在扫描界面
  int currentIndex = -1; // 新增
  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    blueClassicProvider =
        Provider.of<BlueClassicProvider>(context, listen: false);
    controller = ScrollController();
    combinedProvider = Provider.of<CombinedProvider>(context, listen: false);
    startListening();
    // blueClassicProvider.v6DevicesList.clear();

    ///初始化时监听蓝牙
    // FlutterBluetoothSerial.instance
    //     .onStateChanged()
    //     .listen((BluetoothState state) {
    //   bluetoothState = state;
    //   if (blueState) {
    //     _checkBluetoothState(state);
    //   }
    // });

    ///初始化时检查蓝牙
    FlutterBluetoothSerial.instance.state.then((state) {
      if (mounted) {
        setState(() {
          bluetoothState = state;
        });
      }
      _checkBluetoothState(state);
    });
    // 实时监听蓝牙状态变化
    FlutterBluetoothSerial.instance
        .onStateChanged()
        .listen((BluetoothState state) {
      if (mounted) {
        setState(() {
          bluetoothState = state;
        });
      }
      _checkBluetoothState(state);
    });
    if (blueClassicProvider.v6DevicesList.length == 0) {
      // 蓝牙已开启，开始设备发现
      print('蓝牙已开启，开始设备发现');

      blueClassicProvider.v6GetPairedDevices();
      blueClassicProvider.v6StartDiscovery();
    }
  }

  void _checkBluetoothState(BluetoothState state) {
    if (state == BluetoothState.STATE_OFF) {
      // 蓝牙未开启，弹出提示框
      OverlayManager.showBlueOverlay(blueProvider: blueClassicProvider);
    } else {
      // print(blueClassicProvider.v6DevicesList);
      // if (blueClassicProvider.v6DevicesList.length == 0) {
      //   // 蓝牙已开启，开始设备发现
      //   print('蓝牙已开启，开始设备发现');
      //   blueClassicProvider.v6StartDiscovery();
      //   blueClassicProvider.v6GetPairedDevices();
      // }
    }
  }

  /// 蓝牙弹窗
  void _showBluetoothOffDialog() {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('蓝牙未开启'),
          content: const Text('请开启蓝牙以继续使用此功能。'),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(
                  NavigatorProvider.navigatorContext!,
                ).pop();
              },
              child: const Text('确定'),
            ),
            TextButton(
              onPressed: () async {
                await FlutterBluetoothSerial.instance.requestEnable();
                Navigator.of(context).pop();
                blueClassicProvider.v6GetPairedDevices();
                blueClassicProvider.v6StartDiscovery();
              },
              child: const Text('打开'),
            ),
          ],
        );
      },
    );
  }

  void updateCurrentIndex(int index) {
    setState(() {
      currentIndex = index;
      // 将点击的设备移动到列表的第一位
      // final device = blueClassicProvider.v6DevicesList.removeAt(index);
      // blueClassicProvider.v6DevicesList.insert(0, device);
      // currentIndex = 0;
    });
  }

  void reCallCurrentIndex(int index) {
    setState(() {
      // currentIndex = index;
      // 将点击的设备移动到列表的第一位
      final device = blueClassicProvider.v6DevicesList.removeAt(index);
      blueClassicProvider.v6DevicesList.insert(0, device);
      currentIndex = 0;
    });
// 使用 WidgetsBinding 进行延迟调用，确保 ListView 已经构建完成
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (controller.hasClients) {
        // 确保 controller 已经附加到 ListView
        controller.animateTo(
          0, // 滚动到第一个位置
          duration: Duration(milliseconds: 300), // 动画时长
          curve: Curves.easeInOut, // 动画曲线
        );
      }
    });
    widget.onTap();
  }

  /// 停止监听
  void stopListening() {
    _timer?.cancel();
    _timer = null;
  }

  /// 调用 sendControlMusicInfo 方法并更新状态
  Future<void> _fetchMusicInfo() async {
    if (blueClassicProvider.connection != null) {
      try {
        await combinedProvider.sendControlMusicInfo();
      } catch (e) {
        print('获取音乐信息失败: $e');
      }
    }
  }

  /// 启动定时器，每 5 秒调用一次 sendControlMusicInfo
  void startListening() {
    _timer?.cancel(); // 确保没有重复的 Timer 实例
    _timer = Timer.periodic(const Duration(seconds: 10), (timer) async {
      await _fetchMusicInfo();
    });
  }

  @override
  void dispose() {
    stopListening(); // 确保 Provider 销毁时停止定时器
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final combinedProvider = Provider.of<CombinedProvider>(context);

    // 获取初始蓝牙状态
    ///初始化时检查蓝牙
    BlueClassicProvider blueClassicProvider =
        Provider.of<BlueClassicProvider>(context);
    print(blueClassicProvider.v6Discover); // 处理返回按钮
    Future<bool> onWillPop() async {
      if (OverlayManager.isOverlayVisible) {
        OverlayManager.removeOverlay();
        return false; // 阻止默认的返回操作
      }
      return true; // 允许默认的返回操作
    }

    return WillPopScope(
      onWillPop: onWillPop, // 捕捉返回按钮事件
      child: Padding(
        padding: EdgeInsets.symmetric(horizontal: 24.0.w),
        child: Scaffold(
          backgroundColor: AppTheme.v6ThemeColors,
          appBar: AppBar(
            toolbarHeight: 78.h,
            backgroundColor: AppTheme.v6ThemeColors,
            title: Text(
              S.of(context).splash_home_device_title,
              style: TextStyle(fontSize: 28.sp, fontWeight: FontWeight.bold),
            ),
            leadingWidth: 0,
            leading: SizedBox(),
            actions: [
              GestureDetector(
                onTap: () async {
                  ///初始化时检查蓝牙
                  await FlutterBluetoothSerial.instance.state.then((state) {
                    if (mounted) {
                      setState(() {
                        bluetoothState = state;
                      });
                    }
                  });
                  if (bluetoothState == BluetoothState.STATE_OFF) {
                    print('11111111111111111');
                    await FlutterBluetoothSerial.instance.requestEnable();
                  } else {
                    print('222222222222222222222222');
                    blueClassicProvider.v6DevicesList.clear();
                    blueScan = true;
                    await Future.delayed(Duration(milliseconds: 500));
                    await blueClassicProvider.v6GetPairedDevices();
                    setState(() {});
                    blueClassicProvider.v6StartDiscovery();
                  }
                },
                child: Container(
                  width: 33.w,
                  height: 33.w,
                  decoration: BoxDecoration(
                      shape: BoxShape.circle, color: AppTheme.v6A2),
                  child: Image.asset(
                    'assets/icons/png/pngX4/home_add_withe.png',
                    scale: 4,
                  ),
                ),
              ),
            ],
          ),
          body: SizedBox(
            width: 1.sw,
            child: PageView(
              children: [
                ///这里的显示应该为当扫描时显示的是扫描界面和弹窗,点击
                // blueClassicProvider.v6Discover
                // blueClassicProvider.v6DevicesList.length == 0
                // blueClassicProvider.v6Discover &&
                //         blueClassicProvider.blueInfoList.length == 0
                blueScan
                    ? V6ScanDeviceView(
                        blueClassicProvider: blueClassicProvider,
                        onTap: () {
                          blueScan = !blueScan;
                          setState(() {});
                        },
                      )
                    : blueClassicProvider.blueInfoList.isNotEmpty
                        ? ListView.builder(
                            controller: controller,
                            itemCount: blueClassicProvider.blueInfoList.length,
                            itemBuilder: (BuildContext context, int index) {
                              if (index == 0) {
                                return Column(
                                  children: [
                                    GestureDetector(
                                      child: Image.asset(
                                        'assets/design/splash/Component_3.png',
                                        scale: 2,
                                        width: 342.w,
                                        height: 174.h,
                                        fit: BoxFit.contain,
                                      ),
                                      onTap: () async {
                                        final url =
                                            Uri.parse('https://monarai.cn');
                                        if (await canLaunchUrl(url)) {
                                          await launchUrl(url,
                                              mode: LaunchMode
                                                  .externalApplication);
                                        } else {
                                          throw '无法打开 URL: $url';
                                        }
                                      },
                                    ),
                                    SizedBox(
                                      height: 20.h,
                                    ),
                                    MonarCard(
                                      blueClassicProvider: blueClassicProvider,
                                      blueDevice: blueClassicProvider
                                          .blueInfoList[index],
                                      currentIndex: currentIndex, // 传递索引
                                      onTap: () =>
                                          updateCurrentIndex(index), // 更新索引
                                      recall: () =>
                                          reCallCurrentIndex(index), // 更新索引
                                      index: index,
                                      combinedProvider: combinedProvider,
                                    ),
                                  ],
                                );
                              }

                              return MonarCard(
                                blueClassicProvider: blueClassicProvider,
                                blueDevice:
                                    blueClassicProvider.blueInfoList[index],
                                currentIndex: currentIndex, // 传递索引
                                onTap: () => updateCurrentIndex(index), // 更新索引
                                recall: () => reCallCurrentIndex(index), // 更新索引
                                index: index,
                                combinedProvider: combinedProvider,
                              );
                            },
                          )
                        : ListView.builder(
                            controller: controller,
                            itemCount: 1,
                            itemBuilder: (BuildContext context, int index) {
                              return GestureDetector(
                                child: Image.asset(
                                  'assets/design/splash/Component_3.png',
                                  scale: 2,
                                  width: 342.w,
                                  height: 174.h,
                                  fit: BoxFit.contain,
                                ),
                                onTap: () async {
                                  final url = Uri.parse('https://monarai.cn');
                                  if (await canLaunchUrl(url)) {
                                    await launchUrl(url,
                                        mode: LaunchMode.externalApplication);
                                  } else {
                                    throw '无法打开 URL: $url';
                                  }
                                },
                              );
                            },
                          )
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class MonarCard extends StatefulWidget {
  const MonarCard({
    super.key,
    required this.blueClassicProvider,
    required this.blueDevice,
    required this.currentIndex, // 新增
    required this.onTap,
    required this.recall,
    required this.index,
    required this.combinedProvider,
  });
  final CombinedProvider combinedProvider;
  final BlueClassicProvider blueClassicProvider;
  final BlueInformation blueDevice;
  final int currentIndex; // 当前索引
  final VoidCallback onTap; // 点击事件
  final VoidCallback recall; // 点击事件
  final int index; // 当前卡片索引
  @override
  State<MonarCard> createState() => _MonarCardState();
}

class _MonarCardState extends State<MonarCard> {
  bool connecting = false;
  BluetoothConnection? connection; //当前连接的设备
  late BlueInformation blueInfo;
  @override
  Widget build(BuildContext context) {
    connection = widget.blueClassicProvider
        .v6GetConnection(widget.blueDevice.address ?? '');

    ///获取蓝牙信息
    blueInfo = widget.blueClassicProvider.blueInfoList.firstWhere(
        (blueInfo) => blueInfo.address == widget.blueDevice.address,
        orElse: () => BlueInformation());
    // 判断是否是当前选中的卡片
    bool isSelected = widget.index == widget.currentIndex;
    return GestureDetector(
      onTap: () {
        widget.onTap();
      },
      child: AnimatedContainer(
        duration: Duration(milliseconds: 300),
        width: 342.w,
        height: isSelected ? 393.h : 96.h,
        padding: EdgeInsets.all(15.w),
        margin: EdgeInsets.only(bottom: 20.h),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(24.r),
          color: isSelected
              ? AppTheme.v6ThemeBlank
              : Color(0xFF19191A).withOpacity(0.7),
        ),
        child: Stack(
          children: [
            Positioned(
              right: 0,
              child: SizedBox(
                width: isSelected ? 302.w : 250.w,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    SizedBox(
                      width: 200.w,
                      child: Text(
                        '${widget.blueDevice.name}',
                        softWrap: false,
                        overflow: TextOverflow.ellipsis,
                        style: TextStyle(
                            fontSize: 20.sp,
                            color: AppTheme.white,
                            fontWeight: FontWeight.bold),
                      ),
                    ),
                    GestureDetector(
                      onTap: () {
                        Navigator.pushNamed(context, MonarRoutes.v6ProductHome,
                            arguments: widget.blueDevice.address);
                      },
                      child: Image.asset(
                        'assets/icons/png/pngX4/more_withe.png',
                        width: 16.w,
                        height: 16.h,
                        scale: 4,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            Positioned(
              top: isSelected ? 30.h : 25.h,
              left: isSelected ? 16.w : 60.w,
              child: Row(
                children: [
                  connection?.isConnected ?? false
                      ? Row(
                          children: [
                            GestureDetector(
                              onTap: () {
                                OverlayManager.showTipOverlay(
                                    str: 'IP_address', tip: '${blueInfo.ip}');
                              },
                              child: Row(
                                children: [
                                  Image.asset(
                                    blueInfo.wifiConnect ?? false
                                        ? 'assets/icons/png/pngX4/wifi (2).png'
                                        : 'assets/icons/png/Vector (1).png',
                                    scale: 4,
                                  ),
                                  SizedBox(
                                    width: 5.w,
                                  ),
                                  Text(
                                    blueInfo.wifiName ?? '',
                                    style: TextStyle(
                                        color: AppTheme.white, fontSize: 12.sp),
                                  ),
                                ],
                              ),
                            ),
                            Container(
                              height: 15.h,
                              width: 2.w,
                              margin: EdgeInsets.symmetric(horizontal: 5.w),
                              color: AppTheme.v6C4,
                            ),
                            GestureDetector(
                                onTap: () {
                                  widget.blueClassicProvider
                                      .v6disconnectFromDevice(
                                          widget.blueDevice.address ?? '');
                                },
                                child: productOfflineOff(context))
                            // blueInfo.wifiConnect ?? false
                            //     ? productOfflineOff(context)
                            //     : productOfflineOn(context)
                          ],
                        )
                      : productOfflineOn(context),
                ],
              ),
            ),
            AnimatedPositioned(
              top: isSelected ? 50.h : -20.h,
              left: isSelected ? (320 - 202) / 2.w : -10.w,
              height: isSelected ? 210.h : 110.h,
              duration: Duration(milliseconds: 200),
              child: Stack(
                alignment: Alignment.center,
                children: [
                  AnimatedOpacity(
                    opacity: connection?.isConnected ?? false ? 1 : 0.5,
                    duration: Duration(milliseconds: 300),
                    child: GestureDetector(
                      onTap: connection?.isConnected ?? false
                          ? () {
                              widget.onTap();
                            }
                          : () async {
                              bool result;
                              connecting = true;
                              setState(() {});

                              ///如果当前没有设备连接connectingNew=false,进行连接
                              if (!widget.blueClassicProvider.v6ConnectingNew) {
                                result = await widget.blueClassicProvider
                                    .v6ConnectImageFinalToDevice(
                                        BlueInformation(
                                            name: widget.blueDevice.name,
                                            address: widget.blueDevice.address),
                                        context);
                                connecting = false;
                                setState(() {});
                              } else {
                                result = false;
                                connecting = false;
                                setState(() {});
                              }
                              if (result) {
                                print('连接成功');
                                widget.recall();
                                widget.blueClassicProvider
                                    .getWifiNetworksAndSetIP(
                                        widget.blueDevice.address ?? '');
                              } else {
                                print('连接失败');
                              }
                            },
                      child: Image.asset(
                        'assets/design/splash/珍珠耳环_20.png',
                        height: isSelected ? 210.h : 64.h,
                        width: isSelected ? 202.w : 70.w,
                        fit: BoxFit.cover,
                      ),
                    ),
                  ),
                  Visibility(
                    visible: connecting,
                    child: CupertinoActivityIndicator(
                      radius: 40.w,
                      color: AppTheme.v6White,
                    ),
                  ),
                ],
              ),
            ),
            Positioned(
              width: 310.w,
              top: 290.h,
              child: AnimatedOpacity(
                opacity: connection?.isConnected ?? false ? 1 : 1,
                duration: Duration(milliseconds: 300),
                child: connection?.isConnected ?? false
                    ? Stack(
                        clipBehavior: Clip.none,
                        children: [
                          Container(
                            height: 60.h,
                            padding: EdgeInsets.all(5.w),
                            decoration: BoxDecoration(
                              color: AppTheme.v6ThemeRed,
                              borderRadius: BorderRadius.circular(100.r),
                              // boxShadow: [
                              //   BoxShadow(
                              //     blurRadius: 1,
                              //     spreadRadius: 1,
                              //     color: AppTheme.v6ThemeRed,
                              //     offset: Offset(0, 3.h),
                              //   ),
                              // ],
                            ),
                          ),
                          Positioned(
                            top: -5.h,
                            child: ClipRRect(
                              borderRadius: BorderRadius.circular(100.r),
                              child: BackdropFilter(
                                filter:
                                    ImageFilter.blur(sigmaX: 10, sigmaY: 10),
                                child: Container(
                                  width: 310.w,
                                  height: 60.h,
                                  decoration: BoxDecoration(
                                    color: Colors.white.withOpacity(0.1),
                                    border: Border.all(
                                        color: Colors.white.withOpacity(0.44),
                                        width: 2.w),
                                    borderRadius: BorderRadius.circular(100.r),
                                  ),
                                ),
                              ),
                            ),
                          ),
                          Positioned(
                            top: -5.h,
                            child: Container(
                              height: 60.h,
                              padding: EdgeInsets.all(5.w),
                              decoration: BoxDecoration(
                                color: Colors.transparent,
                                // border: Border.all(
                                //     color: Colors.white.withOpacity(0.5)),
                                borderRadius: BorderRadius.circular(100.r),
                              ),
                              child: Row(
                                children: [
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  Image.asset(
                                    'assets/icons/png/pngX4/有音频.png',
                                    width: 22.w,
                                    height: 22.w,
                                  ),
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  ClipRRect(
                                    child: Image.asset(
                                      'assets/design/dynamic/album.png',
                                      width: 42.w,
                                      height: 42.w,
                                    ),
                                    borderRadius: BorderRadius.circular(8.r),
                                  ),
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  Column(
                                    crossAxisAlignment:
                                        CrossAxisAlignment.start,
                                    mainAxisAlignment: MainAxisAlignment.center,
                                    children: [
                                      SizedBox(
                                        width: 120.w,
                                        child: Text(
                                          widget.combinedProvider.infoResult
                                                      .title ==
                                                  ''
                                              ? "Die with a smile"
                                              : widget.combinedProvider
                                                      .infoResult.title ??
                                                  "Die with a smile",
                                          style: TextStyle(
                                              fontSize: 16.sp,
                                              overflow: TextOverflow.ellipsis,
                                              fontWeight: FontWeight.bold,
                                              color: Colors.white),
                                        ),
                                      ),
                                      SizedBox(
                                        width: 120.w,
                                        child: Text(
                                          widget.combinedProvider.infoResult
                                                      .artist ==
                                                  ''
                                              ? "Lady gaga & Bruno Mars"
                                              : widget.combinedProvider
                                                      .infoResult.artist ??
                                                  "Lady gaga & Bruno Mars",
                                          overflow: TextOverflow.ellipsis,
                                          style: TextStyle(
                                            fontSize: 12.sp,
                                            color:
                                                Colors.white.withOpacity(0.5),
                                          ),
                                        ),
                                      ),
                                    ],
                                  ),
                                  SizedBox(
                                    width: 20.w,
                                  ),
                                  Row(
                                    mainAxisAlignment:
                                        MainAxisAlignment.spaceBetween,
                                    children: [
                                      // InkWell(
                                      //   child: Container(
                                      //       width: 20.w,
                                      //       height: 20.h,
                                      //       child: Image.asset(
                                      //         'assets/icons/png/pngX4/上一首_1.png',
                                      //         fit: BoxFit.contain,
                                      //       )),
                                      //   onTap: () async {
                                      //     setState(() {});
                                      //     // await skipPrevious();
                                      //     await widget.combinedProvider
                                      //         .sendControlMusic(3);
                                      //   },
                                      // ),
                                      // combinedProvider.infoResult.status == "playing"
                                      widget.combinedProvider.play
                                          ? InkWell(
                                              onTap: () async {
                                                await widget.combinedProvider
                                                    .sendControlMusic(1);
                                                // await pause();
                                                // isplay = false;
                                                widget.combinedProvider.play =
                                                    false;

                                                setState(() {});
                                              },
                                              child: Container(
                                                width: 20.w,
                                                height: 20.h,
                                                child: Image.asset(
                                                    'assets/icons/png/播放器-暂停_44_1.png'),
                                              ))
                                          : InkWell(
                                              onTap: () async {
                                                await widget.combinedProvider
                                                    .sendControlMusic(0);
                                                // await resume();
                                                // isplay = true;
                                                widget.combinedProvider.play =
                                                    true;
                                                setState(() {});
                                              },
                                              child: Container(
                                                width: 33.w,
                                                height: 33.w,
                                                decoration: BoxDecoration(
                                                    color: Colors.white,
                                                    shape: BoxShape.circle),
                                                child: Center(
                                                  child: Image.asset(
                                                    'assets/icons/png/pngX4/Polygon_1 (2).png',
                                                    fit: BoxFit.contain,
                                                    scale: 4,
                                                  ),
                                                ),
                                              )),
                                      // InkWell(
                                      //     onTap: () async {
                                      //       setState(() {});
                                      //       // await skipNext();
                                      //       await widget.combinedProvider
                                      //           .sendControlMusic(2);
                                      //     },
                                      //     child: Container(
                                      //         width: 20.w,
                                      //         height: 20.h,
                                      //         child: Image.asset(
                                      //           'assets/icons/png/pngX4/上一首_2.png',
                                      //           fit: BoxFit.contain,
                                      //         ))),
                                    ],
                                  ),
                                ],
                              ),
                            ),
                          ),
                        ],
                      )
                    : Stack(
                        clipBehavior: Clip.none,
                        children: [
                          Container(
                            height: 60.h,
                            padding: EdgeInsets.all(5.w),
                            decoration: BoxDecoration(
                              color: AppTheme.v6ThemeRed,
                              borderRadius: BorderRadius.circular(100.r),
                              // boxShadow: [
                              //   BoxShadow(
                              //     blurRadius: 1,
                              //     spreadRadius: 1,
                              //     color: AppTheme.v6ThemeRed,
                              //     offset: Offset(0, 3.h),
                              //   ),
                              // ],
                            ),
                          ),
                          Positioned(
                            top: -5.h,
                            child: ClipRRect(
                              borderRadius: BorderRadius.circular(100.r),
                              child: BackdropFilter(
                                filter:
                                    ImageFilter.blur(sigmaX: 10, sigmaY: 10),
                                child: Container(
                                  width: 310.w,
                                  height: 60.h,
                                  decoration: BoxDecoration(
                                    color: Colors.white.withOpacity(0.1),
                                    border: Border.all(
                                        color: Colors.white.withOpacity(0.44),
                                        width: 2.w),
                                    borderRadius: BorderRadius.circular(100.r),
                                  ),
                                ),
                              ),
                            ),
                          ),
                          Positioned(
                            top: -5.h,
                            child: Container(
                              height: 60.h,
                              padding: EdgeInsets.all(5.w),
                              decoration: BoxDecoration(
                                color: Colors.transparent,
                                borderRadius: BorderRadius.circular(100.r),
                              ),
                              child: Row(
                                children: [
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  Image.asset(
                                    'assets/icons/png/pngX4/有音频.png',
                                    width: 22.w,
                                    height: 22.w,
                                  ),
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  ClipRRect(
                                    child: Image.asset(
                                      'assets/design/dynamic/album.png',
                                      width: 42.w,
                                      height: 42.w,
                                    ),
                                    borderRadius: BorderRadius.circular(8.r),
                                  ),
                                  SizedBox(
                                    width: 10.w,
                                  ),
                                  Column(
                                    crossAxisAlignment:
                                        CrossAxisAlignment.start,
                                    mainAxisAlignment: MainAxisAlignment.center,
                                    children: [
                                      SizedBox(
                                        width: 120.w,
                                        child: Text(
                                          "unKnow",
                                          style: TextStyle(
                                              fontSize: 16.sp,
                                              overflow: TextOverflow.ellipsis,
                                              fontWeight: FontWeight.bold,
                                              color: Colors.white),
                                        ),
                                      ),
                                      SizedBox(
                                        width: 120.w,
                                        child: Text(
                                          "unKnow",
                                          overflow: TextOverflow.ellipsis,
                                          style: TextStyle(
                                            fontSize: 12.sp,
                                            color:
                                                Colors.white.withOpacity(0.5),
                                          ),
                                        ),
                                      ),
                                    ],
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
            ),
          ],
        ),
      ),
    );
  }

  Row productOfflineOn(BuildContext context) {
    return Row(
      children: [
        Image.asset(
          'assets/icons/png/pngX4/offine_no.png',
          scale: 4,
        ),
        Text(
          S.of(context).splash_home_device_offline,
          style: TextStyle(color: AppTheme.white, fontSize: 12.sp),
        ),
      ],
    );
  }

  Row productOfflineOff(BuildContext context) {
    return Row(
      children: [
        Image.asset(
          'assets/icons/png/pngX4/blue_stutas.png',
          scale: 4,
        ),
        Text(
          S.of(context).splash_home_device_offline_off,
          style: TextStyle(color: AppTheme.white, fontSize: 12.sp),
        ),
      ],
    );
  }
}

class V6ScanDeviceView extends StatelessWidget {
  final BlueClassicProvider blueClassicProvider;
  final VoidCallback onTap; // 定义一个接收点击事件的回调函数
  const V6ScanDeviceView({
    super.key,
    required this.blueClassicProvider,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        const ScanColumn(),
        GestureDetector(
          onTap: () {
            onTap();
            print('1');
          },
          child: Container(
            color: Colors.transparent,
            width: 390.w,
            height: 600.h,
          ),
        ),
        if (blueClassicProvider.v6DevicesList.isNotEmpty)
          Positioned(
            top: 210.h,
            right: 30.w,
            left: 30.w,
            child: Container(
              width: 268.w,
              height: 300.h,
              decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(16.r),
                  color: Colors.white),
              child: Column(
                children: [
                  Container(
                    constraints:
                        BoxConstraints(maxHeight: 160.h, maxWidth: 268.w),
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(16.r),
                      color: Color(0xfff6f6f6),
                    ),
                    child: ListView.separated(
                      shrinkWrap: true,
                      itemCount: blueClassicProvider.v6DevicesList.length,
                      itemBuilder: (BuildContext context, int index) {
                        BluetoothDevice item =
                            blueClassicProvider.v6DevicesList[index];
                        return GestureDetector(
                          onTap: () {
                            blueClassicProvider.saveBlueInfoList(
                                BlueInformation(
                                    name: item.name, address: item.address));

                            ///添加到设备存储
                            onTap();
                            print('2');
                          },
                          child: Container(
                            width: 268.w,
                            height: 30.h,
                            child: Text(
                              item.name ?? '未知',
                              softWrap: false,
                              overflow: TextOverflow.ellipsis,
                              style: TextStyle(
                                  fontSize: 22.sp, fontWeight: FontWeight.bold),
                            ),
                          ),
                        );
                      },
                      separatorBuilder: (BuildContext context, int index) {
                        return Divider();
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),
      ],
    );
  }
}

class ScanColumn extends StatelessWidget {
  const ScanColumn({
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(
          S.of(context).scan_device_immerse,
          style:
              TextStyle(fontSize: 16.sp, color: AppTheme.splashContainerText),
        ),
        SizedBox(
          height: 50.h,
        ),
        Stack(
          children: [
            RippleEffectWidget(),
          ],
        ),
        SizedBox(
          height: 50.h,
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            SizedBox(
              width: 210.w,
              child: Row(
                mainAxisAlignment: MainAxisAlignment.start,
                children: [
                  SizedBox(
                    width: 180.w,
                    child: Text(
                      S.of(context).scan_device_search_monar_device,
                      style: TextStyle(
                          fontSize: 16.sp, color: AppTheme.splashContainerText),
                    ),
                  ),
                  SizedBox(width: 5.w),
                  SizedBox(
                    width: 25.w,
                    child: AnimatedTextKit(repeatForever: true, animatedTexts: [
                      TyperAnimatedText(
                        '...',
                        textStyle: TextStyle(
                          color: AppTheme.splashContainerText,
                          fontSize: 16.sp,
                        ),
                        speed: const Duration(milliseconds: 300),
                      ),
                    ]),
                  )
                ],
              ),
            ),
          ],
        ),
      ],
    );
  }
}

```

```
import 'dart:async';  
import 'dart:ffi';  
import 'dart:math';  
  
import 'package:flutter/cupertino.dart';  
import 'package:flutter/material.dart';  
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';  
import 'package:flutter_screenutil/flutter_screenutil.dart';  
import 'package:flutter_volume_controller/flutter_volume_controller.dart';  
import 'package:provider/provider.dart';  
  
import '../../app_theme.dart';  
import '../../generated/l10n.dart';  
import '../../models/public/wifi_model.dart';  
import '../../utils/blue_classic_provider.dart';  
import '../../utils/overlay_manage.dart';  
import '../../utils/wifi_blue_combined.dart';  
import '../../widgets/image_animation.dart';  
  
class V6ProductsDetailsView extends StatefulWidget {  
  final String address;  
  const V6ProductsDetailsView({super.key, required this.address});  
  
  @override  
  State<V6ProductsDetailsView> createState() => _V6ProductsDetailsViewState();  
}  
  
class _V6ProductsDetailsViewState extends State<V6ProductsDetailsView>  
    with SingleTickerProviderStateMixin {  
  ScrollController _scrollController = ScrollController(); //监听展开的值  
  double _expandedHeight = 540.h; // 设置初始展开高度  
  double currentExtent = 0.h;  
  late CombinedProvider combinedProvider;  
  late BlueClassicProvider blueClassicProvider;  
  late BlueInformation blueInfo;  
  late BluetoothConnection? blueInfoConnect;  
  late PageController controller;  
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
  ///亮度模式  
  int lightMode = 0;  
  List<LightMode> lightModes = [  
    LightMode(  
      mode: 0,  
      img: 'assets/icons/png/pngX4/正面_2.png',  
      str: 'splash_home_device_details_Speaker_brightness_display_daytime',  
      light: 255,  
    ),  
    LightMode(  
      mode: 1,  
      img: 'assets/icons/png/pngX4/Group_1921 (1).png',  
      str: 'splash_home_device_details_Speaker_brightness_display_cloudy',  
      light: 128,  
    ),  
    LightMode(  
      mode: 2,  
      img: 'assets/icons/png/pngX4/Group_1921.png',  
      str: 'splash_home_device_details_Speaker_brightness_display_At_night',  
      light: 32,  
    ),  
    LightMode(  
      mode: 3,  
      img: 'assets/icons/png/pngX4/Group_1921 (2).png',  
      str:  
          'splash_home_device_details_Speaker_brightness_display_Turn_off_the_screen',  
      light: 0,  
    ),  
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
    blueClassicProvider =  
        Provider.of<BlueClassicProvider>(context, listen: false);  
    combinedProvider = Provider.of<CombinedProvider>(context, listen: false);  
    // 获取蓝牙信息  
    blueInfo = blueClassicProvider.blueInfoList.firstWhere(  
      (blueInfo) => blueInfo.address == widget.address,  
      orElse: () => BlueInformation(),  
    );  
  
    print(blueInfo.name);  
    print(blueInfo.address);  
  
    // 获取蓝牙连接  
    blueInfoConnect = blueClassicProvider.v6GetConnection(widget.address);  
  
    // 检查是否有现有连接  
    if (blueInfoConnect != null) {  
      blueClassicProvider.connection = blueInfoConnect;  
    }  
  
    // 检查连接状态并获取Wi-Fi信息  
    if (blueInfoConnect?.isConnected ?? false) {  
      WidgetsBinding.instance.addPostFrameCallback((_) {  
        blueClassicProvider.getWifiNetworksAndSetIP(widget.address);  
      });  
    }  
    startListening();  
  }  
  
  @override  
  void dispose() {  
    _scrollController.removeListener(_scrollListener);  
    _scrollController.dispose();  
    _controller.dispose();  
    controller.dispose();  
    stopListening();  
    super.dispose();  
  }  
  
// 滚动监听器  
  void _scrollListener() {  
    currentExtent = _scrollController.position.extentBefore;  
    print(currentExtent);  
    setState(() {  
      _expandedHeight = 540.h - (currentExtent.h / 1.4); // 计算当前展开高度  
    });  
  }  
  
  // 初始化音量  
  Future<void> _initVolume() async {  
    currentVolume = await FlutterVolumeController.getVolume() ??  
        0.1; // _currentVolume = (await FlutterVolumeController.getVolume())!;  
  }  
  
  /// 停止监听  
  void stopListening() {  
    _timer?.cancel();  
    _timer = null;  
  }  
  
  /// 调用 sendControlMusicInfo 方法并更新状态  
  Future<void> _fetchMusicInfo() async {  
    try {  
      await combinedProvider.sendControlMusicInfo();  
    } catch (e) {  
      print('获取音乐信息失败: $e');  
    }  
  }  
  
  /// 启动定时器，每 2 秒调用一次 sendControlMusicInfo  void startListening() {  
    _timer?.cancel(); // 确保没有重复的 Timer 实例  
    _timer = Timer.periodic(const Duration(seconds: 15), (timer) async {  
      await _fetchMusicInfo();  
    });  
  }  
  
  @override  
  Widget build(BuildContext context) {  
    final blueClassicProvider = Provider.of<BlueClassicProvider>(context);  
    final combinedProvider = Provider.of<CombinedProvider>(context);  
// 处理返回按钮  
    Future<bool> onWillPop() async {  
      if (blueClassicProvider.blueOverlayVisible) {  
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
      child: Scaffold(  
        backgroundColor: AppTheme.v6ThemeBlank,  
        body: CustomScrollView(  
          controller: _scrollController,  
          slivers: [  
            SliverAppBar(  
              floating: true,  
              pinned: false, // 固定住标题，避免滚动时被遮住  
              expandedHeight: 600.h,  
              backgroundColor: AppTheme.v6ThemeBlank,  
              centerTitle: true, stretch: true, // 启用拉伸效果  
              toolbarHeight: 100,  
              title: Text('1222'),  
              flexibleSpace: FlexibleSpaceBar(  
                // expandedTitleScale: 2,  
                titlePadding: EdgeInsets.only(),  
                background: Stack(  
                  fit: StackFit.expand,  
                  children: [  
                    Image.asset(  
                      'assets/design/splash/image_297.png',  
                      scale: 1,  
                      fit: BoxFit.fill,  
                    ),  
                    Positioned(  
                      top: _expandedHeight,  
                      child: SizedBox(width: 390.w, child: builditemMode()),  
                    ),  
                  ],  
                ),  
                // centerTitle: true,  
                title: Container(  
                  margin: EdgeInsets.only(  
                    top: 260.h - (currentExtent * 200 / 248), //这里是60-260  
                    bottom: 0.h,  
                  ),  
                  child: Align(  
                    alignment: Alignment.topCenter,  
                    child: AnimatedBuilder(  
                      animation: _scrollController,  
                      builder: (context, child) {  
                        double scale = 1.0; // 默认的缩放倍数  
                        scale =  
                            1.0 - (currentExtent / 248) * 0.5; // 假设最小缩放倍数为 0.5                        return Transform.scale(  
                          scale: scale, // 设置缩放倍数  
                          child: Image.asset(  
                            'assets/design/splash/珍珠耳环_20.png',  
                            scale: 4,  
                          ), // 你的标题内容  
                        );  
                      },  
                    ),  
                  ),  
                ),  
                stretchModes: [StretchMode.zoomBackground], // 拉伸背景并放大  
              ),  
            ),  
            SliverToBoxAdapter(  
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
                      wifiCard(context, blueClassicProvider),  
                      // Container(  
                      //   child: Center(                      //     child: Text(                      //       S                      //           .of(context)                      //           .splash_home_device_details_modes_no_create,                      //       style:                      //           TextStyle(color: Colors.black, fontSize: 32.sp),                      //     ),                      //   ),                      // ),                      LightContainer(context),  
                      musicCard(context, combinedProvider),  
                      ShutdownCard(blueClassicProvider: blueClassicProvider),  
                    ],  
                  ),  
                ),  
              ),  
            ),  
          ],  
        ),  
      ),  
    );  
  }  
  
  Container musicCard(BuildContext context, CombinedProvider combinedProvider) {  
    return Container(  
      height: 360.h,  
      width: 360.w,  
      padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
      decoration: BoxDecoration(  
        color: AppTheme.v6ThemeColors,  
        borderRadius: BorderRadius.circular(15.r),  
      ),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        children: [  
          Text(  
            S.of(context).splash_home_device_details_enjoy_the_music,  
            style: AppTheme.v6blank16bold,  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 163.h,  
            padding: EdgeInsets.symmetric(vertical: 10.w, horizontal: 18.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Column(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: [  
                Row(  
                  children: [  
                    ClipRRect(  
                      child: Image.asset(  
                        'assets/design/dynamic/album.png',  
                        width: 55.w,  
                        height: 55.w,  
                      ),  
                      borderRadius: BorderRadius.circular(8.r),  
                    ),  
                    SizedBox(  
                      width: 15.w,  
                    ),  
                    Column(  
                      crossAxisAlignment: CrossAxisAlignment.start,  
                      mainAxisAlignment: MainAxisAlignment.start,  
                      children: [  
                        SizedBox(  
                          width: 220.w,  
                          child: Text(  
                            combinedProvider.infoResult.title == ''  
                                ? "Die with a smile"  
                                : combinedProvider.infoResult.title ??  
                                    "Die with a smile",  
                            style: TextStyle(  
                                fontSize: 20.sp,  
                                overflow: TextOverflow.ellipsis,  
                                fontWeight: FontWeight.bold,  
                                color: Colors.black),  
                          ),  
                        ),  
                        SizedBox(  
                          width: 220.w,  
                          child: Text(  
                            combinedProvider.infoResult.artist == ''  
                                ? "Lady gaga & Bruno Mars"  
                                : combinedProvider.infoResult.artist ??  
                                    "Lady gaga & Bruno Mars",  
                            overflow: TextOverflow.ellipsis,  
                            style: TextStyle(  
                              fontSize: 13.sp,  
                              color: AppTheme.v6AF,  
                            ),  
                          ),  
                        ),  
                      ],  
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
                              await combinedProvider.sendControlMusic(1);  
                              // await pause();  
                              // isplay = false;                              combinedProvider.play = false;  
  
                              setState(() {});  
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
                              await combinedProvider.sendControlMusic(0);  
                              // await resume();  
                              // isplay = true;                              combinedProvider.play = true;  
                              setState(() {});  
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
                      'assets/icons/png/pngX4/volume_select.png',  
                      scale: 4,  
                    ),  
                    Expanded(  
                      child: Slider(  
                        min: 0,  
                        max: 1,  
                        value: currentVolume,  
                        activeColor: AppTheme.v6Blank,  
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
                          //     currentVolume);                          // combinedProvider.sendLuminosity(                          //     currentLight.toInt());                          // sendLuminosity                        },  
                      ),  
                    ),  
                  ],  
                ),  
              ],  
            ),  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 90.h,  
            padding: EdgeInsets.symmetric(vertical: 12.w, horizontal: 18.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
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
                    Image.asset(  
                      'assets/icons/png/pngX4/均衡器_(1)_9.png',  
                      scale: 4,  
                    ),  
                  ],  
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,  
                ),  
                SingleChildScrollView(  
                  scrollDirection: Axis.horizontal,  
                  child: Row(  
                    mainAxisAlignment: MainAxisAlignment.start,  
                    children: [  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/音乐_10.png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_STANDARD,  
                              style: AppTheme.v6blank16bold,  
                            ),  
                          ],  
                        ),  
                      ),  
  
                      ///  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/语音_9 (1).png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_VOCAL,  
                              style: AppTheme.v6blank16bold,  
                            ),  
                          ],  
                        ),  
                      ),  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/跳舞_10.png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_PARTY,  
                              style: AppTheme.v6blank16bold,  
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
  Widget wifiCard(  
      BuildContext context, BlueClassicProvider blueClassicProvider) {  
    return Container(  
      width: 360.w,  
      height: 360.h,  
      padding: EdgeInsets.symmetric(vertical: 15.w, horizontal: 15.w),  
      decoration: BoxDecoration(  
          borderRadius: BorderRadius.circular(15.r),  
          color: AppTheme.v6ThemeColors),  
      margin: EdgeInsets.only(bottom: 10.h),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        mainAxisAlignment: MainAxisAlignment.start,  
        children: [  
          SizedBox(  
            height: 10.h,  
          ),  
  
          Text(  
            S.of(context).splash_home_device_details_net_connect_current,  
            style: TextStyle(  
                fontSize: 20.sp,  
                color: AppTheme.v6Blank,  
                fontWeight: FontWeight.bold),  
          ),  
          Container(  
            alignment: Alignment.centerLeft,  
            padding: EdgeInsets.symmetric(horizontal: 10.w, vertical: 1.h),  
            margin: EdgeInsets.symmetric(vertical: 15.h),  
            height: 50.h,  
            decoration: BoxDecoration(  
              color: AppTheme.v6White,  
              borderRadius: BorderRadius.circular(8.r),  
            ),  
            child: Row(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: [  
                Text(  
                  'Wi-Fi',  
                  style: TextStyle(  
                      color: AppTheme.v6Blank,  
                      fontSize: 16.sp,  
                      fontWeight: FontWeight.bold),  
                ),  
                SizedBox(  
                  width: 200.w,  
                  child: Text(  
                    blueClassicProvider.wifiName,  
                    softWrap: false,  
                    overflow: TextOverflow.ellipsis,  
                    textAlign: TextAlign.end,  
                    style: TextStyle(  
                        color: AppTheme.v6Blank,  
                        fontSize: 16.sp,  
                        fontWeight: FontWeight.bold),  
                  ),  
                ),  
              ],  
            ),  
          ),  
          Row(  
            mainAxisAlignment: MainAxisAlignment.spaceBetween,  
            children: [  
              Text(  
                S.of(context).splash_home_device_details_other_net,  
                style: TextStyle(  
                    fontSize: 20.sp,  
                    color: AppTheme.v6Blank,  
                    fontWeight: FontWeight.bold),  
              ),  
              GestureDetector(  
                onTap: () async {  
                  if (_isDebounced) return; // 如果已经在处理，直接返回  
                  print('111');  
                  _isDebounced = true;  
                  blueClassicProvider.networksData?.clear();  
                  _controller.repeat();  
                  await blueClassicProvider  
                      .getWifiNetworksAndSetIP(widget.address); // 停止旋转动画  
                  _controller.stop();  
                  _controller.reset();  
                  _isDebounced = false;  
                },  
                child: AnimatedBuilder(  
                  animation: _controller,  
                  builder: (BuildContext context, Widget? child) {  
                    return Transform.rotate(  
                      angle: _controller.value * 2 * pi, // 旋转角度  
                      child: child,  
                    );  
                  },  
                  child: Image.asset(  
                    width: 30.w,  
                    height: 30.h,  
                    'assets/icons/png/pngX4/刷新_(1)_4.png',  
                    fit: BoxFit.contain,  
                  ),  
                ),  
              )  
            ],  
          ),  
  
          /// wifi列表  
          Container(  
            width: 360.w,  
            height: (blueClassicProvider.networksData?.length ?? 0) * 50.h,  
            constraints: BoxConstraints(maxHeight: 250.h),  
            padding: EdgeInsets.symmetric(horizontal: 10.w, vertical: 1.h),  
            margin: EdgeInsets.symmetric(vertical: 15.h),  
            decoration: BoxDecoration(  
              color: AppTheme.v6White,  
              borderRadius: BorderRadius.circular(8.r),  
            ),  
            child: ListView.separated(  
              padding: EdgeInsets.zero, // 去掉默认的 padding              itemCount: blueClassicProvider.networksData?.length ?? 0,  
              separatorBuilder: (BuildContext context, int index) {  
                return Divider(  
                  height: 0,  
                );  
              },  
              itemBuilder: (context, index) {  
                String? network = blueClassicProvider.networksData?[index];  
                return GestureDetector(  
                  onTap: () {  
                    blueClassicProvider.ssidController.text = network!;  
                    final WifiModel info =  
                        blueClassicProvider.wifiModelData.firstWhere(  
                      (wifiModel) => wifiModel.wifiName == network,  
                      orElse: () => WifiModel(wifiName: ''),  
                    );  
                    blueClassicProvider.passwordController.text =  
                        info.wifiPassword ?? '';  
                    OverlayManager.showWifiOverlay(  
                        blueProvider: blueClassicProvider);  
                  },  
                  child: Container(  
                    alignment: Alignment.centerLeft,  
                    height: 50.h,  
                    decoration: BoxDecoration(  
                      color: AppTheme.v6White,  
                      borderRadius: BorderRadius.circular(8.r),  
                    ),  
                    child: Row(  
                      // mainAxisAlignment:  
                      //     MainAxisAlignment.spaceBetween,                      children: [  
                        SizedBox(  
                          width: 200.w,  
                          child: Text(  
                            network ?? S.of(context).wifi_Unknown_network,  
                            softWrap: false,  
                            overflow: TextOverflow.ellipsis,  
                            style: TextStyle(  
                                color: AppTheme.v6Blank, fontSize: 16.sp),  
                          ),  
                        ),  
                        Expanded(child: SizedBox()),  
                        Image.asset('assets/icons/png/锁_1.png'),  
                        SizedBox(  
                          width: 15.w,  
                        ),  
                        Image.asset('assets/icons/png/Frame_39.png'),  
                      ],  
                    ),  
                  ),  
                );  
              },  
            ),  
          ),  
        ],  
      ),  
    );  
  }  
  
  // Container SetWifi() {  
  //   return Container(  //     width: 390.w,  //     height: 191.h,  //     decoration: BoxDecoration(  //         color: AppTheme.v6ThemeColors,  //         borderRadius: BorderRadius.circular(16.r)),  //     padding: EdgeInsets.all(5.w),  //     child: Column(  //       children: [  //         Row(  //           mainAxisAlignment: MainAxisAlignment.center,  //           children: [  //             Text(  //               'cannel',  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //             Text(  //               S.of(context).splash_home_device_details_enter_net,  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //             Text(  //               'Join',  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //           ],  //         ),  //         TextField(),  //       ],  //     ),  //   );  // }  
  ///亮度模块  
  Widget LightContainer(BuildContext context) {  
    return Container(  
      // margin: EdgeInsets.only(top: 10.h),  
      height: 360.h,  
      width: 360.w,  
      padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
      decoration: BoxDecoration(  
        color: AppTheme.v6ThemeColors,  
        borderRadius: BorderRadius.circular(15.r),  
      ),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        children: [  
          Text(  
            S.of(context).splash_home_device_details_light,  
            style: AppTheme.v6blank16bold,  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 43.h,  
            padding: EdgeInsets.symmetric(vertical: 2.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Row(  
              children: [  
                Image.asset('assets/icons/png/15A亮度.png'),  
                Expanded(  
                  child: Slider(  
                    min: 0,  
                    max: 255,  
                    value: currentLight,  
                    activeColor: AppTheme.v6Blank,  
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
            margin: EdgeInsets.only(top: 10.h),  
            padding: EdgeInsets.symmetric(vertical: 5.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Row(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: List.generate(lightModes.length, (index) {  
                return buildSpeakDisplayItem(  
                    context: context, mode: lightModes[index]);  
              }),  
            ),  
          ),  
          Container(  
            height: 43.h,  
            margin: EdgeInsets.only(top: 20.h),  
            padding: EdgeInsets.symmetric(vertical: 2.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
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
        combinedProvider.sendLuminosity(mode.light);  
        setState(() {});  
      },  
      child: Column(  
        children: [  
          Image.asset(  
            mode.img,  
            width: 80.w,  
            height: 114.h,  
            fit: BoxFit.contain,  
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
            width: 24.w,  
            height: 24.w,  
            decoration: BoxDecoration(  
              border: Border.all(color: Colors.black),  
              color: lightMode == mode.mode ? Colors.black : Colors.transparent,  
              shape: BoxShape.circle,  
            ),  
            child: Center(child: Image.asset('assets/icons/png/对_2.png')),  
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
        width: 70.w,  
        height: 70.h,  
        decoration: BoxDecoration(  
          color: Colors.transparent,  
          border: currentPage == page  
              ? Border.all(color: Colors.white, width: 2)  
              : Border.all(color: Colors.transparent),  
          borderRadius: BorderRadius.circular(16.r),  
        ),  
        child: Center(  
          child: Container(  
            width: 55.w,  
            height: 55.h,  
            decoration: BoxDecoration(  
              color: AppTheme.v6E7,  
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
  
class ShutdownCard extends StatelessWidget {  
  const ShutdownCard({  
    super.key,  
    required this.blueClassicProvider,  
  });  
  
  final BlueClassicProvider blueClassicProvider;  
  
  @override  
  Widget build(BuildContext context) {  
    return Container(  
        height: 360.h,  
        width: 360.w,  
        padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
        decoration: BoxDecoration(  
          color: AppTheme.v6ThemeColors,  
          borderRadius: BorderRadius.circular(15.r),  
        ),  
        child: Center(  
          child: Row(  
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,  
            children: [  
              ShutButton(  
                onTap: blueClassicProvider.shutdown,  
                icon: 'assets/icons/png/pngX4/电源_12.png',  
                text: S.of(context).splash_home_device_details_shutdown,  
                title: 'dialog_inquire_shutdown',  
                content: 'dialog_inquire_shutdown_why',  
                yes: 'dialog_inquire_shutdown',  
                no: 'dialog_inquire_no',  
              ),  
              ShutButton(  
                onTap: blueClassicProvider.reboot,  
                icon: 'assets/icons/png/pngX4/刷新_1.png',  
                text: S.of(context).splash_home_device_details_reboot,  
                title: 'dialog_inquire_reboot',  
                content: 'dialog_inquire_reboot_why',  
                yes: 'dialog_inquire_reboot',  
                no: 'dialog_inquire_no',  
              ),  
            ],  
          ),  
        ));  
  }  
}  
  
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
  final int light;  
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


///设备详情页uI
```
import 'dart:async';  
import 'dart:ffi';  
import 'dart:math';  
  
import 'package:flutter/cupertino.dart';  
import 'package:flutter/material.dart';  
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';  
import 'package:flutter_screenutil/flutter_screenutil.dart';  
import 'package:flutter_volume_controller/flutter_volume_controller.dart';  
import 'package:provider/provider.dart';  
  
import '../../app_theme.dart';  
import '../../generated/l10n.dart';  
import '../../models/public/wifi_model.dart';  
import '../../utils/blue_classic_provider.dart';  
import '../../utils/overlay_manage.dart';  
import '../../utils/wifi_blue_combined.dart';  
import '../../widgets/image_animation.dart';  
  
class V6ProductsDetailsView extends StatefulWidget {  
  final String address;  
  const V6ProductsDetailsView({super.key, required this.address});  
  
  @override  
  State<V6ProductsDetailsView> createState() => _V6ProductsDetailsViewState();  
}  
  
class _V6ProductsDetailsViewState extends State<V6ProductsDetailsView>  
    with SingleTickerProviderStateMixin {  
  ScrollController _scrollController = ScrollController(); //监听展开的值  
  double _expandedHeight = 540.h; // 设置初始展开高度  
  double currentExtent = 0.h;  
  late CombinedProvider combinedProvider;  
  late BlueClassicProvider blueClassicProvider;  
  late BlueInformation blueInfo;  
  late BluetoothConnection? blueInfoConnect;  
  late PageController controller;  
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
  ///亮度模式  
  int lightMode = 0;  
  List<LightMode> lightModes = [  
    LightMode(  
      mode: 0,  
      img: 'assets/icons/png/pngX4/正面_2.png',  
      str: 'splash_home_device_details_Speaker_brightness_display_daytime',  
      light: 255,  
    ),  
    LightMode(  
      mode: 1,  
      img: 'assets/icons/png/pngX4/Group_1921 (1).png',  
      str: 'splash_home_device_details_Speaker_brightness_display_cloudy',  
      light: 128,  
    ),  
    LightMode(  
      mode: 2,  
      img: 'assets/icons/png/pngX4/Group_1921.png',  
      str: 'splash_home_device_details_Speaker_brightness_display_At_night',  
      light: 32,  
    ),  
    LightMode(  
      mode: 3,  
      img: 'assets/icons/png/pngX4/Group_1921 (2).png',  
      str:  
          'splash_home_device_details_Speaker_brightness_display_Turn_off_the_screen',  
      light: 0,  
    ),  
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
    blueClassicProvider =  
        Provider.of<BlueClassicProvider>(context, listen: false);  
    combinedProvider = Provider.of<CombinedProvider>(context, listen: false);  
    // 获取蓝牙信息  
    blueInfo = blueClassicProvider.blueInfoList.firstWhere(  
      (blueInfo) => blueInfo.address == widget.address,  
      orElse: () => BlueInformation(),  
    );  
  
    print(blueInfo.name);  
    print(blueInfo.address);  
  
    // 获取蓝牙连接  
    blueInfoConnect = blueClassicProvider.v6GetConnection(widget.address);  
  
    // 检查是否有现有连接  
    if (blueInfoConnect != null) {  
      blueClassicProvider.connection = blueInfoConnect;  
    }  
  
    // 检查连接状态并获取Wi-Fi信息  
    if (blueInfoConnect?.isConnected ?? false) {  
      WidgetsBinding.instance.addPostFrameCallback((_) {  
        blueClassicProvider.getWifiNetworksAndSetIP(widget.address);  
      });  
    }  
    startListening();  
  }  
  
  @override  
  void dispose() {  
    _scrollController.removeListener(_scrollListener);  
    _scrollController.dispose();  
    _controller.dispose();  
    controller.dispose();  
    stopListening();  
    super.dispose();  
  }  
  
// 滚动监听器  
  void _scrollListener() {  
    currentExtent = _scrollController.position.extentBefore;  
    print(currentExtent);  
    setState(() {  
      _expandedHeight = 540.h - (currentExtent.h / 1.4); // 计算当前展开高度  
    });  
  }  
  
  // 初始化音量  
  Future<void> _initVolume() async {  
    currentVolume = await FlutterVolumeController.getVolume() ??  
        0.1; // _currentVolume = (await FlutterVolumeController.getVolume())!;  
  }  
  
  /// 停止监听  
  void stopListening() {  
    _timer?.cancel();  
    _timer = null;  
  }  
  
  /// 调用 sendControlMusicInfo 方法并更新状态  
  Future<void> _fetchMusicInfo() async {  
    try {  
      await combinedProvider.sendControlMusicInfo();  
    } catch (e) {  
      print('获取音乐信息失败: $e');  
    }  
  }  
  
  /// 启动定时器，每 2 秒调用一次 sendControlMusicInfo  void startListening() {  
    _timer?.cancel(); // 确保没有重复的 Timer 实例  
    _timer = Timer.periodic(const Duration(seconds: 15), (timer) async {  
      await _fetchMusicInfo();  
    });  
  }  
  
  @override  
  Widget build(BuildContext context) {  
    final blueClassicProvider = Provider.of<BlueClassicProvider>(context);  
    final combinedProvider = Provider.of<CombinedProvider>(context);  
// 处理返回按钮  
    Future<bool> onWillPop() async {  
      if (blueClassicProvider.blueOverlayVisible) {  
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
      child: Scaffold(  
        backgroundColor: AppTheme.v6ThemeBlank,  
        body: CustomScrollView(  
          controller: _scrollController,  
          slivers: [  
            SliverAppBar(  
              floating: true,  
              pinned: false, // 固定住标题，避免滚动时被遮住  
              expandedHeight: 600.h,  
              backgroundColor: AppTheme.v6ThemeBlank,  
              centerTitle: true, stretch: true, // 启用拉伸效果  
              toolbarHeight: 100,  
              title: Text('1222'),  
              flexibleSpace: FlexibleSpaceBar(  
                // expandedTitleScale: 2,  
                titlePadding: EdgeInsets.only(),  
                background: Stack(  
                  fit: StackFit.expand,  
                  children: [  
                    Image.asset(  
                      'assets/design/splash/image_297.png',  
                      scale: 1,  
                      fit: BoxFit.fill,  
                    ),  
                    Positioned(  
                      top: _expandedHeight,  
                      child: SizedBox(width: 390.w, child: builditemMode()),  
                    ),  
                  ],  
                ),  
                // centerTitle: true,  
                title: Container(  
                  margin: EdgeInsets.only(  
                    top: 260.h - (currentExtent * 200 / 248), //这里是60-260  
                    bottom: 0.h,  
                  ),  
                  child: Align(  
                    alignment: Alignment.topCenter,  
                    child: AnimatedBuilder(  
                      animation: _scrollController,  
                      builder: (context, child) {  
                        double scale = 1.0; // 默认的缩放倍数  
                        scale =  
                            1.0 - (currentExtent / 248) * 0.5; // 假设最小缩放倍数为 0.5                        return Transform.scale(  
                          scale: scale, // 设置缩放倍数  
                          child: Image.asset(  
                            'assets/design/splash/珍珠耳环_20.png',  
                            scale: 4,  
                          ), // 你的标题内容  
                        );  
                      },  
                    ),  
                  ),  
                ),  
                stretchModes: [StretchMode.zoomBackground], // 拉伸背景并放大  
              ),  
            ),  
            SliverToBoxAdapter(  
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
                      wifiCard(context, blueClassicProvider),  
                      // Container(  
                      //   child: Center(                      //     child: Text(                      //       S                      //           .of(context)                      //           .splash_home_device_details_modes_no_create,                      //       style:                      //           TextStyle(color: Colors.black, fontSize: 32.sp),                      //     ),                      //   ),                      // ),                      LightContainer(context),  
                      musicCard(context, combinedProvider),  
                      ShutdownCard(blueClassicProvider: blueClassicProvider),  
                    ],  
                  ),  
                ),  
              ),  
            ),  
          ],  
        ),  
      ),  
    );  
  }  
  
  Container musicCard(BuildContext context, CombinedProvider combinedProvider) {  
    return Container(  
      height: 360.h,  
      width: 360.w,  
      padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
      decoration: BoxDecoration(  
        color: AppTheme.v6ThemeColors,  
        borderRadius: BorderRadius.circular(15.r),  
      ),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        children: [  
          Text(  
            S.of(context).splash_home_device_details_enjoy_the_music,  
            style: AppTheme.v6blank16bold,  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 163.h,  
            padding: EdgeInsets.symmetric(vertical: 10.w, horizontal: 18.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Column(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: [  
                Row(  
                  children: [  
                    ClipRRect(  
                      child: Image.asset(  
                        'assets/design/dynamic/album.png',  
                        width: 55.w,  
                        height: 55.w,  
                      ),  
                      borderRadius: BorderRadius.circular(8.r),  
                    ),  
                    SizedBox(  
                      width: 15.w,  
                    ),  
                    Column(  
                      crossAxisAlignment: CrossAxisAlignment.start,  
                      mainAxisAlignment: MainAxisAlignment.start,  
                      children: [  
                        SizedBox(  
                          width: 220.w,  
                          child: Text(  
                            combinedProvider.infoResult.title == ''  
                                ? "Die with a smile"  
                                : combinedProvider.infoResult.title ??  
                                    "Die with a smile",  
                            style: TextStyle(  
                                fontSize: 20.sp,  
                                overflow: TextOverflow.ellipsis,  
                                fontWeight: FontWeight.bold,  
                                color: Colors.black),  
                          ),  
                        ),  
                        SizedBox(  
                          width: 220.w,  
                          child: Text(  
                            combinedProvider.infoResult.artist == ''  
                                ? "Lady gaga & Bruno Mars"  
                                : combinedProvider.infoResult.artist ??  
                                    "Lady gaga & Bruno Mars",  
                            overflow: TextOverflow.ellipsis,  
                            style: TextStyle(  
                              fontSize: 13.sp,  
                              color: AppTheme.v6AF,  
                            ),  
                          ),  
                        ),  
                      ],  
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
                              await combinedProvider.sendControlMusic(1);  
                              // await pause();  
                              // isplay = false;                              combinedProvider.play = false;  
  
                              setState(() {});  
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
                              await combinedProvider.sendControlMusic(0);  
                              // await resume();  
                              // isplay = true;                              combinedProvider.play = true;  
                              setState(() {});  
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
                      'assets/icons/png/pngX4/volume_select.png',  
                      scale: 4,  
                    ),  
                    Expanded(  
                      child: Slider(  
                        min: 0,  
                        max: 1,  
                        value: currentVolume,  
                        activeColor: AppTheme.v6Blank,  
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
                          //     currentVolume);                          // combinedProvider.sendLuminosity(                          //     currentLight.toInt());                          // sendLuminosity                        },  
                      ),  
                    ),  
                  ],  
                ),  
              ],  
            ),  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 90.h,  
            padding: EdgeInsets.symmetric(vertical: 12.w, horizontal: 18.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
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
                    Image.asset(  
                      'assets/icons/png/pngX4/均衡器_(1)_9.png',  
                      scale: 4,  
                    ),  
                  ],  
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,  
                ),  
                SingleChildScrollView(  
                  scrollDirection: Axis.horizontal,  
                  child: Row(  
                    mainAxisAlignment: MainAxisAlignment.start,  
                    children: [  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/音乐_10.png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_STANDARD,  
                              style: AppTheme.v6blank16bold,  
                            ),  
                          ],  
                        ),  
                      ),  
  
                      ///  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/语音_9 (1).png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_VOCAL,  
                              style: AppTheme.v6blank16bold,  
                            ),  
                          ],  
                        ),  
                      ),  
                      Container(  
                        height: 33.h,  
                        margin: EdgeInsets.only(right: 10.w),  
                        padding: EdgeInsets.symmetric(horizontal: 10.w),  
                        decoration: BoxDecoration(  
                            color: AppTheme.v6ED,  
                            borderRadius: BorderRadius.circular(16.r)),  
                        child: Row(  
                          children: [  
                            Image.asset(  
                              'assets/icons/png/pngX4/跳舞_10.png',  
                              scale: 4,  
                            ),  
                            Text(  
                              S  
                                  .of(context)  
                                  .splash_home_device_details_sound_mode_PARTY,  
                              style: AppTheme.v6blank16bold,  
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
  Widget wifiCard(  
      BuildContext context, BlueClassicProvider blueClassicProvider) {  
    return Container(  
      width: 360.w,  
      height: 360.h,  
      padding: EdgeInsets.symmetric(vertical: 15.w, horizontal: 15.w),  
      decoration: BoxDecoration(  
          borderRadius: BorderRadius.circular(15.r),  
          color: AppTheme.v6ThemeColors),  
      margin: EdgeInsets.only(bottom: 10.h),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        mainAxisAlignment: MainAxisAlignment.start,  
        children: [  
          SizedBox(  
            height: 10.h,  
          ),  
  
          Text(  
            S.of(context).splash_home_device_details_net_connect_current,  
            style: TextStyle(  
                fontSize: 20.sp,  
                color: AppTheme.v6Blank,  
                fontWeight: FontWeight.bold),  
          ),  
          Container(  
            alignment: Alignment.centerLeft,  
            padding: EdgeInsets.symmetric(horizontal: 10.w, vertical: 1.h),  
            margin: EdgeInsets.symmetric(vertical: 15.h),  
            height: 50.h,  
            decoration: BoxDecoration(  
              color: AppTheme.v6White,  
              borderRadius: BorderRadius.circular(8.r),  
            ),  
            child: Row(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: [  
                Text(  
                  'Wi-Fi',  
                  style: TextStyle(  
                      color: AppTheme.v6Blank,  
                      fontSize: 16.sp,  
                      fontWeight: FontWeight.bold),  
                ),  
                SizedBox(  
                  width: 200.w,  
                  child: Text(  
                    blueClassicProvider.wifiName,  
                    softWrap: false,  
                    overflow: TextOverflow.ellipsis,  
                    textAlign: TextAlign.end,  
                    style: TextStyle(  
                        color: AppTheme.v6Blank,  
                        fontSize: 16.sp,  
                        fontWeight: FontWeight.bold),  
                  ),  
                ),  
              ],  
            ),  
          ),  
          Row(  
            mainAxisAlignment: MainAxisAlignment.spaceBetween,  
            children: [  
              Text(  
                S.of(context).splash_home_device_details_other_net,  
                style: TextStyle(  
                    fontSize: 20.sp,  
                    color: AppTheme.v6Blank,  
                    fontWeight: FontWeight.bold),  
              ),  
              GestureDetector(  
                onTap: () async {  
                  if (_isDebounced) return; // 如果已经在处理，直接返回  
                  print('111');  
                  _isDebounced = true;  
                  blueClassicProvider.networksData?.clear();  
                  _controller.repeat();  
                  await blueClassicProvider  
                      .getWifiNetworksAndSetIP(widget.address); // 停止旋转动画  
                  _controller.stop();  
                  _controller.reset();  
                  _isDebounced = false;  
                },  
                child: AnimatedBuilder(  
                  animation: _controller,  
                  builder: (BuildContext context, Widget? child) {  
                    return Transform.rotate(  
                      angle: _controller.value * 2 * pi, // 旋转角度  
                      child: child,  
                    );  
                  },  
                  child: Image.asset(  
                    width: 30.w,  
                    height: 30.h,  
                    'assets/icons/png/pngX4/刷新_(1)_4.png',  
                    fit: BoxFit.contain,  
                  ),  
                ),  
              )  
            ],  
          ),  
  
          /// wifi列表  
          Container(  
            width: 360.w,  
            height: (blueClassicProvider.networksData?.length ?? 0) * 50.h,  
            constraints: BoxConstraints(maxHeight: 250.h),  
            padding: EdgeInsets.symmetric(horizontal: 10.w, vertical: 1.h),  
            margin: EdgeInsets.symmetric(vertical: 15.h),  
            decoration: BoxDecoration(  
              color: AppTheme.v6White,  
              borderRadius: BorderRadius.circular(8.r),  
            ),  
            child: ListView.separated(  
              padding: EdgeInsets.zero, // 去掉默认的 padding              itemCount: blueClassicProvider.networksData?.length ?? 0,  
              separatorBuilder: (BuildContext context, int index) {  
                return Divider(  
                  height: 0,  
                );  
              },  
              itemBuilder: (context, index) {  
                String? network = blueClassicProvider.networksData?[index];  
                return GestureDetector(  
                  onTap: () {  
                    blueClassicProvider.ssidController.text = network!;  
                    final WifiModel info =  
                        blueClassicProvider.wifiModelData.firstWhere(  
                      (wifiModel) => wifiModel.wifiName == network,  
                      orElse: () => WifiModel(wifiName: ''),  
                    );  
                    blueClassicProvider.passwordController.text =  
                        info.wifiPassword ?? '';  
                    OverlayManager.showWifiOverlay(  
                        blueProvider: blueClassicProvider);  
                  },  
                  child: Container(  
                    alignment: Alignment.centerLeft,  
                    height: 50.h,  
                    decoration: BoxDecoration(  
                      color: AppTheme.v6White,  
                      borderRadius: BorderRadius.circular(8.r),  
                    ),  
                    child: Row(  
                      // mainAxisAlignment:  
                      //     MainAxisAlignment.spaceBetween,                      children: [  
                        SizedBox(  
                          width: 200.w,  
                          child: Text(  
                            network ?? S.of(context).wifi_Unknown_network,  
                            softWrap: false,  
                            overflow: TextOverflow.ellipsis,  
                            style: TextStyle(  
                                color: AppTheme.v6Blank, fontSize: 16.sp),  
                          ),  
                        ),  
                        Expanded(child: SizedBox()),  
                        Image.asset('assets/icons/png/锁_1.png'),  
                        SizedBox(  
                          width: 15.w,  
                        ),  
                        Image.asset('assets/icons/png/Frame_39.png'),  
                      ],  
                    ),  
                  ),  
                );  
              },  
            ),  
          ),  
        ],  
      ),  
    );  
  }  
  
  // Container SetWifi() {  
  //   return Container(  //     width: 390.w,  //     height: 191.h,  //     decoration: BoxDecoration(  //         color: AppTheme.v6ThemeColors,  //         borderRadius: BorderRadius.circular(16.r)),  //     padding: EdgeInsets.all(5.w),  //     child: Column(  //       children: [  //         Row(  //           mainAxisAlignment: MainAxisAlignment.center,  //           children: [  //             Text(  //               'cannel',  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //             Text(  //               S.of(context).splash_home_device_details_enter_net,  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //             Text(  //               'Join',  //               style: TextStyle(fontSize: 20.sp, fontWeight: FontWeight.w600),  //             ),  //           ],  //         ),  //         TextField(),  //       ],  //     ),  //   );  // }  
  ///亮度模块  
  Widget LightContainer(BuildContext context) {  
    return Container(  
      // margin: EdgeInsets.only(top: 10.h),  
      height: 360.h,  
      width: 360.w,  
      padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
      decoration: BoxDecoration(  
        color: AppTheme.v6ThemeColors,  
        borderRadius: BorderRadius.circular(15.r),  
      ),  
      child: Column(  
        crossAxisAlignment: CrossAxisAlignment.start,  
        children: [  
          Text(  
            S.of(context).splash_home_device_details_light,  
            style: AppTheme.v6blank16bold,  
          ),  
          Container(  
            margin: EdgeInsets.only(top: 10.h, bottom: 10.h),  
            height: 43.h,  
            padding: EdgeInsets.symmetric(vertical: 2.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Row(  
              children: [  
                Image.asset('assets/icons/png/15A亮度.png'),  
                Expanded(  
                  child: Slider(  
                    min: 0,  
                    max: 255,  
                    value: currentLight,  
                    activeColor: AppTheme.v6Blank,  
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
            margin: EdgeInsets.only(top: 10.h),  
            padding: EdgeInsets.symmetric(vertical: 5.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
                borderRadius: BorderRadius.circular(12.r)),  
            child: Row(  
              mainAxisAlignment: MainAxisAlignment.spaceBetween,  
              children: List.generate(lightModes.length, (index) {  
                return buildSpeakDisplayItem(  
                    context: context, mode: lightModes[index]);  
              }),  
            ),  
          ),  
          Container(  
            height: 43.h,  
            margin: EdgeInsets.only(top: 20.h),  
            padding: EdgeInsets.symmetric(vertical: 2.w, horizontal: 10.h),  
            decoration: BoxDecoration(  
                color: AppTheme.white,  
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
        combinedProvider.sendLuminosity(mode.light);  
        setState(() {});  
      },  
      child: Column(  
        children: [  
          Image.asset(  
            mode.img,  
            width: 80.w,  
            height: 114.h,  
            fit: BoxFit.contain,  
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
            width: 24.w,  
            height: 24.w,  
            decoration: BoxDecoration(  
              border: Border.all(color: Colors.black),  
              color: lightMode == mode.mode ? Colors.black : Colors.transparent,  
              shape: BoxShape.circle,  
            ),  
            child: Center(child: Image.asset('assets/icons/png/对_2.png')),  
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
        width: 70.w,  
        height: 70.h,  
        decoration: BoxDecoration(  
          color: Colors.transparent,  
          border: currentPage == page  
              ? Border.all(color: Colors.white, width: 2)  
              : Border.all(color: Colors.transparent),  
          borderRadius: BorderRadius.circular(16.r),  
        ),  
        child: Center(  
          child: Container(  
            width: 55.w,  
            height: 55.h,  
            decoration: BoxDecoration(  
              color: AppTheme.v6E7,  
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
  
class ShutdownCard extends StatelessWidget {  
  const ShutdownCard({  
    super.key,  
    required this.blueClassicProvider,  
  });  
  
  final BlueClassicProvider blueClassicProvider;  
  
  @override  
  Widget build(BuildContext context) {  
    return Container(  
        height: 360.h,  
        width: 360.w,  
        padding: EdgeInsets.symmetric(vertical: 15.h, horizontal: 15.w),  
        decoration: BoxDecoration(  
          color: AppTheme.v6ThemeColors,  
          borderRadius: BorderRadius.circular(15.r),  
        ),  
        child: Center(  
          child: Row(  
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,  
            children: [  
              ShutButton(  
                onTap: blueClassicProvider.shutdown,  
                icon: 'assets/icons/png/pngX4/电源_12.png',  
                text: S.of(context).splash_home_device_details_shutdown,  
                title: 'dialog_inquire_shutdown',  
                content: 'dialog_inquire_shutdown_why',  
                yes: 'dialog_inquire_shutdown',  
                no: 'dialog_inquire_no',  
              ),  
              ShutButton(  
                onTap: blueClassicProvider.reboot,  
                icon: 'assets/icons/png/pngX4/刷新_1.png',  
                text: S.of(context).splash_home_device_details_reboot,  
                title: 'dialog_inquire_reboot',  
                content: 'dialog_inquire_reboot_why',  
                yes: 'dialog_inquire_reboot',  
                no: 'dialog_inquire_no',  
              ),  
            ],  
          ),  
        ));  
  }  
}  
  
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
  final int light;  
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