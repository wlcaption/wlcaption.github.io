---
title: Unity开发必须掌握的Android知识
date: 2017-09-06 09:58:46
categories: Unity
type: "categories"
comments: false
tags:
- Unity

---
### 简介
随着项目开发到尾期的时候,必要考虑的一个因素就是Sdk, 那就是接入android部分,和ios部分, 作为一个Unity开发来说这些东西你可以不学,你觉得很难,一想到要学Java和Oc好多人都懵了,那几天我就在这里给大家解忧,我将详细写Android和Ios与Unity之间的通信,和互相调用,以最直接的方式来帮助大家,下面我是我写的教程:
工具方面我用的是Unity5.6.1f, android分两个环境分是Eclipse和Android Stuidio,Xcode使用的版本是8,其实版本不是最重要的思路和方法才是最重要的.随着Unity、cocos2dx等优秀跨平台游戏引擎的出现，开发者可以把自己从繁重的Android、iOS原生台开发中解放出来，把精力放在游戏的创作。原来做一款跨平台的游戏可能需要开发者懂得Java、Objective-C、C#甚至是C、C++，现在借助Unity我们开发者只需要懂得很少的原生应用开发知识就能够打造一款优秀的游戏。可能每个项目组只需要有1-2个人懂Android，iOS开发就够了。但是也正因为如此，很多同事有了充足的理由不去学习、接触Android和iOS的开发，等到真正需要做接入的时候才开始找人找资料，难免会踩坑。基于此，本文的目的就是通过介绍基础的Android开发知识以及部分的实际操作，让大家有一定的Android基础知识储备。又或者是当作一份Unity接入Android SDK/插件的基础教程，只要照着做，就基本上不会错了。
本文将会从大家熟悉的Unity为出发点来介绍如何将自己写的或者第三方的Android插件集成到自己的游戏中。
1. Unity是怎么打包APK文件的？
2. Android开发基础以及导入到Unity
3. Unity导出Android项目到Eclipse和AndroidStudio
4. Unity导出Xcode项目并且打包成.ipa
5. Android和Ios回调的处理


### Unity是怎么打包APK文件的？
大家看过一些第三方组件的接入文档都知道，在Unity里面有几个特殊的文件夹是跟打包APK有关的。首先我们就来了解一下，这些文件夹里面的内容是经历了哪些操作才被放到APK里面的呢？

在Unity的Assets目录下，Plugins/Android无疑是其中的重中之重，首先我们先来看一个常见的Plugins/Android目录是什么样子的。

-android 

--assets

--libs

--res

--AndroidManifest.xml

#### 接下来我们分别来看看Android打包工具都会做什么样的操作。
● libs文件夹里面有很多*.jar文件，以及被放在固定名字的文件夹里面的*.so文件。*.jar文件是Java编译器把.java代码编译后的文件，Android在打包的时候会把项目里面的所有jar文件进行一次合并、压缩、重新编译变成classes.dex文件被放在APK根目录下。当应用被执行的时候Android系统内的Java虚拟机（Dalvik或者Art），会去解读classes.dex里面的字节码并且执行。把众多jar包编译成classes.dex文件是打包Android应用不可或缺的一步。

看到这里有人可能会想不对啊，这一步只将jar包打成dex文件，那之前的java文件生成jar文件难道不是在这一步做吗？没错，这里用的jar包一般是由其他Android的IDE生成完成后再拷贝过来的。本文后面的部分会涉及到怎么使用Android的IDE并且生成必要的文件。

● libs文件夹的*.so文件则是可以动态的被Android系统加载的库文件，一般是由C/C++撰写而成然后编译成的二进制文件。要注意的是，由于实际执行这些二进制库的CPU的架构不一样，所以同样的C\C++代码一般会针对不同的CPU架构生成几分不同的文件。这就是为什么libs文件夹里面通常都有armeabi-v7a、armeabi、x86等几个固定的文件夹，而且里面的.so文件也都是有相同的命名方式。Java虚拟机在加载这些动态库的时候会根据当前CPU的架构来选择对应的so文件。有时候这些so文件是可以在不同的CPU架构上执行的，只是在不对应的架构上执行速度会慢一些，所以当追求速度的时候可以给针对每个架构输出对应的so文件，当追求包体大小的时候输出一个armeabi的so文件就可以了,这也是为了安全性着想，毕竟.so库很难被反编译，一些重要的处理函数都会被打成.so库。

● assets文件夹，这个里面的东西最简单了，在打包APK的时候，这些文件里面的内容会被原封不动的被拷贝到APK根目录下的assets文件夹，一般情况会放置一些不变得配置文件。这个文件夹有几个特性。

√ 里面的文件基本不会被Android的打包工具修改，应用里面要用的时候可以读出来。
√ 打出包以后，这个文件夹是只读的，不能修改。
√ 读取这个文件夹里面的内容的时候要通过特定的Android API来读取，参考getAssets()。
√ 基于上述两点，在Unity中，要读取这部分内容要通过WWW来进行加载。

除了Plugins/Android内的所有assets文件夹里面的文件会连同StreamingAssets目录下的文件一起被放到APK根目录下的assets文件夹。

● res文件夹里面一般放的是xml文件以及一些图片素材文件。xml文件一般来说有以下几种：
√ 布局文件，被放在res中以layout开头的文件夹中，文件里描述的一般都是原生界面的布局信息。由于Unity游戏的显示是直接通过GL指令来完成的，所以我们一般不会涉及到这些文件。
√ 字符串定义文件，一般被放到values文件夹下，这个里面可以定义一些字符串在里面，方便程序做国际
化还有本地化用。当然有时候被放到里面的还有其他xml会引用到的字符串，一般常见的是app的名称。
√ 动画文件，一般定义的是Android原生界面元素的动画，对于Unity游戏，我们一般也不会涉及他。
√ 图片资源，一般放在以drawable为开头的文件夹内。这些文件夹的后缀一般会根据手机的像素密度来来进行区分，这样我们可以往这些文件夹内放入对应像素密度的图片资源。
例如后缀为ldpi的drawable文件夹里面的图片的尺寸一般来说会是整个系列里面最小的，因为这个文件夹的内容会被放到像素密度最低的那些手机上运行。而一般1080p或者2k甚至4k的手机在读取图片的时候会从后缀为xxxxhdpi的文件夹里面去读，这样才可以保证应用内的图像清晰。图片资源在打包过程中会被放到APK的res文件夹内的对应目录。
√ Android还有其他一些常见的xml文件，这里就不一一列举了。

