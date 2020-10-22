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
[Android Debug Log](./debug_log.md)

## Layout
[layout](./Layout.md)

## Component
Android中有四大组件，activity, service，content provider, broadcast receiver.
- Activity: 一个Activity就是一个窗口
- Service: Service组件通常用于为其他组件提供后台服务或监控其他组件的运行状态。
- Content provider: content provider 的主要作用是进程间通信，主要的应用场景是一个app中的数据会被其他app使用的情况，典型的case是通讯录，日历，短信等等。
- broadcast receiver: broadcast recevier的主要作用是对于外部事件的响应，比如说，一款音乐类的app，在听音乐过程中，如果有电话进来应该暂停音乐的播放等等。 

## Activity
[Activity](./activity.md)

## Service
[Service](./Service.md)

## UI
[UI](./UI.md)

## Anroid 语音识别


## Android SMS Sending
[SMS](./SMS.md)

## Misc
[misc](./misc.md)

