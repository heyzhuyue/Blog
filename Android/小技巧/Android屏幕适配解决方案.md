# Android屏幕适配解决方案

### 屏幕相关基本概念

#### 屏幕分辨率

手机在横向和纵向上的像素点数总和，单位是像素（pixel)，1px = 1像素点，举个例子，1080x1920，即宽度方向上有1080个像素点，在高度方向上有1920个像素点。

#### 屏幕尺寸

屏幕尺寸指屏幕的对角线的长度，单位是英寸，1英寸=2.54厘米
，比如常见的屏幕尺寸有2.4、2.8、3.5、3.7、4.2、5.0、5.5、6.0等

#### 屏幕密度(density)

PPI：每英寸像素（Pixels Per Inch）又被称为DPI（Dots Per Inch）

Density Independent Pixels，即密度无关像素。
dip和dp是一个意思，都是Density Independent Pixels的缩写，即密度无关像素，上面我们说过，dpi是屏幕像素密度，假如一英寸里面有160个像素，这个屏幕的像素密度就是160dpi，那么在这种情况下，dp和px如何换算呢？在Android中，规定以160dpi为基准，1dp=1px，如果屏幕密度是320dpi，则1dp=2px，以此类推。

-  120dpi  1dp=0.75*1px 
-  160dpi, 1dp = 1dp 
-  240dpi, 1dp = 1.5*1px 
-  320dpi, 1dp = 2*1px 
-  480dpi, 1dp = 3*1px 
-  640dpi, 1dp = 4*1px   

```java
private void getScreenInfo() {
    // 获取屏幕分辨率
    int screenWidth  = getWindowManager().getDefaultDisplay().getWidth();		// 屏幕宽
    int screenHeight = getWindowManager().getDefaultDisplay().getHeight();		// 屏幕高
    
    Log.e( "屏幕分辨率", "screenWidth=" + screenWidth + "; screenHeight=" + screenHeight);
    
    // 获取像素密度和屏幕密度
    DisplayMetrics dm = new DisplayMetrics();
    dm = getResources().getDisplayMetrics();
    
    float density  = dm.density;		// 屏幕密度（像素比例：0.75/1.0/1.5/2.0）
    int densityDPI = dm.densityDpi;		// 像素密度（每寸像素：120/160/240/320）
    float xdpi = dm.xdpi;               //X轴方向的像素密度
    float ydpi = dm.ydpi;                //Y轴方向的像素密度
    
    Log.e("XY轴方向上的像素密度", "xdpi=" + xdpi + "; ydpi=" + ydpi);
    Log.e( " 像素密度和屏幕密度", "density=" + density + "; densityDPI=" + densityDPI);
    
    screenWidth  = dm.widthPixels;		// 屏幕宽
    screenHeight = dm.heightPixels;		// 屏幕高
    
    Log.e("屏幕分辨率", "screenWidth=" + screenWidth + "; screenHeight=" + screenHeight);
    
    // 获取屏幕密度（方法3）
    dm = new DisplayMetrics();
    getWindowManager().getDefaultDisplay().getMetrics(dm);
    
    density  = dm.density;
    densityDPI = dm.densityDpi;
    xdpi = dm.xdpi;
    ydpi = dm.ydpi;
    
    Log.e("屏幕XY轴方向上的像素密度", "xdpi=" + xdpi + "; ydpi=" + ydpi);
    Log.e("屏幕像素密度和屏幕密度", "density=" + density + "; densityDPI=" + densityDPI);
    
    int screenWidthDip = dm.widthPixels;
    int screenHeightDip = dm.heightPixels;
    
    Log.e("屏幕XY轴方向上的像素密度", "screenWidthDip=" + screenWidthDip + "; screenHeightDip=" + screenHeightDip);
    
    screenWidth  = (int)(dm.widthPixels * density + 0.5f);
    screenHeight = (int)(dm.heightPixels * density + 0.5f);
    
    Log.e(" 屏幕分辨率", "screenWidth=" + screenWidth + "; screenHeight=" + screenHeight); 
}
```

