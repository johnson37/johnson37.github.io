# Misc

## Error Fix

### Didn't find class Error inflating class android.support.design.widget.BottomNavigationView

[Solution](https://stackoverflow.com/questions/45672547/didnt-find-class-error-inflating-class-android-support-design-widget-bottomnavi)

### Frame & Navigate

[Solution](https://medium.com/@oluwabukunmi.aluko/bottom-navigation-view-with-fragments-a074bfd08711)

### Check permission in fragments
[Solution](https://stackoverflow.com/questions/40760625/how-to-check-permission-in-fragment)
```java
if (ActivityCompat.checkSelfPermission(getContext(),
            android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED &&
            ActivityCompat.checkSelfPermission(getContext(),
                    android.Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
         requestPermissions(getActivity(),
                new String[]{android.Manifest.permission.ACCESS_COARSE_LOCATION,
                        android.Manifest.permission.ACCESS_FINE_LOCATION},
                REQUEST_LOCATION);
    } else {
        Log.e("DB", "PERMISSION GRANTED");
    }
```
#### How to use dp/sp for margin
sp for font sizes, dp for everything else. sp stands for Scale-independent Pixels, dp stands for dip=density independent pixels.

#### 如何去掉App上方的actionbar
在res-> style.xml, 修改AppTheme
Default:
```c
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
```
Modify
```c
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
```

主要去掉标题栏后，屏幕上方还会有一块长条，需要通过全屏的方式去掉。
** 全屏的设置要在setContentView之前。**
```c
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_main);
```

#### 如何降低图片的像素。
Win10中默认绘图软件--Edit -- Resize -- 调整。

#### EditText 如何使得密码不可见。
```c
android:inputType="textPassword"
```
