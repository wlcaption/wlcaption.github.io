---
title: Unity使用SharedSDK有关微信开放平台回调的问题
date: 2017-09-05 11:00:11
categories: Sdk
type: "categories"
comments: false
tags:
- Sdk
---
最近在做一个项目Unity棋牌项目，项目里用到了SharedSDK来做登陆和一些分享相关，只要是使用微信的。
在使用SharedSDk中发现，其他平台都是好的，可以登陆分享，之后也会收到回调，唯独微信不可以，登陆没有授权页面显示，分享后收不到回调，完全不可用。
重SharedSDK官网下载了官方Demo也是不行，研究了一下午，可能初步找到了原因，但是还没有验证成功，因为微信开放平台的应用申请需要时间。现在写下来以备后用。

首先，也是最重要的一点，要使用微信第三方登陆，需要在微信开放平台注册，并完成企业认证。只有通过了企业认证才能在开放平台中申请移动应用。申请时IOS平台需要填写应用的Bundle ID。Android平台比较麻烦，需要填写包名即Unity里AndroidManifest.xml中的package名，并且需要应用签名，这个可以使用开放平台提供的签名生成工具安装在手机上，然后把自己的应用也安装好，通过包名来查询，得到一串字符字符串就是应用签名了。现在还没办法验证是否修改就是因为目前申请的应用还没有审核通过，只能等了。

另外，AndroidManifest.xml中的package名需要和Unity里面adroid设置的Package Name一致，并且AndroidManifest.xml中
```
<activity
android:name=”包名.wxapi.WXEntryActivity”
android:theme=”@android:style/Theme.Translucent.NoTitleBar”
android:configChanges=”keyboardHidden|orientation|screenSize”
android:exported=”true” />
```
.wxapi.WXEntryActivity一定要在包名下面，这个也是微信的特殊要求，WXEntryActivity这个文件一定要在包名目录下，SharedSDK自带的Plugins\Android\ShareSDK\libs\DemoCallback.jar包含了这个文件，但是肯定不在你自己的包名目录下，这就需要重新制作替换相应文件了。可以通过SharedSDK官方Unity Demo下载Android_Java_Demo，自己生成相应jar包。
首先，如果没有Android开发环境可以先下载adt-bundle，运行eclipse文件夹中的eclipse，打开刚才现在的Android_Java_Demo工程。
启动eclipse时，可能会因为Java SDK安装目录的问题无法启动，需要在eclipse.ini的openFile后面添加
-vm
C:/Program Files/Java/jdk1.8.0_112/bin/javaw.exe 指定自己的Java SDK安装目录。
修改WXEntryActivity.java所在目录到包名正确的地址包名.wxapi.WXEntryActivity，然后通过eclipse将WXEntryActivity导出为DemoCallback.jar，具体导出方法可以在网上去查。

微信分享时，如果设置的ShareContent.SetImageUrl地址无法访问的话，游戏中会出现文字提示“分享操作正在后台进行”之后回调会收到错误消息，所以设置时一定要设置正确地址。

这样应该就可以了，之后测试成功，我在来确认。
今天开放平台的移动应用申请通过了，事实证明确实是这个问题，授权登陆页面可以显示了，回调也能收到了，至此微信登陆算是正常了。
另外上面提到的xml中的.wxapi.WXEntryActivity前面也不需要加完整路径，这样也是没有问题的。

另外，在解决这个问题的时候，微信的官网文档发现，微信登陆不管是开放平台还是公众号，每个用户都会有一个对应该不同应用的不同openid，这个id是不唯一的，要想在不同应用中获取用户的唯一id来辨识用户如，应用和公众号联动，首选同一开放品台帐号下面的所有应用，用户的unionid字段都是相同的，如果需要公众号也有相同的unionid，需要把公众号和开放平台帐号绑定，这样通过unionid就可以唯一表示用户了。

