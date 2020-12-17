---
title: Android Camera1 教程 · 第一章 · 开启相机
date: 2018-11-08 23:12:06
categories:
- Android
tags:
---

> 本教程已经停更，《Camera2 教程》持续更新中。

从事相机的开发已经三年多了，这两年来积累了很多相机相关的开发经验，所以想做个总结，同时也希望对那些想了解这块知识的人有所帮助。

本文所使用的相机 API 是 android.hardware.Camera，至于 Camera2 的知识我打算另外写。我会开发一个具有完整相机功能的应用程序，并且将相机知识分成多个篇章进行介绍，而本章所要介绍的就是相机的开启流程。

阅读本章之后，你将学会以下几个知识点：

1. 如何注册相机相关的权限
2. 如何配置相机特性要求
3. 如何获取摄像头的个数
4. 如何开启摄像头
5. 如何关闭摄像头

你可以在 [https://github.com/darylgo/Camera1Sample](https://github.com/darylgo/Camera1Sample) 下载相关的源码。

# 1 创建相机项目

正如前所说的，我会开发一个具有完整相机功能的应用程序，所以第一步要做的就是创建一个相机项目，这里我用 AS 创建了一个叫 Camera1Sample 的项目，并且有一个 Activity 叫 MainActivity。

为了降低源码的阅读难度，我不打算引入任何的第三方库，不去关注性能问题，也不进行任何模式上的设计，大部分的代码我都会写在这个 MainActivity 里面，所有的功能的实现都尽可能简化，让阅读者可以只关注重点。

# 2 注册相关权限

在使用相机 API 之前，必须在 AndroidManifest.xml 注册相机权限 android.permission.CAMERA，声明我们开发的应用程序需要相机权限，另外如果你有保存照片的操作，那么读写 SD 卡的权限也是必须的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.darylgo.camera.sample">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

</manifest>
```

需要注意的是 6.0 以上的系统需要我们在程序运行的时候进行动态权限申请，所以我们需要在程序启动的时候去检查权限，有任何一个必要的权限被用户拒绝时，我们就弹窗提示用户程序因为权限被拒绝而无法正常工作：

```java
public class MainActivity extends AppCompatActivity {

    private static final int REQUEST_PERMISSIONS_CODE = 1;
    private static final String[] REQUIRED_PERMISSIONS = {Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onResume() {
        super.onResume();
        // 动态权限检查
        if (!isRequiredPermissionsGranted() && Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            requestPermissions(REQUIRED_PERMISSIONS, REQUEST_PERMISSIONS_CODE);
        }
    }

    /**
     * 判断我们需要的权限是否被授予，只要有一个没有授权，我们都会返回 false。
     *
     * @return true 权限都被授权
     */
    private boolean isRequiredPermissionsGranted() {
        for (String permission : REQUIRED_PERMISSIONS) {
            if (ContextCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_DENIED) {
                return false;
            }
        }
        return true;
    }
}
```

# 3 配置相机特性要求

你一定不希望用户在一台没有任何摄像头的手机上安装你的相机应用程序吧，因为那样做是没有意义的。所以接下来要做的就是在 AndroidManifest.xml 中配置一些程序运行时必要的相机特性，如果这些特性不支持，那么用户在安装 apk 的时候就会因为条件不符合而无法安装。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.darylgo.camera.sample">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <uses-feature
        android:name="android.hardware.camera"
        android:required="true" />

</manifest>
```

我们通过 <uses-feature> 标签声明了我们的应用程序必须在具有摄像头的手机上才能运行。另外你还可以配置更多的特性要求，例如必须支持自动对焦的摄像头才能运行你的应用程序，更多的特性可以在 [官方文档](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#hw-features) 上查询。

# 4 获取摄像头的个数

因为市面上的安卓手机千千万，并不是所有的手机都支持前后置摄像头，甚至有手机一个摄像头都没有，所以我们首先要确定当前的设备支持多少个摄像头。我们可以通过 Camera.getNumberOfCameras() 获取设备支持的摄像头个数，它是一个静态方法，返回一个 int 值代表摄像头个数。

```java
int numberOfCameras = Camera.getNumberOfCameras();// 获取摄像头个数
```

# 5 根据 ID 获取 CameraInfo

Camera.CameraInfo 正如其名，里面存储了几个相机相关的信息，信息并不多：

* **facing**

  摄像头的方向，可选值有 Camera.CameraInfo.CAMERA_FACING_BACK 和Camera.CameraInfo.CAMERA_FACING_FRONT。

* **orientation**

  摄像头的画面经过顺时针旋转多少度之后是正常画面，这个属性比较难以理解，我们在后面会专门解释。

* **canDisableShutterSound**

  是否支持静音拍照，也就是说在用户拍照的时候是否会“咔嚓”一声，例如在日本由于隐私政策，所有手机都是不允许静音拍照的，那么该字段就会返回 false。该字段一般配合 Camera.enableShutterSound(boolean) 使用，当它返回 false 的时候，即使你调用  Camera.enableShutterSound(false)，相机在拍照的时候也会发出声音。

好了，Camera.CameraInfo 其实就这么几个字段，最重要的就是 facing 和 orientation 了，我们可以通过 facing 判断摄像头是前置还是后置，通过 orientation 值取校正摄像头的画面。接下来，我们看下如何获取 Camera.CameraInfo 实例：

```java
@Nullable
private Camera.CameraInfo mFrontCameraInfo = null;
private int mFrontCameraId = -1;

@Nullable
private Camera.CameraInfo mBackCameraInfo = null;
private int mBackCameraId = -1;

/**
 * 初始化摄像头信息。
 */
private void initCameraInfo() {
    int numberOfCameras = Camera.getNumberOfCameras();// 获取摄像头个数
    for (int cameraId = 0; cameraId < numberOfCameras; cameraId++) {
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        Camera.getCameraInfo(cameraId, cameraInfo);
        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
            // 后置摄像头信息
            mBackCameraId = cameraId;
            mBackCameraInfo = cameraInfo;
        } else if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT){
            // 前置摄像头信息
            mFrontCameraId = cameraId;
            mFrontCameraInfo = cameraInfo;
        }
    }
}
```

从上面的代码可以看出 Camera.CameraInfo 的获取非常简单，只需要你创建一个 Camera.CameraInfo 实例，然后通过 Camera.getCameraInfo(int, Camera.CameraInfo) 方法就可以将相机信息填充到你创建的实例中。另外值得注意的一点是，相机的 ID 实际上就是从 0 到 numberOfCameras 递增，大部分手机的后置摄像头的 ID 是 0，前置摄像头的 ID 是 1，但是我们最好还是通过 facing 字段去判断比较靠谱。

> Tips：Camera.getNumberOfCameras() 和 Camera.getCameraInfo(int, Camera.CameraInfo) 都不需要相机权限。

# 6 开启相机

接下来我们要做的就是调用 Camera.open(int) 方法开启相机了，需要注意的是你必须确保在开启相机之前已经被授予了相机权限，否则会抛权限异常。一个比较稳妥的做法就是每次开启相机之前检查相机权限。下面是主要代码前段：

```java
/**
 * 开启指定摄像头
 */
private void openCamera() {
    if (mCamera != null) {
        throw new RuntimeException("相机已经被开启，无法同时开启多个相机实例！");
    }

    if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED) {
        if (hasFrontCamera()) {
            // 优先开启前置摄像头
            mCamera = Camera.open(mFrontCameraId);
        } else if (hasBackCamera()) {
            // 没有前置，就尝试开启后置摄像头
            mCamera = Camera.open(mBackCameraId);
        } else {
            throw new RuntimeException("没有任何相机可以开启！");
        }
    }
}
```

# 7 关闭相机

和其他硬件资源的使用一样，当我们不再需要使用相机时记得调用 Camera.release() 方法及时关闭相机回收资源。关闭相机的操作至关重要，因为如果你一直占用相机资源，其他基于相机开发的功能都会无法正常使用，严重情况下直接导致其他相机相关的 APP 无法正常使用。那么在什么时候关闭相机最合适呢？我个人的建议是在 onPause() 的时候就一定要关闭相机，因为在这个时候相机页面已经不是用户关注的焦点，大部分情况下已经可以关闭相机了。

```java
@Override
protected void onPause() {
    super.onPause();
    closeCamera();
}

/**
 * 关闭相机。
 */
private void closeCamera() {
    if (mCamera != null) {
        mCamera.release();
        mCamera = null;
    }
}
```

至此，我们关于开启相机的教程就结束了，下一章我们会介绍如何开启预览。