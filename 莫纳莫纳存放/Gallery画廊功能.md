要实现的功能为
默认初始页面为画廊主页,下方是bottomBar,可以切换搜索,画廊主页和收藏

![[Pasted image 20240921212255.png]]
## 结构分析
具体实现上面的代码,那么大致的框架应该为
```
class GalleryHome extends StatefulWidget {
  const GalleryHome({super.key});

  @override
  State<GalleryHome> createState() => _GalleryHomeState();
}

class _GalleryHomeState extends State<GalleryHome> {
  @override
  Widget build(BuildContext context) {
    Size size = MediaQuery.of(context).size;
    return Scaffold(
      appBar: AppBar(
        title: Text('$size'),
      ),
      body: PageView(
        children: [
          Column(
            children: [
              Container(
                color: Colors.blue,
                width: 300,
                height: 200,
                child: Text('300 200'),
              ),
            ],
          ),
          Column(
            children: [
              Container(
                color: Colors.blue,
                width: 300,
                height: 200,
                child: Text('300 200'),
              ),
            ],
          ),
          Column(
            children: [
              Container(
                color: Colors.blue,
                width: 300,
                height: 200,
                child: Text('300 200'),
              ),
            ],
          ),
        ],
      ),
      bottomNavigationBar: BottomNavigationBar(items: const [
        BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),
        BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),
        BottomNavigationBarItem(icon: Icon(Icons.add), label: '1'),
      ]),
    );
  }
}
```
## 问题一,在Flutter的Windows桌面应用中,`PageView`不能左右拖动页面
### 原因
通常是因为`PageView`在桌面平台上的默认交互行为与移动平台不同。Flutter的`PageView`在桌面端没有自动启用鼠标拖动功能
### 解决原因
通过手动添加鼠标监听或使用`GestureDetector`来实现这一功能。
### 解决
通过以下步骤实现鼠标拖动切换页面：

1. 使用`PageController`来管理`PageView`的页面控制。
2. 添加`GestureDetector`来监听左右拖动手势。
3. 根据拖动手势手动调用`PageController`的`animateToPage`方法切换页面。