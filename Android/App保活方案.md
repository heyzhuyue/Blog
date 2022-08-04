## App保活方案

### 一像素Activity保活

创建1像素透明的Activity,监听屏幕点亮/屏幕关闭,在屏幕关闭的时候,开启1像素的Activity,屏幕点亮后,如果1像素Activity存在,销毁Activity

```plain
class OnePixelActivity : Activity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //设定一像素的activity
        val window = window
        window.setGravity(Gravity.START or Gravity.TOP)
        val params = window.attributes
        params.x = 0
        params.y = 0
        params.height = 1
        params.width = 1
        window.attributes = params
    }

    override fun onResume() {
        super.onResume()
        checkScreenOn()
    }

    /**
     * 检测屏幕是否常亮
     */
    private fun checkScreenOn() {
        try {
            val pm =
                applicationContext.getSystemService(Context.POWER_SERVICE) as PowerManager
            val isScreenOn = pm.isScreenOn
            if (isScreenOn) {
                finish()
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```



```plain
class ScreenOnOffReceiver : BroadcastReceiver() {
    /**
     * Handler
     */
    private val mHandler: Handler = Handler(Looper.getMainLooper())

    /**
     * 屏幕是否亮着
     */
    private var screenOn = true

    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_SCREEN_OFF) {
            //屏幕关闭的时候接受到广播
            screenOn = false
            mHandler.postDelayed({
                if (!screenOn) {
                    val onePxIntent = Intent(context, OnePixelActivity::class.java)
                    onePxIntent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
                    onePxIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
                    val pendingIntent =
                        PendingIntent.getActivity(context, 0, onePxIntent, 0)
                    try {
                        pendingIntent.send()
                    } catch (e: Exception) {
                        e.printStackTrace()
                    }
                }
            }, 1000)
            //通知屏幕已关闭，开始播放无声音乐
            context.sendBroadcast(Intent("_ACTION_SCREEN_OFF"))
        } else if (intent.action == Intent.ACTION_SCREEN_ON) {
            //屏幕打开的时候发送广播  结束一像素
            screenOn = true
            //通知屏幕已点亮，停止播放无声音乐
            context.sendBroadcast(Intent("_ACTION_SCREEN_ON"))
        }
    }

}
```



### 双进程守护



开启2个Service服务,分别位于2个进程中，通过bindService进行互相唤起保活



