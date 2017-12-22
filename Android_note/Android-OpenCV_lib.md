---
title: Android 使用OpenCV - 将OpenCV相关库做成模块形式
date: 2017-12-22 16:59:14
category: Android_note
---

* win7
* Android Studio 3.0.1

## 将OpenCV库做成一个模块
官网[OpenCV4Android SDK](https://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html)
可以找到下载地址 https://sourceforge.net/projects/opencvlibrary/files/opencv-android/  
这里使用的是2.4.9版本

下载得到 OpenCV-2.4.9-android-sdk，需要的代码和库在这里在里面的`sdk/native`目录
```
OpenCV-2.4.9-android-sdk/sdk/native
$ tree -L 2
.
|-- 3rdparty
|   `-- libs
|-- jni
|   `-- ......
`-- libs
    |-- armeabi
    |-- armeabi-v7a
    |-- mips
    `-- x86
```

新建一个Android项目`TestOpenCVProj`，`File->New->Import Module...`，选择`OpenCV-2.4.9-android-sdk\sdk\java`，
导入模块名为`openCVLibrary249`

将目录`OpenCV-2.4.9-android-sdk\sdk\native\libs`复制到模块`openCVLibrary249`下，与`src`同级
```
OpenCVTest/TestOpenCVProj/openCVLibrary249
$ tree -L 1
.
|-- build
|-- build.gradle
|-- libs
|-- lint.xml
|-- openCVLibrary249.iml
`-- src
```
修改模块openCVLibrary249的`build.gradle`文件
```
apply plugin: 'com.android.library'

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.3" // 根据实际情况修改

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 26
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        }
    }

    sourceSets {
        main {
            jni.srcDirs = []
            jniLibs.srcDirs = ['libs'] // 指定so库的位置
        }
    }
}
```
至此，OpenCV相关库已经成了一个Android Studio的模块

在App中调用，添加模块依赖后，调用`OpenCVLoader.initDebug()`进行初始化

## 参考资料

* [Android 使用OpenCV的三种方式(Android Studio) - CSDN](http://blog.csdn.net/sbsujjbcy/article/details/49520791)
* [Android Studio中配置及使用OpenCV示例（一） - CSDN](http://blog.csdn.net/gao_chun/article/details/49359535)
