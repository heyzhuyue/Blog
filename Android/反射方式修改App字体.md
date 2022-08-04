# 反射方式修改App字体

1.  自定义Applicaiton,实现全局App修改,对于字体也是同理,通过Typeface设置加载App全局字体,对于自定义的Applicaiton需在AndroidManifest.xml的tag这个标签中进行设置.  

```plain
     public class App extends Application {
 	private static final String TAG = "软件商店";
         public static Typeface typeFace;

 	@Override
     public void onCreate() {// Application启动时调用
 // TODO Auto-generated method stub
     	super.onCreate();
     	setTypeface();
     }
     	public void setTypeface() {
 // 微软雅黑，加载外部字体assets/front/huawen_caiyun.ttf
     typeFace = Typeface.createFromAsset(getAssets(), "fonts/mi_font.ttf");
 	try {
 	        Field field_3 = Typeface.class.getDeclaredField("SANS_SERIF");
         	field_3.setAccessible(true);
         	field_3.set(null, typeFace);
             	} catch (NoSuchFieldException e) {
                 	e.printStackTrace();
         	} catch (IllegalAccessException e) {
             	e.printStackTrace();
 	}
  }
 }
```

2. 自定义Applicaiton的Theme,在Values文件夹下style文件下自定义Theme,并且设置sans属性 
   这里android:typeface可以设置的仅仅有normal、sans、serif、monospace可以设置，因为我在SetAppTypeface类中设置的是Typeface.class.getDeclaredField("SANS_SERIF");所以我这里便设置成sans，如果getDeclaredField()设置的是其他的类型，则要选择同类型的其他诸如serif、monospace等等 

```plain
 <style name="AppTheme" parent="@android:style/Theme.NoTitleBar">
      <item name="android:typeface">sans</item>
      <!-- All customizations that are NOT specific to a particular API-level can go here. -->
 </style>
```

 在AndroidManifest.xml的标签中使用自定义Theme  

```plain
 <application
 android:allowBackup="true"
 android:name=".App"
 android:icon="@drawable/ic_launcher"
 android:label="@string/app_name"
 android:theme="@style/AppTheme" >
  <activity
     android:name=".MainActivity"
     android:label="@string/app_name" >
     <intent-filter>
         <action android:name="android.intent.action.MAIN" />
         <category android:name="android.intent.category.LAUNCHER" />
     </intent-filter>
     </activity>
 </application>
```