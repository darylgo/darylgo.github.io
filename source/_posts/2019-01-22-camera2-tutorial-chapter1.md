---
title: Android Camera2 教程 · 第一章 · 概览
date: 2019-01-22 10:05:19
categories:
- Android
tags:
---

从 Android 5.0 开始，Google 引入了一套全新的相机框架 Camera2（android.hardware.camera2）并且废弃了旧的相机框架 Camera1（android.hardware.Camera）。作为一个专门从事相机应用开发的开发者来说，这一刻我等了太久了，Camera1 那寥寥无几的 API 和极差的灵活性早已不能满足日益复杂的相机功能开发。Camera2 的出现给相机应用程序带来了巨大的变革，因为它的目的是为了给应用层提供更多的相机控制权限，从而构建出更高质量的相机应用程序。本文是 Camera2 教程的开篇作，本章将介绍以下几个内容：

* 一些 Camera2 的重要概念
* 一些只有 Camera2 才支持的高级特性
* 一些从 Camera1 迁移到 Camera2 的建议

> 本章涉及的代码很少，因为我们会在接下来的教程中深入介绍 Camera2 的 API。

# 1 Pipeline

Camera2 的 API 模型被设计成一个 Pipeline（管道），它按顺序处理每一帧的请求并返回请求结果给客户端。下面这张来自官方的图展示了 Pipeline 的工作流程，我们会通过一个简单的例子详细解释这张图。

{% asset_img pipeline.png %}

为了解释上面的示意图，假设我们想要同时拍摄两张不同尺寸的图片，并且在拍摄的过程中闪光灯必须亮起来。整个拍摄流程如下：

1. 创建一个用于从 Pipeline 获取图片的 CaptureRequest。
2. 修改 CaptureRequest 的闪光灯配置，让闪光灯在拍照过程中亮起来。
3. 创建两个不同尺寸的 Surface 用于接收图片数据，并且将它们添加到 CaptureRequest 中。
4. 发送配置好的 CaptureRequest 到 Pipeline 中等待它返回拍照结果。

一个新的 CaptureRequest 会被放入一个被称作 Pending Request Queue 的队列中等待被执行，当 In-Flight Capture Queue 队列空闲的时候就会从 Pending Request Queue 获取若干个待处理的 CaptureRequest，并且根据每一个 CaptureRequest 的配置进行 Capture 操作。最后我们从不同尺寸的 Surface 中获取图片数据并且还会得到一个包含了很多与本次拍照相关的信息的 CaptureResult，流程结束。

# 2 Supported Hardware Level

相机功能的强大与否和硬件息息相关，不同厂商对 Camera2 的支持程度也不同，所以 Camera2 定义了一个叫做 Supported Hardware Level 的重要概念，其作用是将不同设备上的 Camera2 根据功能的支持情况划分成多个不同级别以便开发者能够大概了解当前设备上 Camera2 的支持情况。截止到 Android P 为止，从低到高一共有 LEGACY、LIMITED、FULL 和 LEVEL_3 四个级别：

1. **LEGACY**：向后兼容的级别，处于该级别的设备意味着它只支持 Camera1 的功能，不具备任何 Camera2 高级特性。
2. **LIMITED**：除了支持 Camera1 的基础功能之外，还支持部分 Camera2 高级特性的级别。
3. **FULL**：支持所有 Camera2 的高级特性。
4. **LEVEL_3**：新增更多 Camera2 高级特性，例如 YUV 数据的后处理等。

# 3 Capture

相机的所有操作和参数配置最终都是服务于图像捕获，例如对焦是为了让某一个区域的图像更加清晰，调节曝光补偿是为了调节图像的亮度。因此，在 Camera2 里面所有的相机操作和参数配置都被抽象成 Capture（捕获），所以不要简单的把 Capture 直接理解成是拍照，因为 Capture 操作可能仅仅是为了让预览画面更清晰而进行对焦而已。如果你熟悉 Camera1，那你可能会问 `setFlashMode()` 在哪？`setFocusMode()` 在哪？`takePicture()` 在哪？告诉你，它们都是通过 Capture 来实现的。

Capture 从执行方式上又被细分为【单次模式】、【多次模式】和【重复模式】三种，我们来一一解释下：

* **单次模式（One-shot）**：指的是只执行一次的 Capture 操作，例如设置闪光灯模式、对焦模式和拍一张照片等。多个一次性模式的 Capture 会进入队列按顺序执行。

* **多次模式（Burst）**：指的是连续多次执行指定的 Capture 操作，该模式和多次执行单次模式的最大区别是连续多次 Capture 期间不允许插入其他任何 Capture 操作，例如连续拍摄 100 张照片，在拍摄这 100 张照片期间任何新的 Capture 请求都会排队等待，直到拍完 100 张照片。多组多次模式的 Capture 会进入队列按顺序执行。