在编译IOS版本时需要加入如下依赖库
```
libicucore.dylib
libz.dylib
libstdc++.dylib
JavaScriptCore.framework
CoreTelephony.framework
微信还需要
libsqlite3.dylib
```
Libraries Search Paths 里的”$(SRCROOT)/Libraries”的双引号去掉
并把SharedSDK的IOS下载中的Support和SharedSDK.framework加入到工程中。
在unity的Editor/SDKPorter/ShareSDKPostProcessBuild.cs中EditInfoPlist添加
//URL Scheme 添加
```
string PlistAdd = @”CFBundleURLTypes CFBundleURLName xxxx CFBundleURLSchemes wxxxxxxxxxxxxx“;
```
//白名单添加
```
string LSAdd = @”LSApplicationQueriesSchemes wechat weixin“;
```
如果使用微信支付，使用AFNetworking框架，还需要
MObileCoreServices.framework
如果解析json用到JSONKit
JSONKit.m需要在compile sources中添加-fno-objc-arc
ShareSDKUnity3DBridge.m中如果初始化__iosShareSDKRegisterAppAndSetPltformsConfig后调用__iosShareSDKGetUserInfo会crash在MTA中，可以注释掉__iosShareSDKRegisterAppAndSetPltformsConfig中的[ShareSDKConnector connectWeChat:[WXApi class]]自己再添加[WXApi registerApp:@”xxxxx” enableMTA:false];屏蔽掉MTA具体什么愿意引起MTA崩溃目前还不是太清楚。
另外在UnityAppController.mm中添加
```
#import “WXApi.h”
#import “WeiPayService.h”
```
函数openURL
return YES;
改为return [WXApi handleOpenURL:url delegate:[WeiPayService instance]];
WeiPayService类要实现WXApiDelegate并包含两个定义
```
-(void) onReq:(BaseReq*)req {
}
– (void)onResp:(BaseResp *)resp {
if ([resp isKindOfClass:[PayResp class]]) {
switch (resp.errCode) {
case WXSuccess:
break;
default:
break;
}
}
}
添加函数
– (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
return [WXApi handleOpenURL:url delegate:[WeiPayService instance]];
}
```
ios9以上版本废弃了handleOpenURL使用下面的版本
```
– (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options ｛
return [WXApi handleOpenURL:url delegate:[WeiPayService instance]];
｝
```
包括打开第三方app的回调
– (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
在ios9以后也废弃了，都是用上面的函数。

如果需要使用定位信息，需要在info.plist中加入
NSLocationWhenInUseDescription
NSLocationAlwaysUsageDescription

如果想通过分享连接打开app，需要如下操作
Android
在AndroidManifest.xml文件添加如下代码，
```
activity中加入android:launchMode=”singleTask”
<activity android:configChanges=”” android:label=”@string/app_name” android:launchMode=”singleTask” android:name=”xxxx.MainActivity” android:screenOrientation=”sensorLandscape”>
<intent-filter>
<action android:name=”android.intent.action.VIEW” />
<category android:name=”android.intent.category.DEFAULT” />
<category android:name=”android.intent.category.BROWSABLE” />
<data android:scheme=”xxxx” android:host=”xxxxxx” android:pathPrefix=”/openwith”/>
</intent-filter>

Java代码，UnityPlayerActivity重载onCreate和onNewIntent，openurl为具体处理函数，用来处理url传入的参数。
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);

openurl();
}
protected void onNewIntent(Intent intent) {
super.onNewIntent(intent);
setIntent(intent);
openurl()
}
```
IOS
在工程配置的info的URL Types中加入URL Schemes xxxx
重载openURL，处理url传入参数。

在调用是，只需要在浏览器中调用
xxxx:xxxxxx/openwith?x=x
就可以启动app了。
因为微信屏蔽了url schemes，所以上述方法只能在浏览器中使用，如果想在微信启动应用，ios上可以用Universal Links来实现，具体方法如下，
工程配置Capabilities中找到Associated Domains，在里面加入
applinks:yourdomain.com
app就会在第一次启动是在这个域名的空间下载一个apple-app-site-association文件，文件格式如下
```
{
“applinks”: {
“apps”: [],
“details”: {
“yourteamid.bundleid”: {
“paths”:[ “*” ]
}
}
}
}
```
yourteamid就是你的开发者帐号的teamid，bundleid就是ios工程中的bundle id，
paths是app相应什么url路径可以用通配符*，或者具体路径如”/xxxx/openwith”
把这个apple-app-site-association文件上传到你的http域名的目录下就可以了,
ios9.3以后，请求地址变为了.well-known/apple-app-site-association。
另外网站需要使用ssl访问，并且保证证书不会提示警告，就是要用正式证书。
如果没有证书apple-app-site-association需要签名。
cat apple-app-site-association-unsigned.js | openssl smime -sign -inkey g01-server.key -signer g01-server.crt -certfile 　g01-dvcacert.cer -noattr -nodetach -outform DER > apple-app-site-association
可以在地址中校验文件是否有效。
在微信中打开你的域名就会启动app，如打开
```
http://yourteamid/xxxx/openwith
```
---
