---
title: Android BuildVariant+Flavor 构建变体
tags: Android BuildVariant
categories: Android
---

### Android BuildVariant+Flavor 构建变体

#### 一、背景：
我们的项目中，一套代码需要发布多个版本，包括B站版本、Google Play版本、国内版本等，而每个版本的差异仅存在很小差异，差异如下：

| 版本        | 图标  |  登录注册页面  | 接入sdk |
| --------   | -----:  | -----:  |  :----:  | 
| 国内版本      | 正常图标   |   正常登录注册     | bugly |
| Google Play        |   google定制图标  |   定制登录+谷歌 推特 飞书三方登录   | adjust |
| B站版本       |   B站定制图标    |  正常登录注册  | bugly |

#### 二、旧的解决方法
采用分支管理的方式，
发版分支：分别拉取bizhan、china、google分支
开发分支：在主干master分支进行开发，需求开发、bug修复都合入master
最终发版分支rebase开发分支后，就可以在对应发版分支打包，发布版本
老的方式的优点是灵活管理，分支相互隔离，可以很好地定制差异，但缺点也很明显，那就是开发复杂，而且整个过程，需要人为rebase操作，存在发版分支忘记rebase开发分支，存在隐患。

基于以上问题，需要对打包方式进行改进优化。

#### 三、新的发版方式
在了解了gradle语法之后，BuildVariant是一个很好能实现我们的需求，通读了官方文档（https://developer.android.com/studio/build/build-variants?hl=zh-cn ） 之后，最主要的技术点有下面这些。

##### 3.1 BuildVariant功能概览
AndroidStudio在编译版本时，执行的其实是gradle中定义的task，点击AndroidStudio左侧的BuildVariant，可以选择构建的版本，比如选择bzDebug，实际上在run时，执行的是`./gradlew assembleBZDebug`指令。
![AnroidStudio-BuildVariant.png](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-1.png)


BuildVariant编译变体由两部分决定：
- BuildType 编译类型：debug、release
- Product Flavor 产品特性：china、google、bz

BuildVariant的种类= BuildType * Product Flavor，在我们的业务中，BuildVariant有6种。

##### 3.2 BuildType 编译类型
我们可以在app moudle模块下面的 `build.gradle` 文件中的 `android` 代码块内创建和配置 build 类型。
默认会有“debug”类型和“release”两种类型类型。一般在release会打开混淆，这样能够保护代码。如果需要新增编译类型，比如pre类型，代表预发之类的含义，可以在此处直接新增

``` 
android {
    buildTypes {
        release {
            debuggable false
            signingConfig signingConfigs.myConfig
            minifyEnabled true
            zipAlignEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }

        debug {
            debuggable true
            signingConfig signingConfigs.myConfig
        }
    }
}
```

附官方文档（https://google.github.io/android-gradle-dsl/3.4/com.android.build.gradle.internal.dsl.BuildType.html）
![BuildType属性](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-11.png)


