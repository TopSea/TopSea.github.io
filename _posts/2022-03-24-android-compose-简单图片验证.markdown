---
layout: post
title: Android Compose简单图片验证
date: 2022-03-24
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: image-verify.png # Add image post (optional)
tags: [Android, Compose, Jetpack] # add tag
---
最近在学Jetpack Compose，简单整个图片验证试试手。基本逻辑就是用户滑动小球，图片跟着缩放，
当图片大小与其原图相似且用户在此停留两秒左右即算成功。

ProgressBar
---
目前的 Jetpack Compose 里面是没有ProgressBar组件的。只能自己做一个了，这里偷个懒就不用 Canvas 去画了，
就用 `Divider()` 加一个 Box 的小球凑合了。代码如下所示：   

~~~kotlin
@Composable
fun ProgressBar(
        onScroll: (Float) -> Unit
) {
 //滑动的位置
  var offsetX by remember { mutableStateOf(0f) }
  val configuration = LocalContext.current.resources.displayMetrics
  val ballSize = LocalDensity.current.run { 20.dp.toPx() }
 //屏幕宽度
  val screenWidth = configuration.widthPixels
  val padding = (screenWidth * 0.15f).toInt()
 //滑动的边界
  val ballEnd = (screenWidth - padding - ballSize).toInt()
 
  Box(contentAlignment = Alignment.Center, modifier = Modifier.fillMaxWidth()) {
     Divider(modifier = Modifier
             .fillMaxWidth(0.85f)
             .size(2.dp))
     Box(contentAlignment = Alignment.CenterStart, modifier = Modifier.fillMaxWidth(0.85f)) {
        Box(modifier = Modifier
           .offset { IntOffset(offsetX.roundToInt(), 0) }
           .size(20.dp)
           .background(Color.Red, CircleShape)
           .draggable(
              orientation = Orientation.Horizontal,
              state = rememberDraggableState { delta ->
                 onScroll(delta)
                 offsetX = if ((offsetX + delta).toInt() in 0..ballEnd) {
                    offsetX + delta
                 } else {
                    offsetX
                 }
              }
           )
        )
     }
   }
}
~~~    

代码也不复杂，内部消化小球的移动状态然后通过状态提升来处理滑动时要处理的事情，
还不了解状态提升的同学可以点这里
[状态提升](https://developer.android.google.cn/jetpack/compose/state#state-hoisting)
。
 
图片操作和判断   
---
这里用 Canvas 的原因是 Canvas 能够获取自己的宽度和高度，比较好用来做判断。   
不要用太大的图不然滑动的时候会卡。代码如下：
~~~kotlin
@Composable
fun MyCanvas(
    width: Float
) {
   val context = LocalContext.current
   //注意上太大的图会卡
   val bitmap = BitmapFactory.decodeResource(context.resources, R.drawable.dog)
   var success by remember { mutableStateOf(false) }
   val dst = bitmap.height / (bitmap.width).toFloat()
  
   val current = mutableListOf<Long>()
   val executor = Executors.newSingleThreadExecutor()
  
   Column {
      if(success) {
          Text(text = "success!", fontSize = 30.sp, color = Color.Green)
      }
      Canvas(modifier = Modifier
          .fillMaxWidth(width)
          .height(200.dp)
          .background(Color.LightGray)
      ) {
         drawImage(
             image = bitmap.asImageBitmap(),
             srcOffset = IntOffset.Zero,
             dstOffset = IntOffset.Zero,
             dstSize = IntSize((size.width).toInt(), (size.height).toInt()),
             alpha = 1.0f,
             style = Fill,
             blendMode = DefaultBlendMode
         )
         val nowSize = size.height / size.width
         if (nowSize in dst - 0.1.. dst + 0.1) {
            current.add(System.currentTimeMillis())
         } else {
            current.clear()
         }
         if (current.size > 0) {
            executor.execute {
             //成功匹配后等待两秒再判定
                while (!success) {
                   success = System.currentTimeMillis() - current[0] in 1800..2200
                }
             current.clear()
             executor.shutdown()
            }
         } else {
            success = false
         }
      }
   }
}
~~~    
传入变化的 Canvas 的宽度，根据宽度和高度的比值和图片的比值相比较（我这个设置的高比较小，
所以最好用高大于宽的图片）。如果值相似就存下当前的 `System.currentTimeMillis()` ,
然后进入判断。我试着用 scope 来做判断，但是会卡不知道为什么，所以就用了 executor 。

调用方
---
~~~kotlin
var width by remember { mutableStateOf(1f) }
val onScroll = { delta: Float ->
    width -= delta / 1000
}
Column(
    horizontalAlignment = Alignment.CenterHorizontally,
    verticalArrangement = Arrangement.Center,
    modifier = Modifier
        .fillMaxWidth(0.85f)
        .height(200.dp)
) {
    MyCanvas(width)
    ProgressBar(onScroll)
}
~~~
真的很简单，又水了一篇文章，哈哈哈。