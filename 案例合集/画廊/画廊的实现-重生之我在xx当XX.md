这个文档是专门拿来存储制作的记录信息
# 第一天
首先创建一个flutter项目`gallery_sms`[杉木笙/gallery_sms (gitee.com)](https://gitee.com/fir-sheng/gallery_sms)

首先导入我们需要的插件在`pubspec.yaml`中
```
permission_handler: ^10.2.0  
path_provider: ^2.1.3  
provider: ^6.1.2  
shared_preferences: ^2.3.2  
flutter_screenutil: ^5.9.3  
image_picker: ^1.1.2
```
![[Pasted image 20241018141827.png]]
然后在`android/app/src/main/AndroidManifest.xml`中添加相应的权限
```
<manifest
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-feature
        android:name="android.hardware.camera"
        android:required="false" />

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.WRITE_SETTINGS"
        tools:ignore="ProtectedPermissions" />
```
![[Pasted image 20241018142138.png]]
这样最基本的插件和权限都已经设置好了
然后开始页面的基本搭建
## 页面搭建
在搭建页面前要对整体的设计进行一个约定,这里是`flutter_screenutil`来实现,由于我的设计稿是(360,690)所以这里我的设计稿就是这个,这一步请根据你的设计稿来修改
在`MaterialApp`外包裹`ScreenUtilInit`并设置`designSize`参数为你的设计稿大小
添加后
![[Pasted image 20241018142758.png]]
创建一个widget用来存放主页面
```
class Home extends StatefulWidget {  
  const Home({super.key});  
  
  @override  
  State<Home> createState() => _HomeState();  
}  
  
class _HomeState extends State<Home> {  
  @override  
  Widget build(BuildContext context) {  
    return Scaffold(  
      body: PageView(  
        children: [  
          Placeholder(),//暂时替代页面  
          Placeholder(),  
          Placeholder(),  
        ],  
      ),  
    );  
  }  
}
```
小提示,如果你的开发是用的桌面端,会发现你的鼠标并不能像手指一样去滚动这个界面[FLutter桌面端怎么不能触摸滑动这个原因是在Flutter桌面端，PageView 默认并不支持触摸滑动，主要是因为 - 掘金 (juejin.cn)](https://juejin.cn/post/7418075685824462898) 这篇文章里简单明确的指出了解决方案
所以在加上
```
MaterialApp(
...
scrollBehavior: DesktopScrollBehavior(),
...
)

class DesktopScrollBehavior extends MaterialScrollBehavior {  
  @override  
  Set<PointerDeviceKind> get dragDevices => {  
        PointerDeviceKind.touch,  
        PointerDeviceKind.mouse,  
      };  
}
```
![[Pasted image 20241018143624.png]]
这样就解决了
利用`PageController`和`bottomNavigationBar`来给这个`PageView`加上切换页面功能
```
final PageController _pageController = PageController(initialPage: 2);  
int pageIndex = 0;

PageView(  
  controller: _pageController,  
  onPageChanged: (value) {  
    pageIndex = value;  
    setState(() {});  
  },


bottomNavigationBar: SizedBox(  
  height: 80.h,  
  child: BottomNavigationBar(  
    items: const [  
      BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),  
      BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),  
      BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),  
    ],  
    onTap: (index) {  
      print(index);  
      _pageController.jumpToPage(index);  
    },  
  ),  
),
```
![[Pasted image 20241018150851.png]]
接下来是设计最主要的manager,去管理所有图片的信息和储存
## manager