##### 3.3 ProductFlavor 产品变种
`ProductFlavor`支持与 `defaultConfig` 相同的属性，这是因为，`defaultConfig` 实际上属于 [`ProductFlavor`](https://developer.android.com/reference/tools/gradle-api/current/com/android/build/api/dsl/ProductFlavor?hl=zh-cn) 类。所以DefaultConfig的属性都可以在ProductFlavor覆盖，如 `applicationId`。
```
android {
    defaultConfig {
        applicationId versions.applicationId
        minSdkVersion versions.minSdkVersion
        targetSdkVersion versions.targetSdkVersion
        multiDexEnabled true
        versionCode versions.versionCode
        versionName versions.versionName

        ndk {
            abiFilters "armeabi-v7a", "arm64-v8a"
        }

    }


    buildTypes {
        release {
            debuggable false
            signingConfig signingConfigs.myConfig
            minifyEnabled true
            zipAlignEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }

        debug {
            debuggable true
            signingConfig signingConfigs.myConfig
        }
    }

    productFlavors {
        / **
        * 国内
        * */
        china {
            buildConfigField "boolean", "ISDEBUG", "true"
        }

        / **
        * 海外
        * */
        google {
            buildConfigField "boolean", "ISDEBUG", "false"
        }

        // B站渠道
        bz {
            buildConfigField "boolean", "ISDEBUG", "false"
        }
    }
}

```

创建并配置ProductFlavor后，点击gradle sync。sync完成后，Gradle 会根据 build 类型和产品变种自动创建 build 变体，并按照 <product-flavor><Build-Type> 为其命名：
- chinaDebug
- chinaRelease
- googleDebug
- googleRelease
- bzDebug
- bzRelease

productFlavor还有个用法，是flavorDimension，可以用于减少手写代码的数量，这个暂时不做介绍，可以去官方文档查看（https://developer.android.com/studio/build/build-variants?hl=zh-cn#flavor-dimensions）

##### 3.3 SourceSet
默认配置，所有 build 变体之间共享的所有内容创建 `main/` [源代码集](https://developer.android.com/studio/build?hl=zh-cn#sourcesets)，包含src/main、src/res、src/assets、AndroidManifest.xml等内容。
我们可以新建文件夹，并在build.gradle中指定文件夹路径，用来指定变体的指定目录。比如在src下面，创建china、google、bz等目录，这样在构建时，会将main目录与对应目录下的文件进行合并编译。



![图片.png](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-3.png)

这里要重点注意几个事情：
- 1、java目录下面的代码差异，只能在china变体文件夹下面，比如LoginActivity.java，只能存在于china、google、bz等变体目录，在main目录下面不需要定义，否则在编译是会出现文件重复定义的错误。
- 2、assets、res目录是求合集，比如执行./gradlew assembleBzRelease，会将main/res和bz/res下的资源进行合并，都打到包里，如果存在重名文件，比如bz下的图标ic_launcher.png，与main下的图标，其实是同名的，那么编译时，会优先以变体目录下的文件为准。
- 3、基于2，建议认真整理assets目录，将差异缩短到最小，放到各个变体目录。

![图片.png](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-2.png)

现在回过头来，看我们的目录一，整理了各个版本的差异，这里再贴一下：
| 版本        | 图标  |  登录注册页面  | 接入sdk |
| --------   | -----:  | -----:  |  :----:  | 
| 国内版本      | 正常图标   |   正常登录注册     | bugly |
| Google Play        |   google定制图标  |   定制登录+谷歌 推特 飞书三方登录   | adjust |
| B站版本       |   B站定制图标    |  正常登录注册  | bugly |

我们会发现，对于B站版本，它只是在res方面与china版本不同，java代码一直，但是如果按照上面的方式，就需要在bz和china两个变体目录下面，都有java代码。这样不是不行，但是结构上很丑，而且如果java代码有变动，需要同时修改两个目录下面的文件，存在漏修改的风险。

看官方文档，提供了另一种方式，用来指定变体差异目录，比如我们想把变体目录整体放在BuildVariant下面：
![图片.png](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-4.png)

代码示例：
```
android {
 sourceSets {
        main {
            setRoot('src/main')
        }

        china {
            java.srcDirs = ['src/buildVariant/china/java']
        }

        bz {
            java.srcDirs = ['src/buildVariant/china/java']
            res.srcDirs = ['src/buildVariant/bzzzzz/res']
        }

        google {
            java.srcDirs = ['src/buildVariant/google/java']
            res.srcDirs = ['src/builddVariant/google/res']
        }
    }
}
```
通过上面的代码，就可以实现我们的需求，其中，china下面是java代码差异，google是java+图标差异，bz下面是图标差异。
需要注意的时，外面的src/bz，仍然会在编译阶段被打到包里，比如在此目录下放置如下内容，最终的apk包也会有这个文件。
![图片.png](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariant-5.png)

这里要说明一下java代码差异，以我们的登录注册页面来说，china和google版本页面完全不一样，而且还有三方登录的差异，所以一开始我们就是写了两个Activity页面，china版本启动LoginChinaActivity，google版本启动LoginGoogleActivity。   在新的模式下，两个Activity都重命名为LoginActivity，在main下面启动的都是LoginActivity，根据打包指令决定是启动google目录下面的LoginActivity，还是china目录下面的LoginActivity。


#### 3.4 声明依赖项implemention
在上面的各个版本差异中，最后一项是依赖项的差异，比如国内上报用的是bugly，而google版本必须用adjust。当然最简单的解决方法就是在所有版本中，都同时依赖bugly和adjust，但是这样是不美观的，而且可能会存在一定的合规风险。所以要做到依赖项隔离。官方提供的方法也非常简单：
```
    // 海外需要的依赖
    googleImplementation 'com.adjust.sdk:adjust-android:4.28.1'

    // 国内的依赖
    chinaImplementation bugly

    // B站渠道
    bzImplementation bugly
```


以上就是我们此次做的发版改造，总结一下主要涉及两个重要的点：
- 1. 通过sourceSet来指定代码
- 2.依赖项声明