```plain
class LocalService : Service() {
    /**
     * 一像素Receiver
     */
    private var mScreenOnOffReceiver: ScreenOnOffReceiver? = null

    /**
     * 屏幕关闭/开启Receiver
     */
    private var screenStateReceiver: ScreenStateReceiver? = null

    /**
     * 控制暂停
     */
    private var isPause = true

    /**
     * 媒体播放器
     */
    private var mMediaPlayer: MediaPlayer? = null

    /**
     * MyBilder
     */
    private var mBilder: MyBilder? = null

    /**
     * Handler
     */
    private var mHandler: Handler? = null

    /**
     * 是否绑定了守护进程
     */
    private var mIsBoundRemoteService = false

    /**
     * ServiceConnection
     */
    private val connection: ServiceConnection = object : ServiceConnection {
        override fun onServiceDisconnected(name: ComponentName) {
            if (isServiceRunning(
                    applicationContext,
                    " keeplive.service.LocalService"
                )
            ) {
                val remoteService = Intent(
                    this@LocalService,
                    RemoteService::class.java
                )
                startService(remoteService)
                val intent = Intent(this@LocalService, RemoteService::class.java)
                mIsBoundRemoteService = this@LocalService.bindService(
                    intent, this,
                    Context.BIND_ABOVE_CLIENT
                )
            }
            val pm =
                applicationContext.getSystemService(Context.POWER_SERVICE) as PowerManager
            val isScreenOn = pm.isScreenOn
            if (isScreenOn) {
                sendBroadcast(Intent("_ACTION_SCREEN_ON"))
            } else {
                sendBroadcast(Intent("_ACTION_SCREEN_OFF"))
            }
        }

        override fun onServiceConnected(name: ComponentName, service: IBinder) {
            try {
                if (mBilder != null && KeepLive.foregroundNotification != null) {
                    val guardAidl = GuardAidl.Stub.asInterface(service)
                    guardAidl.wakeUp(
                        KeepLive.foregroundNotification?.title,
                        KeepLive.foregroundNotification?.content,
                        KeepLive.foregroundNotification?.icon!!
                    )
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    override fun onCreate() {
        super.onCreate()
        if (mBilder == null) {
            mBilder = MyBilder()
        }
        val pm =
            applicationContext.getSystemService(Context.POWER_SERVICE) as PowerManager
        isPause = pm.isScreenOn
        if (mHandler == null) {
            mHandler = Handler()
        }
    }

    override fun onBind(intent: Intent): IBinder? {
        return mBilder
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        if (KeepLive.useSilenceMusic) {
            //播放无声音乐
            if (mMediaPlayer == null) {
                mMediaPlayer = MediaPlayer.create(this, R.raw.novioce)
                if (mMediaPlayer != null) {
                    mMediaPlayer!!.setVolume(0f, 0f)
                    mMediaPlayer!!.setOnCompletionListener {
                        if (!isPause) {
                            if (KeepLive.runMode == RunMode.ROGUE) {
                                play()
                            } else {
                                if (mHandler != null) {
                                    mHandler!!.postDelayed({ play() }, 5000)
                                }
                            }
                        }
                    }
                    mMediaPlayer!!.setOnErrorListener { mp, what, extra -> false }
                    play()
                }
            }
        }

        //像素保活
        if (mScreenOnOffReceiver == null) {
            mScreenOnOffReceiver = ScreenOnOffReceiver()
        }
        val onePxIntentFilter = IntentFilter()
        onePxIntentFilter.addAction("android.intent.action.SCREEN_OFF")
        onePxIntentFilter.addAction("android.intent.action.SCREEN_ON")
        registerReceiver(mScreenOnOffReceiver, onePxIntentFilter)

        //屏幕点亮状态监听，用于单独控制音乐播放
        if (screenStateReceiver == null) {
            screenStateReceiver = ScreenStateReceiver()
        }
        val screenIntentFilter = IntentFilter()
        screenIntentFilter.addAction("_ACTION_SCREEN_OFF")
        screenIntentFilter.addAction("_ACTION_SCREEN_ON")
        registerReceiver(screenStateReceiver, screenIntentFilter)

        //启用前台服务，提升优先级
        if (KeepLive.foregroundNotification != null) {
            val notificationIntent =
                Intent(applicationContext, NotifyClickReceiver::class.java)
            notificationIntent.action = NotifyClickReceiver.CLICK_NOTIFICATION
            val notification =
                AppFrontNotification.Builder(this).title(KeepLive.foregroundNotification?.title!!)
                    .content(KeepLive.foregroundNotification?.content!!)
                    .icon(KeepLive.foregroundNotification?.icon!!)
                    .intent(notificationIntent)
                    .build()
            startForeground(13691, notification)
        }

        //绑定守护进程
        try {
            val bindServiceIntent = Intent(this@LocalService, RemoteService::class.java)
            mIsBoundRemoteService = this.bindService(
                bindServiceIntent,
                connection!!,
                Context.BIND_ABOVE_CLIENT
            )
        } catch (e: Exception) {
            e.printStackTrace()
        }

        //隐藏服务通知
        try {
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N_MR1) {
                //Android7.1.1
                startService(Intent(this, HideForegroundService::class.java))
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
        if (KeepLive.keepLiveService != null) {
            KeepLive.keepLiveService?.onWorking()
        }
        return START_STICKY
    }

    override fun onDestroy() {
        super.onDestroy()
        if (connection != null) {
            try {
                if (mIsBoundRemoteService) {
                    unbindService(connection)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
        try {
            unregisterReceiver(mScreenOnOffReceiver)
            unregisterReceiver(screenStateReceiver)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        if (KeepLive.keepLiveService != null) {
            KeepLive.keepLiveService?.onStop()
        }
    }

    /**
     * 开启声音
     */
    private fun play() {
        if (KeepLive.useSilenceMusic) {
            if (mMediaPlayer != null && !mMediaPlayer!!.isPlaying) {
                mMediaPlayer!!.start()
            }
        }
    }

    /**
     * 关闭声音
     */
    private fun pause() {
        if (KeepLive.useSilenceMusic) {
            if (mMediaPlayer != null && mMediaPlayer!!.isPlaying) {
                mMediaPlayer!!.pause()
            }
        }
    }

    /**
     * 屏幕关闭/开启Receiver
     */
    private inner class ScreenStateReceiver : BroadcastReceiver() {
        override fun onReceive(
            context: Context,
            intent: Intent
        ) {
            if (intent.action == "_ACTION_SCREEN_OFF") {
                isPause = false
                play()
            } else if (intent.action == "_ACTION_SCREEN_ON") {
                isPause = true
                pause()
            }
        }
    }

    /**
     * Service IBinder
     */
    private class MyBilder : GuardAidl.Stub() {
        override fun wakeUp(
            title: String,
            description: String,
            iconRes: Int
        ) {
        }
    }
}
```



