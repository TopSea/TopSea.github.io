---
layout: post
title: Android 计算器 [Kotlin实现]
date: 2021-1-10
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
video: player.html?aid=844260870&bvid=BV1h54y1Y7Ku&cid=299650993&page=1 # 哔哩哔哩视频
small-img: android-calculator.webp # 主页的博客封面
tags: [Js, Conference] # add tag
---    

<font size="5">之前看老师的算法视频，用Java写了普通表达式如何转后缀表达式并且计算的方法。
正好最近有空，用Kotlin在Android上写个计算器练练手吧。
</font>    

```kotlin
    //转List
    fun toList(formula: String): MutableList<String> {
        var a = -1
        val items: MutableList<String> = LinkedList()
        for (i in formula.indices) {
            val c = formula[i]
            if (!(c.toInt() in 48..57 || c.toInt() == 46)) {
                items.add(formula.substring(a + 1, i))
                a = i
                items.add("" + c)
            }
        }
        items.add(formula.substring(a + 1, formula.length))
        return items
    }
```    

```kotlin
    //转后缀
    private fun toSuffix(items: List<String>): List<String> {
        val s1: Stack<String> = Stack()
        val s2: MutableList<String> = LinkedList()
        for (item in items) {
            if (item == "(") {
                s1.push(item)
            } else if (item == ")") {
                while (!s1.peek().equals("(")) {
                    s2.add(s1.pop())
                }
                s1.pop()
            } else if (item == "/" || item == "*" || item == "+" || item == "-" || item == "%") {
                if (s1.isEmpty() || s1.peek().equals("(")) {
                    s1.push(item)
                } else if (comparePriority(s1.peek()) < comparePriority(item)) {
                    s1.push(item)
                } else {
                    while (comparePriority(s1.peek()) >= comparePriority(item)) {
                        s2.add(s1.pop())
                        if (s1.isEmpty()) break
                    }
                    s1.push(item)
                }
            } else {
                s2.add(item)
            }
        }
        while (!s1.isEmpty()) {
            s2.add(s1.pop())
        }
        return s2
    }

    //比较优先级
    private fun comparePriority(operation: String): Int {
        return when (operation) {
            "%" -> 1
            "/" -> 1
            "*" -> 1
            "-" -> 0
            "+" -> 0
            else -> 0
        }
    }


    fun Calculating(suffixString: String): Any {
        val suffixList: List<String> = toSuffix(toList(suffixString))
        val s: Stack<String> = Stack()
        var a: Double
        var b: Double
        for (item in suffixList) {
            if (!(item == "/" || item == "*" || item == "+" || item == "-" || item == "%")) {
                s.push(item)
            } else {
                when (item) {
                    "%" -> {
                        a = s.pop().toDouble()
                        b = s.pop().toDouble()
                        s.push((b % a).toString())
                    }
                    "/" -> {
                        a = s.pop().toDouble()
                        b = s.pop().toDouble()
                        s.push((b / a).toString())
                    }
                    "*" -> {
                        a = s.pop().toDouble()
                        b = s.pop().toDouble()
                        s.push((b * a).toString())
                    }
                    "-" -> {
                        a = s.pop().toDouble()
                        b = s.pop().toDouble()
                        s.push((b - a).toString())
                    }
                    "+" -> {
                        a = s.pop().toDouble()
                        b = s.pop().toDouble()
                        s.push((b + a).toString())
                    }
                    else -> {
                    }
                }
            }
        }
        return when {
            s.peek().isEmpty() -> {
                ""
            }
            s.peek().toDouble() > s.peek().toDouble().toInt() -> {
                s.pop()
            }
            else -> {
                s.pop().toDouble().toInt().toString()
            }
        }
    }

```