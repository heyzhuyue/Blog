# App Startup

## Jetpack App Startup 库

### 一、前言说明

应用启动时间是应用性能的关键衡量指标。应用启动后，用户期望能够得到快速响应并加载内容，当不符合预期时用户会感到失望，甚至会卸载App。在开发过程中，随着业务的不断迭代，难免会引入一些三方库和工具，例如平台分享、支付、消息推送、热更新 …等功能库，这些库初始化大部分放在**Application的Oncreate**方法中进行初始化，这样会造成增加App启动时间，造成性能下降，因此可以使用**App Startup库**减少应用启动时间。

[Jetpack App Startup](https://developer.android.google.cn/topic/libraries/app-startup) 库在应用启动时以一种简单、高效的方法来初始化组件。库开发者和应用开发者都可以使用 App Startup 简化启动流程，并显式指定初始化顺序。

### 二、集成方法

#### 配置Gradle

```groovy
dependencies {
   implementation "androidx.startup:startup-runtime:1.0.0"
}
```

#### 实现 Initializer继承

为了在应用中使用 App Startup，框架底层定义一个 [Initializer](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.android.google.cn%2Ftopic%2Flibraries%2Fapp-startup%23implement-initializers)接口。开发者可以通过继承**Initializer**接口，创建多个Initializer实现类，实现不同类型框架的初始化

```kotlin
interface Initializer<out T: Any> {
    fun create(context: Context): T
    fun dependencies(): List<Class<out Initializer<*>>>
}
```

在实际使用中，我们以**WorkManager**为例子，讲解初始化 **WorkManager** 的 Initializer。实现如下:

```kotlin
class WorkManagerInitializer : Initializer<WorkManager> {
    override fun create(context: Context): WorkManager {
        val configuration = Configuration.Builder()
            .setMinimumLoggingLevel(Log.DEBUG)
            .build()
        WorkManager.initialize(context, configuration)
        return WorkManager.getInstance(context)
    }
    // 此组件无需任何依赖
    override fun dependencies() = emptyList<Class<out Initializer<*>>>()
}
```

最后，我们需要在 `AndroidManifest.xml` 中增加 `WorkManagerInitializer` 条目:

```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes WorkManagerInitializer discoverable. -->
    <meta-data android:name="com.std.stdlink.initializer.WorkManagerInitializer"
          android:value="androidx.startup" />
</provider>
```

### 三、工作原理

------

App Startup 使用了一个名为 InitializationProvider 的 ContentProvider。该 ContentProvider 在合并后的 AndroidManifest.xml 文件中查找 条目来发现 Initializer。此过程发生在 Application.onCreate() 被调用之前。

**ContentProvider的onCreate的调用时机介于Application的attachBaseContext和onCreate之间**，Provider的onCreate优先于Application的onCreate执行，并且此时的Application已经创建成功，而Provider里的context正是Application的对象,Lifecycle这么做，把init的逻辑放到库内部，让调用方完全不需要在Application里去进行初始化了。同样还可以减小**Application的onCreate**初始化压力，小幅度提升应用启动速度。

**使用ContentProvider初始化的方式，我们需要注意一下几点：**

首先，在Manifest里设置ContentProvider的时候要设置一个**authorities，这个authorities相当于ContentProvider的标识**，是不能重复的，为了保证不重复，我们再设置这个值的时候最好不要硬编码，而是使用以下的这种方式：使用appid+xxx

### 四、延迟初始化

------

我们强烈推荐您使用延迟初始化来进一步提升启动性能，您可以通过如下方式实现组件的延迟初始化，在 `<meta-data>` 条目下为 `Initializer` 增加 `tools:node="remove"` 属性，这将禁用即时初始化:

```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes WorkManagerInitializer discoverable. -->
    <meta-data android:name="com.std.stdlink.initializer.WorkManagerInitializer"
                tools:node="remove"
          android:value="androidx.startup" />
</provider>
```

为了实现 `WorkManagerInitializer` 的延迟初始化，您可以进行如下操作:

```java
// 此处返回一个 WorkManager 的实例
AppInitializer.getInstance(context)
    .initializeComponent(WorkManagerInitializer.class);
```

[官方集成文档](https://developer.android.google.cn/topic/libraries/app-startup#groovy)

### 五、延伸知识点

------

#### ContentProvider如何共享数据

ContentProvider通过uri来标识其它应用要访问的数据，通过ContentResolver的增、删、改、查方法实现对共享数据的操作。还可以通过注册ContentObserver来监听数据是否发生了变化来对应的刷新页面。下面分别说说每个类的作用。

#### ContentProvider讲解

ContentProvider是一个抽象类，如果我们需要开发自己的内容提供者我们就需要继承这个类并复写其方法，需要实现的主要方法如下：

```plain
public boolean onCreate()
```

在创建ContentProvider时使用

```java
public Cursor query()
```

用于查询指定uri的数据返回一个Cursor

```java
public Uri insert()
```

用于向指定uri的ContentProvider中添加数据

```java
public int delete()
```

用于删除指定uri的数据

```java
public int update()
```

用户更新指定uri的数据

```java
public String getType()
```

用于返回指定的Uri中的数据MIME类型

数据访问的方法insert，delete和update可能被多个线程同时调用，此时必须是线程安全的

#### Uri分析

ContentResolver通过uri来定位自己要访问的数据，所有需要我们了解Uri组成已经各个域的意义。Uri的格式为`[scheme:][//host:port][path][?query]`

可能这么看大家不一定了解的，打个比方 URI:`http://www.baidu.com:8080/wenku/jiatiao.html?id=123456&name=jack`

scheme：根据格式我们很容易看出来scheme为http

host：[www.baidu.com](http://www.baidu.com/)

port：就是主机名后面path前面的部分为8080

path：在port后面？的前面为wenku/jiatiao.html

query:?之后的都是query部分为 id=123456$name=jack

uri的各个部分在安卓中都是可以通过代码获取的，下面我们就以上面这个uri为例来说下获取各个部分的方法：

getScheme() :获取Uri中的scheme字符串部分，在这里是http

getHost():获取Authority中的Host字符串，即 [www.baidu.com](http://www.baidu.com/)

getPost():获取Authority中的Port字符串，即 8080

getPath():获取Uri中path部分，即 wenku/jiatiao.html

getQuery():获取Uri中的query部分，即 id=15&name=du

```java
public class StudentContentProvider extends ContentProvider {
    //这里的AUTHORITY就是我们在AndroidManifest.xml中配置的authorities
    private static final String AUTHORITY = "com.std.stdlink.studentProvider";
    //匹配成功后的匹配码
    private static final int MATCH_CODE = 100;
    private static UriMatcher uriMatcher;
    private StudentDao studentDao;
    //数据改变后指定通知的Uri
    private static final Uri NOTIFY_URI = Uri.parse("content://" + AUTHORITY + "/student");

    static {
        //匹配不成功返回NO_MATCH(-1)
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        //添加我们需要匹配的uri
        uriMatcher.addURI(AUTHORITY,"student", MATCH_CODE);
    }


    @Override
    public boolean onCreate() {
        studentDao = StudentDao.getInstance(getContext());
        return false;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection,
                        @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        int match = uriMatcher.match(uri);
        if (match == MATCH_CODE){
            Cursor cursor = studentDao.queryStudent();
            return cursor;
        }
        return null;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        if (uriMatcher.match(uri) == MATCH_CODE){
            studentDao.insertStudent(values);
            notifyChange();
        }
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        if (uriMatcher.match(uri) == MATCH_CODE){
            int delCount = studentDao.deleteStudent();
            notifyChange();
            return delCount;
        }
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection,
                      @Nullable String[] selectionArgs) {
        return 0;
    }

    private void notifyChange(){
        getContext().getContentResolver().notifyChange(NOTIFY_URI,null);
    }
}
```

```java
<provider
        android:authorities="com.std.stdlink.studentProvider"
        android:name=".StudentContentProvider"
        android:exported="true"/>
```

authorities唯一标识该内容提供者，作为其他访问应用找到当前ContentProvider的索引，通过这个才能要操作的ContentProvider并且提供数据操作。

- 在query、insert和delete方法中都是先调用uriMatcher.match(uri) 判断当前uri是不是匹配，如果匹配才能操作数据（该例子没有添加update功能，方式与其它三个方法一样），MATCH_CODE是我们在调用
  public void addURI (String authority, String path, int code)
  方法添加uri时设置的，当外部应用传递过来的uri与对应add的uri一致时，会返回我们设置的code。
- 该例子中整个数据的操作都是通过SQLite数据库完成的，在onCreate方法中我们先获得数据库的操作对象，通过该对象完成数据库的增、删、改、查，数据库操作。
- 在数据库发生变化的时候我们调用notifyChange方法

```java
private void notifyChange(){
        getContext().getContentResolver().notifyChange(NOTIFY_URI,null);
    }
```

通过其他的应用操作ContentProvider中的数据

- 获得ContentObserver

```java
contentResolver = getContentResolver();
```

- 注册ContentObserver来监听对应uri的数据变化，这步不是必须的，如果不需要监听数据变化也可以不注册

```java
private static final String AUTHORITY = "com.std.stdlink.studentProvider";
private static final Uri STUDENT_URI = Uri.parse("content://" + AUTHORITY + "/student");
contentResolver.registerContentObserver(STUDENT_URI, true, new MyContentObserver(handler));
public class StdLinkContentObserver extends ContentObserver {

    Handler mHandler;
    public MyContentObserver(Handler handler) {
        super(handler);
        mHandler = handler;
    }

    @Override
    public void onChange(boolean selfChange) {
        super.onChange(selfChange);
    }

    @Override
    public void onChange(boolean selfChange, Uri uri) {
        super.onChange(selfChange, uri);
        Message message = Message.obtain();
        message.obj = uri;
        mHandler.sendMessage(message);
    }
}
```

- 通过ContentProvider来操作数据

```java
Cursor cursor = contentResolver.query(STUDENT_URI, null, null, null, null, null);
        if (cursor != null && cursor.getCount() > 0) {
            while (cursor.moveToNext()) {
                String name = cursor.getString(cursor.getColumnIndex("name"));
                int age = cursor.getInt(cursor.getColumnIndex("age"));
                String desc = cursor.getString(cursor.getColumnIndex("desc"));
                Log.e(getClass().getSimpleName(), "Student:" + "name = " + name + ",age = " + age + ",desc = " + desc);
            }
            cursor.close();
        }

 ContentValues contentValues = new ContentValues();
        contentValues.put("name", "新插入");
        contentValues.put("age", "-100");
        contentValues.put("desc", "我是新插入的呦。。。");
        contentResolver.insert(STUDENT_URI, contentValues);

 contentResolver.delete(STUDENT_URI, null, null);
```