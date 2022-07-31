---
title: 在Android项目中使用GitHub Packages
date: 2022-08-01 19:54:37
tags:
---

在Android项目开发过程中，我们会使用各种第三方依赖包，其中大多数依赖包存放在maven仓库中。通过在Gradle中配置maven仓库，可以方便地管理和使用依赖。GitHub Packages也提供了maven仓库功能，对于一些常用的公共组件，可以选择发布到GitHub Packages，这样可以将源码和依赖包放在同一个Repository 下，管理起来更加便捷。

**GitHub果然优秀，集SourceCode/Library/Page于一身，免费的一条龙服务真香！**

下面记录GitHub Packages依赖包的发布和使用流程，本文涉及的Gradle脚本均为Kotlin编写，如果你使用Groovy语言会有一些语法上的差异。

##### 一、准备Repository

GitHub Packages有单独的页面，可以看到所有已发布的依赖包，但每个依赖包都有自己所归属的Repository。也就是说，发布依赖包的时候需要指定一个Repository作为容器，这个Repository可以是某个已存在的代码仓库，也可以新建一个名为Library的Repository专门用于存放依赖包。

第一步非常简单，简言之，只要账号下有一个Repository即可，此处假定我们的Repository名为**Library**。

##### 二、生成Access Token

想要上传或下载GitHub Packages中的依赖包，必须在配置maven仓库时设置`username`和`password`，其中`username`是GitHub账号的用户名，`password`是该账号生成的Access Token而**不是**登录密码。

生成Access Token的方式为GitHub - Settings - Developer Settings - Personal access tokens - Generate new token，注意在配置权限时务必勾选write/read packages。

这里有个注意事项，假设用户A在GitHub Packages上传了public权限的依赖包，用户B想要在项目中使用用户A的依赖包，此时用户B可以在不登录GitHub的情况下在网页浏览到用户A的依赖包，但在项目中配置maven仓库时，必须配置`username`和`password`才能正常下载依赖包，否则会出现`unauthorized`错误。此处配置用户B自己的账号和Access Token即可正常使用依赖包。

整体maven仓库的配置如下：
```kotlin
maven {
    url = uri("https://maven.pkg.github.com/USER_NAME/REPOSITORY_NAME")
    credentials {
        username = "USER_NAME"
        password = "ACCESS_TOKEN"
    }
}
```

其中，`url`里的host是固定的，而**不是**依赖包所属的Repository地址，`url`中的`USER_NAME`是依赖包所属账号的User Name；`credentials`中的`USER_NAME`则可以是任意账户的User Name(前提是当前依赖包所属仓库是public的)，`ACCESS_TOKEN`则与`credentials`中`USER_NAME`匹配的Access Token。

如果作为上传方上传到自己的仓库，那么此处`USER_NAME`是相同的；如果作为下载方从自己的仓库下载，那么此处`USER_NAME`也是相同的。一句话总结，`USER_NAME`是否相同取决于依赖包上传/下载方与依赖包所属方是否一致。

考虑到安全性问题，这里配置的`ACCESS_TOKEN`请务必从`local.properties`或者环境变量读取。

##### 三、配置Maven Publish插件
首先给想要发布的Module添加`maven-publish`插件：
```kotlin
plugins {
    ...
    id("maven-publish")
}
```

然后配置artifacts以及publishing节点：
```kotlin
val sourcesJar by tasks.creating(Jar::class) {
    archiveClassifier.set("sources")
    from(android.sourceSets.getByName("main").java.srcDirs)
}

artifacts {
    archives(sourcesJar)
}

afterEvaluate {
    publishing {
        publications {
            create<MavenPublication>("release") {
                from(components["release"])
                groupId = "cn.xxx.library"
                artifactId = "example"
                version = "1.0.0"
                artifact(sourcesJar)
            }
        }
        repositories {
            maven {
                url = uri("https://maven.pkg.github.com/USER_NAME/REPOSITORY_NAME")
                credentials {
                    username = "USER_NAME"
                    password = "ACCESS_TOKEN"
                }
            }
        }
    }
}
```

配置完成后，执行Gradle Sync，然后执行`./gradlew publishReleasePublicationToMavenRepository`即可完成依赖包的发布了。

##### 四、测试发布的依赖包

测试依赖包是否正常可用，可以在发布依赖包项目的其他Module或者其他独立项目中进行验证，首先在项目级gradle的仓库里添加上述的Maven仓库：
```kotlin
repositories {

    // other repositories like google(), mavenCentral()...

    maven {
        repositories {
            maven {
                url = uri("https://maven.pkg.github.com/USER_NAME/REPOSITORY_NAME")
                credentials { // 这里一定要配置，即使仓库是public的！
                    username = "USER_NAME"
                    password = "ACCESS_TOKEN"
                }
            }
        }
    }
}
```

然后在Module级gradle的依赖里添加依赖包：
```kotlin
dependencies {
    
    // other dependencies
    
    implementation "cn.xxx.library:example:1.0.0"
}
```

最后，执行Gradle Sync，此时在Module里就可以访问到这个Library了。
