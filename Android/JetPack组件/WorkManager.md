# WorkManager

## JectPack WorkManager库

### 前言

在日常开发过程中，我们会遇到某些需要后台执行上传本地数据到服务器这样的需求，通过后台服务的方式，当我们把应用从后台杀掉或者遇到系统杀死App或限制其网络环境情况下，将降低数据上传的成功性。因此Google提供了WorkManager。Google给的描述是：它对可延期任务操作非常简单，同时稳定性非常强，对于异步任务，即使App退出运行或者设备重启，它都能够很好的保证任务的顺利执行。

WorkManager对于当有一个后台任务在执行的过程中，app突然退出或者手机断网时，实现特殊场景的异步任务执行有简单操作，稳定性强的特点。

### 集成方法

#### Gradle配置

```groovy
    implementation "androidx.work:work-runtime:2.5.0"
    implementation "androidx.work:work-runtime-ktx:2.5.0"
    implementation "androidx.work:work-rxjava2:2.5.0"
```

#### 构建Work类

自定义Worker类，继承自Worker，然后复写doWork() 方法，返回当前任务的结果 Result。doWork方法是执行在子线程的。

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters):
       Worker(appContext, workerParams) {
   override fun doWork(): Result {
       // Do the work here--in this case, upload the images.
       uploadImages()
       // Indicate whether the work finished successfully with the Result
       return Result.success()
   }
}
```

#### 执行任务

（1）使用 OneTimeWorkRequest.Builder 创建对象Worker，将任务加入WorkManager队列。并且OneTimeWorkRequest.Builder创建的是一个单次执行的任务。
（2）将任务排入WorkManager队列，等待执行。
Worker不一定是立即执行的。WorkManager会选择适当的时间运行Worker，平衡诸如系统负载，设备是否插入等考虑因素。但是如果我们没有指定任何约束条件，WorkManager会立即运行我们的任务。

```kotlin
 var request = OneTimeWorkRequest.Builder(UploadWorker::class.java).build()
 WorkManager.getInstance(this).enqueue(request)
```

#### 重复执行任务

（1）使用PeriodicWorkRequest.Builder类创建循环任务，创建一个PeriodicWorkRequest对象
（2）然后将任务添加到WorkManager的任务队列，等待执行。
（3）最小时间间隔是15分钟。

```kotlin
var pRequest = PeriodicWorkRequest.Builder(UploadWorker::class.java,1,TimeUnit.SECONDS).build()
WorkManager.getInstance(this).enqueue(pRequest)
```

#### 任务约束Constraints

WorkManager 允许我们指定任务执行的环境，比如网络已连接、电量充足时等，在满足条件的情况下任务才会执行。
（1）使用Constraints.Builder()创建并配置Constraints对象，可以指定上诉任务运行时间时的约束。
（2）创建Worker时调用setConstraints指定约束条件。

```kotlin
var constraint = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(true)
            .setRequiresCharging(true).build()
            
 var request = OneTimeWorkRequest.Builder(UploadWorker::class.java)
            .setConstraints(constraint)
            .build()
```

WorkManger提供了以下的约束作为Work执行的条件：
（1）setRequiredNetworkType：网络连接设置
（2）setRequiresBatteryNotLow：是否为低电量时运行 默认false
（3）setRequiresCharging：是否要插入设备（接入电源），默认false
（4）setRequiresDeviceIdle：设备是否为空闲，默认false
（5）setRequiresStorageNotLow：设备可用存储是否不低于临界阈值

#### 取消任务

（1）从WorkRequest()获取Worker的ID
（2 调用WorkManager.getInstance().cancelWorkById(workRequest.id)根据ID取消任务。
WorkManager 对于已经正在运行或完成的任务是无法取消任务的。

```kotlin
WorkManager.getInstance(this).cancelWorkById(request.id)
```

#### 添加TAG

通过为WorkRequest对象分配标记字符串来对任务进行分组

```kotlin
var twoRequest = OneTimeWorkRequest.Builder(UploadWorker::class.java)
            .setConstraints(constraint)
            .addTag("jetpack")
            .build()
```

WorkManager.getStatusesByTag() 返回该标记的所有任务的列表信息

```kotlin
WorkManager.getInstance(this).getWorkInfosByTag("jetpack")
```

WorkManager.cancelAllWorkByTag() 取消具有特定标记的所有任务

```kotlin
 WorkManager.getInstance(this).cancelAllWorkByTag("jetpack")
```

通过获取LiveData查看具有特定标记的所有任务的状态WorkInfo.State

```kotlin
WorkManager.getInstance(this).getWorkInfosByTagLiveData("jetpack").observe(this, Observer { 
            
        })
```

### 进阶使用

#### 数据交互

WorkManager可以将参数传递给任务，并让任务返回结果。传递和返回值都是键值对形式

(1) 使用 Data.Builder创建 Data 对象，保存参数的键值对。
(2) 在创建WorkQuest之前调用WorkRequest.Builder.setInputData()传递Data的实例

```kotlin
var requestData = Data.Builder().putString("jetpack", "workermanager").build()
var request = OneTimeWorkRequest.Builder(UploadWorker::class.java)
            .setConstraints(constraint)
            .setInputData(requestData)
            .build()
```

(3) 在JetpackWork.doWork方法中通过getInputData获取传递值。
(4) 在构建Data对象，跟随着任务的结果返回

```kotlin
class UploadWorker(context: Context,workerParameters: WorkerParameters) : Worker(context,workerParameters){
    override fun doWork(): Result {
        Log.e("workermanager","work start:"+inputData.getString("jetpack"))
        Thread.sleep(2_000)
        Log.e("workermanager","do work thread msg :"+Thread.currentThread().name)
        var outData = Data.Builder().putString("back","hi,jetpack").build()
        return Result.success(outData)
    }
}
```

(5) 通过LiveData监听Worker返回的数据

```kotlin
  WorkManager.getInstance(this).getWorkInfoByIdLiveData(request.id).observe(this, Observer {
            Log.e("workermanager", "out data :" + it?.outputData?.getString("back"))

        })
```

#### 链式任务

WorkManager允许拥有多个任务的工作序列按照顺序执行任务

(1) 使用该WorkManager.beginWith() 方法创建一个序列 ，并传递一个OneTimeWorkRequest对象;，该方法返回一个WorkContinuation对象。
(2) 使用 WorkContinuation.then()添加剩余的任务。
(3) 最后调用WorkContinuation.enqueue()将整个序列排入队列 。
如果中间有任何任务返回 Result.failure()，则整个序列结束。并且上一个任务的结果数据可以作为下一个任务的输入数据，实现任务之间的数据传递。

```kotlin
WorkManager.getInstance(this).beginWith(request).then(twoRequest).then(threeRequest).enqueue()
```

### 技术抛析

### 总结

[Goodel Developer](https://developer.android.google.cn/topic/libraries/architecture/workmanager)

[CSDN博客](https://blog.csdn.net/u011060103/article/details/99679972)