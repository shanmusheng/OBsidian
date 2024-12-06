```
final List<String> imageUrls = [  
  'assets/design/image_12.png',  
  'assets/design/home/image_114.png', 
  'assets/design/home/Group_1811.png',  
  'assets/design/home/image_127.png',];

Future<void> _preloadImages() async {  
  try {  
    await Future.wait(imageUrls.map((url) {  
      print('url$url');  
      return precacheImage(AssetImage(url), context).catchError((e) {  
        print("Failed to load image: $url");  
      });  
    }));  
  } catch (e) {  
    print("Error preloading images: $e");  
  }  
}




Future<void> _preloadImagesAndNavigate(BuildContext context) async {  
  try {  
    // 预加载图片  
    // 记录开始时间  
    DateTime startTime = DateTime.now();  
    await _preloadImages();  
    // 记录结束时间  
    DateTime endTime = DateTime.now();  
    Duration difference = endTime.difference(startTime);  
    print(  
        'Method execution time: ${difference.inMilliseconds} ms startTime${startTime}');  
    await Future.delayed(Duration(milliseconds: 1500));  
    // 图片加载完成后跳转到目标页面  
    Navigator.pushReplacementNamed(context, MonarRoutes.splashHome);  
  } catch (e) {  
    print("Error preloading images: $e");  
    await Future.delayed(Duration(milliseconds: 1500));  
    // 如果发生错误，也可以直接跳转到目标页面  
    Navigator.pushReplacementNamed(context, MonarRoutes.splashHome);  
  }  
}
```

## 如果你有provider什么的
这样去获取
```
_preloadImagesAndNavigate(  
  context,  
  preferencesProvider.imagePaths.map((e) => e.img).toList(),  
  preferencesProvider.gifPaths.map((e) => e.img).toList(),  
  preferencesProvider.songPaths.map((e) => e.img).toList(),  
);
```