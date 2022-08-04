## Android版本适配

### Android M（6.0） 适配

运行时权限动态申请

### Android N（7.0） 适配

#### **FileProvider获取Uri**

在Android7.0系统上，Android 框架强制执行了 StrictMode API 政策禁止向你的应用外公开 file:// URI。 如果一项包含文件 file:// URI类型 的 Intent 离开你的应用，应用失败，并出现 FileUriExposedException 异常，如调用[系统相机拍照录制视频，或裁切照片](undefined)。

若要在应用间共享文件，可以发送 content:// URI类型的Uri，并授予 URI 临时访问权限。 进行此授权的最简单方式是使用 [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)类。

使用FileProvider的大致步骤如下：

**1.在manifest清单文件中注册provider**

```xml
<provider 

android:name="android.support.v4.content.FileProvider" 

android:authorities="com.jph.takephoto.fileprovider" 

android:grantUriPermissions="true" 

android:exported="false"> 

<meta-data 

android:name="android.support.FILE_PROVIDER_PATHS" 

android:resource="@xml/file_paths" />
```

exported:要求必须为false，为true则会报安全异常。grantUriPermissions:true，表示授予 URI 临时访问权限。

**2.指定共享的目录**

为了指定共享的目录我们需要在资源(res)目录下创建一个xml目录，然后创建一个名为“file_paths”(名字可以随便起，只要和在manifest注册的provider所引用的resource保持一致即可)的资源文件，内容如下：

```xml
<!-- 内部存储空间应用私有目录下的files/目录，等同于Context.getFilesDir() 所获取的目录路径 /data/data/包名/files目录-->
    <files-path
        name="DocDir"
        path="/" />
    <!-- 内部存储空间应用私有目录下的cache/目录，等同于Context.getCacheDir() 所获取的目录路径 /data/data/包名/cache目录-->
    <cache-path
        name="CacheDocDir"
        path="/" />
    <!--外部存储空间应用私有目录下的files/目录，等同于Context.getExternalFilesDir(null) 所获取的目录路径 /storage/sdcard/Android/data/包名/files-->
    <external-files-path
        name="ExtDocDir"
        path="/" />
    <!--外部存储空间应用私有目录下的cache/目录，等同于Context.getExternalCacheDir() /storage/sdcard/Android/data/包名/cache-->
    <external-cache-path
        name="ExtCacheDir"
        path="/" />
```

**3.使用FileProvider**

```java
File file=new File(Environment.getExternalStorageDirectory(), "/temp/"+System.currentTimeMillis() + ".jpg"); 
if (!file.getParentFile().exists()){
   file.getParentFile().mkdirs(); 
}
//通过FileProvider创建一个content类型的Uri 
Uri imageUri = FileProvider.getUriForFile(context, "com.jph.takephoto.fileprovider", file);
Intent intent = new Intent(); 
//添加这一句表示对目标应用临时授权该Uri所代表的文件 
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);   
//设置Action为拍照
intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);
//将拍取的照片保存到指定URI
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
startActivityForResult(intent,1006);
```

### Android O（8.0） 适配

#### 通知适配

