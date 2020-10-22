# Permission

权限是一种安全机制。Android权限主要用于限制应用程序内部某些具有限制性特性的功能使用以及应用程序之间的组件访问。
比如网络权限 <user-permission android:name="android.permission.INTERNET"/>当我们的程序需要访问网络的时候，必须添加这个权限，不添加则无法访问网络。

在Android6.0之前，我们只需要在AndroidManifest.xml文件中直接添加权限即可，但是在Android6.0之后，我们只在AndroidManifest.xml文件中配置是不够的，
还需要在Java代码中进行动态获取权限。当然这里不是所有的权限都需要动态获取，只需要获取危险权限就可以了。

** Note: 不是所有的权限都是需要动态申请的，只有重要的才需要动态申请**

![Permission](permission.png)

## Usage

### Step 1 请求授权
```java
 /**
     * 请求授权
     */
    private void requestPermission(){

        if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED){ //表示未授权时
            //进行授权
            ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.CALL_PHONE},1);
        }else{
            //调用打电话的方法
            makeCall();
        }
    }
```

### Step 2 我们需要重写onRequestPermissionsResult这个方法
```java
/**
 * 权限申请返回结果
 * @param requestCode 请求码
 * @param permissions 权限数组
 * @param grantResults  申请结果数组，里面都是int类型的数
 */
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    switch (requestCode) {
        case 1:
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){ //同意权限申请
                makeCall();
            }else { //拒绝权限申请
                Toast.makeText(this,"权限被拒绝了",Toast.LENGTH_SHORT).show();
            }
            break;
        default:
            break;
    }
}

```