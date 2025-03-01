[卡片交互动效还可以这么做，好惊艳🌟 - 小红书](https://www.xiaohongshu.com/explore/67884815000000001601a923?app_platform=ios&app_version=8.69.4&share_from_user_hidden=true&xsec_source=app_share&type=video&xsec_token=CBMJx8BsH7y0FXLU912eRBBy-bEitsfKV3Bdqz1M1bmJo=&author_share=1&xhsshare=WeixinSession&shareRedId=ODc0ODY8PUo2NzUyOTgwNjY0OThKNkg9&apptime=1740123504&share_id=dad9d3a2af494245ac84db606b7aad5e)![[1040g2so31cmp1aj410705n1lgi9hv8i8duj48lg.mp4]]
如上所示
要考虑的是滑动时,上下间隔角度是固定的

所有的delta都是上滑动为负,下滑为正,也对应角度的大小


a long time

和ai对抗了一会,,,,,

a long time


ai甘拜下风


a long time

```
double angleFun4(int layer) {  
  // 设定固定夹角  
  const double theta = pi * 30 / 180; // 30°  
  const double alpha = pi * 60 / 180; // 60°  
  
  // 计算整体平移量，让所有层的角度一起变化  
  double shift = pi * 15 / 180 * sin(delta * pi / 2);  
  
  if (layer == 3) {  
    // 中间层始终为 0°，只受 shift 影响  
    return shift;  
  } else if (layer == 2) {  
    // 上层 `2` 为正 `theta`，下层 `2` 为负 `theta`    return (delta < 0 ? theta : -theta) + shift;  
  } else if (layer == 1) {  
    // 上层 `1` 为正 `alpha`，下层 `1` 为负 `alpha`    return (delta < 0 ? alpha : -alpha) + shift;  
  } else {  
    return 0;  
  }  
}
```

虽然这个方法并不会很好,但是也拿到了思路
思路:
默认角度加变化角度
3变2的时候,并不是初始角度,而是初始角度加变化角度=初始角度

在极限一点

3变2的临界点时
3的初始角度加上变化角度\==2的初始角度
先考虑上划
3   delta:     0--->-0.5
2   delta:    -1--->-1.5