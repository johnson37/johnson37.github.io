## Service

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