Android 8.0 引入了通知渠道，其允许您为要显示的每种通知类型创建用户可自定义的渠道。用户界面将通知渠道称之为通知类别。[详见](https://developer.android.google.cn/training/notify-user/channels)

```java
private void createNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        NotificationManager notificationManager = (NotificationManager)
                getSystemService(Context.NOTIFICATION_SERVICE);
        //分组（可选）
        //groupId要唯一
        String groupId = "group_001";
        NotificationChannelGroup group = new NotificationChannelGroup(groupId, "广告");
        //创建group
        notificationManager.createNotificationChannelGroup(group);
        //channelId要唯一
        String channelId = "channel_001";
        NotificationChannel adChannel = new NotificationChannel(channelId,
                "推广信息", NotificationManager.IMPORTANCE_DEFAULT);
        //补充channel的含义（可选）
        adChannel.setDescription("推广信息");
        //将渠道添加进组（先创建组才能添加）
        adChannel.setGroup(groupId);
        //创建channel
        notificationManager.createNotificationChannel(adChannel);
        //创建通知时，标记你的渠道id
        Notification notification = new Notification.Builder(MainActivity.this, channelId)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                .setContentTitle("一条新通知")
                .setContentText("这是一条测试消息")
                .setAutoCancel(true)
                .build();
        notificationManager.notify(1, notification);
    }
}
```

删除渠道代码如下：

```java
private void deleteNotificationChannel(String channelId){
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        NotificationManager mNotificationManager = (NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);
        mNotificationManager.deleteNotificationChannel(channelId);
    }
}
```

#### 后台限制执行

[详见](https://developer.android.google.cn/about/versions/oreo/background)

应用在两个方面受到限制：

**后台服务限制:**处于空闲状态时，应用可以使用的后台服务存在限制。 这些限制不适用于前台服务，因为前台服务更容易引起用户注意。

**广播限制**:除了有限的例外情况，应用无法使用清单注册隐式广播。 它们仍然可以在运行时注册这些广播，并且可以使用清单注册专门针对它们的显式广播。

在大多数情况下，应用都可以使用 JobScheduler 克服这些限制。 这种方式让应用安排为在未活跃运行时执行工作，不过仍能够使系统可以在不影响用户体验的情况下安排这些作业。

```java
 private void sendNotification() {
        if (Build.VERSION.SDK_INT>=26) {
            Intent intent = new Intent(this, StopActivity.class);
            //创建NotificationChannel并与NotificationManager、Notification关联，否则系统会
            //提示Failed to post notification on channel “null”
            String channelId = "channel1";
            Notification nf = new Notification.Builder(this, channelId)
                    .setContentTitle("通知")
                    .setContentText("一个服务正在运行")
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setContentIntent(PendingIntent.getActivities(this, 100, new Intent[]{intent}, PendingIntent.FLAG_CANCEL_CURRENT))
                    .build();
            startForeground(10, nf);
        }
    }
 if (Build.VERSION.SDK_INT>=26) {
    startForegroundService(new Intent(this, MyService.class));
 }else{
   startService(new Intent(this, MyService.class));
 }
```

关于的用法可以参考官方例子：android-JobScheduler

https://github.com/googlesamples/android-JobScheduler

当然还有后台位置的限制需要去注意。

[详见](https://developer.android.google.cn/about/versions/oreo/background-location-limits)

#### APK文件下载成功没有正常跳到应用安装界面

Android O (Android 8.0) 中，Google 移除掉了容易被滥用的“允许未知来源”应用的开关，在安装 Play Store 之外的第三方来源的 Android 应用的时候，竟然没有了“允许未知来源”的检查框，如果你还是想要安装某个被自己所信任的开发者的 app，则需要在每一次都手动授予“安装未知应用”的许可。

首先在AndroidManifest.xml 清单文件中添加安装未知来源应用的权限

```xml
  <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```

然后在用户点击更新时判断是否开启了该应用的“允许安装未知来源”的权限，没有的话，就引导用户去开启该应用的“允许安装未知来源”的权限

```java
private void downloadAPK(){
      boolean hasInstallPerssion = getPackageManager().canRequestPackageInstalls();
            if (hasInstallPerssion ) {
               //安装应用的逻辑
            } else {
               //跳转至“安装未知应用”权限界面，引导用户开启权限，可以在onActivityResult中接收权限的开启结果
                Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
                startActivityForResult(intent, REQUEST_CODE_UNKNOWN_APP);
            }
          }

//接收“安装未知应用”权限的开启结果
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQUEST_CODE_UNKNOWN_APP) {
            downloadAPK();
        }
    }
```

这样点击更新时引导用户开启“允许安装未知来源”的权限后，APK文件下载成功后也成功的跳转到应用安装界面。第三个问题也得到了解决。

### Android P（9.0） 适配

#### Http请求失败

在9.0中默认情况下启用网络传输层安全协议 (TLS)，默认情况下已停用明文支持。也就是不允许使用http请求，要求使用https。

解决方法是需要我们添加网络安全配置。首先在res 目录下新建xml文件夹，添加network_security_config.xml文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

 AndroidManifest.xml中的 application添加

以上这是一种简单粗暴的配置方法，要么支持http，要么不支持http。为了安全灵活，我们可以指定支持的http域名：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
	<!-- Android 9.0 上部分域名时使用 http -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">secure.example.com</domain>
        <domain includeSubdomains="true">cdn.example1.com</domain>
    </domain-config>
</network-security-config>
```

#### 前台服务

Android P（9.0）要求创建一个前台服务需要请求 FOREGROUND_SERVICE 权限，否则系统会引发 SecurityException

```plain
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

#### 其他

在 Android 9 中，调用Build.SERIAL 会始终返回 UNKNOWN 以保护用户的隐私。如果你的应用需要访问设备的硬件序列号，那么需要先请求 READ_PHONE_STATE 权限，然后调用 Build.getSerial

### Android Q（10.0） 适配

#### 分区存储

应用只能看到本应用专有的目录（通过 Context.getExternalFilesDir() 访问）以及特定类型的媒体。除非您的应用需要访问存放在应用的专有目录以及 MediaStore 之外的文件，否则最好使用分区存储。

要点：
1.Android Q文件存储机制修改成了沙盒模式
2.APP只能访问自己目录下的文件和公共媒体文件
3.Android Q版本以下机型，还是使用老的文件存储方式
4.Android Q及以上版本机型，所有应用均需要分区存储, 所以应用需要提前确保支持分区存储

*需要注意：在适配AndroidQ的时候还要兼容Q系统版本以下的，使用SDK_VERSION区分*
*外部存储：被分为应用私有目录以及共享目录两个部分*

**应用私有目录：存储应用私有数据，外部存储应用私有目录对应**

1.外部存储应用私有目录对应/Android/data/包名/，内部存储应用私有目录对应/data/data/包名/
2.应用私有目录文件访问方式与之前Android版本一致，可以通过File path获取资源。

**共享目录：**

1.存储其他应用可访问文件， 包含媒体文件、文档文件以及其他文件，对应设备DCIM、Pictures、Alarms, Music, Notifications,Podcasts, Ringtones、Movies、Download等目录。

2.共享目录文件需要通过MediaStore API或者Storage Access Framework方式访问。

3.MediaStore API在共享目录指定目录下创建文件或者访问应用自己创建文件，不需要申请存储权限

4.MediaStore API访问其他应用在共享目录创建的媒体文件(图片、音频、视频)， 需要申请存储权限，未申请存储权限，通过ContentResolver查询不到文件Uri，即使通过其他方式获取到文件Uri，读取或创建文件会抛出异常

5.MediaStore API不能够访问其他应用创建的非媒体文件(pdf、office、doc、txt等)， 只能够通过Storage Access Framework方式访问

**受影响的变更**

图片位置信息
一些图片会包含位置信息，因为位置对于用户属于敏感信息， Android 10应用在分区存储模式下图片位置信息默认获取不到，应用通过以下两项设置可以获取图片位置信息：
1.1在manifest中申请ACCESS_MEDIA_LOCATION
1.2调用MediaStore setRequireOriginal(Uri uri)接口更新图片Uri

**兼容模式**

应用未完成外部存储适配工作，可以临时以兼容模式运行， 兼容模式下应用申请存储权限，即可拥有外部存储完整目录访问权限，通过Android10之前文件访问方式运行，以下设置应用以兼容模式运行。

tagretSDK 大于等于Android 10（API level 29）， 在manifest中设置requestLegacyExternalStorage属性为true

```xml
<manifest ...>
...
<application android:requestLegacyExternalStorage="true" ... >
...
</manifest>
```



~~~plain
	**判断兼容模式接口**
```
//返回值
//true : 应用以兼容模式运行
//false：应用以分区存储特性运行
Environment.isExternalStorageLegacy();
```

	**File Path路径访问受影响接口**

开启分区存储新特性， Andrioid 10不能够通过File Path路径直接访问共享目录下资源，以下接口通过File 路径操作文件资源，功能会受到影响，应用需要使用MediaStore或者SAF方式访问。

**存储特性Android版本差异概览**
~~~



**适配指导**

AndroidQ中使用ContentResolver进行文件的增删改查。

**1）获取(创建)私有目录下的文件夹**

```plain
File apkFile = context.getExternalFilesDir("apk");
```

**2）创建私有目录文件**

生成需要下载的路径，通过输入输出流读取写入

```java
String apkFilePath = context.getExternalFilesDir("apk").getAbsolutePath();
File newFile = new File(apkFilePath + File.separator + "demo.apk");
OutputStream os = null;
try {
    os = new FileOutputStream(newFile);
    if (os != null) {
        os.write("file is created".getBytes(StandardCharsets.UTF_8));
        os.flush();
    }
} catch (IOException e) {
} finally {
    try {
        if (os != null) {
        os.close();
    }catch (IOException e1) {
    }
}
```

**3)创建共享目录文件或文件夹**

主要是在公共目录下创建文件或文件夹拿到本地路径uri，不同的Uri，可以保存到不同的公共目录中。接下来使用输入输出流就可以写入文件。

重点：AndroidQ中不支持file://类型访问文件，只能通过uri方式访问。

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    ContentResolver resolver = context.getContentResolver();
    ContentValues values = new ContentValues();
    values.put(MediaStore.Downloads.DISPLAY_NAME, fileName);
    values.put(MediaStore.Downloads.DESCRIPTION, fileName);
    //设置文件类型
    values.put(MediaStore.Downloads.MIME_TYPE, "application/vnd.android.package-archive");
    //注意MediaStore.Downloads.RELATIVE_PATH需要targetVersion=29,
    //故该方法只可在Android10的手机上执行
    values.put(MediaStore.Downloads.RELATIVE_PATH, "Download" + File.separator + "apk");
    Uri external = MediaStore.Downloads.EXTERNAL_CONTENT_URI;
    String status = Environment.getExternalStorageState();
    // 判断是否有SD卡,优先使用SD卡存储,当没有SD卡时使用手机存储
    if (status.equals(Environment.MEDIA_MOUNTED)) {
        return resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,values);
    } else {
        return resolver.insert(MediaStore.Images.Media.INTERNAL_CONTENT_URI, values);
    }
}
```

#### 定位权限

用户可以更好地控制应用何时可以访问设备位置。当在Android Q上运行的应用程序请求位置访问时，会通过对话框的形式给用户进行授权提示。此对话框允许用户授予对两个不同范围的位置访问权限：在使用中（仅限前台）或始终（前台和后台）

新增权限 ACCESS_BACKGROUND_LOCATION

如果你的应用针对 Android Q 并且需要在后台运行时访问用户的位置，则必须在应用的清单文件中声明新权限

```xml
<manifest>
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
</manifest>
```

#### 新增 ACCESS_MEDIA_LOCATION 权限

一些照片在其数据中会包含位置信息，允许用户查看拍摄照片的位置。由于此位置信息是敏感的，因此我们想获取位置信息需要以下几步：

- 将新的 ACCESS_MEDIA_LOCATION 权限添加到AndroidManifest
- 获取位置信息

```java
photoUri = MediaStore.setRequireOriginal(photoUri);
//从流中读取位置信息
InputStream stream = getContentResolver().openInputStream(photoUri);
```

### Android R（11.0） 适配

#### Scoped Storage（分区存储）

不过需要注意的是，应用targetSdkVersion >= 30，强制执行分区存储机制。之前在AndroidManifest.xml中添加 android:requestLegacyExternalStorage="true"的适配方式已不起作用。

还有一个变化：Android 11 允许使用除 MediaStore API 之外的 API 通过文件路径直接访问共享存储空间中的媒体文件。其中包括：

- File API
- 原生库，例如 fopen()

如果你之前没有适配Android 10，这一点对你来说是个好消息。Android 10在AndroidManifest.xml中添加 android:requestLegacyExternalStorage="true"来适配，Android 11上直接使用File API访问媒体文件。

不过，使用原始文件路径直接访问共享存储空间中的媒体文件会重定向到 MediaStoreAPI，这次重定向会造成性能影响（随机读写慢一倍左右）。而且直接使用原始文件路径，并不会比使用 MediaStore API 有更多优势，因此官方强烈建议直接使用 MediaStore API。

当然还有一种简单粗暴的适配方法，获取外部存储管理权限。如果你的应用是手机管家、文件管理器这类需要访问大量文件的app，可以申请MANAGE_EXTERNAL_STORAGE权限，将用户引导至系统设置页面开启。代码如下：

```plain
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"
        tools:ignore="ScopedStorage" />
