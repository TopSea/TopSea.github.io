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