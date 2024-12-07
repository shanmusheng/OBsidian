`PreferredSize` 是一个非常实用的组件，特别用于那些没有固定高度的组件。它通过 `preferredSize` 属性允许你控制组件的大小，使得你能够灵活地调整界面的布局和外观。

### `PreferredSize` 的作用

- **用来指定组件的首选尺寸（size）**，使其根据你指定的宽度和高度进行显示。
- 常常用于自定义 `AppBar`、`TabBar` 或 `SliverAppBar` 等组件的尺寸，因为这些组件的默认尺寸可能无法满足你的需求。

```
return Scaffold(
      appBar: PreferredSize(
        preferredSize: Size.fromHeight(120.0), // 设置AppBar的高度为120.0
        child: AppBar(
          title: Text('Custom AppBar Height'),
          backgroundColor: Colors.blue,
          centerTitle: true,
        ),
      ),
```
比如这样
