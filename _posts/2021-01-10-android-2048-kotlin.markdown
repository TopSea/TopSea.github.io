---
layout: post
title: Android 2048小游戏
date: 2021-1-10
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: js-1.png # Add image post (optional)
tags: [Android, 游戏, Java] # add tag
---     


<font size=5>2048小游戏中最重要的部分就是小方块的移动和合并的逻辑。游戏的操作
界面是`4 * 4`小方块组成的，所以用二维数组来模拟是最合适不过了。
我们以向左滑动为例子来简单分析一下要如何处理它的逻辑，代码如下所示：
</font>

```java
public static int[][] operationLeft(int[][] a) { 
    int[][] result = new int[a.length][a[0].length];
    for (int i = 0; i < a.length; i++) {
        index = 0;
        first = true;
        sign = true;
        for (int j = 0; j < a[0].length; j++) {
            if (a[i][j] != 0) {
                if (first) {
                    result[i][index] = a[i][j];
                    first = false;
                    continue;
                }
                if (a[i][j] == result[i][index] && sign) {
                    result[i][index] *= 2;
                    sign = false;
                } else {
                    index++;
                    result[i][index] = a[i][j];
                    sign = true;
                }
            }
        }
    }
    return result;
}
```   

<font size=5>代码不多，看起来并不复杂。向左滑动只涉及了水平方向的移动，所以
`result[i]`是不需要变化的。设置`index = 0`来处理每一行数字的移动，设置`sign`来记录
每一行是否有合并操作（`false`为有合并操作）。设置判断找到每一行的第一个非零的数字，把它移到最左边。第一个非零的
数字只需要移动不需要合并， 移动之后`continue;`直接开始寻找下一个非零数字。
</font>   

<font size=5>中间有零的话应该跳过，所以`index`只记录非零的数字。找到下一个数字时
与之前的数字比较一下：如果相等且没有过合并操作，之前的数字乘二（合并）、`index`不变且`sign = false`；
如果不满足条件，`index + 1`并把数字移到左边。
</font>  

<font size=5>同理可得：
</font>  