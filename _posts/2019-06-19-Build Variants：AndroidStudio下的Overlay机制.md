---
title: Build Variants：AndroidStudio下的Overlay机制
tags: Android Androi Studio
categories: Android
---

之前在做Android ROM开发时，有一些需求，比如美洲国家的Camera APP，桌面图标需要定制为美洲特色，相机按钮需要定制为美洲特色等等。
在Android源码中，我们经常使用overlay机制，进行资源的覆盖。

> Android Overlay是一种资源替换机制，它能在不重新打包apk的情况下，实现资源文件的替换（res目录非assert目录），Overlay又分为静态Overlay(Static Resource Overlay)与运行时Overlay(Runtime Resource Overlay)。

这篇文章对于overlay机制的介绍比较详细：https://www.jianshu.com/p/9304089c513d

目前我们APP，直接在Android Studio中开发，同时需要对不同渠道进行定制，比如针对美洲市场的图标进行定制。

这就对我们的overlay机制提出了新的要求。由于APP代码是在Android Studio中直接进行调试，如果使用Android 源码中的Overlay，会导致代码的编译验证仍然需要到源码中去进行，失去了AndroidStudio代码调试的优势。

为解决这个问题，我们专门去研究了Android Studio的Overlay机制。

针对这种定制化的需求，我首先想到的是新建一个分支，在独立分支上面进行代码的定制开发。
缺点：
1.这样相当于需要维护两个项目，而且随着定制渠道的增多，需要维护的项目会大幅度增加，不符合我们的要求。

2.定制分支bug与master有共性，bug解决后还需要cherry-pick等操作，太过麻烦。

3.测试需要测试多个apk，bug归档分类等都会浪费人力。

经过研究，发现了Android Studio的Build Variants可以满足我们的需求。

### 定制APP名称

**需求**：对美洲、印度、非洲市场定制打包Gallery应用，APP名称分别命名为Gallery_America、Gallery_India、Gallery_Africa。

#### 1.APP名称修改方法：
AndroidManifest.xml中，引用了string.xml的值，我们只要修改string.xml里面app_name的值即可。

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/app_name.png)

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/string.png)

#### 2. 在productFlavors里面新建渠道：America、India、Africa。

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/buildvariants.png)

#### 3.在src目录下，也就是main目录的同级，创建America、India、Africa三个文件夹，并分别创建string.xml文件，修改app_name。

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/resource.png)

#### 4. 点击Studio左下方的Build Variants，即可选择相应的定制版本进行调试

![在这里插入图片描述](https://raw.githubusercontent.com/FrankdeBoers/blog/master/static/img/build.png)

通过上面的步骤，就可以实现定制渠道发布版本了。当然实际开发中，不只是这么简单的修改应用名称，Overlay不止可以覆盖res资源文件，也可以覆盖java文件。（Amazing！）