```

#### 单次权限授权

从 Android 11 开始，每当应用请求与位置信息、麦克风或摄像头相关的权限时，面向用户的权限对话框会包含仅限这一次选项。如果用户在对话框中选择此选项，系统会向应用授予临时的单次授权。

单次权限授权的应用可以在一段时间内访问相关数据，具体时间取决于应用的行为和用户的操作：

- 当应用的 Activity 可见时，应用可以访问相关数据
- 如果用户将应用转为后台运行，应用可以在短时间内继续访问相关数据
- 如果您在 Activity 可见时启动了一项前台服务，并且用户随后将您的应用转到后台，那么您的应用可以继续访问相关数据，直到该前台服务停止
- 如果用户撤消单次授权（例如在系统设置中撤消），无论您是否启动了前台服务，应用都无法访问相关数据。与任何权限一样，如果用户撤消了应用的单次授权，应用进程就会终止

当用户下次打开应用并且应用中的某项功能请求访问位置信息、麦克风或摄像头时，系统会再次提示用户授予权限。

#### 请求位置权限

这部分在Android 10的适配有过调整，当时规则如下：

请求ACCESS_FINE_LOCATION或 ACCESS_COARSE_LOCATION权限表示在前台时拥有访问设备位置信息的权限。在请求弹框中，选择“始终允许”表示前后台都可以获取位置信息，选择“仅在应用使用过程中允许”只表示拥有前台的权限。

在Android 11中，请求弹框中取消了“始终允许”这一选项。也就是说默认不会授予你后台访问设备位置信息的权限。如果尝试请求ACCESS_BACKGROUND_LOCATION权限的同时请求任何其他权限，系统会抛出异常，不会向应用授予其中的任一权限。

官方给出的适配建议及原因如下：

建议应用对位置权限执行递增请求，先请求前台位置信息访问权限，再请求后台位置信息访问权限。执行递增请求可以为用户提供更大的控制权和透明度，因为他们可以更好地了解应用中的哪些功能需要后台位置信息访问权限。

总结一下得出两点：

-  先请求前台位置信息访问权限，再请求后台位置信息访问权限。 
-  单独请求后台位置信息访问权限，不要与其他权限一同请求。 

#### 软件包可见性

软件包可见性是Android 11上提升系统隐私安全性的一个新特性。它的作用是限制app随意获取其他app的信息和安装状态。避免病毒软件、间谍软件利用，引发网络钓鱼、用户安装信息泄露等安全事件。

软件包可见性是Android 11上提升系统隐私安全性的一个新特性。它的作用是限制app随意获取其他app的信息和安装状态。避免病毒软件、间谍软件利用，引发网络钓鱼、用户安装信息泄露等安全事件。

解决方法很简单，在AndroidManifest.xml 中添加queries元素，里面添加需要可见的应用包名。

```xml
<manifest package="com.example.app">
   <queries>
   <!-- 微博 -->
   <package android:name="com.sina.weibo" />
   <!-- QQ -->
   <package android:name="com.tencent.mobileqq" />
   <!-- 支付宝 -->
   <package android:name="com.eg.android.AlipayGphone" /> 
   <!-- AlipayHK -->
   <package android:name="hk.alipay.wallet" />
   </queries>