* **重复模式（Repeating）**：指的是不断重复执行指定的 Capture 操作，当有其他模式的 Capture 提交时会暂停该模式，转而执行其他被模式的 Capture，当其他模式的 Capture 执行完毕后又会自动恢复继续执行该模式的 Capture，例如显示预览画面就是不断 Capture 获取每一帧画面。该模式的 Capture 是全局唯一的，也就是新提交的重复模式 Capture 会覆盖旧的重复模式 Capture。

# 4 CameraManager

CameraManager 是一个负责查询和建立相机连接的系统服务，它的功能不多，这里列出几个 CameraManager 的关键功能：

1. 将相机信息封装到 CameraCharacteristics 中，并提获取 CameraCharacteristics 实例的方式。
2. 根据指定的相机 ID 连接相机设备。
3. 提供将闪光灯设置成手电筒模式的快捷方式。

# 5 CameraCharacteristics

CameraCharacteristics 是一个只读的相机信息提供者，其内部携带大量的相机信息，包括代表相机朝向的 `LENS_FACING`；判断闪光灯是否可用的 `FLASH_INFO_AVAILABLE`；获取所有可用 AE 模式的 `CONTROL_AE_AVAILABLE_MODES` 等等。如果你对 Camera1 比较熟悉，那么 CameraCharacteristics 有点像 Camera1 的 `Camera.CameraInfo` 或者 `Camera.Parameters`。

# 6 CameraDevice

CameraDevice 代表当前连接的相机设备，它的职责有以下四个：

1. 根据指定的参数创建 CameraCaptureSession。
2. 根据指定的模板创建 CaptureRequest。
3. 关闭相机设备。
4. 监听相机设备的状态，例如断开连接、开启成功和开启失败等。

熟悉 Camera1 的人可能会说 CameraDevice 就是 Camera1 的 Camera 类，实则不是，Camera 类几乎负责了所有相机的操作，而 CameraDevice 的功能则十分的单一，就是只负责建立相机连接的事务，而更加细化的相机操作则交给了稍后会介绍的 CameraCaptureSession。

# 7 Surface

Surface 是一块用于填充图像数据的内存空间，例如你可以使用 SurfaceView 的 Surface 接收每一帧预览数据用于显示预览画面，也可以使用 ImageReader 的 Surface 接收 JPEG 或 YUV 数据。每一个 Surface 都可以有自己的尺寸和数据格式，你可以从 CameraCharacteristics 获取某一个数据格式支持的尺寸列表。

# 8 CameraCaptureSession

CameraCaptureSession 实际上就是配置了目标 Surface 的 Pipeline 实例，我们在使用相机功能之前必须先创建 CameraCaptureSession 实例。一个 CameraDevice 一次只能开启一个 CameraCaptureSession，绝大部分的相机操作都是通过向 CameraCaptureSession 提交一个 Capture 请求实现的，例如拍照、连拍、设置闪光灯模式、触摸对焦、显示预览画面等等。

# 9 CaptureRequest

CaptureRequest 是向 CameraCaptureSession 提交 Capture 请求时的信息载体，其内部包括了本次 Capture 的参数配置和接收图像数据的 Surface。CaptureRequest 可以配置的信息非常多，包括图像格式、图像分辨率、传感器控制、闪光灯控制、3A 控制等等，可以说绝大部分的相机参数都是通过 CaptureRequest 配置的。值得注意的是每一个 CaptureRequest 表示一帧画面的操作，这意味着你可以精确控制每一帧的 Capture 操作。

# 10 CaptureResult

CaptureResult 是每一次 Capture 操作的结果，里面包括了很多状态信息，包括闪光灯状态、对焦状态、时间戳等等。例如你可以在拍照完成的时候，通过 CaptureResult 获取本次拍照时的对焦状态和时间戳。需要注意的是，CaptureResult 并不包含任何图像数据，前面我们在介绍 Surface 的时候说了，图像数据都是从 Surface 获取的。

# 11 一些只有 Camera2 才支持的高级特性

如果要我给出强有力的理由解释为什么要使用 Camera2，那么通过 Camera2 提供的高级特性可以构建出更加高质量的相机应用程序应该是最佳理由了。

1. **在开启相机之前检查相机信息**
  出于某些原因，你可能需要先检查相机信息再决定是否开启相机，例如检查闪光灯是否可用。在 Caemra1 上，你无法在开机相机之前检查详细的相机信息，因为这些信息都是通过一个已经开启的相机实例提供的。在 Camera2 上，我们有了和相机实例完全剥离的 CameraCharacteristics 实例专门提供相机信息，所以我们可以在不开启相机的前提下检查几乎所有的相机信息。

