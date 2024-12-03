 相关链接 [Flutter 教程 Key-1 没有 Key 会发生什么奇怪现象_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1b54y1z7iD/?spm_id_from=333.999.0.0&vd_source=30c5e78b65f0821d46fd06f4e6c729a4)
 在渲染树里,flutter并不能通过颜色或者其他数值来确定当前组件是什么,所以需要一个类似id的东西去来识别.
 这个就是==key==
 
 ![[Pasted image 20241203144325.png]]
 在不穿key的时候,蓝图的渲染是1对1
 如果更改这里的颜色
 ![[Pasted image 20241203144425.png]]
 ,换个角度看其实并没有更改类型,还是box对box,所以这里怎么更换颜色,这里都没有变化,类型还是类型,状态没有改变
 ![[Pasted image 20241203144659.png]]
 
 
 
 但是如果加上key之后,在进行更换虽然box没有变化,但是key发生了变化,所以检查完类型之后,还要检查key,
 ![[Pasted image 20241203144621.png]]
 ![[Pasted image 20241203144815.png]]
 ![[Pasted image 20241203144826.png]]
 ![[Pasted image 20241203144839.png]]
 ![[Pasted image 20241203144846.png]]
 
 
key的类型
分为局部和全局
 ![[Pasted image 20241203145139.png]]