res文件夹下的xml文件在被打包的时候会被转换成一种读取效率更高的一种特殊格式（也是二进制的格式），命名的时候还是以xml为结尾被放到APK包里面的res文件夹下，其目录结构会跟打包之前的目录结构相对应。

除了转换xml之外，Android的打包工具还会把res文件夹下的资源文件跟代码静态引用到的资源文件的映射给建立起来，放到APK根目录的resources.arsc文件。这一步可以确保安卓应用启动的时候可以加载出正确的界面，是打包Android应用不可或缺的一步。

● AndroidManifest.xml，这份文件太重要了，这是一份给Android系统读取的指引，在Android系统安装、启动应用的时候，他会首先来读取这个文件的内容，分析出这个应用分别使用了那些基本的元素，以及应该从classes.dex文件内读取哪一段代码来使用又或者是应该往桌面上放哪个图标，这个应用能不能被拿来debug等等。在后面的部分会有详细解释。打包工具在处理Unity项目里面的AndroidManifest文件时会将所有AndroidManifest文件的内容合并到一起，也就是说主项目引用到的库项目里面如果也有AndroidManifest文
件，都会被合并到一起。这样就不需要手动复制粘贴。需要说明的是，这份文件在打包Android程序的时候是必不可少的，但是在Unity打包的时候，他会先检查Plugins目录下有没有这份文件，如果没有就会用一个自带的AndroidManifest来代替。此外，Unity还会自动检查项目中AndroidManifest里面的某些信息是不是默认值，如果是的话，会拿Unity项目中的值来进行替换。例如，游戏的App名称以及图标等。

● project.properties，这份文件一般只有在库项目里面能看得到，里面的内容极少，就只有一句话android.library=true。但是少了这份文件Android的打包工具就不会认为这个文件夹里面是个Android的库项目，从而在打包的时候整个文件夹会被忽略。这有时候不会影响到打包的流程，打包过程中也不会报错，但是打出的APK包缺少资源或者代码，一跑就崩溃。关于这份文件，其实在Unity的官方文档上并没有详细的描述（因为他实际上是Android项目的基础知识），导致很多刚刚接触Unity-Android开发的开发者在这里栽坑。曾
经有个很早就开始用Unity做Android游戏的老前辈告诉我要搞定Unity中的Android库依赖的做法是用Eclipse打开Plugins/Android文件夹，把里面的所有的项目依赖处理好就行了。殊不知这样将Unity项目跟Eclipse项目耦合在一起的做法是不太合理的，会造成Unity项目开启的时候缓慢。

● 其他文件夹例如aidl以及jni在Unity生成APK这一步一般不会涉及到，这里不展开。

看到了上述介绍的Unity打包APK的基础知识我们知道了往Plugins/Android目录下放什么样的文件会对APK包产生什么样的影响。但是实际上上述的内容只是着重的讲了Unity是怎么打包APK，所以接下来会简述一下打包这个步骤到底是怎么完成的。

Android提供了一个叫做aapt的工具，这个工具的全称是Android Asset Packaging Tool，这个工具完成了上述大部分的对资源文件处理的工作，而Unity则是通过对Android提供的工具链（Android Build Tools）的一系列调用从而完成打包APK的操作。这里感觉有点像我们写了个bat/bash脚本，这个脚本按照顺序调用Android提供的工具一样。在一些常见的Android IDE里面，这样的“bat/bash脚本”往往是一个完整的构建系统。最早的Android IDE是Eclipse，他的构建系统是Ant，是基于XML配置的构建系统。后来Android团队推出了Android专用的IDE——Android Studio（这个在文章后面会有详述），他的构建系统则是换成了gradle，从基于xml的配置一下子升级到了语言（DSL, Domain Specific Language）的层级，给使用Android Studio的人带来更多的弹性。

写到这里我想很多人都清楚了要怎么把Android的SDK/插件放到Unity里面并且打包到Unity里面。这时候应该有人会说，光会放这些文件不够啊，我还需要知道自己怎么写Android的代码并且输出相应的SDK/插件给Unity使用啊。

本文接下来的内容将会一步一步描述怎么写Android代码并且输出库文件给Unity。

####  Android开发基础以及导入到Unity
那就开始我们的第一个Android App,这里我不会讲太多的Android教程，毕竟我们不是Android开发，我们的重点还是Unity开发，那就开始我们的Android基础之旅吧。

