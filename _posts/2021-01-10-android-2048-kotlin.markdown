---
layout: post
title: Android 2048小游戏
date: 2021-1-10
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: js-1.png # Add image post (optional)
tags: [Android, 游戏, Java] # add tag
---
最近迷上了玩小游戏，贪吃蛇、俄罗斯方块、数字华容道还有2048。不过小程序里面各种广告，下的软件也遍地是广告而且安装包还贼大甚至还要我权限，鬼知道是个什么东西。那就自己整一个吧，先写个我觉得最简单的2048。


滑动逻辑
---
2048小游戏中最重要的部分就是小方块的移动和合并的逻辑。游戏的操作
界面是 `4 * 4` 小方块组成的，所以用二维数组来模拟是最合适不过了。
我们以向左滑动为例子来简单分析一下要如何处理它的逻辑，代码如下所示：   

~~~java
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
~~~    

代码不多，看起来并不复杂。向左滑动只涉及了水平方向的移动，所以
 `result[i]` 是不需要变化的。设置 `index = 0` 来处理每一行数字的移动，
设置 `sign` 来记录 每一行是否有合并操作（ `false` 为有合并操作）。
设置判断找到每一行的第一个非零的数字，把它移到最左边。第一个非零的
数字只需要移动不需要合并， 移动之后`continue;`直接开始寻找下一个非零数字。   

中间有零的话应该跳过，所以 `index` 只记录非零的数字。找到下一个数字时
与之前的数字比较一下：如果相等且没有过合并操作，之前的数字乘二（合并）、
 `index` 不变且 `sign = false` ； 如果不满足条件， `index + 1` 并把数字移到左边。

同理可得，当向右滑时：    
`index = a[0].length - 1;`  为最大值，列的循环从大到小遍历。其他的基本相似。   
~~~java
public static int[][] operationRight(int[][] a) {
        int[][] result = new int[a.length][a[0].length];

        for (int i = 0; i < a.length; i++) {
            index = a[0].length - 1;
            first = true;
            sign = true;
            for (int j = a[0].length - 1; j >= 0; j--) {
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
                        index--;
                        result[i][index] = a[i][j];
                        sign = true;
                    }
                }
            }
        }
        return result;
    }
~~~   

上下滑动时，把 `i` 和 `j` 换一下就好了。

判断游戏结束   
---

判断游戏结束也比较简单，只要游戏没办法进行下一步操作的时候就结束了。

也就是在玩家滑动之后判断没有 `0` 了就结束了。
~~~java
public static int CountZeros(int[][] array) {
   int countZeros = 0;
   for (int i = 0; i < array.length; i++) {
      for (int j = 0; j < array[0].length; j++) {
         if (array[i][j] == 0)
             countZeros++;
         }
      }
   if(countZeros != 0)
       return countZeros;
   return -1;
}
~~~    
不过这里会有一个小bug，如果已经没有 `0` 了，但是有两个 `4` 竖着连在一起，这是可以有下一步操作的。
但这时玩家却横着滑了一下，程序还是会判断游戏结束。   
所以当 `CountZeros` 的返回是 `-1` 时，我们需要自己做一次与玩家滑动方向垂直的方向的滑动。
然后再判断一次 `CountZeros` ，这时如果返回 `-1` 就真的结束了。