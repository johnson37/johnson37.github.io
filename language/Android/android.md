# Android Study Note

## Android Study Project 
Android Project上有两个主要的文件夹，app和gradle。
- app: 源代码。app中有libs/src 两个文件夹。一般情况源码都在src中。src中源码主要在main中。main中包含java和res两个文件夹和AndroidManifest.xml.
- gradle: 用于管理三方库和打包使用。

![Android Code Structure](./pic1.PNG)

### Manifest.xml Style
Manifest 主框架是一个application，application下有application的属性，application就是我们整个应用的系统设置，application下有各个activity，activity
 的level等同于页面的level。
#### Android 设置名字
```c
<application
android:label="@string/app_name"
</application>

```
在res values string.xml 中储存了字符串string
```c
<resources>
    <string name="app_name">EEducationOL</string>
</resources>
```

#### Android 设置图标
```c
<application
	android:icon="@mipmap/theme"
</application>
```

#### AndroidManifest.xml Example
```c
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.johnsonz.myapplication">

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/theme"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".TestActivity" />
        <activity
            android:name=".VideoActivity"
            android:screenOrientation="sensor" />
        <activity android:name=".SurfaceVideo" />
        <activity android:name=".socket"></activity>
    </application>

</manifest>
```

## Android Debug LOG
- Log.v("Johnson", "log content")
- Log.d()
- Log.i()
- Log.w()
- Log.e()

## Layout
- ConstraintLayout

## Component
Android中有四大组件，activity, service，content provider, broadcast receiver.
- Activity: 一个Activity就是一个窗口
- Service: Service组件通常用于为其他组件提供后台服务或监控其他组件的运行状态。
- Content provider: content provider 的主要作用是进程间通信，主要的应用场景是一个app中的数据会被其他app使用的情况，典型的case是通讯录，日历，短信等等。
- broadcast receiver: broadcast recevier的主要作用是对于外部事件的响应，比如说，一款音乐类的app，在听音乐过程中，如果有电话进来应该暂停音乐的播放等等。 

### Activity
Activity 是Android中的一个重要概念，Activity代表一个页面。

#### Create one Activity
** new --> Activity --> xxx Activity**
在xxx activity中设置activity的名字，以及对应的layout。对应的layout会默认生成在res-- layout -- xxx.xml

新创的activity会默认创建onCreate Function.
```c
public class test extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test2);
    }
}
```

#### Android main activity
一个Android应用会有多个activity页面，页面之间是通过跳转完成的，但是Android需要一个入口Activity。通常情况下， Android Studio在
project中创建的第一个activity就是主Activity。Android Studio是通过下面的语法实现的：

- android.intent.action.MAIN：决定应用的入口Activity，也就是我们启动应用时首先显示哪一个Activity。
- android.intent.category.LAUNCHER：表示activity应该被列入系统的启动器(launcher)(允许用户启动它)。Launcher是安卓系统中的桌面启动器，是桌面UI的统称。
```c
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

#### Android Activity States
- running: 可见，可交互
- pause: 可见，不可交互
- stopped: 不可见，不可交互
- killed: activity处于暂停或停止状态，若内存不足，从内存中删除；

#### Activity switch
- 生成一个意图对象 Intent
- 调用setClass方法设置所要启动的Activity
- 调用startActivity方法启动Activity

```c
    public void onClick(View v) {
        Intent intent = new Intent();
        //intent.putExtra("teacher", "Johnson");
        intent.setClass(MainActivity.this,socket.class);
        startActivity(intent);
    }
```
Activity Switch中activity的活动周期，原activity切换成pause状态，并压入activity的堆栈，新的activity create出来并成为running状态。当我们回退的时候，
新的activity会被destroy掉，原来成为pause的activity重新成为running状态。

![activity_switch](./activity.png)

### Service

Service主要实现一些不需要进行交互的内容，比如说网络请求，文件读写，数据库操作等等。
Service有两种实现方式，一种是startservice，另外一种是bindservice。
- Startservice: 主进程start service之后，主进程跟service之间不再有任何关系。即便退出应用，service还依然存活。
- bindservice: activity 对service进行了绑定之后，多个activity可以对service进行绑定，可以调用service中的api获取一些信息。当所有activity都unbind service，service就down了。

#### start service
```c
public void onClick(View v) {
    TextView hellotv = (TextView)findViewById(R.id.textView2);
    hellotv.setText(R.string.interact_message);
    //This is an example of startservice
    Intent intent = new Intent(MainActivity.this, MyService.class);
    switch(v.getId())
    {
        case R.id.button2:
            startService(intent);
            break;
        case R.id.button3:
            Log.i("Johnson", "Activity plan to stop service");
            stopService(intent);
            break;
    }
}
	
