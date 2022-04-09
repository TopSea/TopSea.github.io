---
layout: post
title: GitHub发布Android依赖包
date: 2022-04-09
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: 20220409.png # Add image post (optional)
tags: [Android, AAB, Github] # add tag
---
最近整了个ComposeChart（以后会介绍）想把它分享到GitHub上以后做依赖包来用。在网上找了好多教程，试了下都不能用了。   
JitPack打印出来的log如下：
```shell
Init SDKMan
Found Android manifest
Android SDK version: . Build tools: 
Found gradle
Gradle build script
Found gradle version: 7.2.
Using gradle wrapper
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Downloading https://services.gradle.org/distributions/gradle-7.2-bin.zip
.10%.20%.30%.40%.50%.60%.70%.80%.90%.100%

------------------------------------------------------------
Gradle 7.2
------------------------------------------------------------

Build time:   2021-08-17 09:59:03 UTC
Revision:     a773786b58bb28710e3dc96c4d1a7063628952ad

Kotlin:       1.5.21
Groovy:       3.0.8
Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
JVM:          1.8.0_292 (Private Build 25.292-b10)
OS:           Linux 4.14.63-xxxx-std-ipv6-64 amd64

0m3.385s
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~16.04.1-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)
Getting tasks: ./gradlew tasks --all
WARNING:  > Android Gradle plugin requires Java 11 to run. You are currently using Java 1.8.
Please specify Java version in jitpack.yml
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
JVM:          11.0.2 (Oracle Corporation 11.0.2+9)
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Tasks: 

 ⚠️   WARNING:
 Gradle 'publishToMavenLocal' task not found. Please add the 'maven-publish' or 'maven' plugin.
 See the documentation and examples: https://docs.jitpack.io

Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Running: ./gradlew clean -Pgroup=com.github.TopSea -Pversion=0.1-beta -xtest -xlint publishToMavenLocal
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2

FAILURE: Build failed with an exception.

* What went wrong:
Task 'publishToMavenLocal' not found in root project 'ComposeChart'.

* Try:
Run gradlew tasks to get a list of available tasks. Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 742ms
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Build tool exit code: 0
Looking for artifacts...
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Looking for pom.xml in build directory and ~/.m2
2022-04-09T02:11:06.573695454Z
Exit code: 0

⚠️ ERROR: No build artifacts found
```

好像就是这个出了问题：`Gradle 'publishToMavenLocal' task not found. Please add the 'maven-publish' or 'maven' plugin.` 网上
的解法都不适用，还是老老实实跟着官方的[例子](https://docs.jitpack.io/android) 来吧

![这是图片](/assets/img/in_post/official-plain.png "Magic Gardens")

这还是说得很清楚的，只要项目里有一个 build file 就行。至于他的 build 配置，从我打印出来的日志来看确实是这样。
那要怎么写这个 build file 呢？可以参考 Android 的官方[例子](https://developer.android.google.cn/studio/build/maven-publish-plugin) 啦

直接把我的配置贴出来吧
```groovy
plugins {
    id 'com.android.library'
//    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'maven-publish'
}

group = 'top.topsea'
version = '0.1-beta'

android {
    compileSdk 32

    defaultConfig {
//        applicationId "top.topsea.composechart"
        minSdk 21
        targetSdk 32
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary true
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion compose_version
    }
    packagingOptions {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
}

dependencies {

    implementation 'androidx.core:core-ktx:1.7.0'
    implementation "androidx.compose.ui:ui:$compose_version"
    implementation "androidx.compose.material:material:$compose_version"
    implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.3.1'
    implementation 'androidx.activity:activity-compose:1.3.1'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
    debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"
}

afterEvaluate {
    publishing {
        publications {
            composeChart(MavenPublication) {
                from components.release

                groupId = 'top.topsea.compose-chart'
                artifactId = 'compose-chart'
                version = '0.1-beta'
            }
        }
    }
}
```
这个是 module 里面的 build.gradle 需要使用 com.android.library 插件，并且注释掉 com.android.application 和 applicationId 。    
因为 components 只会在 afterEvaluate 阶段创建，所以只能在这个方法里面设置。在其中的配置为

- publishing{}是由插件为项目创建的扩展（PublishingExtension类型），这个扩展提供了两个block：publications{}（MavenPublication类型）
  和repositories{}（MavenArtifactRepository 类型），分别用于配置要发布的内容和目标仓库（均可配置多个）。在这里我只需要发布到本地所以没有设置
  repositories{} ，而且需要在 settings.gradle 中添加 mavenLocal() 
- composeChart{}被称为发布组件Component，用于配置一项要发布的内容，composeChart 只是一个命名，没有特殊含义，你也可以将其更换为别的名称，
  但不能与其它发布组件名称重复。components.release表示这个组件是通过 release（buildTypes 中） 插件添加的组件，可以简单地理解为就是以 release 的配置作为发布配置。

另外

- repositories{}里面用maven{}指定要发布的目标仓库，url "$buildDir/repo" 指定了当前项目的build目录下的repo目录，即发布到本地
- composeChart块中还配置了要发布的artifact的group、id及version，如果不显式指定，则默认采用当前项目的配置

整完这些，到命令行跑一个 `gradle publishToMavenLocal` 或者 `gradle publishReleasePublicationToMavenLocal` （前提是已经在 Path
中配置了 Gradle ）。如果一切正常，在 `C:\Users\你电脑的用户名\.m2\repository` 中就能找到这个包了。

![这是图片](/assets/img/in_post/mavenLocal.png "Magic Gardens")

再提交到 GitHub 新增 Release ，就没啥问题了。