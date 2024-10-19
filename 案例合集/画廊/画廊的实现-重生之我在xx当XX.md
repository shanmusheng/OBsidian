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
# 第二天
 >写这个还是蛮窒息的,不过在没做成之前,一切都没意义啊
## manager
这是初步的manage代码,后续还会对看具体进行修改,注释都在代码里
```
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:path_provider/path_provider.dart';
import 'package:shared_preferences/shared_preferences.dart';

class ImageInformation {
  String name; //图片名称
  List<String> tags; //图片的tags
  String path; //图片路径
  String description; //图片描述
  ImageInformation({
    required this.name,
    required this.tags,
    required this.path,
    required this.description,
  });
  // 将类对象转换为 Map，用于 JSON 序列化
  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'tags': tags,
      'path': path,
      'description': description,
    };
  }

// 从 Map 创建类实例，用于反序列化
  factory ImageInformation.fromJson(Map<String, dynamic> json) {
    return ImageInformation(
      name: json['name'],
      tags: List<String>.from(
          json['tags'] ?? []), // 显式转换为 List<String>,不然后面修改tag会导致数据类型不匹配
      path: json['path'],
      description: json['description'],
    );
  }
}

class ImageManager extends ChangeNotifier {
  SharedPreferences? _preferences; //存储
  final ImagePicker _picker = ImagePicker(); //选择图片
  List<String> _imageTags = []; //存放所有搜索tags
  List<String> get imageTags => _imageTags;
  List<ImageInformation> _imageInfoList = [];
  List<ImageInformation> get imageInfoList => _imageInfoList;
  GalleryManage() {
    _init();
  }

  Future<void> _init() async {
    _preferences = await SharedPreferences.getInstance();
    await getImageTags();
    _imageInfoList = await getImageInfoList();
    notifyListeners(); //通知更新
  }

  //加载tags
  Future<List<String>> getImageTags() async {
    // 从 SharedPreferences 获取存储的 JSON 字符串列表
    _imageTags = _preferences!.getStringList('image_tags') ?? [];
    notifyListeners();
    return _imageTags;
  }

  //加载图片集
  Future<List<ImageInformation>> getImageInfoList() async {
    // 从 SharedPreferences 获取存储的 JSON 字符串列表
    List<String>? jsonList = _preferences?.getStringList('image_info_list');
    if (jsonList != null) {
      return jsonList.map((jsonString) {
        Map<String, dynamic> json = jsonDecode(jsonString);
        return ImageInformation.fromJson(json);
      }).toList();
    }
    notifyListeners();
    return [];
  }

  //把图片存储到图片集
  Future<void> saveImageInfoList(ImageInformation info) async {
    _imageInfoList.add(info);
    // 将每个 ImageInfo 对象转换为 JSON，并将整个列表转换为 JSON 字符串
    List<String> jsonList = _imageInfoList
        .map((imageInfo) => jsonEncode(imageInfo.toJson()))
        .toList();
    // 存储 JSON 字符串列表到 SharedPreferences
    await _preferences?.setStringList('image_info_list', jsonList);
    print('ImageInfo list saved.');
    notifyListeners();
  }

  //用相册来选择图片,并添加到图片集里
  Future<void> pickImageInfoFromGallery() async {
    File? _image;
    final pickedFile = await _picker.pickImage(source: ImageSource.gallery);
    if (pickedFile != null) {
      _image = File(pickedFile.path);
      // 使用 image.path 来获取图片的路径
      print('选择的图片路径: ${_image.path}');
      final directory = await getApplicationDocumentsDirectory();
      print('文档目录路径: ${directory.path}');
      String uniquePathName = '${DateTime.now().millisecondsSinceEpoch}'; //避免相同
      Directory newDir = Directory('${directory.path}/A');
      if (!await newDir.exists()) {
        await newDir.create(recursive: false); // 创建目录及其父目录
      }
      String newPath = '${newDir.path}/image_$uniquePathName.png';
      await _image.copy(newPath); //把原图_image复制到创建的路径newPath
      print('图片已保存到: $newPath');
      saveImageInfoList(ImageInformation(
          name: uniquePathName, tags: [], path: newPath, description: '默认'));
    }
  }

  //删除图片
  Future<void> deleteImageInfo(String path) async {
    // 2. 查找要删除的 ImageInfo 对象
    ImageInformation? imageInfoToDelete;
    for (var imageInfo in imageInfoList) {
      if (imageInfo.path == path) {
        imageInfoToDelete = imageInfo;
        break;
      }
    }

    // 3. 如果找到了 ImageInfo 对象
    if (imageInfoToDelete != null) {
      // 删除本地文件
      File file = File(imageInfoToDelete.path);
      if (await file.exists()) {
        await file.delete();
        print('本地图片文件已删除: ${imageInfoToDelete.path}');
      } else {
        print('文件未找到: ${imageInfoToDelete.path}');
      }

      // 4. 从列表中删除该 ImageInfo 对象
      imageInfoList.remove(imageInfoToDelete);
      notifyListeners();
      // 5. 将更新后的列表重新保存到 SharedPreferences 中
      await reloadImageInfoList();
      print('ImageInfo 已删除并更新列表.');
    } else {
      print('未找到指定路径的 ImageInfo 对象.');
    }
  }

  //重新加载_imageInfoList变化后的图片集(_preferences更新最新的图片数据)
  Future<void> reloadImageInfoList() async {
    // 将每个 ImageInfo 对象转换为 JSON，并将整个列表转换为 JSON 字符串
    List<String> jsonList = _imageInfoList
        .map((imageInfo) => jsonEncode(imageInfo.toJson()))
        .toList();
    // 存储 JSON 字符串列表到 SharedPreferences
    await _preferences?.setStringList('image_info_list', jsonList);
    print('ImageInfo list reload.');
    notifyListeners();
  }

  // 修改指定 newPath 的 ImageInfo 对象的tags，并更新 SharedPreferences 中的列表
  Future<void> updateImageInfoTags(String newPath, String? newTag) async {
    print('添加之前的tag${_imageInfoList.first.tags}');
    //查找需要修改的 ImageInfo 对象
    for (var imageInfo in _imageInfoList) {
      if (imageInfo.path == newPath) {
        // 修改对象的属性
        if (newTag != null) {
          print(newTag);
          imageInfo.tags.add(newTag); // 修改 tags
          ///这里存放图片的tag,当每次更新时做一个判断
          List<String> list1 = [..._imageTags, newTag].toSet().toList();
          _imageTags = list1;
          print([..._imageTags]);
          reloadImageTags();
        }
        break; // 找到并修改后退出循环
      }
    }
    // 打印修改后的 tags
    print('修改后的 tags: ${_imageInfoList.first.tags}');
    // 将更新后的列表重新保存到 SharedPreferences 中
    notifyListeners();
    await reloadImageInfoList();
    print('ImageInfo 已更新并保存.');
  }

  // 修改指定 newPath 的 ImageInfo 对象的名称，并更新 SharedPreferences 中的列表
  Future<void> updateImageInfoName(String newPath, String? newName) async {
    print('添加之前的tag${_imageInfoList.first.tags}');
//查找需要修改的 ImageInfo 对象
    for (var imageInfo in _imageInfoList) {
      if (imageInfo.path == newPath) {
        // 修改对象的属性
        if (newName != null) {
          imageInfo.name = newName; // 修改 path
        }
        break; // 找到并修改后退出循环
      }
    }
    // 将更新后的列表重新保存到 SharedPreferences 中
    notifyListeners();
    await reloadImageInfoList();
    print('ImageInfo 已更新并保存.');
  }

  //重新加载_imageTags变化后的图片集
  Future<void> reloadImageTags() async {
    // 存储 JSON 字符串列表到 SharedPreferences
    await _preferences?.setStringList('image_tags', _imageTags);
    print('image_tags list saved.');
    notifyListeners();
  }

  // 检查文件夹中的文件路径是否都存在于 imageInfos 中
  Future<void> checkMissingPaths() async {
    List<ImageInformation> missingPaths = _imageInfoList; // 创建 _imagePaths 的副本
    Directory _directory = await getApplicationDocumentsDirectory();
    Directory newDir = Directory('${_directory.path}/A');
    // 获取文件夹中的所有文件
    List<FileSystemEntity> files = newDir.listSync();
    List<String> filePaths = files
        .map((file) => file.path.replaceAll(r'\', '/'))
        .toList(); // 将反斜杠替换为正斜杠
    var imgPaths =
        List.from(_imageInfoList.map((info) => info.path)); //获取所有的图片路径
    for (var img in imgPaths) {
      print(img);
      String standardizedImg =
          img.replaceAll(r'\', '/'); // 也标准化 _imagePaths 中的路径
      if (!filePaths.contains(standardizedImg)) {
        // 使用路径字符串进行比较
        missingPaths.removeWhere((info) {
          if (info.path == img) {
            return true;
          }
          return false;
        });
        print('被移除的图片为 $img');
      }
    }
    print('checkMissingPaths');
    _imageInfoList = missingPaths; // 更新 _imagePaths 列表
    reloadImageInfoList();
    notifyListeners(); // 通知监听者数据已更新
  }
}

```
## 页面搭建