| 名称    | 屏幕密度 | 通常分辨率 |
| ------- | -------- | ---------- |
| ldpi    | 120dpi   | 320*240    |
| mhdpi   | 160dpi   | 320*480    |
| hdpi    | 240dpi   | 480*800    |
| xhdpi   | 320dpi   | 720*1280   |
| xxhdpi  | 480dpi   | 1080*1920  |
| xxxhdpi | 640dpi   | 1440*2560  |

| drawable名称    | dpi     | density      |
| --------------- | ------- | ------------ |
| drawable-ldpi   | dpi=120 | density=0.75 |
| drawable-mdpi   | dpi=160 | density=1    |
| drawable-hdpi   | dpi=240 | density=1.5  |
| drawable-xhdpi  | dpi=320 | density=2    |
| drawable-xxhdpi | dpi=480 | density=3    |

屏幕像素密度是指每英寸上的像素点数，单位是dpi，即“dot per inch”的缩写。屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小、分辨率越高，像素密度越大，反之越小。假设1英寸上像素点为160个，那么该屏幕像素密度为160dpi,同理可知其余屏幕像素密度。

为简便起见，Android 将所有屏幕密度分组为六种通用密度： 低、中、高、超高、超超高和超超超高。

- ldpi（低）~  120dpi
- mdpi（中）~  160dpi
- hdpi（高）~  240dpi
- xhdpi（超高）~  320dpi
- xxhdpi（超超高） ~ 480dpi
- xxxhdpi（超超超高）~ 640dpi

