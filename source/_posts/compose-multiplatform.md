---
title: Jetpack Compose跨平台实践（一）（二）
date: 2024-05-08 17:16:58
tags:
---

#### （一）心路历程

##### 零、写在前面

[本文AI含量为0]

Jetpack Compose是一个在Android平台被大力推荐的声明式UI框架，正式版发布已经接近3年，记得刚在项目上接触Compose的时候还是beta版本，Compose的简单高效让我在开发工作中非常乐于探索不同的功能和特性。
对于Jetpack Compose的跨平台支持特性我一直都有所关注，但是没有实际应用过，现在Compose已经达到了足够高的稳定性和易用性，我也在跨平台的Compose应用上做出了第一次实践。

在切入正文之前，我要强调的事情是，我并不是一个跨平台开发者，对于iOS，Desktop以及Web平台，我只是有一些简单的仅限于基础知识层面的了解，没有任何深入的开发经验，所以对我而言，要想开发一个跨平台的应用，本身就是一个很大的挑战。

本篇文章仅分享一些主观感受和想法，Jetpack Compose跨平台开发技术请看下篇-开发手记。

##### 一、实践初衷
在Android平台的应用开发中，我已经进行了足够多的Compose编程实践，对于Compose的各种组件和开发模式有足够的了解，我非常看好Compose对于Android开发者编程体验的提升，同时我也在持续关注Jetpack Compose的Roadmap，Compose的功能迭代非常迅速。
既然Jetpack Compose在Android平台如此出色，并且Compose在设计之初就面向跨平台，我也想试试Compose在其他平台能有怎样的体验。

##### 二、实践目的
相信每个开发者在接触一门新技术的时候，第一个项目大多都是`Hello World`。
做到这一步显然不是我的目的，我想探究的是Jetpack Compose跨平台体验，那么最好还是能从实际需求出发，做一个有一些实际功能并且达到可用程度的App，最好能涉及到一些网络请求、图片加载等功能来提高开发的复杂度。
除了业务需求之外，我制定的一个规则是开发中使用纯粹的跨平台方案实现，尽量不引入特定平台的实现，我想知道纯粹的跨平台实现完成度有多少。（当然，还有个很大的原因是，如果涉及到特定平台的实现，我可能写不出来）

最终我想到的需求是做一个新闻客户端App，包含列表和详情页面，分为窄屏和宽屏两个场景：

窄屏场景下，采用两个独立页面，默认显示新闻列表页面，点击新闻卡片切换到文章详情页面：
![竖屏原型图](./compose-multiplatform/prototype-portrait.png)

宽屏场景下，只有一个页面，左侧显示文章列表，右侧显示新闻详情：
![屏原型图](./compose-multiplatform/prototype-landscape.png)


总结一下我想实现的需求：
1. 面向Android、iOS、Desktop和Web四个平台
2. 使用纯粹的跨平台实现，尽量不针对特定平台编写代码
3. 使用响应式布局适配不同尺寸的屏幕
4. 实现网络请求、图片加载等较为复杂的功能
5. 适配Dark Mode，提供流畅优雅的用户体验

##### 三、成果展示

先来一张全平台演示图：
![全平台演示图](./compose-multiplatform/showcase-all-in-one-screen.png)
【注：Web端Compose尚在实验阶段，中文无法正常显示，并且加载网络图片也遇到了问题】

