# Debug Log

Usually, we will add one parameter for "Johnson", such as,
```java
private String TAG = "Johnson";
Log.v(TAG, "log content")
```

```java
- Log.v("Johnson", "log content")
- Log.d()
- Log.i()
- Log.w()
- Log.e()

```

## Log 添加参数

Android采用的是Java的语法，数字和string相加，会自动转换成String类型。
```c
Log.i("Johnson", "open camera type: "+ currentCameraType);
```