```plain
class RemoteService : Service() {
    /**
     * MyBilder
     */
    private var mBilder: MyBilder? = null

    /**
     * 是否绑定主服务
     */
    private var mIsBoundLocalService = false

    /**
     * ServiceConnection
     */
    private val connection: ServiceConnection? = object : ServiceConnection {
        override fun onServiceDisconnected(name: ComponentName) {
            if (isRunningTaskExist(
                    applicationContext,
                    "$packageName:remote"
                )
            ) {
                val localService = Intent(
                    this@RemoteService,
                    LocalService::class.java
                )
                startService(localService)
                mIsBoundLocalService = this@RemoteService.bindService(
                    Intent(
                        this@RemoteService,
                        LocalService::class.java
                    ), this, Context.BIND_ABOVE_CLIENT
                )
            }
            val pm =
                this@RemoteService.getSystemService(Context.POWER_SERVICE) as PowerManager
            val isScreenOn = pm.isScreenOn
            if (isScreenOn) {
                sendBroadcast(Intent("_ACTION_SCREEN_ON"))
            } else {
                sendBroadcast(Intent("_ACTION_SCREEN_OFF"))
            }
        }

        override fun onServiceConnected(
            name: ComponentName,
            service: IBinder
        ) {
        }
    }

    override fun onCreate() {
        super.onCreate()
        if (mBilder == null) {
            mBilder = MyBilder()
        }
    }

    override fun onBind(intent: Intent): IBinder? {
        return mBilder
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        try {
            mIsBoundLocalService = this.bindService(
                Intent(this@RemoteService, LocalService::class.java),
                connection!!, Context.BIND_ABOVE_CLIENT
            )
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return START_STICKY
    }

    override fun onDestroy() {
        super.onDestroy()
        if (connection != null) {
            try {
                if (mIsBoundLocalService) {
                    unbindService(connection)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    private inner class MyBilder : GuardAidl.Stub() {
        override fun wakeUp(
            title: String,
            description: String,
            iconRes: Int
        ) {
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N_MR1) {
                //Android7.1.1
                val notifyIntent = Intent(applicationContext, NotifyClickReceiver::class.java)
                notifyIntent.action = NotifyClickReceiver.CLICK_NOTIFICATION
                val notification =
                    AppFrontNotification.Builder(this@RemoteService)
                        .title(KeepLive.foregroundNotification?.title!!)
                        .content(KeepLive.foregroundNotification?.content!!)
                        .icon(KeepLive.foregroundNotification?.icon!!)
                        .intent(notifyIntent)
                        .build()
                this@RemoteService.startForeground(13691, notification)
            }
        }
    }
}
```



### JobService保活



JobService也是一个service，和普通的service不同的是，JobService是一个任务回调类，通过JobScheduler设置任务给系统，系统来调用JobService中的方法，具体处理什么任务需要我们自己在JobService中的回调方法中实现。那么关于任务的管理和进程的维护、调度当然是由系统来统一管理。



```plain
@RequiresApi(api = 21)
class JobHandlerService : JobService() {

    companion object {
        const val JOB_ID = 100
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        this.startService(this)
        if (VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            //Android5.0
            val mJobScheduler = this.getSystemService(Context.JOB_SCHEDULER_SERVICE) as JobScheduler
            mJobScheduler.cancel(JOB_ID)
            val builder = JobInfo.Builder(
                JOB_ID,
                ComponentName(this.packageName, JobHandlerService::class.java.name)
            )
            if (VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                //Android7.0
                builder.setMinimumLatency(30000L) //经过30000毫秒后再开始执行任务
                builder.setOverrideDeadline(30000L) //设置最长等待时间，即使其他条件未满足，经过30000毫秒后任务都会执行
                builder.setBackoffCriteria(30000L, JobInfo.BACKOFF_POLICY_LINEAR) //设置重试机制的时间策略
            } else {
                builder.setPeriodic(30000L) //经过30000毫秒后再开始执行任务
            }
            builder.setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY) //设置网络情况为任何网络
            mJobScheduler.schedule(builder.build())
        }
        return START_STICKY
    }

    override fun onStartJob(jobParameters: JobParameters): Boolean {
        if (!isServiceRunning(
                this.applicationContext,
                "com.zy.keeplive.service.service.LocalService"
            )
            || !isRunningTaskExist(
                this.applicationContext,
                this.packageName + ":remote"
            )
        ) {
            this.startService(this)
        }
        return false
    }

    override fun onStopJob(jobParameters: JobParameters): Boolean {
        if (!isServiceRunning(
                this.applicationContext,
                "com.zy.keeplive.service.service.LocalService"
            )
            || !isRunningTaskExist(
                this.applicationContext,
                this.packageName + ":remote"
            )
        ) {
            this.startService(this)
        }
        return false
    }

    /**
     * 开启服务
     *
     * @param context 上下问
     */
    private fun startService(context: Context) {
        if (VERSION.SDK_INT >= Build.VERSION_CODES.O && KeepLive.foregroundNotification != null) {
            //Android8.0 不允许开通后台服务，只允许开通前台服务
            val notifyIntent =
                Intent(this.applicationContext, NotifyClickReceiver::class.java)
            notifyIntent.action = "CLICK_NOTIFICATION"
            val notification =
                AppFrontNotification.Builder(this).title(KeepLive.foregroundNotification?.title!!)
                    .content(KeepLive.foregroundNotification?.content!!)
                    .icon(KeepLive.foregroundNotification?.icon!!)
                    .intent(notifyIntent)
                    .build()
            this.startForeground(13691, notification)
        }
        val localIntent = Intent(context, LocalService::class.java)
        val remoteIntent = Intent(context, RemoteService::class.java)
        if (VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            this.startForegroundService(localIntent)
            this.startService(remoteIntent)
        } else {
            this.startService(localIntent)
            this.startService(remoteIntent)
        }

    }
}
```