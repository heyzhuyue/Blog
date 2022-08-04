# Android沉浸式状态栏

-  前言  

在使用App的过程中,如果细心观察,我们会发现,某些应用顶部菜单栏颜色会延伸到系统状态栏中，使得菜单栏和状态栏呈现统一颜色，国内把这样状态栏叫做沉浸式状态栏，ios中几乎所有App都统一实现这样的状态栏，使用沉浸状态栏会使App更好的融合到系统中，提高App颜值，虽然大部分应用不愿去实现沉浸，因为好奇，所以打算搜集资料学习了解一下

-  前提  

沉浸式是[android](http://lib.csdn.net/base/android) 4.4及以上才有的，在后续的5.0及6.0上面都增加了一些相关支持,所以如何更好适配从4.4到最新系统，是沉浸式最大问题，废话不多说 ，开始沉浸式实现，Let's go !

-  手把手教你实现
  首选: android在4.4版本上添加了WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS 和 WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION，即透明的状态栏和导航栏，在使用过程中，一般会配合fitsSystemWindows和clipToPadding属性一起使用 
  当然，如果你使用了Toolbar,可以通过以下方式设置
  1.在App主题中添加 **windowTranslucentStatus** 和 **windowIsTranslucent** 属性 
  也可以在Activity中通过代码设置实现上述功能,因为在部分机型上,xml设置属性无效 
  2.在Toolbar添加相关属性,使得Toolbar不受状态栏影响变形 
  使用**android:fitsSystemWindows="true"**,就可以调整内容布局(估计也是在根布局上加padding)恢复到原来位置
  在开发过程中，有事我们还会使用到Dawerlayout,针对Drawerlayout我们也需要实现沉浸式，所以如果开发中使用了Drawerlayout，需要设置以下代码实现Drawerlayout沉浸式  

```xml
android:fitsSystemWindows="true"  
android:clipToPadding="true"
 <style name="AppTheme.NoActionBar" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <!--沉浸状态栏添加-->
        <item name="android:windowTranslucentStatus">true</item>  <!--状态栏透明-->
        <item name="android:windowIsTranslucent">true</item>
    	<!--Android 5.x开始需要把颜色设置透明，否则导航栏会呈现系统默认的浅灰色-->  
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            WindowManager.LayoutParams localLayoutParams = getWindow().getAttributes();
            localLayoutParams.flags = (WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS | localLayoutParams.flags);
        }
<android.support.design.widget.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:clipToPadding="true"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay"
        toolbar:elevation="0.5dp">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:clipToPadding="true"
            android:fitsSystemWindows="true"
            android:minHeight="?attr/actionBarSize"
            toolbar:popupTheme="@style/AppTheme.PopupOverlay" />
    </android.support.design.widget.AppBarLayout>
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WindowManager.LayoutParams localLayoutParams = getWindow().getAttributes();    
    local LayoutParams.flags = (WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS | localLayoutParams.flags);
    if(Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP){
        //将侧边栏顶部延伸至status bar
        mDrawerLayout.setFitsSystemWindows(true);        
        //将主页面顶部延伸至status bar;虽默认为false,但经测试,DrawerLayout需显示设置
        mDrawerLayout.setClipToPadding(false);
    }
}
```

-  那些容易掉进去坑
  1.上面的**FLAG_TRANSLUCENT_STATUS** 只是把状态栏设置为透明的，但是！但是，状态栏是有背景色的，一些手机的状态栏背景色为透明，而一些手机的状态栏背景色为半透明的黑色。
  于是在5.0上增加了WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS 和 getWindow().setStatusBarColor(int color)，一般使用如下: 
  2.状态栏字体颜色
  状态栏的字体颜色默认为白色的，但是我们应用的主题色为黄色，白色字体在黄色背景上是不易分辨的，所以这里得把状态栏的字体改为深色，想想这个也是不太好做（其实这个在[iOS](http://lib.csdn.net/base/ios)上面自带的效果），但是在6.0增加了View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR，这个字段就是把状态栏标记为浅色，然后状态栏的字体颜色自动转换为深色。 

```java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
getWindow().clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
getWindow().setStatusBarColor(Color.TRANSPARENT);// SDK21
```

-  总结
  沉浸式是提高App融合性的好主意，建议更多应用支持沉浸式，给用户更好的App体验，最后，退出一端代码，对调用方法做一个简单注释  
  http://blog.csdn.net/brian512/article/details/52096445
  http://www.jianshu.com/p/a89bff39e8ca
  **最后一句,知识点就到这里,溜了！** 

```java
/**
     * 初始化沉浸状态栏
     */
    public void initTransparentBar() {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT && Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) { // Android4.4 < = SDK版本 < Android5.0
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS); //状态栏栏透明
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION); //导航栏透明
            getWindow().clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS); //强制状态栏透明
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) { //SDK版本 > = Android5.0
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS); //状态栏栏透明
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION); //导航栏透明
            getWindow().clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS); //强制状态栏透明
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) { // 状态栏字体设置为深色，SYSTEM_UI_FLAG_LIGHT_STATUS_BAR 为SDK23增加
                getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
            }
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            }
            getWindow().setStatusBarColor(Color.TRANSPARENT); //设置StatusBar颜色为透明
        }
    }
```

## 参考文章

[Android实现沉浸式状态栏的那些坑](http://blog.csdn.net/brian512/article/details/52096445)

[Android 沉浸式状态栏 +DrawerLayout+Toolbar,适配4.4X及以上](http://www.jianshu.com/p/a89bff39e8ca)