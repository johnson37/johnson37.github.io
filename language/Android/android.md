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
 
## Activity
Activity 是Android中的一个重要概念，Activity代表一个页面。

### Create one Activity
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

## Main Class

- Integer.toString(position)

### Set the applicaiton Name

How to set application Name?


### Modify the label

- Put the png pic to the related document directory:  app/src/main/res/
- AndroidManfest.xml: android:icon="@mipmap/theme"
