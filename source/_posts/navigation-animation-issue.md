---
title: 给Compose官方修个小Bug
date: 2022-08-12 23:02:14
tags:
---

Jetpack Compose在正式版本中提供了Navigation功能，但并不支持页面切换动画，生硬的页面过渡效果基本劝退了使用者。好在Google官方同时还在维护另外一个[Accompanist支持库](https://google.github.io/accompanist/)，这个库包括一些复杂Feature的Compose版本实现，在这些Feature迭代稳定之后，会直接集成到Compose核心Library中，比如Window Inset的控制，早期在Accompanist中就有实现，但在最新版本中已经标为废弃，原因就是已经集成到Compose核心库中。
今天的主角是Accompanist Library里的[Navigation Animation](https://google.github.io/accompanist/navigation-animation/)。顾名思义，也就是带有动画效果的Navigation。

**本篇文章基于Accompanist v0.26.1-alpha版本，后续版本更新后可能随时失效。**

#### 一、发现问题
在刚接触到Navigation Animation的时候，就发现设置的动画效果有些不对劲儿，但当时以为是动画属性设置问题，影响不大，没有深究。最近使用Navigation时又注意到这个奇怪的动画效果，并且在查阅文档之后确信我设置的参数完全正确，所以就想探究一下问题到底出在哪里。

Navigation Animation的基本用法如下：
```kotlin
AnimatedNavHost(
    navController = rememberAnimatedNavController(),
    startDestination = "A",
    enterTransition = {...},
    exitTransition = {...},
    popEnterTransition = {...},
    popExitTransition = {...}
) {
    composable(route = "A") {...}
    composable(route = "B") {...}
}
```

可以看到，AnimatedNavHost有4个动画相关的参数，这里我们设置了起始路由是A，所有可到达路由为A、B，假设场景是A跳转到B，然后执行popStack退回到A，那么动画的执行过程如下：
- enterTransition，A跳转到B时，B会执行
- exitTransition，A跳转到B时，A会执行
- popEnterTransition，B退回A时，A会执行
- popExitTransition，B退回A时，B会执行

了解清楚每个动画的含义后，我想实现的动画效果是这样的：

- A跳转到B时，B完全从右侧屏幕外移入
- A跳转到B时，A向左移出屏幕50%
- B退回A时，A从屏幕左侧外50%处移入
- B退回A时，B向右完全移出屏幕外

根据上面的参数说明，很容易就可以得出下面的实现代码：
```kotlin
AnimatedNavHost(
    navController = rememberAnimatedNavController(),
    startDestination = "A",
    enterTransition = {
        slideIntoContainer(
            towards = AnimatedContentScope.SlideDirection.Left,
            animationSpec = tween(1000)
        )
    },
    exitTransition = {
        slideOutOfContainer(
            towards = AnimatedContentScope.SlideDirection.Left,
            animationSpec = tween(1000),
            targetOffset = { it / 2 }
        )
    },
    popEnterTransition = {
        slideIntoContainer(
            towards = AnimatedContentScope.SlideDirection.Right,
            animationSpec = tween(1000),
            initialOffset = { it / 2 }
        )
    },
    popExitTransition = {
        slideOutOfContainer(
            towards = AnimatedContentScope.SlideDirection.Right,
            animationSpec = tween(1000)
        )
    }
) {
    composable(route = "A") { ScreenA() }
    composable(route = "B") { ScreenB() }
}
```

效果演示：

---
![动画遮盖问题演示](./navigation-animation-issue/navigation-animation-issue.gif)
---

仔细观察，可以看到A在向左退出时是在B的下层，但在B退回A的时候，A却盖在了B的上层，层级发生了错乱。

#### 二、原因分析
`AnimatedNavHost`是所有路由页面的Container，并且动画相关属性也是通过它来设置，那么问题一定出在AnimatedNavHost内部。

在Clone[官方Accompanist源码](https://github.com/google/accompanist)后，惊喜地发现AnimatedNavHost内部实现基于AnimatedContent，关于AnimatedContent可以在{% post_link compose-animation Compose Animation %}这篇文章进行了解。

AnimatedContent接收的`ContentTransform`除了支持动画自定义之外，还支持设置`targetContentZIndex`，也就是可以自定义每个页面的层级，默认每个页面的targetContentZIndex为0，并且实际效果是将要进入的页面在上层，将要退出的页面在下层，也就是上面演示的动画效果。

到这里问题原因就非常清楚了，AnimatedNavHost没有设置AnimatedContent里每个页面的zIndex。

#### 三、解决方案
**完整内容请见[Pull Request](https://github.com/google/accompanist/pull/1282)。**

##### 1. 增加contentZIndex参数
因为每个页面需要控制自己的zIndex，因此需要通过AnimatedContent里的`composable(...)`来设置contentZIndex，`composable(...)`接收到zIndex后，存储在contentZIndices map里，如下：
```kotlin
// 原始代码保存动画的逻辑
enterTransition?.let { enterTransitions[route] = enterTransition }
exitTransition?.let { exitTransitions[route] = exitTransition }
popEnterTransition?.let { popEnterTransitions[route] = popEnterTransition }
popExitTransition?.let { popExitTransitions[route] = popExitTransition }
// 下面是新增的保存zIndex的逻辑
contentZIndices[route] = contentZIndex
```

这里参考了动画的存储方案，每种动画都存在以route为key的map里：
```kotlin
// 原始代码存储动画的map
@ExperimentalAnimationApi
internal val enterTransitions =
    mutableMapOf<String?,
        (AnimatedContentScope<NavBackStackEntry>.() -> EnterTransition?)?>()
@ExperimentalAnimationApi
internal val exitTransitions =
    mutableMapOf<String?, (AnimatedContentScope<NavBackStackEntry>.() -> ExitTransition?)?>()
@ExperimentalAnimationApi
internal val popEnterTransitions =
    mutableMapOf<String?, (AnimatedContentScope<NavBackStackEntry>.() -> EnterTransition?)?>()
@ExperimentalAnimationApi
internal val popExitTransitions =
    mutableMapOf<String?, (AnimatedContentScope<NavBackStackEntry>.() -> ExitTransition?)?>()
// 下面是新添加的存储zIndex的map
@ExperimentalAnimationApi
internal val contentZIndices = mutableMapOf<String?, Float>()
```

##### 2. 传递contentZIndex
由于原始代码比较复杂，所以不太方便描述具体的改动，大致的思路就是按照route从map中取出对应的`zIndex`，然后将其设置给AnimatedContent，具体实现请参考[Pull Request](https://github.com/google/accompanist/pull/1282)。

##### 3. 效果测试
在原先的Navigation实现代码增加zIndex参数：
```kotlin
AnimatedNavHost(
    navController = rememberAnimatedNavController(),
    startDestination = "A",
    enterTransition = {...},
    exitTransition = {...},
    popEnterTransition = {...},
    popExitTransition = {...}
) {
    composable(route = "A", contenZIndex = 0f) { ScreenA() }
    composable(route = "B", contenZIndex = 1f) { ScreenB() }
}
```

效果演示：

---
![动画问题修复演示](./navigation-animation-issue/navigation-animation-fixed.gif)
---

#### 四、写在最后
已经在Accompanist提交了[PR](https://github.com/google/accompanist/pull/1282)，或许在某天会被采纳。如果你正在寻找快速的解决方案，一个简单的办法是复制Accompanist里Navigation的实现，共有4个文件，然后按照[PR](https://github.com/google/accompanist/pull/1282)进行一些改动，总耗时大概5分钟。（或者直接clone我修改过的[版本](https://github.com/ommiao/accompanist)）
