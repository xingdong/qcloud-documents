
## 准备工作

在开始使用移动开发平台（MobileLine） Crash 服务前，确保您已经完成：[安装和配置 SDK](https://cloud.tencent.com/document/product/666/14305)

## 添加 SDK

如果希望将 Crash 库集成至自己的某个项目中，可以通过 gradle 远程依赖或者 jar 包两种方式集成。

### 通过 gradle 远程依赖集成

如果您使用 Android Studio 作为开发工具或者使用 gradle 编译系统，**我们推荐您使用此方式集成依赖。**

#### 1. 使用 jcenter 作为仓库来源。

在工程根目录下的 build.gradle 使用 jcenter 作为远程仓库：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        ...
    }
}

allprojects {
    repositories {
         jcenter()
    }
}
```

#### 2. 添加 Crash 库依赖。

在您的应用级 build.gradle（通常是 app/build.gradle）添加 Crash 的依赖：

```
dependencies {
    //增加这行
    compile 'com.tencent.tac:tac-crash:1.0.0'
}
```

#### 3. 如果需要上报 Native 异常，添加 Native Crash 库依赖。
 
如果您的工程有 Native 代码（C/C++）或者集成了其他第三方 SO 库，您可以集成 native 异常上报库。

在您的应用级 build.gradle（通常是 app/build.gradle）添加 Native Crash 的依赖：

```
android {
    defaultConfig {
        ndk {
            // 设置支持的SO库架构
            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
        }
    }
}

dependencies {
    //增加这行
    compile 'com.tencent.tac:tac-nativecrash:1.0.0'
}
```

然后，点击您 IDE 的 gradle 同步按钮，会自动将依赖包同步到本地。

### 手动集成

如果您无法采用远程依赖的方式，您可以通过以下方式手动集成。

#### 1. 下载服务资源压缩包。

1. 下载 [移动开发平台（MobileLine）核心框架资源包](http://tac-android-libs-1253960454.cosgz.myqcloud.com/1.0.0/tac-core-1.0.0.zip)，并解压。
2. 下载 [移动开发平台（MobileLine） Crash 资源包](http://tac-android-libs-1253960454.cosgz.myqcloud.com/1.0.0/tac-crash-1.0.0.zip)，并解压。

#### 2. 集成 jar 包。
- 将资源文件中的所有 jar 包拷贝到您工程的 `libs` 目录。

#### 3. 如果需要上报 Native 异常，集成 Native Crash 包。
 
如果您的工程有 Native 代码（C/C++）或者集成了其他第三方 SO 库，您可以集成 [native crash 上报库](http://tac-android-libs-1253960454.cosgz.myqcloud.com/1.0.0/tac-nativecrash-1.0.0.zip)。
 
- 如果您是采用 Eclipse 开发，将 `native crash 上报库`解压后的 `jni` 目录下的内容 拷贝到您工程您工程的 `libs` 目录。
- 如果您是采用 Android Studio 开发，将`native crash 上报库`解压后的 `jni` 目录下的内容 拷贝到 app 模块的 `main` 文件夹下的 `jniLibs` 目录下 。如果不存在该目录，请新建一个。

#### 4. 修改您工程的 AndroidManifest.xml 文件。

请按照下面的示例代码修改您工程下的 AndroidManifest.xml 文件：

```
<!-- 添加 Crash 需要的权限 -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_LOGS" />
```

## 避免混淆

如果您的应用开启了混淆，请在 Proguard 混淆文件中增加以下配置，避免 Crash 被混淆：

```
-dontwarn com.tencent.bugly.**
-keep public class com.tencent.bugly.**{*;}
```

## 添加符号表和 mapping 文件上传插件

如果您的工程使用了 so 文件或者对代码进行了混淆，您需要添加插件来上传符号表和 mapping 文件。

### 通过 gradle 远程依赖上传

#### 1. 在工程根目录下的 build.gradle 文件中添加依赖。
 
```
buildscript {
	 ......
    dependencies {
        ......
        classpath 'com.tencent.tac:tac-crash-plugin:1.0.0'
    }
}
```

#### 2. 在您应用 module 下的 build.gradle 文件中添加插件依赖。
 
请加在您 build.gralde 文件的头部。

```
apply plugin: 'com.android.application'
// 添加这一行
apply plugin: 'com.tencent.tac.crash'
```

依赖完成之后，我们会在您编译打包的过程中自动上传符号表，您不需要其他操作。

### 手动上传

1. 下载 [符号表工具](https://bugly.qq.com/v2/sdk?id=37d1ad19-a4b0-4eed-9146-55d87fc79f8d)。
2. 根据 UUID 定位 Debug SO 文件，具体可参考工具包中的使用文档。
3. 使用工具生成符号表文件（zip 文件），具体的使用方法可参考工具包中的使用文档。
4. 在移动开发平台（MobileLine）的控制台上传符号表文件。
 
如果您的项目只使用了混淆代码 (Proguard)，而没有 Native 工程，只需要直接上传 Proguard 生成的 Mapping 文件即可。


## 配置服务

Crash 服务使用默认参数即可，不需要额外配置。如果您已经配置好 TACApplication 单例，这个过程已经自动完成。

### 高级配置

如果您需要自定义服务的策略，您可以使用 TACCrashOptions 修改一些具体的策略：

```
TACApplicationOptions applicationOptions = TACApplication.options();

TACCrashOptions crashOptions = applicationOptions.sub("crash");
```

具体的 API 请参考编程手册文档。

>**注意：**
>请在 Crash 服务启动前完成它对应的参数配置，一旦服务启动，后续所有对它的参数修改都不会生效。
