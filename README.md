# MuPdfSo
Android 构建MuPdf 编译so库 并使用
[MuPdf官网](https://mupdf.com/index.html)
[MuPdfAndroid开发文档](https://mupdf.com/docs/android-sdk.html)
本次使用版本 14


这篇文章记录编译mupdf-android-viewer库，打包成so库，并在项目中使用.[请参考](https://www.jianshu.com/p/33d454933a98)

#编译so库
1. 首先还是获取mupdf-android-viewer库到本地，使库能够正常运行，并安装到手机上，看看效果。

```
git clone --recursive git://git.ghostscript.com/mupdf-android-viewer.git
```

2. 生成必要文件

在`mupdf-android-viewer/jni/libmupdf `目录下执行` make generate`命令。

```
➜  libmupdf git:(d63bd227) pwd
/Users/fengxing/openProject/mupdf-android-viewer/jni/libmupdf
➜  libmupdf git:(d63bd227) make generate
```

3. 添加`local.properties` 文件
在工程跟目录下创建`local.properties`文件配置sdk，ndk路径
```
ndk.dir=/Users/fengxing/Library/Android/sdk/ndk-bundle
sdk.dir=/Users/fengxing/Library/Android/sdk
```

4. cmake编译
在根目录执行make进行编译，如果编译成功，那so文件会生成在这里。
![image.png](https://upload-images.jianshu.io/upload_images/4118241-15f1a739e4b371ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目录
`mupdf-android-viewer/jni/build/intermediates/ndkBuild/debug/obj/local`

#使用so库
现在我们已经得到了so库，现在我们就使用它。

1. 创建新项目

创建一个新项目，项目的名字就叫MuPdfSo

2. 导入so库
把编译的so库导入新项目中
![image.png](https://upload-images.jianshu.io/upload_images/4118241-5c17234b667e05e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在gradle中添加代码

```
defaultConfig {
        applicationId "cn.picc.com.mupdfso"
        minSdkVersion 16
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

        ndk {
            // 设置支持的SO库架构 
            abiFilters 'armeabi'
        }
    }
```

3. 创建两个module
看mupdf-android-viewer的目录结构

![image.png](https://upload-images.jianshu.io/upload_images/4118241-cd8f731f6326d028.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看出来mupdf-android-viewer由三部分组成，

app：显示pdf
lib：UI显示层
jni：MuPdf核心库


调用顺序：
```app-->lib库-->jni库<-->  JNI <--> .h -->.c```

我们也根据这个模式创建两个module

一个名字叫PdfSo，定义native方法的。
一个名字叫PdfUi，定义显示UI的。

#####PdfSo

首先来搞PdfSo这个module，PdfSo主要是定义了一些native，和c文件进行沟通，这里我们复制mupdf-android-viewer定义好的类。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-8deddf7d4e322b3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把`com.artifex.mupdf.fitz`这个包，整个全部复制出来，记住 是全部哦。放到PdfSo库中，类似这样。
![image.png](https://upload-images.jianshu.io/upload_images/4118241-28199f20cfc6abd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为c代码库中定义的包就这个`com.artifex.mupdf.fitz`  所以native也必须是这个包。
我们来看下c代码和h代码。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-f4db3b9830862f11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/4118241-90bbaaccf92c35b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


看到没 在c代码中有个pkg，这个就是咱们定义的报名，可以修改成我们的报名，就不需要使用他们的包明了。

.c文件可以修改报名，是因为  看图

![image.png](https://upload-images.jianshu.io/upload_images/4118241-42559d4052c3156e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是因为根据包名去找字段，但是.h中的字段是写死的，只改c代码的包名可能还会有问题。

好吧，那暂时先使用他们的包名。


##### PdfUi库

这个就简单了：把lib库的代码直接占过去就行

![image.png](https://upload-images.jianshu.io/upload_images/4118241-64cd7191df5dae2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

别忘了资源文件。

编译一下，也就是 包明的问题，分分钟解决。

在app主模块测试一下，可以使用了。。。。

特此记录。

# 自定义界面

目前我没有这方便需求。
简单显示就可以的
使用so库 api大小增加8m左右
如果需要自定义界面或者需要什么功能
源码都有了  自己搞把。。。。。。。。

[其它PDF请参考](https://www.jianshu.com/p/33d454933a98)

[demo地址](https://github.com/fengxing1234/MuPdfSo)