public class MyService extends Service {
    public MyService() {
    }

    public void onCreate()
    {
        Log.i("Johnson","Johnson create");
        super.onCreate();
    }
    public void onDestroy()
    {
        Log.i("Johnson","Johnson Destroy");
        super.onDestroy();
    }
    public int onStartCommand(Intent intent, int flags, int startId)
    {
        Log.i("Johnson","Johnson on StartCommand");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

### bind service

#### Service
Service中要初始化一个Binder，通过Binder能够获取到该Service，从而调用该service的api。
```c
public class bind_Service extends Service {
    public bind_Service() {
    }
    public class MyBinder extends Binder {

        public bind_Service getService(){
            return bind_Service.this;
        }

    }
    //通过binder实现调用者client与Service之间的通信
    private MyBinder binder = new MyBinder();

    private final Random generator = new Random();

    @Override
    public void onCreate() {
        Log.i("Johnson","TestService -> onCreate, Thread: " + Thread.currentThread().getName());
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i("Johnson", "TestService -> onStartCommand, startId: " + startId + ", Thread: " + Thread.currentThread().getName());
        return START_NOT_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        Log.i("Johnson", "TestService -> onBind, Thread: " + Thread.currentThread().getName());
        return binder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Log.i("Johnson", "TestService -> onUnbind, from:" + intent.getStringExtra("from"));
        return false;
    }

    @Override
    public void onDestroy() {
        Log.i("Johnson", "TestService -> onDestroy, Thread: " + Thread.currentThread().getName());
        super.onDestroy();
    }

    //getRandomNumber是Service暴露出去供client调用的公共方法
    public int getRandomNumber(){
        return generator.nextInt();
    }
}


```
#### Activity bind service.

Activity 需要ServiceConnection 去获取binder，进而获取到service。
```c
    private boolean isBound = false;
    private bind_Service.MyBinder myBinder = null;
    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder binder) {
            isBound = true;
            myBinder = (bind_Service.MyBinder)binder;
            service = myBinder.getService();
            Log.i("Johnson", "ActivityA onServiceConnected");
            int num = service.getRandomNumber();
            Log.i("Johnson", "ActivityA 中调用 TestService的getRandomNumber方法, 结果: " + num);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            isBound = false;
            Log.i("Johnson", "ActivityA onServiceDisconnected");
        }
    };
	    public void onClick(View v) {
        TextView hellotv = (TextView)findViewById(R.id.textView2);
        hellotv.setText(R.string.interact_message);
        //This is an example of startservice
        //Intent intent = new Intent(MainActivity.this, MyService.class);
        Intent intent = new Intent(MainActivity.this, bind_Service.class);
        switch(v.getId())
        {
            case R.id.button2:
                //startService(intent);
                if (isBound) {
                    service = myBinder.getService();
                    Log.i("Johnson", "ActivityA onServiceConnected");
                    int num = service.getRandomNumber();
                    Log.i("Johnson", "ActivityA 中调用 TestService的getRandomNumber方法, 结果: " + num);
                }
                break;
            case R.id.button_bind:
                Log.i("Johnson", "Bind Service");
                bindService(intent, conn, BIND_AUTO_CREATE);
                break;
            case R.id.button_unbind:
                Log.i("Johnson", "Unbind Service");
                unbindService(conn);
                break;
        }
    }
```
### Service 和 Thread的选择

如果任务只在应用程序与用户有交互的情况下产生，并且任务很占时间或者会引起阻塞，则另起thread或者HandlerThread。
如果在应用程序与用户无交互的情况下仍需处理任务，那么用service，因为service是在后台运行的，同时仍要注意是否需要在service中另开thread。