</manifest>
```

除了直接添加包名的方式外，我们可以按intent和provider来添加：

```xml
<manifest package="com.example.app">
    <queries>
        <intent>
            <action android:name="android.intent.action.SEND" />
            <data android:mimeType="image/jpeg" />
        </intent>

        <provider android:authorities="com.example.settings.files" />
    </queries>
```

具体的规则参见：[管理软件包可见性](https://developer.android.google.cn/training/basics/intents/package-visibility#intent-signature)

当然，还有一种简单粗暴的方式，可以直接申请权限QUERY_ALL_PACKAGES。如果你的应用需要上架Google Play，那么可能要注意相关政策。为了尊重用户隐私，建议我们的应用按正常工作所需的最小软件包可见性来适配。

最后需要注意的是，使用queries元素需要Android Gradle 插件版本是 4.1及以上，因为旧版本的插件并不兼容此元素，出现合并 manifest 的错误。

#### 前台服务类型

Android 10中，在前台服务访问位置信息，需要在对应的service中添加 location 服务类型。

同样的，Android 11中，在前台服务访问摄像头或麦克风，需要在对应的service中添加camera或microphone 服务类型。

```xml
<manifest>
   <service 
       android:name="MyService"
       android:foregroundServiceType="location|microphone|camera" />
</manifest>
```



这一限制的变更，使得程序无法在后台启动服务访问摄像头和麦克风。如需使用，只能是前台开启前台服务。除非有如下情况：

- 服务由系统组件启动
- 服务是通过应用小部件启动
- 服务是通过与通知交互启动的
- 服务是PendingIntent启动的，它是从另一个可见的应用程序发送过来的
- 服务由一个提供VoiceInteractionService的应用启动
- 服务由一个具有START_ACTIVITIES_FROM_BACKGROUND权限的应用启动

#### 权限自动重置

如果应用以 Android 11 或更高版本为目标平台并且数月未使用，系统会通过自动重置用户已授予应用的运行时敏感权限来保护用户数据。如下图所示：

注意上图中有一个启动自动重置的开关。如果我们的应用有特殊需要，可以引导用户关闭它。示例代码如下：

```java
public void checkAutoRevokePermission(Context context) {
    // 判断是否开启
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R &&
            !context.getPackageManager().isAutoRevokeWhitelisted()) {
        // 跳转设置页    
        Intent intent = new Intent(Intent.ACTION_AUTO_REVOKE_PERMISSIONS);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setData(Uri.fromParts("package", context.getPackageName(), null));
        context.startActivity(intent);
    }
}
```

#### 读取手机号

如果你是通过TelecomManager的getLine1Number方法，或TelephonyManager的getMsisdn方法获取电话号码。那么在Android 11中需要增加READ_PHONE_NUMBERS权限。使用其他方法不受限。

```xml
<manifest>
    <!-- 如果应用仅在 Android 10及更低版本中使用该权限，可以添加 maxSdkVersion="29" -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"
                     android:maxSdkVersion="29" />
    <uses-permission android:name="android.permission.READ_PHONE_NUMBERS" />
</manifest>
```

参考文章
[Android R（11.0） 适配](https://mp.weixin.qq.com/s/VN6juSAvLpziZlE2VaN6Lw)