然后是在不同平台设备上的响应式布局，这里我演示了PC端调整窗口大小以及在Mobile端切换横竖屏的效果，请看演示视频：
响应式布局演示视频：[On Desktop](https://drive.google.com/file/d/1gBDsZ_fBSHwqGogS0_QLOAJ7QbmaW3iu/view?usp=sharing) | [On Mobile](https://drive.google.com/file/d/1Cq3oykErf54ZoBb-HRC8BHE-xPp-zdGC/view?usp=sharing)

最后展示一下Dark Mode的支持情况，为了直观地看到切换Dark Mode时的体验，请看演示视频：
Dark Mode演示视频：[Dark Mode](https://drive.google.com/file/d/1MWuVvP5edpcTs9g7RMHKu6kjEnb8WyzB/view?usp=sharing)

项目源码：
Github：https://github.com/ommiao/ComposeMultiplatformDemo

##### 四、体验报告
经过一系列环境搭建、文档查阅和参考各种Compose跨平台的Example之后，终于实现了预期的功能，但比想象中要困难很多，甚至在中途有过放弃某些feature的想法。

**WELL：**
1. 开发环境搭建比较简单，Android Studio和Xcode都可以傻瓜式安装，并且官方提供了[KDoctor](https://github.com/Kotlin/kdoctor)工具，可以一键检查开发环境是否完善可用。
2. Compose开发直接在Android Studio中进行，对Android开发者十分友好。
3. Compose基础功能比较完善，比如State驱动UI、页面元素动画、使用kotlin协程、以及Dark模式检测等功能都能很好地支持。
4. 对平台差异有比较好的抽象，比如尺寸单位在不同的平台上可能对应px、em、pt等不同的逻辑概念，在Compose中统一为dp，此外一些图片、字体资源也有统一的定义方式，不需要针对不同平台进行差异化处理。
5. 支持Kotlin Multiplatform的第三方库越来越多，涵盖了存储、网络、测试、绘制等各个方面，详情可见[汇总列表](https://github.com/terrakok/kmp-awesome)。

**LESS WELL：**
1. Web端的支持程度比预想中差很多，目前Web平台被标注为实验性功能，甚至基本的文字显示、网络请求都还存在问题。
2. 有些平台差异必须针对特定平台处理，比如在Web上访问网络图片会遇到跨域问题，虽然这是个浏览器常见问题，但是会让跨平台编程体验变得割裂。
3. Kotlin跨平台项目有很多Example，但不同的Example有着不同的项目结构pattern，应该是基于不同Kotlin Multiplatform版本创建的，这不但增加了学习成本，也可能会增加项目后续的维护成本。
4. 第三方库的平台支持范围参差不齐，并不是全都支持全平台，有时候找到一个适合自己项目的支持库并不容易。

##### 五、总结展望
整体来讲，使用Compose跨平台的编程体验是比较舒适的，如果仅考虑UI部分，可以说基本是没有障碍的，但是距离能在正式项目中被使用还有很长的路要走。
作为Android开发者，不妨幻想一下：在Android端学好Compose，说不定哪天就可以在全平台使用了；只要Compose官方足够努力，Android开发者躺着就能变成全栈开发不是梦！

如果让我来推荐Compose跨平台技术是否值得了解和学习，我给Android开发者的推荐学习指数是：
★★★★★★☆☆☆☆

给其他技术栈开发者的推荐学习指数是：
★★★☆☆☆☆☆☆☆



#### （二）开发手记

> 上篇文章中我大致介绍了在Compose跨平台开发中的主观体验，这篇文章从技术角度详细讲解一下如何从零开始开发一个Compose跨平台项目。

为了更好地理解文章内容，这里我重复一下项目需求：

0. 做一个新闻阅读App，包含新闻列表和详情两个页面
1. 面向Android、iOS、Desktop和Web四个平台
2. 使用纯粹的跨平台实现，尽量不针对特定平台编写代码
3. 使用响应式布局适配不同尺寸的屏幕
4. 实现网络请求、图片加载等较为复杂的功能
5. 适配Dark Mode，提供流畅优雅的用户体验

预览图：
![全平台演示图](./compose-multiplatform/showcase-all-in-one-screen.png)

##### 一、环境搭建
Compose跨平台开发需要的环境其实就是Android和iOS开发环境的合集，针对Desktop和Web平台则不需要引入特定的开发工具，综合下来就是Android Studio和XCode两个开发平台，并且在开发过程中是不需要使用XCode的，只是为了能提供iOS平台必要的编译环境。

Android Studio可以直接在官网下载安装包，XCode可以在App Store直接安装。

此外，Kotlin Platform平台还[kdoctor](https://github.com/Kotlin/kdoctor)工具用来检查你的Mac电脑上是否具备了开发条件:
![kdoctor](./compose-multiplatform/kdoctor.jpg)

搭建开发环境比较简单，没有任何特殊的配置，一切准备就绪之后就可以进行开发了！

##### 二、创建项目
在Android Studio中创建一个项目，通常是直接在Android Studio中新建，选择对应的模板就可以得到一个可运行的Hello World项目。

但在Kotlin Platform中创建新项目是另外一种方式，通过官方提供的静态网站[Kotlin Multiplatform Wizard](https://kmp.jetbrains.com/#newProject)，勾选需要引入的平台，然后就可以下载到对应的项目模板。
![Wizard](./compose-multiplatform/kotlin-multiplatform-wizard.png)

这种方式可以脱离Android Studio的版本限制，便于模板的升级维护工作，也就是说我们每次下载的模板都基于最新版本，包括一些编译脚本的pattern、各种依赖库，当然还是集成到Android Studio更加方便一些。

##### 三、项目结构
在正式实现功能之前，我们先来了解一下这个项目的结构：
![Project Structure](./compose-multiplatform/project-structure.jpg)

每个部分代表的含义分别是：
- `Kotlin Multiplatform Project` - 整个项目的根目录，是一个标准的Gradle项目
- `composeApp` - 这是一个Gradle的kotlin跨平台Module，同时也兼职了Android application的角色；根据创建项目时选择的不同平台，这个目录下会包含多个不同平台的子目录，每个平台我们视为一个`target`
  - `commonMain`：可以在不同平台共享的部分
  - `androidMain`：Android target，本质上是标准的Android项目中app module，其中包含Android App的入口MainActivity
  - `iosMain`：iOS target，其中包含MainViewController，在`iosApp`中被引用
  - `desktopMain`：Desktop target，其中包含`Window{ ... }`
  - `wasmMain`：Web target, 其中包含`CanvasBasedWindow{ ... }`
- `iosApp`：iOS平台项目的根目录，也是iOS App的入口，编译iOS App的必需组成部分

在编译不同平台的应用时，这些模块会有不同的组合方式：
- Android：`commonMain` + `androidMain`
- iOS：`commonMain` + `iosMain` + `iosApp`
- Desktop：`commonMain` + `desktopMain`
- Web：`commonMain` + `wasmMain`

了解了项目结构，相信大家应该对每个部分的代码有了大概的认知，我们这次重点要做的是UI部分，理想情况下这些UI是可以被4个平台共享的，因此这些代码应该放在`commonMain`目录下。

##### 四、开发指南

完整的开发过程是比较繁琐的，这里仅挑选几个比较有代表性的内容详细讲解。

其他没有提及的部分，大家可以参考项目源码：
Github：https://github.com/ommiao/ComposeMultiplatformDemo

说明：想要成功运行需要在代码里配置新闻的API key，如果需要的话可以在[这里](https://www.mxnzp.com/doc/detail?id=12)免费申请，如果只是想简单体验一下，也可以联系我，使用我已经申请好的API key。

###### 1）创建跨平台模块
在上述生成的项目模板中，有且仅有一个Gradle模块 - `composeApp`，并且这是整个项目的application module，如果我们想创建其他的跨平台模块，应该怎么做呢？

在创建自定义跨平台模块之前，先来了解一下`composeApp`是怎么成为一个跨平台模块的。既然这是一个Gradle模块，那么我们重点关注一下这个模块的`build.gradle.kts`。

```kotlin
// build.gradle.kts of composeApp module

// 首先声明了3个gradle plugin: kotlin platform, android app和compose
plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidApplication)
    alias(libs.plugins.jetbrainsCompose)
}

// kotlin跨平台模块的核心配置
kotlin {
    
  // 当前module支持的平台，我们创建项目时勾选了4个平台，因此这里有4种target
  wasmJs {}
  androidTarget { ... }
  jvm("desktop")
  listOf(
    iosX64(),
    iosArm64(),
    iosSimulatorArm64()
  ).forEach { iosTarget ->
    ...
  }

  // 当前module的依赖，其中commonMain里定义了可以共享的依赖，比如compose foundation等，其他则是针对不同平台的特定实现
  sourceSets {
    commonMain.dependencies {
      // shared libs
      // 这里依赖的库必须是支持当前module所有target的，缺一不可
    }
    androidMain.dependencies {
      // android libs
    }
    iosMain.dependencies {
      // ios libs
    }
    desktopMain.dependencies {
      // desktop libs
    }
    wasmJsMain.dependencies {
      // js libs
    }
  }
}

// Android application相关配置，与Android native项目中一致
android {
  ...
}

// Desktop application相关配置，定义了mainClass入口，包名等信息
compose.desktop {
  application {
    ...
  }
}

// Web application相关配置
compose.experimental {
  web.application { ... }
}

// 这里没有iOS application相关的配置，因为iOS有单独的application入口iosApp

```

简单总结一下，application级别的kotlin跨平台模块的配置主要是声明了
- 编译必需的plugin
- 支持的目标平台（target）
- 共享依赖以及不同target的特定依赖
- 各个平台application的编译信息（iOS有单独的application目录）

假设我们要创建一个data模块，用来为composeApp模块提供新闻的数据源。 由于这个模块只是library而不是application，所以可以省去application的配置信息。（对于Android则是需要将application的配置更改为library）

这里直接给出data module的gradle配置与application module做个对比：
```kotlin
// build.gradle.kts of data module

// 首先声明了3个gradle plugin: kotlin platform, android library和kotlin serialization
// 相比于application module，android从application变成了library；compose是UI library这里不需要引入
// 此外，引入了用于json反序列化的kotlin serialization
plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidLibrary)
    alias(libs.plugins.kotlinSerialization)
}

// kotlin跨平台模块的核心配置，与application一致
kotlin {

    wasmJs {}
    androidTarget { ... }
    jvm("desktop")
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        ...
    }

    sourceSets {
        commonMain.dependencies { }
        androidMain.dependencies { }
        iosMain.dependencies { }
        desktopMain.dependencies { }
        wasmJsMain.dependencies { }
    }
}

// Android library，与Android native项目中一致
android {
    ...
}
```

完成了module的配置之后，就可以在`commonMain`下使用kotlin实现请求新闻api的http client了，具体实现细节可以参考源码，这里就不展开讲了。

整个data module的结构如图所示：
![data module structure](./compose-multiplatform/data-module-structure.png)


###### 2）实现响应式布局
Compose中的响应式布局非常容易实现，并且在编程时不需要考虑当前App运行在哪个平台，我们只需要设计在不同空间下如何展示UI即可。

在Compose中有个非常好用的API - `BoxWithConstraints { ... }`，在这个方法提供的scope里，可以方便地获取当前节点可以占有的最大尺寸，并且在设备的布局发生变化时以新的尺寸信息触发compose界面重绘，我们只需要根据不同空间大小摆按照不同方式摆放UI组件就能实现响应式布局。

比如我们在`BoxWithConstraints`中检测最大可用空间宽度maxWidth，并将阈值设置为450dp，一般的Android手机宽度在400dp以内，因此达到450dp以上的设备可以认为是平板或者手机的横屏模式。

在maxWidth大于450时，将`NewsListScreen`和`NewsDetailScreen`摆放在一个`Row`里，即左右摆放；在maxWidth小于等于450时，将`NewsListScreen`和`NewsDetailScreen`摆放在一个`Box`里，即上下叠加摆放。（Row和Box都是Compose里的基础布局组件）

这样就以很小的代价实现了多个平台的响应式布局，在调节窗口大小或者切换手机横竖屏时可以实时刷新布局。

响应式布局演示视频：[On Desktop](https://drive.google.com/file/d/1gBDsZ_fBSHwqGogS0_QLOAJ7QbmaW3iu/view?usp=sharing) | [On Mobile](https://drive.google.com/file/d/1Cq3oykErf54ZoBb-HRC8BHE-xPp-zdGC/view?usp=sharing)

示例代码：
```kotlin
BoxWithConstraints {
    if (maxWidth.value > 450) {
        Row {
            NewsListScreen(...)
            NewsDetailScreen(...)
        }
    } else {
        Box {
            NewsListScreen(...)
            if (shouldShowDetail) {
                NewsDetailScreen(...)
            }
        }
    }
}
```

Compose中还有很多强大的功能，比如动画、主题、懒加载组件等等，这些功能我都在这个跨平台项目中实际应用到了，体验基本与Android端Compose一致。

###### 3）引入第三方依赖

以data module为例，为了实现多平台网络请求，可以引入kotlin官方的[ktor](https://ktor.io/docs/client-create-multiplatform-application.html)来使用HttpClient。

通常在某个特定平台进行开发时，一个三方库只需要引入一次依赖，但在kotlin跨平台项目中，可能要为不同的平台同时引入同一个依赖的不同target。

Gradle引入ktor依赖示例：
```kotlin
// build.gradle.kts of data module

...

kotlin {
    ...

    sourceSets {
        commonMain.dependencies {
            implementation(libs.ktor.client.core) // 没有平台差异的client core
        }
        androidMain.dependencies {
            implementation(libs.ktor.client.okhttp) // 适用于Android的client实现
        }
        iosMain.dependencies {
            implementation(libs.ktor.client.darwin) // 适用于iOS的client实现
        }
        wasmJsMain.dependencies {
            implementation(libs.ktor.client.js) // 适用于js的client实现
        }
    }
}

...

```

引入第三方依赖并不复杂，单独进行讲解是为了大家能够了解这种一对多的依赖引入pattern。

值得一提的是，使用第三方库最棘手的问题是有时候很难找到适用于当前项目所有target的版本，找不到的合适的版本就要考虑这个第三方库能否被用于当前项目了。

##### 五、写在最后

这篇文章只是从很浅显的角度介绍了Compose在跨平台应用中的实践，想真正体会Compose的魅力，不妨亲自上手体验一下。

我相信Compose的功能会越来越强大，除了基于个人主观体验上的判断，官方的[更新日志](https://developer.android.com/jetpack/androidx/versions/stable-channel)也非常具有说服力：
![compose releases](./compose-multiplatform/compose-release-note.png)

自从2021年7月发布第一个正式版本之后，几乎每个月都会有Compose相关的更新，充分说明了Compose的重要性。

有任何Compose相关的问题，欢迎随时和我讨论！