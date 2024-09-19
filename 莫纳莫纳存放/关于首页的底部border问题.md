#flutter_screenutil 
当随意拖拉时,底部会发生很大的形变,
原先在没有尺寸模版时,采用的是固定写死的宽度
```
topLeft: Radius.circular(60),
topRight: Radius.circular(60)),
```
当实际大小为size(391,844)时
![[Pasted image 20240920052352.png]]
当实际大小为size(1188,844)
![[Pasted image 20240920052424.png]]
当实际大小为size(132,844)时
![[Pasted image 20240920052503.png]]
看以看出当写死时,虽然在符合设计图纸的尺寸下很完美,但是一旦设备的尺寸发生变化,就会导致不美观,并且突兀,
这个时候就要用到屏幕适配来解决这个情况
这个时候就要获取屏幕尺寸
```
Size size = MediaQuery.sizeOf(context);
```
然后利用这个size来更改大小(60/390约等于0.153)
```
topLeft: Radius.circular(size.width * 0.153),
topRight: Radius.circular(size.width * 0.153)
```
当实际大小为size(391,844)时
![[Pasted image 20240920053120.png]]
当实际大小为size(1188,844)\
![[Pasted image 20240920053139.png]]
当实际大小为size(132,844)时
![[Pasted image 20240920053223.png]]
但是还可以利用插件来解决这个情况
#flutter_screenutil 
因为目前这个情境下的效果和用size是一样的,这里不再多做演示
```
topLeft: Radius.circular(60.w),  
topRight: Radius.circular(60.w)),
```
所以所以,为什么要用这个插件呢,因为这个插件可以直接用设计稿的比例啊!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!