2. **在不开启预览的情况下拍照**
  在 Camera1 上，开启预览是一个很重要的环节，因为只有在开启预览之后才能进行拍照，因此即使显示预览画面与实际业务需求相违背的时候，你也不得不开启预览。而 Camera2 则不强制要求你必须先开启预览才能拍照。

3. **一次拍摄多张不同格式和尺寸的图片**
  在 Camera1 上，一次只能拍摄一张图片，更不同谈多张不同格式和尺寸的图片了。而 Camera2 则支持一次拍摄多张图片，甚至是多张格式和尺寸都不同的图片。例如你可以同时拍摄一张 1440x1080 的 JPEG 图片和一张全尺寸的 RAW 图片。

4. **控制曝光时间**
  在暗环境下拍照的时候，如果能够适当延长曝光时间，就可以让图像画面的亮度得到提高。在 Camera2 上，你可以在规定的曝光时长范围内配置拍照的曝光时间，从而实现拍摄长曝光图片，你甚至可以延长每一帧预览画面的曝光时间让整个预览画面在暗环境下也能保证一定的亮度。而在 Camera1 上你只能 YY 一下。

5. **连拍**
  连拍 30 张图片这样的功能在 Camera2 出现之前恐怕只有系统相机才能做到了（通过 OpenGL 截取预览画面的做法除外），也可能是出于这个原因，市面上的第三方相机无一例外都不支持连拍。有了 Camera2，你完全可以让你的相机应用程序支持连拍功能，甚至是连续拍 30 张使用不同曝光时间的图片。

6. **灵活的 3A 控制**
  3A（AF、AE、AWB）的控制在 Camera2 上得到了最大化的放权，应用层可以根据业务需求灵活配置 3A 流程并且实时获取 3A 状态，而 Camera1 在 3A 的控制和监控方面提供的接口则要少了很多。例如你可以在拍照前进行 AE 操作，并且监听本这次拍照是否点亮闪光灯。

#  12 一些从 Camera1 迁移到 Camera2 的建议

如果你熟悉 Camera1，并且打算从 Camera1 迁移到 Camera2 的话，希望以下几个建议可以对你起到帮助：

1. Camera1 严格区分了预览和拍照两个流程，而 Camera2 则把这两个流程都抽象成了 Capture 行为，只不过一个是不断重复的 Capture，一个是一次性的 Capture 而已，所以建议你不要带着过多的 Camera1 思维使用 Camera2，避免因为思维上的束缚而无法充分利用 Camera2 灵活的 API。

2. 如同 Camera1 一样，Camera2 的一些 API 调用也会耗时，所以建议你使用独立的线程执行所有的相机操作，尽量避免直接在主线程调用 Camera2 的 API，HandlerThread 是一个不错的选择。

3. Camera2 所有的相机操作都可以注册相关的回调接口，然后在不同的回调方法里写业务逻辑，这可能会让你的代码因为不够线性而错综复杂，建议你可以尝试使用子线程的阻塞方式来尽可能地保证代码的线性执行（熟悉 Dart 的人一定很喜欢它的 async 和 await 操作）。例如在子线程阻塞等待 CaptureResult，然后继续执行后续的操作，而不是将代码拆分到到 `CaptureCallback.onCaptureCompleted()` 方法里。

4. 你可以认为 Camera1 是 Camera2 的一个子集，也就是说 Camera1 能做的事情 Camera2 一定能做，反过来则不一定行得通。

5. 如果你的应用程序需要同时兼容 Camera1 和 Camera2，个人建议分开维护，因为 Camera1 蹩脚的 API 设计很可能让 Camera2 灵活的 API 无法得到充分的发挥，另外将两个设计上完全不兼容的东西搅和在一起带来的痛苦可能远大于其带来便利性，多写一些冗余的代码也许还更开心。

6. 官方说 Camera2 的性能会更好，这句话听听就好，起码在较早期的一些机器上运行 Camera2 的性能并没有比 Camera1 好。

7. 当设备的 Supported Hardware Level 低于 FULL 的时候，建议还是使用 Camera1，因为 FULL 级别以下的 Camera2 能提供的功能几乎和 Camera1 一样，所以倒不如选择更加稳定的 Camera1。

# 13 结束语

本章到此结束，主要是介绍了 Camera2 的一些基础概念，让大家能够基本了解 Camera2 的工作流程和基础概念，并且知道使用 Camera2 能够做些什么。如果你对 Camera2 还是感到很陌生，不要紧，后续的教程会带领大家逐步深入了解 Camera2。