![img](https://cdn.nlark.com/yuque/0/2022/png/12856485/1658237585487-bd35ced0-4f7a-419a-ab52-a966a95b9cb6.png)

#### 像素密度(densityDpi)

像素密度(dpi，dots per inch；或PPI，pixels per inch)：每英寸上的像素点数，结合屏幕大小和屏幕分辨率如果5.0英寸的手机的屏幕分辨率为1280x720，那么像素密度为192dpi（计算过程为首先计算对角线上的像素数）

#### 屏幕相关单位

-  px：像素单位，屏幕上像素点的大小不是固定的，像素点可大可小 
-  dp：长度单位，与具体屏幕像素无关，显示的时候根据具体平台屏幕密度的不同最终转换为相应的像素长度，具体转换规则是: dp = （dpi/标准密度<160dpi>）*px,标准密度为160dpi，例如，1dp长度在密度为160dpi的平台表示一个像素的长度，而在240dpi的平台则表示1.5个像素的长度  

Density-independent pixel (dp)独立像素密度。标准是160dip.即1dp对应1个pixel，计算公式如：px = dp * (dpi / 160)，屏幕密度越大，1dp对应 的像素点越多。
上面的公式中有个dpi，dpi为DPI是Dots Per Inch（每英寸所打印的点数），也就是当设备的dpi为160的时候1px=1dp；

-  sp：Scale Independent Pixels, 即sp或sip,Android开发时用此单位设置文字大小，可根据字体大小首选项进行缩放，推荐使用12sp、14sp、18sp、22sp作为字体设置的大小，不推荐使用奇数和小数，容易造成精度的丢失问题,小于12sp的字体会太小导致用户看不清。 

### 适配方案

#### 屏幕适配之图片适配



![img](http://cc.cocimg.com/api/uploads/20151029/1446101501383824.jpg)



![img](http://upload-images.jianshu.io/upload_images/1779681-5a2e95b7ec92ff43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在设计图标时，对于5种主流的像素密度(mdpi,hdpi,xhdpi,xxhdpi和xxxdpi)应按照2:3:4:6:8的比例进行缩放。例如一个启动图片ic_launcher.png,它在各个像素密度文件夹下大小为：



- ldpi（低）36*36 (0.75x)
- mdpi（中）48*48 (1x)
- hdpi（高）72*72 (1.5x)
- xhdpi（超高）96*96 (2x)
- xxhdpi（超超高）144*144 (3x)
- xxxhdpi（超超超高）192*192 (4x)



对于每种密度下的icon应该设计成什么尺寸其实Android也是给出了最佳建议，icon的尺寸最好不要随意设计，因为过低的分辨率会造成图标模糊，而过高的分辨率只会徒增APK大小。建议尺寸如下表所示：



```plain
|屏幕密度|建议尺寸|
|:----:|:--------:|
|ldpi|32*32|
|mdpi|48*48|
|hdpi|72*72|
|xhdpi|96*96|
|xxhdpi|144 *144 |
|xxxhdpi|192 *192 |
```



**存在问题**



- 每套分辨率出一套图，为美工或者设计增加了许多工作量
- 对Android工程文件的apk包变的很大



**解决办法**



-  Android SDK加载图片流程 

1. 1. Android SDK会根据屏幕密度自动选择对应的资源文件进行渲染加载，比如说，SDK检测到你手机的分辨率是xhdpi，会优先到xhdpi文件夹下找对应的图片资源
   2. 如果xhdpi文件夹下没有图片资源，那么就会去分辨率高的文件夹下查找，比如xxhdpi，直到找到同名图片资源，将它按比例缩小成xhpi图片
   3. 如果往上查找图片还是没有找到，那么就会往低分辨率的文件夹查找，比如hdpi，直到找到同名图片资源，将它按比例放大成xhpi图片

-  推荐使用尺寸:xhdpi  

xhdpi目前主流，如果选用太低分辨率的图片，那么在高分辨率手机上显示就会模糊
iPhone主流的屏幕dpi约等于320, 刚好属于xhdpi，设计只需切一套图就好了。



#### 今日头条屏幕适配方案原理

以下适配方案均基于以下公式进行的计算



核心公式：`px值 = dp值 * metrics.density`  `densityDpi = 160 * density`



```kotlin
 "dpi=${resources.displayMetrics.densityDpi}" //屏幕像素密度
 "scaledDensity=${resources.displayMetrics.scaledDensity}" //屏幕密度
 "density=${resources.displayMetrics.density}" //屏幕密度
 "widthPixels=${resources.displayMetrics.widthPixels}" //屏幕宽度
 "heightPixels=${resources.displayMetrics.heightPixels}" //屏幕高度
```

- 其实方案的原理其实很简单，首先我们要明白一点，无论我们在`xml`中使用何种尺寸单位（dp、sp、pt......）,最后在绘制时都会给我们转成`px`!
- 知道这点后，剩下的容易了，我们选定一种尺寸单位（dp、sp、pt ......）作为我们的适配单位，然后篡改这个单位与`px`之间的转化比例，然后在`xml`中使用选定的单位做适配！就是这么简单！！！

#### 方案实现

下面，我们看下面的单位转换源码

```java
 public static float applyDimension(int unit, float value,
                                       DisplayMetrics metrics)
    {
        switch (unit) {
        case COMPLEX_UNIT_PX:
            return value;
        case COMPLEX_UNIT_DIP:
            return value * metrics.density;
        case COMPLEX_UNIT_SP:
            return value * metrics.scaledDensity;
        case COMPLEX_UNIT_PT:
            return value * metrics.xdpi * (1.0f/72);
        case COMPLEX_UNIT_IN:
            return value * metrics.xdpi;
        case COMPLEX_UNIT_MM:
            return value * metrics.xdpi * (1.0f/25.4f);
        }
        return 0;
    }
```



- `今日头条`选用`dp`作为适配单位，给出的理由是项目中大部分都是使用`dp`做单位。

- 推荐`dp`做适配。因为在适配出现异常的时候，选用`dp`至少很大程度可以防止出现页面显示不全，显示效果太差的问题！



在屏幕适配中，我们一般只对宽度适配，毕竟高度可以滑动解决！所以方案中给出的是用`dp`适配宽度。下面举个小例子：

- 由源码可知：`px值 = dp值 * metrics.density`，这里的`density`是指的手机的屏幕密度，由系统提供，不同的手机的`density`可能不同；所以我们不能直接使用系统的`density`，需要篡改`density`来达到适配的目的
- 这里假设UI设计给我们一张`640dp（高） x 360dp（宽）`的设计图，那么我们如果要适配所有屏幕，则`density = 设备屏幕的真实宽度(单位：px) / 360`，这样我们的`1dp`在所有设备屏幕的宽中所占的比例都是一样的，为`1/360`，然后我们只要直接照抄设计图中的`dp`值，宽度就适配啦！
- 获取到我们计算出来的`density`后，再修改下`densityDpi（公式：density = dpi / 160，这是系统计算 dp 的方法，在很多场景会用到，所以我们连 dpi 一并改了）`，所以`densityDpi = 160 * density`
- 除此之外，不要忘记了我们的`sp`。公式中`px值 = sp值 * metrics.scaledDensity`，一般情况下，`scaledDensity`与`density`是相等的，但是如果我们在系统设置中改变字体大小，`scaledDensity`与`density`就不相等了。所以此时我们的`scaledDensity`应该这样计算：`scaledDensity = 人为修改的density * (系统的ScaledDensity / 系统的Density)`（假如你不要字体大小随系统设置而改变，就直接使用`dp`做单位好了）

下面是`今日头条`给出的案例代码：

```java
// 系统的Density
 private static float sNoncompatDensity;
 // 系统的ScaledDensity
 private static float sNoncompatScaledDensity;

 public static void setCustomDensity(Activity activity, Application application) {
        DisplayMetrics displayMetrics = application.getResources().getDisplayMetrics();
        if (sNoncompatDensity == 0) {
            sNoncompatDensity = displayMetrics.density;
            sNoncompatScaledDensity = displayMetrics.scaledDensity;
            // 监听在系统设置中切换字体
            application.registerComponentCallbacks(new ComponentCallbacks() {
                @Override
                public void onConfigurationChanged(Configuration newConfig) {
                    if (newConfig != null && newConfig.fontScale > 0) {
                        sNoncompatScaledDensity=application.getResources().getDisplayMetrics().scaledDensity;
                    }
                }

                @Override
                public void onLowMemory() {

                }
            });
        }
        // 此处以360dp的设计图作为例子
        float targetDensity=displayMetrics.widthPixels/360;
        float targetScaledDensity=targetDensity*(sNoncompatScaledDensity/sNoncompatDensity);
        int targetDensityDpi= (int) (160 * targetDensity);
        displayMetrics.density = targetDensity;
        displayMetrics.scaledDensity = targetScaledDensity;
        displayMetrics.densityDpi = targetDensityDpi;

        DisplayMetrics activityDisplayMetrics = activity.getResources().getDisplayMetrics();
        activityDisplayMetrics.density = targetDensity;
        activityDisplayMetrics.scaledDensity = targetScaledDensity;
        activityDisplayMetrics.densityDpi = targetDensityDpi;
    }
```



[今日头条适配方案原文章链接](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fd9QCoBP6kV9VSWvVldVVwA)

基于以上的方法，我们会出现这样的疑惑，当需要做高度适配或者宽度、高度一起适配的时候，我们应该如何实现，如果出现的适配Bug情况，如何恢复到默认情况，这些都是需要考虑的问题，因此我们需要基于`今日头条实现方案`进行二次封装。

以下是我的思路：

- `dp`、`sp`的适配依然保留
- 使用冷门的`pt`单位来做备用，当需要同时用宽度适配与高度适配时，一个`dp`适配是绝对不够用的，使用`pt`做为高度适配的补充，`dp`做宽度适配
- 为了方便，不用每个`Activity`中去执行适配方案，直接在`Application.ActivityLifecycleCallbacks`中监听`Activity`被创建时，调用适配方案
- 另外提供恢复系统原来单位转换比例的方法

*封装*

- 使用时，必须先在application中使用`setup()`方法初始化一下
- `register()`方法不是必须使用，可以手动在`Activity`中调用`match()`方法做适配，必须在`setContentView()`之前
- 建议使用`dp`做宽度适配，`pt`做高度适配，毕竟宽度适配才是主流需要
- 如果设计图给的不是`dp`单位的设计图，甚至不是安卓设计图（小公司经常发生），尽量将设计图的尺寸换算为`dp`后，使用`dp`做适配（原因上面已经说过了）
- 对于老项目，可以直接使用`pt`做适配（防止使用`dp`，破环原来的项目布局，毕竟`dp`辣么普及）
- 使用`pt`做适配时，没有那么讲究，个人觉得可以直接照抄设计图尺寸



```java
/**
 * 屏幕适配方案
 * <br>
 * <p>PS: 提供 dp、sp 以及 pt 作为适配单位，建议开发中以 dp、sp 适配为主，pt 可作为 dp、sp 的适配补充</p>
 * <p>PS: 由今日头条适配方案修改而来: https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA</p>
 */
public final class ScreenAdapter {
    /**
     * 屏幕适配的基准
     */
    public static final int MATCH_BASE_WIDTH = 0;
    public static final int MATCH_BASE_HEIGHT = 1;
    /**
     * 适配单位
     */
    public static final int MATCH_UNIT_DP = 0;
    public static final int MATCH_UNIT_PT = 1;

    // 适配信息
    private static MatchInfo sMatchInfo;
    // Activity 的生命周期监测
    private static Application.ActivityLifecycleCallbacks mActivityLifecycleCallback;

    private ScreenAdapter() {
        throw new UnsupportedOperationException("u can't instantiate me...");
    }

    /**
     * 初始化
     *
     * @param application
     */
    public static void setup(@NonNull final Application application) {
        DisplayMetrics displayMetrics = application.getResources().getDisplayMetrics();
        if (sMatchInfo == null) {
            // 记录系统的原始值
            sMatchInfo = new MatchInfo();
            sMatchInfo.setScreenWidth(displayMetrics.widthPixels);
            sMatchInfo.setScreenHeight(displayMetrics.heightPixels);
            sMatchInfo.setAppDensity(displayMetrics.density);
            sMatchInfo.setAppDensityDpi(displayMetrics.densityDpi);
            sMatchInfo.setAppScaledDensity(displayMetrics.scaledDensity);
            sMatchInfo.setAppXdpi(displayMetrics.xdpi);
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            // 添加字体变化的监听
            application.registerComponentCallbacks(new ComponentCallbacks() {
                @Override
                public void onConfigurationChanged(Configuration newConfig) {
                    // 字体改变后,将 appScaledDensity 重新赋值
                    if (newConfig != null && newConfig.fontScale > 0) {
                        sMatchInfo.setAppScaledDensity(application.getResources().getDisplayMetrics().scaledDensity);
                    }
                }

                @Override
                public void onLowMemory() {
                }
            });
        }
    }

    /**
     * 在 application 中全局激活适配（也可单独使用 match() 方法在指定页面中配置适配）
     */
    @RequiresApi(api = Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public static void register(@NonNull final Application application, final float designSize, final int matchBase, final int matchUnit) {
        if (mActivityLifecycleCallback == null) {
            mActivityLifecycleCallback = new Application.ActivityLifecycleCallbacks() {
                @Override
                public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                    if (activity != null) {
                        match(activity, designSize, matchBase, matchUnit);
                    }
                }

                @Override
                public void onActivityStarted(Activity activity) {

                }

                @Override
                public void onActivityResumed(Activity activity) {

                }

                @Override
                public void onActivityPaused(Activity activity) {

                }

                @Override
                public void onActivityStopped(Activity activity) {

                }

                @Override
                public void onActivitySaveInstanceState(Activity activity, Bundle outState) {

                }

                @Override
                public void onActivityDestroyed(Activity activity) {

                }
            };
            application.registerActivityLifecycleCallbacks(mActivityLifecycleCallback);
        }
    }

    /**
     * 全局取消所有的适配
     */
    @RequiresApi(api = Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public static void unregister(@NonNull final Application application, @NonNull int... matchUnit) {
        if (mActivityLifecycleCallback != null) {
            application.unregisterActivityLifecycleCallbacks(mActivityLifecycleCallback);
            mActivityLifecycleCallback = null;
        }
        for (int unit : matchUnit) {
            cancelMatch(application, unit);
        }
    }


    /**
     * 适配屏幕（放在 Activity 的 setContentView() 之前执行）
     *
     * @param context
     * @param designSize
     */
    public static void match(@NonNull final Context context, final float designSize) {
        match(context, designSize, MATCH_BASE_WIDTH, MATCH_UNIT_DP);
    }

    /**
     * 适配屏幕（放在 Activity 的 setContentView() 之前执行）
     *
     * @param context
     * @param designSize
     * @param matchBase
     */
    public static void match(@NonNull final Context context, final float designSize, int matchBase) {
        match(context, designSize, matchBase, MATCH_UNIT_DP);
    }

    /**
     * 适配屏幕（放在 Activity 的 setContentView() 之前执行）
     *
     * @param context
     * @param designSize 设计图的尺寸
     * @param matchBase  适配基准
     * @param matchUnit  使用的适配单位
     */
    public static void match(@NonNull final Context context, final float designSize, int matchBase, int matchUnit) {
        if (designSize == 0) {
            throw new UnsupportedOperationException("The designSize cannot be equal to 0");
        }
        if (matchUnit == MATCH_UNIT_DP) {
            matchByDP(context, designSize, matchBase);
        } else if (matchUnit == MATCH_UNIT_PT) {
            matchByPT(context, designSize, matchBase);
        }
    }

    /**
     * 重置适配信息，取消适配
     */
    public static void cancelMatch(@NonNull final Context context) {
        cancelMatch(context, MATCH_UNIT_DP);
        cancelMatch(context, MATCH_UNIT_PT);
    }

    /**
     * 重置适配信息，取消适配
     *
     * @param context
     * @param matchUnit 需要取消适配的单位
     */
    public static void cancelMatch(@NonNull final Context context, int matchUnit) {
        if (sMatchInfo != null) {
            final DisplayMetrics displayMetrics = context.getResources().getDisplayMetrics();
            if (matchUnit == MATCH_UNIT_DP) {
                if (displayMetrics.density != sMatchInfo.getAppDensity()) {
                    displayMetrics.density = sMatchInfo.getAppDensity();
                }
                if (displayMetrics.densityDpi != sMatchInfo.getAppDensityDpi()) {
                    displayMetrics.densityDpi = (int) sMatchInfo.getAppDensityDpi();
                }
                if (displayMetrics.scaledDensity != sMatchInfo.getAppScaledDensity()) {
                    displayMetrics.scaledDensity = sMatchInfo.getAppScaledDensity();
                }
            } else if (matchUnit == MATCH_UNIT_PT) {
                if (displayMetrics.xdpi != sMatchInfo.getAppXdpi()) {
                    displayMetrics.xdpi = sMatchInfo.getAppXdpi();
                }
            }
        }
    }

    public static MatchInfo getMatchInfo() {
        return sMatchInfo;
    }

    /**
     * 使用 dp 作为适配单位（适合在新项目中使用，在老项目中使用会对原来既有的 dp 值产生影响）
     * <br>
     * <ul>
     * dp 与 px 之间的换算:
     * <li> px = density * dp </li>
     * <li> density = dpi / 160 </li>
     * <li> px = dp * (dpi / 160) </li>
     * </ul>
     *
     * @param context
     * @param designSize 设计图的宽/高（单位: dp）
     * @param base       适配基准
     */
    private static void matchByDP(@NonNull final Context context, final float designSize, int base) {
        final float targetDensity;
        if (base == MATCH_BASE_WIDTH) {
            targetDensity = sMatchInfo.getScreenWidth() * 1f / designSize;
        } else if (base == MATCH_BASE_HEIGHT) {
            targetDensity = sMatchInfo.getScreenHeight() * 1f / designSize;
        } else {
            targetDensity = sMatchInfo.getScreenWidth() * 1f / designSize;
        }
        final int targetDensityDpi = (int) (targetDensity * 160);
        final float targetScaledDensity = targetDensity * (sMatchInfo.getAppScaledDensity() / sMatchInfo.getAppDensity());
        final DisplayMetrics displayMetrics = context.getResources().getDisplayMetrics();
        displayMetrics.density = targetDensity;
        displayMetrics.densityDpi = targetDensityDpi;
        displayMetrics.scaledDensity = targetScaledDensity;
    }

    /**
     * 使用 pt 作为适配单位（因为 pt 比较冷门，新老项目皆适合使用；也可作为 dp 适配的补充，
     * 在需要同时适配宽度和高度时，使用 pt 来适配 dp 未适配的宽度或高度）
     * <br/>
     * <p> pt 转 px 算法: pt * metrics.xdpi * (1.0f/72) </p>
     *
     * @param context
     * @param designSize 设计图的宽/高（单位: pt）
     * @param base       适配基准
     */
    private static void matchByPT(@NonNull final Context context, final float designSize, int base) {
        final float targetXdpi;
        if (base == MATCH_BASE_WIDTH) {
            targetXdpi = sMatchInfo.getScreenWidth() * 72f / designSize;
        } else if (base == MATCH_BASE_HEIGHT) {
            targetXdpi = sMatchInfo.getScreenHeight() * 72f / designSize;
        } else {
            targetXdpi = sMatchInfo.getScreenWidth() * 72f / designSize;
        }
        final DisplayMetrics displayMetrics = context.getResources().getDisplayMetrics();
        displayMetrics.xdpi = targetXdpi;
    }

    /**
     * 适配信息
     */
    public static class MatchInfo {
        private int screenWidth;
        private int screenHeight;
        private float appDensity;
        private float appDensityDpi;
        private float appScaledDensity;
        private float appXdpi;

        public int getScreenWidth() {
            return screenWidth;
        }

        public void setScreenWidth(int screenWidth) {
            this.screenWidth = screenWidth;
        }

        public int getScreenHeight() {
            return screenHeight;
        }

        public void setScreenHeight(int screenHeight) {
            this.screenHeight = screenHeight;
        }

        public float getAppDensity() {
            return appDensity;
        }

        public void setAppDensity(float appDensity) {
            this.appDensity = appDensity;
        }

        public float getAppDensityDpi() {
            return appDensityDpi;
        }

        public void setAppDensityDpi(float appDensityDpi) {
            this.appDensityDpi = appDensityDpi;
        }

        public float getAppScaledDensity() {
            return appScaledDensity;
        }

        public void setAppScaledDensity(float appScaledDensity) {
            this.appScaledDensity = appScaledDensity;
        }

        public float getAppXdpi() {
            return appXdpi;
        }

        public void setAppXdpi(float appXdpi) {
            this.appXdpi = appXdpi;
        }
    }
}
```

### 参考文章

- [CSDN博客](http://blog.csdn.net/u014316462/article/details/52349039)
- [简书文章](https://www.jianshu.com/p/ec5a1a30694b)
- [Google屏幕适配](https://developer.android.com/training/multiscreen/screensizes.html#TaskUseAliasFilters)
- [鸿洋大神](http://blog.csdn.net/lmj623565791/article/details/45460089)
- [Android drawable微技巧，你所不知道的drawable的那些细节](http://blog.csdn.net/guolin_blog/article/details/50727753)
- [Android中的像素密度，屏幕密度，屏幕大小，分辨率，ldpi，mdpi，xhdpi，xxhdpi](https://blog.csdn.net/u010126792/article/details/86624169)