##### 开始你的第一个Android程序
安装好Android studio，设置好我们的Jdk,下载好Android Sdk，Gradle最好自己从官网下载，自己配置，这里具体配置我就不说了，直接百度可以很快的配置好，只要在控制台输入：gradle -v,有提示信息就说明你配置正确了，如果提示不是命令行，那是环境变量的配置，配置好之后再测试。那我们开始创建一个新的Android项目。
在接下来弹出的界面里面输入应用名称，公司域名（这个其实不怎么重要）以包名（Package Name），其中我认为最重要的是包名，毕竟看一个应用的包名可以看得出一个开发者的逼格如何。。。
![1.png](http://upload-images.jianshu.io/upload_images/2463091-32d506b5698d8043.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来选择要开发什么类型的App，这里勾上Phone and Tablet就可以了。SDK的选择一般来说根据项目的需要，最低一般不低于API 9: Android 2.3(Gingerbread)，这也是Unity能接受的最低SDK。如果有些插件不能运行在这么低的Android SDK环境下的话可以酌情考虑提升到API 15: Android 4.3(IceCreamSandwich)，这个等级的API一般也是可以兼容绝大多数近3-4年的机器。

![2.png](http://upload-images.jianshu.io/upload_images/2463091-1fcf55014efdbb4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为我们要输出的内容是给Unity用的，这里可以先选择不带有Activity（就是承载游戏画面的基础部件），后续用到再说。
![3.png](http://upload-images.jianshu.io/upload_images/2463091-434bf4a9e8112a35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击OK以后Android Studio就会开始初始化当前的这个Android项目。初始化会需要一段时间，因为AndroidStudio有可能会去下载一些必要的框架或者更新Android工具的版本。初始化完成以后到左边按照图里面的步骤点开就可以看到整个项目目录树的情况。
![2.png](http://upload-images.jianshu.io/upload_images/2463091-dc0731d5f9e200b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过上图我们可以知道，一个Android Studio的项目（Project）可以由许多小的模块（Module）组成，这些模块可以是带有Activity的应用类模块，也可以是不带有Activity的库模块等等。这些小的模块之间可以有引用关系。我们可以把一些完成基础功能或者容易被复用的模块单独拆出来。

如果要新建一个模块我们可以在上图的列表中点右键选择New Module，在弹出的界面中我们可以选择要新建什么样的模块，或者从Eclipse导入旧的项目也可以。一般来说给Unity游戏开发插件最常用的就是库模块（AndroidLibrary）。同样的，在接下来弹出的窗口中填写好模块名称、包名以及最低运行的SDK。

● libs目录跟本文第一部分介绍的libs目录的功用是一样的，把依赖到的库放在这里面就可以了。
● src/main/res目录也是跟本文第一部分介绍的res目录的功能和结构是一样的，把对应资源放进去就可以了。
● 接下来是java代码所在的目录src/main/java，这个目录有点特殊，他的子路径跟java文件里面定义的包名（package name）要对应的上。
● AndroidManifest.xml也是跟第一部分介绍的AndroidManifest的功能是一样的。
● build文件夹是Android Studio动态生成的，打出的APK包（应用模块）或者AAR包（库模块）会被放到这里面的output文件夹。需要注意的是这个文件夹不应该被放提交到svn里面，要不然会造成项目成员之间的冲突，切记。
● src/test以及src/androidTest是做单元测试用的，本文不涉及。

至此，我们就可以开始动手写代码了，这里我们写一个可以弹出Android的Toast提示的Activity来替换掉Unity默认的Activity。

简述一下Unity跟Activity的关系：在Android系统中，打开一个应用，就是开启该应用指定的启动Activity。
Unity里面有个默认的Activity，他的作用就是在系统启动应用的时候加载Unity的Player，这个Player就是就相当于是Unity应用的“播放器”，他会执行我们在Unity项目中创作的内容，并且通过GL指令渲染到指定的SurfaceView中，而SurfaceView则是被置于Activity里面的一个特殊的View。

创建完成之后我们打开MainActivity这就是我们Android要运行的主Activity，里面的内容如下:
``` Java
    public class MainActivity extends Activity {

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
    }
```
需要注意的是这只是一个最基础的Android Activity，他还不会去加载我们的Unity出来，所以我们要让他继承自Unity的Activity而不是默认的。为此，我们要先将Unity相应的jar包引入到我们的模块当中。首先找到Unity的安装目录，然后找到以下子目录Editor\Data\PlaybackEngines\AndroidPlayer\Variations\il2cpp\Release\Classes\里
面的classes.jar，这个就是被打包成jar包的Unity默认的Activity。我们把这个jar包复制到当前模块的libs目录下（可以把这个jar包改成你想要的名字，便于管理）。（这个jar包的源码在Editor\Data\PlaybackEngines\AndroidPlayer\Source\com\unity3d\player这个目录下。感兴趣的同学可以翻阅一下源码，就可以理解Unity播放器的加载机制。）

接下来，我们可以在Android Studio左边的Project View中找到当前的模块以后点击右键，选择“Open ModuleSetting”或者直接按F4。在弹出的窗口中我们选到最右边的页签“Dependencies”，然后选择右边绿色的加号-JarDependency。


从项目的libs文件夹中找到刚刚导入的jar包，点击OK即可。接下来有一个比较关键的步骤就是，我们改变这个jar包的scope属性，因为默认的scope属性（Compile）是会将该jar包里面的内容跟本模块里面Java代码合并到一起。这在之后Unity打包这个模块的jar包的时候会报错，因为Unity里面内置了刚刚这个jar包。所以我们可以参考下图把这个jar包的scope设置成provided。

搞定了这步骤以后我们就可以回到刚刚新创建出来的Activity把他的父类改成UnityPlayerActivity，同时别忘记引用一下相应的package，改完之后的代码是这样的：
``` java
public class MainActivity extends UnityPlayerActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
    }
```

到这一步，如果我们的Activity如果能被运行的话，他应该能够借助他的父类UnityPlayerActivity里面的代码来运行Unity。接下来，我们添加几个方法，分别是：
``` java
 @Override
    protected void onRestart() {
        super.onRestart();
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
    }

    @Override
    protected void onStop() {
        super.onStop();

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
    }

    public void HolaSdkInit(String appid, String appkey, String privateKey,String oauthLoginServer)
    {
        String msg = "init";
        if(msg!=null){
            MessageHandle.resultCallBack(Util.User,UserWrapper.InitSuccess,"success");
        }else {
            MessageHandle.resultCallBack(Util.User,UserWrapper.InitFalied,"failed");
        }
    }

    public void Login(String p) {
        String username = "wlcaption";
        String password = "835114930";
        //加入网络请求
       ReqTask reqTask = new ReqTask(new ReqTask.Delegate() {
           @Override
           public void execute(String result) {//result 是服务器返回的值
               if(result!=null){
                   MessageHandle.resultCallBack(Util.User,UserWrapper.LoginSuccess,"success");
               }else {
                   MessageHandle.resultCallBack(Util.User,UserWrapper.LoginFailed,"failed");
               }
           }
       },"username="+username+"&password"+password,"http://hzmj.holagames.cn/myphp/login.php");//给服务器传两个字段
        reqTask.execute();


    }

    public void Pay(String payResult){
        String msg = "pay";
        if(msg!=null){
            MessageHandle.resultCallBack(Util.User,UserWrapper.PaySuccess,"success");
        }else {
            MessageHandle.resultCallBack(Util.User,UserWrapper.PayFailed,"failed");
        }
    }

    public void Logout()
    {
        String msg = "logout";
        if(msg!=null){
            MessageHandle.resultCallBack(Util.User,UserWrapper.LogoutSuccess,"success");
        }else {
            MessageHandle.resultCallBack(Util.User,UserWrapper.LogoutFailed,"failed");
        }
    }

    public void CheckBill(String userip)
    {
        String msg = "checkBill";
        if(msg!=null){
            MessageHandle.resultCallBack(Util.User,UserWrapper.InitSuccess,"success");
        }else {
            MessageHandle.resultCallBack(Util.User,UserWrapper.InitFalied,"failed");
        }
    }
```
这四个方法是Sdk必不可少的方法，我会在分别实现这四个方法，然后打成Jar包。导入到Unity中去,看得出来，里面最核心的一个方法其实就只是调用Android里面的Toast组件而已，没啥好解释的。相反，是外面的runOnUiThread是值得大家注意的，在Android编程中，所有涉及到对UI的操作必须要放在UI线程里面来做，否则会造成其他线程修改UI线程里面的数据然后崩溃。由于我们写的这四个个方法最后会被Unity那边调用，而来自Unity的调用可能不是UI线程，所以我们要给他做适当的保护。

在Android中有很多种调度方法可以把某段代码放到UI线程里面来跑。上面这段代码的runOnUiThread的写法是最简便的一种写法。如果遇到比较复杂的逻辑可以考虑使用Messenger或者Handle来调度线程，感兴趣的同学可以上网查一下。

#### Android导出Jar
在该模块的build.gradle添加如下代码:
```
task deleteOldJar(type: Delete) {
    delete 'build/outputs/test.jar'
}

task exportJar(type: Copy) {
    from('build/intermediates/bundles/release/')
    into('build/libs/')
    include('classes.jar')
    rename ('classes.jar', 'test.jar')
}

exportJar.dependsOn(deleteOldJar, build)
```
然后在控制台执行: gradlew exportJar
第一次执行会花费一点时间，第一次嘛难免会加载很多东西，第二次就很快乐，等我们看BUILD　SUCCESSFUL就说明你已经成功打出jar包，是不是很激动。

#### 导入到Unity并且编译
完成Activity的代码编写之后就可以输出这个模块到Unity项目中去。在Android Studio中选择Build - Make Project或者是在左边的项目视图中选中要导出的模块然后选择Build - Make Module。选择完了之后就可以看到下面有个Gradle的进度条，待进度条完成了以后我们就可以到该模块的build/outputs/aar目录下去找输出的文件。打开这个文件夹，可以看到有个*.aar的文件。这个就是该模块所编译出来的结果，如果你用解压缩软件去解压缩它，你会发现他几乎就是一个完整的Android工程。根据本文第一部分所说的内容，我们只要在Unity工程中的Plugins/Android目录下新建一个文件夹，然后把这个文件解压缩以后整个丢进去，再手写一个名字叫project.properties，内容是android.library=true的文件放到新建的文件夹里面就可以了。

胜利在望，我们接下来只要把Unity工程里面的AndroidManifest.xml文件的入口Activity从Unity默认的的改成我们刚刚写的这个就可以了。需要注意的是，如果是旧的Unity工程，可能已经有人写过相关的AndroidManifest文件放在了Plugins/Android目录下，但是如果是全新的Unity项目的话，就没有这份文件了。在打包的时候，如果Unity发现Plugins/Android目录下没有这份文件，他会复制一份默认的文件并且修改其中跟项目有关的内容。这里我们可以从Unity的安装目录的Editor\Data\PlaybackEngines\AndroidPlayer\Apk文件夹内找到AndroidManifest.xml这份文件，把它复制一份到Unity工程的Plugins/Android目录下。接下来就是修改里面的内容。
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="sdk.holagames.com.mywarsdkforandroid"
    android:versionCode="1"
    android:versionName="1.0" >

  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />


  <uses-sdk
      android:minSdkVersion="15"
      android:targetSdkVersion="25" />

  <application
       android:allowBackup="true">
    <activity
        android:name="sdk.holagames.com.mywarsdkforandroid.MainActivity"
        android:configChanges="orientation|keyboardHidden|screenSize"
        android:screenOrientation="landscape"
        >
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
      <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
      <meta-data android:name="unityplayer.ForwardNativeEventsToDalvik" android:value="false" />
    </activity>

  </application>

</manifest>
```
● package="com.example.wei.myapplication"这里的内容如果放着不动，打包的时候Unity会将其修改为Player Setting的Bundle Identifier。
● android:versionCode以及android:versionName这两部分的内容则在打包时会根据Player Setting里面的Version以及Bundle Version Code的内容来进行修改。
● android:icon以及android:label这两个对应的是应用的图标以及应用名称。如果不改的话，Unity也会自动根据Player Setting里面的内容来进行修改。
● android:debuggable="true"这个在打包的时候Unity也会自动根据Build Setting里面的Development Build选项自动进行修改。
● activity里面的android:name，这个name只的是该activity需要运行的哪个Java的Activity的类。如果不修改，加载的就是Unity默认Activity的类。这篇文章需要把默认的Activity改成刚刚我们的实现，所以，我们把刚刚写好的那个Activity的完整名称写上去（包括包名还有类名）。
● activity里面的android:label，这个是在桌面上图标下面写的那一行文字，也是应用的名称。不修改的话Unity会帮你维护。
● meta-data的这一行的name值是key，value值就是这个key对应的内容。meta-data可以根据需要自定义多个，但是key值不能重复，上面代码里面的unityplayer.UnityActivity应该是写给Unity看的，让Unity知道他自己是运行在这个Activity上。

这里我们基本上只要修改activity里面的android:name这一项。修改完成后，我们就可以通过Unity自带Build功能来出Android包了。出包之前请检查一下Player Setting里面的Bundle Identifier，不能留默认的包名在这里，会造成编译失败。编译过程中，可能会出现一些错误，下面罗列几个常见的错误，可以尝试解决：
1. 合并Manifest文件出错，一般来说是在合并所有的AndroidManifest文件的时候出的错，常见的有重复定义了activity、里面的最低sdk写错了。模块的最低sdk不可低于项目的最低sdk。
2. jar文件dex错误，当你的项目中不小心存在了一个以上的相同的jar文件，就会出这个错误，把重复的删掉，只留一个就好了。
3. 找不到Android SDK里面的工具，这个一般来讲是Unity自己的bug，Unity一般不能兼容最新的Android SDK的工具，所以要手动降级才行。

除了上述这些之外，在打包Android项目的过程中还会出现这些那些的错误，大家看到以后不要慌张，会报错总是好的，而且一般的错误你把错误信息贴在万能的Google上，都能找到解决方案。

##### Unity对Android代码的调用
文章到这里为止，说清楚了怎么把Android这边写成的插件打包到Unity的项目中去。但其实并没有涉及到Unity中怎么调用刚刚写好在Android的Activity中的代码。这一部对于一个Unity开发来说其实非常简单，只要以Unity提供的AndroidJavaClass还有AndroidJavaObject来做为中介就可以在Unity和Java中互传数据。这两个类的调用给人一种通过反射来调用Java代码的感觉。只要你能通过包名和类名拿到某个Java对象，就可以直接通过成员变量名称或者方法名称直接调用到Java那边的代码。举个例子，假如要在Unity中调用刚刚我们写的那个类的的几个方法类的话我们需要在Unity中准备以下代码。
先写个单例来定义这个方法：
``` C#
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Text;
using System.Runtime.InteropServices;

namespace Holagames
{
    public enum HolagamesSDKType
    {
        Analytics = 1,
        Share = 2,
        Social = 4,
        IAP = 8,
        Ads = 16,
        User = 32,
        Push = 64,
    };

    public enum UserWrapper
    {
        ExitByGame = 0,
        ExitBySdk = 1,

        InitSuccess = 100,
        InitFalied = 101,

        LoginSuccess = 200,
        LoginFailed = 201,
        LoginCancel = 202,
        Logining = 203,
        LoginSwitch = 204,

        LogoutSuccess = 300,
        LogoutFailed = 301,
        notLogin = 302,
        GameContinue = 351,
        GameExit = 352,

        PaySuccess = 400,
        PayFailed = 401,
        PayCancel = 402,
        PayCheck = 403,

    };

    public class HolagamesSDK
    {
#if UNITY_IPHONE

        /*[DllImport("__Internal")]
        private static extern void _registerActionResultCallback(int type, string gameObject,string func);

        [DllImport("__Internal")]
        private static extern void _LogOut();

        [DllImport("__Internal")]
        private static extern void _entryaGameCenter();
        
        [DllImport("__Internal")]
        private static extern void _Login();

        [DllImport("__Internal")]
        private static extern void _Init();
        
        [DllImport("__Internal")]
        private static extern string _getUserID();
                
        [DllImport("__Internal")]
        private static extern void _hideToolBar();

        [DllImport("__Internal")]
        private static extern void _showToolBar();


          [DllImport("__Internal")]
        private static extern void _pay(string msg);*/


#endif

        private static HolagamesSDK _instance;
        public static HolagamesSDK instance;


        public static HolagamesSDK getInstance()
        {
            if (null == _instance)
            {
                _instance = new HolagamesSDK();
            }
            return _instance;
        }

        public HolagamesSDK()
        {
#if UNITY_ANDROID
            jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
            jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
#endif
        }

        /// <summary>
        /// sdk初始化
        /// </summary>
        /// <param name="appid"></param>
        /// <param name="appkey"></param>
        public void init(string appid, string appkey, string privateKey, string oauthLoginServer)
        {
#if UNITY_ANDROID
            Debug.Log("UNITY_ANDROID===HolaSdkInit!!");
            jo.Call("HolaSdkInit", appid, appkey, privateKey, oauthLoginServer);
#elif UNITY_IPHONE
            //_Init();
#endif
        }

        // 登录
        public void Login(string LoginString)
        {
#if UNITY_ANDROID
            Debug.Log("UNITY_ANDROID===HolaSdk==Login!!");
            jo.Call("Login", LoginString);
#endif

#if UNITY_IPHONE
            // _Login();
#endif
        }

        // 注销
        public void Logout()
        {
#if UNITY_ANDROID
            jo.Call("Logout");
#endif
#if UNITY_IPHONE

            //_LogOut();
#endif
        }

        // 切换帐号
        public void SwitchLogin()
        {
#if UNITY_ANDROID
            jo.Call("SwitchLogin");
#endif
        }

        // 当前登陆状态
        public bool isLogin()
        {
#if UNITY_ANDROID
            return jo.Call<bool>("isLogin");
#else
            return false;
#endif
        }

        // 隐藏悬浮窗
        public void hideToolBar()
        {
#if UNITY_ANDROID
            Debug.Log("hideToolBar");
            jo.Call("hideToolBar");
#endif
        }

        // 显示悬浮窗
        public void showToolBar()
        {
#if UNITY_ANDROID
            Debug.Log("showToolBar");
            jo.Call("showToolBar");
#endif
        }

        // 进入用户中心
        public void entryGameCenter()
        {
#if UNITY_ANDROID
            Debug.Log("entryGameCenter");
            jo.Call("entryGameCenter");
#endif

#if UNITY_IPHONE
            //_entryaGameCenter();
#endif
        }

        public void ExitSDK()
        {
#if UNITY_ANDROID
            jo.Call("Logout");
#endif
        }


        // 支付
        public void Pay(string msg)
        {
#if UNITY_ANDROID
            Debug.Log("andriod pay=" + msg);
            jo.Call("Pay", msg);
#elif UNITY_IPHONE
            //_pay(msg);
#endif
        }


        // 支付
        public void CheckBill(string msg)
        {
#if UNITY_ANDROID
            Debug.Log("andriod CheckBill=" + msg);
            jo.Call("CheckBill", msg);
#elif UNITY_IPHONE
            //_pay(msg);
#endif
        }

        /// <summary>
        /// 获取用户ID
        /// </summary>
        /// <returns></returns>
        public string getUserID()
        {
#if UNITY_ANDROID
            return jo.Call<string>("getUserID");
#elif UNITY_IPHONE
            return "";//_getUserID();
#else
            return string.Empty;
#endif
        }

        /// <summary>
        /// 获取用户ID
        /// </summary>
        /// <returns></returns>
        public string getChannelId()
        {
#if UNITY_ANDROID
            return jo.Call<string>("getChannelId");
#else
            return string.Empty;
#endif
        }


        public void registerActionCallback(int type, MonoBehaviour gameObject, string functionName)
        {
#if UNITY_ANDROID
            if (mAndroidJavaClass == null)
            {
                mAndroidJavaClass = new AndroidJavaClass("com.holagames.sdk.MessageHandle");
            }
            string gameObjectName = gameObject.gameObject.name;
            mAndroidJavaClass.CallStatic("registerActionResultCallback", new object[] { type, gameObjectName, functionName });
#endif

#if UNITY_IPHONE
            //_registerActionResultCallback(type, gameObject.gameObject.name, functionName);
#endif
        }

        public void loginGameRole(string type, string msg)
        {
#if UNITY_ANDROID
            jo.Call("LoginGameRole", type, msg);
#endif

        }

        public void CreateRole(string type, string msg)
        {
#if UNITY_ANDROID
            jo.Call("CreateRole", type, msg);
#endif
        }

#if UNITY_ANDROID
        private AndroidJavaClass jc;
        private AndroidJavaObject jo;

        private AndroidJavaClass mAndroidJavaClass;
#endif
        /**
     	@brief the Dictionary type change to the string type 
    	 @param Dictionary
    	 @return  string
    	*/
        public static Dictionary<string, string> stringToDictionary(string message)
        {
            Dictionary<string, string> param = new Dictionary<string, string>();
            if (null != message)
            {
                    string[] tokens = message.Split('&');

                    for (var i = 0; i < tokens.Length; i++)
                    {
                        string[] _s = tokens[i].Split('=');
                        if (_s.Length == 2)
                        {
                            param.Add(_s[0], _s[1]);
                        }
                        else
                        {
                            string _v = "";
                            int _index = tokens[i].IndexOf("=");
                            _v = tokens[i].Substring(_index + 1);

                            param.Add(_s[0], _v);
                        }
                    }
            }

            return param;
        }


        public static string dictionaryToString(Dictionary<string, string> maps)
        {
            StringBuilder param = new StringBuilder();
            if (null != maps)
            {
                foreach (KeyValuePair<string, string> kv in maps)
                {
                    if (param.Length == 0)
                    {
                        param.AppendFormat("{0}={1}", kv.Key, kv.Value);
                    }
                    else
                    {
                        param.AppendFormat("&{0}={1}", kv.Key, kv.Value);
                    }
                }
            }
            //			byte[] tempStr = Encoding.UTF8.GetBytes (param.ToString ());
            //			string msgBody = Encoding.Default.GetString(tempStr);
            return param.ToString();
        }
    }

}
```
然后就是真正调用方法的地方，代码如下：
``` C#
using UnityEngine;
using System.Collections;
using Holagames;
using System.Collections.Generic;
using System;
using System.Text;

public class HuaWeiGameCenter : MonoBehaviour
{
    public bool IsInit = false;
    public bool IsCheckBill = false;
    public float Timer = 0;
    public string Message = "AAAA";
    public string Message1 = "AAAA";

    void Awake()
    {
        //UnityEngine.NotificationServices.deviceToken.ToString();
        HolagamesSDK.getInstance().registerActionCallback((int)HolagamesSDKType.User, this, "HolaSdkCallBack");
        HolagamesSDK.getInstance().registerActionCallback((int)HolagamesSDKType.IAP, this, "HolaSdkCallBack");
    }

   

    void AndroidReceive(string content)
    {
        Debug.Log(content);
        Message1 = content;
    }

    void AndroidReceiveChangeUser(string ChangeUser)
    {
        Debug.Log("ChangeUser");
        StartSDK();
    }

    void AndroidReceiveLoginFailed(string Value)
    {

    }

    void AndroidReceiveLogin(string Value)
    {
        string Account = Value.Split('_')[0];
        string Password = Value.Split('_')[1];
        Debug.Log(Account + " " + Password);

    }

    public void StartSDK()
    {
#if XIAOMI
        HolagamesSDK.getInstance().init("2882303761517475726", "5331747594726", "HfM9lFxOF5S0lMdWbkXgQQ==", "");  
        return;
#elif OPPO   
        HolagamesSDK.getInstance().init("", "ab03f4746b8C0A1d68624598177f64AD", "", "");  
        return;
#elif UC
        HolagamesSDK.getInstance().init("", "", "", "");  
        return;
#elif QIHOO360
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif VIVO
        HolagamesSDK.getInstance().init("9885e4eaa65e696ed43360ac994a05df", "42816dacc5cfbe9fd0e95cfe35f2eebc", "20160201102033928864", "");  
        return;
#elif QQ
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif BAIDU
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif IQIYI
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif MZW
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif AMIGO
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif MEIZU
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif COOLPAD
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif LESHI
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif DPWNJOY
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif ANZHI
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif WDJ
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif LENOVO
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif PYW
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif HOLA
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif TT
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif HAIMA
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif JOLO
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif DEBUG
        HolagamesSDK.getInstance().init("","","","");
        return;
#elif QIANHUAN
        HolagamesSDK.getInstance().init("","","","");
        return;
#else
        HolagamesSDK.getInstance().init("", "", "", "");
#endif
    }


    private void HolaSdkCallBack(string msg)
    {
        Debug.Log("HolaSdkCallBack==" + msg);
        Message = msg;
        Dictionary<string, string> dic = HolagamesSDK.stringToDictionary(msg);
        int code = Convert.ToInt32(dic["code"]);
        string result = dic["msg"];
        switch (code)
        {
            case (int)UserWrapper.InitFalied:
                Debug.LogError("SDK Init Failed");
                break;

            case (int)UserWrapper.InitSuccess:
                Debug.LogError("SDK InitSuccess");
                Message1 = "SDK InitSuccess";
                IsInit = true;
                break;
            case (int)UserWrapper.LoginCancel:
#if XIAOMI
                HolagamesSDK.getInstance().Login("");
#endif
                break;

            case (int)UserWrapper.LoginSwitch:
                break;

            case (int)UserWrapper.LoginFailed:
                break;

            case (int)UserWrapper.Logining:
                break;

            case (int)UserWrapper.LoginSuccess:
                Debug.LogError("SDK LoginSuccess");
                string _chane = "QQ";

                string Account = _chane + result;
                string Password = "t_" + result;
                Debug.Log(Account + " " + Password);

                Message = Account + " " + Password;

                break;
            case (int)UserWrapper.LogoutFailed:
                break;

            case (int)UserWrapper.LogoutSuccess:
                Message1 = "SDK LogoutSuccess";
                break;

            case (int)UserWrapper.notLogin:
                break;

            case (int)UserWrapper.PayFailed:
                IsCheckBill = false;
                Message = "UserWrapper.PayFailed" + msg;
                break;

            case (int)UserWrapper.PaySuccess:
                IsCheckBill = false;
                Message = "UserWrapper.PaySuccess";
                break;

            case (int)UserWrapper.PayCheck:
                IsCheckBill = true;
                Message = "UserWrapper.PayCheck";
                break;
        }
    }

    void Update()
    {
        if (IsCheckBill)
        {
            Message1 = Timer.ToString();
            Timer += Time.deltaTime;
            if (Timer > 1)
            {
                Timer -= 1;
                using (AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
                {
                    using (AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity"))
                    {
                        jo.Call("CheckBill", "10001-1-1-6");
                    }
                }
            }
        }
    }

    void OnGUI()
    {
        GUILayout.BeginHorizontal();
        if (GUILayout.Button("initSDK", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "Init";
            StartSDK();
        } 

        if (GUILayout.Button("Login", GUILayout.Height(80), GUILayout.Width(100)))
        {

            Message1 = "Login";
#if QQ
        HolagamesSDK.getInstance().Login("WX");
#else
            HolagamesSDK.getInstance().Login("");
#endif

        }

        if (GUILayout.Button("ToGame", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "ToGame";
            HolagamesSDK.getInstance().loginGameRole("create", "roleId=10278&roleName=阳光社&roleLevel=12&zoneId=androids26&zoneName=混服22服幽灵杀手&roleCTime=1481543091&roleLevelMTime=1481547775&vip=0");
        }

        if (GUILayout.Button("CreateRole", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "CreateRole";
#if IQIYI
            HolagamesSDK.getInstance().CreateRole("", "");
#elif VIVO
            HolagamesSDK.getInstance().CreateRole("", "");
#elif HOLA 
            HolagamesSDK.getInstance().CreateRole("", "");
#endif
        }

        if(GUILayout.Button("EnterGameCenter", GUILayout.Height(80),GUILayout.Width(100)))
        {
            Message1 = "EnterGameCenter";
#if HOLA
            HolagamesSDK.getInstance().entryGameCenter();
#endif
        }

        if (GUILayout.Button("ShowFloatView", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "ShowFloatView";
#if PYW
            HolagamesSDK.getInstance().showToolBar();
#elif VIVO
            HolagamesSDK.getInstance().showToolBar();
#elif HUAWEI
            HolagamesSDK.getInstance().showToolBar();
#elif MEIZU
            HolagamesSDK.getInstance().showToolBar();
#elif QIANHUAN
            HolagamesSDK.getInstance().showToolBar();
#endif
        }

        if (GUILayout.Button("Pay", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "Pay";
#if UC
            HolagamesSDK.getInstance().Pay("1_60_10001_1_1_1_1_1_1");  //UC
#elif QQ
            HolagamesSDK.getInstance().Pay("1_60_10001_1_1_1_1_1_1");  //QQ zoneid, diamond, guid, serverid, paytype		
#elif QIHOO360
            HolagamesSDK.getInstance().Pay("100_60钻石_1_xxx_10001_10001" + UnityEngine.Random.Range(1,10000) + "_10001-1-1-6");  //360  Product_Price Product_Name Product_Id Role_Name Role_Id OrderId guid-server_id-pay_type-cash
#elif VIVO
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");  //VIVO diamond, zoneid, guid, name, level, paytype, ProductName
#elif OPPO
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");  //VIVO diamond, zoneid, guid, name, level, paytype, ProductName
#elif BAIDU
             HolagamesSDK.getInstance().Pay("1_AndroidS10_10001_xxx_1_1_60钻石");//BAiDU diamond, zoneid, guid, name, level, paytype, ProductName
#elif HUAWEI
             HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//HUAWEI diamond, zoneid, guid, name, level, paytype, ProductName
#elif IQIYI
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//IQIYI diamond, zoneid, guid, name, level, paytype, ProductName
#elif MZW
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//mzw diamond, zoneid, guid, name, level, paytype, ProductName
#elif AMIGO
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//am diamond, zoneid, guid, name, level, paytype, ProductName
#elif MEIZU
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//meizu diamond, zoneid, guid, name, level, paytype, ProductName
#elif COOLPAD
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//coolpad diamond, zoneid, guid, name, level, paytype, ProductName
#elif LESHI
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//leshi diamond, zoneid, guid, name, level, paytype, ProductName
#elif DOWNJOY
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//downjoy diamond, zoneid, guid, name, level, paytype, ProductName
#elif ANZHI
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//anzhi diamond, zoneid, guid, name, level, paytype, ProductName
#elif WDJ
            HolagamesSDK.getInstance().Pay("1_60_10001_1_1_1_1_1_1");//WDJ diamond, zoneid, guid, name, level, paytype, ProductName
#elif LENOVO
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//lenovo diamond, zoneid, guid, name, level, paytype, ProductName
#elif PYW
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石_1");//pyw diamond, zoneid, guid, name, level, paytype, ProductName
#elif HOLA
            HolagamesSDK.getInstance().Pay("1_AndroidS1_10001_6_1_1_60钻石_1");//hola diamond, zoneid, guid, name, level, paytype, ProductName,servername
#elif TT
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石");//tt diamond, zoneid, guid, name, level, paytype, ProductName
#elif XIAOMI
            HolagamesSDK.getInstance().Pay("1_1_10001_xxx_1_1_60钻石_1");//xiaomi diamond, zoneid, guid, name, level, paytype, ProductName
#elif JOLO
            HolagamesSDK.getInstance().Pay("1_AndroidS1_10001_6_1_1_60钻石_1");//聚乐 diamond, zoneid, guid, name, level, paytype, ProductName
#elif DEBUG
            HolagamesSDK.getInstance().Pay("http://www.baidu.com");//debug url
#elif QIANHUAN
            HolagamesSDK.getInstance().Pay("1_AndroidS1_10001_6_1_1_60钻石_1");//千幻 diamond, zoneid, guid, name, level, paytype, ProductName
#else
            HolagamesSDK.getInstance().Pay("");
#endif
        }

        
        if (GUILayout.Button("CheckBill", GUILayout.Height(80), GUILayout.Width(100)))
        {
//#if DEBUG
            HolagamesSDK.getInstance().CheckBill("http://www.baidu.com");  //debug
//#endif
            Message1 = "CheckBill";
            //HolagamesSDK.getInstance().CheckBill("http://gdown.baidu.com/data/wisegame/ba226d3cf2cfc97b/baiduyinyue_4920.apk");
        }

        if (GUILayout.Button("ExitSDk", GUILayout.Height(80), GUILayout.Width(100)))
        {
#if QQ
            HolagamesSDK.getInstance().Logout();
#else
            HolagamesSDK.getInstance().ExitSDK();
#endif
        }

        if (GUILayout.Button("ChangeAccount", GUILayout.Height(80), GUILayout.Width(100)))
        {
            Message1 = "ChangeAccount";
            HolagamesSDK.getInstance().SwitchLogin();
        }
        GUILayout.EndHorizontal();
        GUILayout.BeginHorizontal();
        GUILayout.Label(Message, GUILayout.Height(200), GUILayout.Width(500));
        GUILayout.Label(Message1, GUILayout.Height(200), GUILayout.Width(500));
        GUILayout.EndHorizontal();
    }



    /// <summary>
    /// 实现php的URLENCODE函数
    /// </summary>
    /// <param name="str"></param>
    /// <returns></returns>
    public string UrlEncode(string str)
    {
        StringBuilder sb = new StringBuilder();

        for (var i = 0; i < str.Length; i++)
        {
            char c = str[i];
            if ((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z'))
            {
                sb.Append(c);
            }
            else
            {
                byte[] bystr = charToByte(str[i]);

                sb.Append(@"%");

                for (int ij = 0; ij < bystr.Length; ij++)
                {
                    if (bystr[ij] != 0)
                    {
                        sb.Append(Convert.ToString(bystr[ij], 16).ToUpper());
                    }
                }
            }
        }
        return (sb.ToString());
    }

    public byte[] charToByte(char c)
    {
        byte[] b = new byte[2];
        b[0] = (byte)((c & 0xFF00) >> 8);
        b[1] = (byte)(c & 0xFF);
        return b;
    }


    /// <summary>
    /// ios登录取帐号
    /// </summary>
    /// <param name="www"></param>
    /// <returns></returns>
    IEnumerator LoginPhp(WWW www)
    {

        yield return www;
        Debug.LogError(www.text);
        Debug.LogError(www.error);
        if (www.error == null && www.text != "0")
        {
            string _chane = "KY_";

#if KY
#elif TBT
            _chane = "TBT_";
            if (www.text == "-1")
            {
                HolagamesSDK.getInstance().Login("");
                return;
            }
#endif

            string Account  = _chane + www.text;
            string Password = "t_" + www.text;
            Debug.Log(Account + " " + Password);

        }

    }
}

```

简单介绍一下这段代码的几个关键点：
1. 通过UnityPlayer可以很方便的拿到当前Activity的Java对象实例。
2. 对Java对象实例的方法的调用实际上很简单，只要调用Call就可以了。
3. 注意用宏来区隔Native代码。UNITY_ANDROID 这个推荐的写法.
4. 推荐在new出AndroidJavaClass还有AndroidJavaObject的地方用using来进行保护，确保执行结束后Unity会自动回收相应的代码。

写了这么多我们来测试一下，将相关资源拷贝的Unity中，然后对Apk进行签名，这里我是自己已经准备好了，签名文件，不会弄签名文件可以去Android Stuidio中自己生成，设置完成之后我们就打出一个apk，如果打包出错的话，先看看控制的报错信息，然后对照控制的报错信息来修改。下面是我测试的数据，分别从初始化,登录,支付,注销这几个接口来测试的，如果你有其它接口，你可以自己添加进去，然后再来测试,我测试的结果如下:
![3.png](http://upload-images.jianshu.io/upload_images/2463091-9660dac5242dcc9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果有我下面截图的数据那就说明你已经成功了。
下篇文章我将对Ios进行接入，本来想写在一起的，但是感觉篇幅太大，很多同学一开始都是Android，Ios难免难度有点大，敬请期待我下篇文章，最近几天会更新的。
源码下载地址:[https://github.com/wlcaption/MyWarHolaSdk](https://github.com/wlcaption/MyWarHolaSdk)

其他的部分在这篇文章里面我们不展开。

    
    