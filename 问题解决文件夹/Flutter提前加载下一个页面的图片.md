跳转时,下一个页面的图片可能还没有加载出来,即使是assets本地图片也会出现这样的情况.
解决方法.在跳转前,用precacheImage提前加载图片url
```
final String nextImagePath = 'assets/design/image_12.png';

@override  
Widget build(BuildContext context) {  
  precacheImage(AssetImage(nextImagePath), context); // 在构建时预加载图片  
  return Scaffold(  
    backgroundColor: AppTheme.splash,  
    body: Center();
```
这样在下一个页面调用nextImagePath的url图片就可以提前加载