---
title: SourceSet：AndroidStudio的分渠道打包
tags: Android Android Studio
categories: Android
---

上一篇从技术上面，利用Gradle实现了Overlay机制。Gradle中还有一个sourceSet，可以实现更骚的操作。

需求：我们的应用，主要有三个模块，主界面模块、设置模块、商城模块，三个模块各有负责人进行开发。  新增欧洲和美洲市场，我们需要根据不同市场，进行定制开发，如欧洲市场不能有开屏广告等等。

我们最终想实现如下的效果：

![目标](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/%E6%A8%A1%E5%9D%97.png)

也就是UI资源单独作为一个目录出现，app目录下面不包含UI资源相关的文件。并且资源文件，根据不同的渠道可以实现动态编译。

这其实是上一节Overlay机制的一个进阶，利用的是productFlavors+sourceSet相配合实现的，优点：

1.是MVP模式的重要一环，View独立出来，包括res资源和自定义控件。

2.方便开发，资源相关的直接进UI目录查看

3.方便迭代，layout或者drawable增多，导致单个目录下文件过多，不利于查看。我们可以新增AppUI、SettingUI、ShopUI等目录，对主界面UI、设置界面UI、商城界面UI进行归类。

4.方便模块化，不同模块的开发，各自维护自己模块，如商城模块，只需要关注ShopUI下的资源。

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/%E6%A8%A1%E5%9D%972.png)

### 1.build.gradle配置
#### 1.在build.gradle中，新增变量app_ui_folder，在后面会用到

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/def.png)

#### 2.利用productFlavors变种两个渠道

```
    flavorDimensions "product"
    productFlavors {
        // @Step1 新增America渠道
        america {
            dimension "product"
        }

        //  @Step2 新增Europe渠道
        europe {
            dimension "product"
        }


    }
```

#### 3.使用sourceSet指定各个渠道下所指向的资源文件

```
sourceSets {
        // 基类所指向的资源
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = [
                    'src',
                    app_ui_folder + 'Common/src',
            ]
            res.srcDirs = [
                    'res',
                    app_ui_folder + 'Common/res',
            ]

            jniLibs.srcDirs = ['jniLibs']
            assets.srcDirs = ['assets']
            java.srcDirs = java.srcDirs
            resources.srcDirs = java.srcDirs
            aidl.srcDirs = java.srcDirs
            renderscript.srcDirs = java.srcDirs
        }

        // Europe渠道指向的资源
        europe {
            java.srcDirs = [
                    app_ui_folder + 'Europe/src',
            ]

            res.srcDirs = [
                    app_ui_folder + 'Europe/res',
                    app_ui_folder + 'Europe/res_overlay',
            ]

            resources.srcDirs = java.srcDirs
            aidl.srcDirs = java.srcDirs
            renderscript.srcDirs = java.srcDirs
        }

        // America渠道指向的资源
        america {
            java.srcDirs = [
                    app_ui_folder + 'America/src',
            ]

            res.srcDirs = [
                    app_ui_folder + 'America/res',
                    app_ui_folder + 'America/res_overlay',
            ]
            resources.srcDirs = java.srcDirs
            aidl.srcDirs = java.srcDirs
            renderscript.srcDirs = java.srcDirs
        }

        debug.setRoot('build-types/debug')
        release.setRoot('build-types/')
    }
```

### 2.新增目录

#### 2.1 Project视图下，在根目录TestGradle右键新增UI目录。

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/%E6%96%B0%E5%A2%9E1.png)

#### 2.2 在UI目录下新增如下目录

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/%E6%96%B0%E5%A2%9E2.png)

#### 2.3 将app目录下的res文件夹，copy至UI\AppUI\Commom目录下，作为基类res

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/%E6%96%B0%E5%A2%9E3.png)

