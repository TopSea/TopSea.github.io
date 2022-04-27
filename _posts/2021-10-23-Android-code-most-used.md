---
layout: post
title: Android 常用代码
date: 2021-10-23
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: android-code.png # Add image post (optional)
tags: [Android] # add tag
---
Android 常用代码
---
设置全面屏：   
~~~kotlin
~~~   

监听网络状态：
~~~kotlin
~~~

创建单例Room数据库：
~~~kotlin
~~~   

判断系统的声音设置：
```
AudioManager audioManager = (AudioManager) context.getSystemService(Context.AUDIO_SERVICE);
final int ringerMode = audioManager.getRingerMode();
switch (ringerMode) {
    case AudioManager.RINGER_MODE_NORMAL:
        System.out.println("gaohai:::normal");
        //没有振动只有响铃
        break;
    case AudioManager.RINGER_MODE_VIBRATE:
        System.out.println("gaohai:::vibrate");
        //静音且振动
        break;
    case AudioManager.RINGER_MODE_SILENT:
        System.out.println("gaohai:::silent");
        //静音且不振动
        break;
}
```

Compose 的 Canvas 画曲线
```kotlin
@Composable
fun CanvasCurve1(
    values: List<Float>
) {
    var stop by remember { mutableStateOf(0f) }

    val animate by animateFloatAsState(
        targetValue = stop,
        animationSpec = tween(durationMillis = 3000, easing = FastOutSlowInEasing)
    )
    Canvas(
        modifier = Modifier.fillMaxSize(),
    ) {
        val yEnd = size.height - ChartConfig.verPadding
        val smooth = 0.2f

        val srcPath = Path()
        val dstPath = Path()

        var prePreviousPointX = Float.NaN
        var prePreviousPointY = Float.NaN
        var previousPointX = Float.NaN
        var previousPointY = Float.NaN
        var currentPointX = Float.NaN
        var currentPointY = Float.NaN
        var nextPointX: Float
        var nextPointY: Float

        val listDot = List(values.size){ index ->
            Offset(
                index * ChartConfig.gridSize.value + ChartConfig.horPadding,
                yEnd - values[index] * ChartConfig.gridSize.value
            )
        }

        listDot.forEachIndexed { index, offset ->
            if (currentPointX.isNaN()) {
                currentPointX = offset.x
                currentPointY = offset.y
            }
            if (previousPointX.isNaN()) {
                //是否是第一个点
                if (index > 0) {
                    previousPointX = listDot[index - 1].x
                    previousPointY = listDot[index - 1].y
                } else {
                    //是的话就用当前点表示上一个点
                    previousPointX = currentPointX
                    previousPointY = currentPointY
                }
            }
            if (prePreviousPointX.isNaN()) {
                //是否是前两个点
                if (index > 1) {
                    prePreviousPointX = listDot[index - 2].x
                    prePreviousPointY = listDot[index - 2].y
                } else {
                    //是的话就用当前点表示上上个点
                    prePreviousPointX = previousPointX
                    prePreviousPointY = previousPointY
                }
            }

            // 判断是不是最后一个点了
            if (offset == listDot.last()) {
                //是的话就用当前点表示下一个点
                nextPointX = currentPointX
                nextPointY = currentPointY
            } else {
                nextPointX = listDot[index + 1].x
                nextPointY = listDot[index + 1].y
            }

            if (offset == listDot.first()) {
                // 将Path移动到开始点
                srcPath.moveTo(currentPointX, currentPointY)
                dstPath.moveTo(currentPointX, currentPointY)
            } else {// 求出控制点坐标
                val firstDiffX = currentPointX - prePreviousPointX
                var firstDiffY = currentPointY - prePreviousPointY
                val secondDiffX = nextPointX - previousPointX
                var secondDiffY = nextPointY - previousPointY

                if (nextPointY - currentPointY == 0f) {
                    secondDiffY = 0f
                }
                if (currentPointY - previousPointY == 0f) {
                    firstDiffY = 0f
                    secondDiffY *= 0.3f
                }

                val ctl1X: Float = previousPointX + smooth * firstDiffX
                val ctl1Y: Float = previousPointY + smooth * firstDiffY
                val ctl2X: Float = currentPointX - smooth * secondDiffX
                val ctl2Y: Float = currentPointY - smooth * secondDiffY

                srcPath.cubicTo(
                    ctl1X, ctl1Y, ctl2X, ctl2Y,
                    currentPointX, currentPointY
                )

            }

            // 更新值,
            prePreviousPointX = previousPointX
            prePreviousPointY = previousPointY
            previousPointX = currentPointX
            previousPointY = currentPointY
            currentPointX = nextPointX
            currentPointY = nextPointY
        }

        val mPathMeasure = PathMeasure()
        mPathMeasure.setPath(srcPath, false)
        stop = mPathMeasure.length

        if (mPathMeasure.getSegment(0f, animate, dstPath, true)) {
            //绘制线
            drawPath(
                path = dstPath,
                color = Color.Red,
                style = Stroke(width = 5f)
            )
        }
    }
}
```

Compose 中 `dp` 和 `px` 互相转换     
```kotlin
import androidx.compose.ui.platform.LocalDensity        //手动导包

val pxValue = with(LocalDensity.current) { 16.dp.toPx() }
// or
val pxValue = LocalDensity.current.run { 16.dp.toPx() }
```

Compose 中处理滑动手势：
```kotlin
var offsetX by remember { mutableStateOf(0F) }
var offsetY by remember { mutableStateOf(0F) }

    Modifier.pointerInput(Unit) {
        detectDragGestures(
            onDragEnd = {
                val x = abs(offsetX)
                val y = abs(offsetY)
                if (x > y) {
                    if (offsetX < 0) {
                        Log.d("", "GaoHai:::detectDragGestures left")
                    } else {
                        Log.d("", "GaoHai:::detectDragGestures right")
                    }
                } else {
                    if (offsetY < 0) {
                        Log.d("", "GaoHai:::detectDragGestures top")
                    } else {
                        Log.d("", "GaoHai:::detectDragGestures bottom")
                    }
                }
                offsetX = 0f
                offsetY = 0f
            }
        ) { _, dragAmount ->
            offsetX += dragAmount.x
            offsetY += dragAmount.y
        }
    }
```