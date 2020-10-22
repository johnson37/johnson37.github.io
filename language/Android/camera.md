# Camera 

## Android 拍照
APP 进行拍照分成两类方法，
- 调用应用自带的拍照app来进行拍照
- 设置自己的拍照功能。设置自己的拍照功能更灵活一点。

### 设置自己的拍照功能。
- Camera.open(), Camera类的open是一个old api，因为没有参数，所以默认打开后置的第一个摄像头。
- Camera.open(id), Camera类的带参数open，可以指定打开哪个摄像头。
- Camera.setPreviewDisplay(holder)


## Xml
```
        <SurfaceView
            android:id = "@+id/camera_surfaceView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <Button
            android:id="@+id/camera_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="30dp"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            android:layout_alignParentBottom="true"
            android:layout_marginBottom="20dp"
            android:background="@drawable/circle"
            app:layout_constraintVertical_bias="0.9"
         />
```

## Code

### Camera 自动调焦

```c
void autoFocus(Camera.AutoFocusCallback cb)
``` 
Starts camera auto-focus and registers a callback function to run when the camera is focused. 即完成自动调焦后的回调函数。

### Camera preview
```c
    surfaceView = findViewById(R.id.camera_surfaceView);
    surfaceHolder = surfaceView.getHolder();
    surfaceHolder.addCallback(new CameraSurfaceCallBack());
```

```c
    public class CameraSurfaceCallBack implements SurfaceHolder.Callback {
        
        @Override
        public void surfaceCreated(SurfaceHolder holder) {
		}
		
		@Override
        public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
            Log.i(TAG,"------surfaceChanged------");
            initCamera(holder);
        }

        @Override
        public void surfaceDestroyed(SurfaceHolder holder) {

        }
	}
```

### Source Code

```java
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        // Setup any handles to view objects here
        // EditText etFoo = (EditText) view.findViewById(R.id.etFoo);
        surfaceView = view.findViewById(R.id.camera_surfaceView);
        surfaceHolder = surfaceView.getHolder();
        surfaceHolder.addCallback(new CameraSurfaceCallBack());

        btn_takePhoto = view.findViewById(R.id.camera_button);
        btn_takePhoto.setOnClickListener(new BtnTakePhotoListener());
    }
	    /**
     * 实现拍照功能
     */
    public void takePhoto(){
        Camera.Parameters parameters;
        //mCamera = Camera.open();
        try{
            parameters = mCamera.getParameters();
        }catch(Exception e){
            Log.i(TAG, "We failed get Parameters?");
            e.printStackTrace();
            return;
        }
        Log.i("Johnson", "After getParameters");
        //获取摄像头支持的各种分辨率,因为摄像头数组不确定是按降序还是升序，这里的逻辑有时不是很好找得到相应的尺寸
        //可先确定是按升还是降序排列，再进对对比吧，我这里拢统地找了个，是个不精确的...
        List<Camera.Size> list = parameters.getSupportedPictureSizes();
        int size = 0;
        for (int i =0 ;i < list.size() - 1;i++){
            if (list.get(i).width >= 480){
                //完美匹配
                size = i;
                break;
            }
            else{
                //找不到就找个最接近的吧
                size = i;
            }
        }
        Log.i(TAG, "set picture size");
        //设置照片分辨率，注意要在摄像头支持的范围内选择
        parameters.setPictureSize(list.get(size).width,list.get(size).height);
        //设置照相机参数
        mCamera.setParameters(parameters);
        //使用takePicture()方法完成拍照
        mCamera.autoFocus(new myAutoFocusCallback());
    }
    private class myAutoFocusCallback implements Camera.AutoFocusCallback
    {
        @Override
        public void onAutoFocus(boolean success, Camera camera) {
            if (success && camera != null){
                //mCamera.takePicture(new ShutterCallback(), null,  new JpegPictureCallback());
                mCamera.takePicture(null, null, new JpegPictureCallback());
            }
        }
    }
    private boolean hasExternalStoragePermission(Context context) {
        int permission = context.checkCallingOrSelfPermission("android.permission.WRITE_EXTERNAL_STORAGE");
        //PERMISSION_GRANTED=0
        return permission == 0;
    }
    private File getOwnCacheDirectory(Context context, String cacheDir) {
        File appCacheDir = null;
        //判断SD卡正常挂载并且拥有根限的时候创建文件
        if(Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) &&
                hasExternalStoragePermission(context)){
            appCacheDir = new File(Environment.getExternalStorageDirectory(),cacheDir);
        }
        if (appCacheDir == null || !appCacheDir.exists() && !appCacheDir.mkdirs()){
            appCacheDir = context.getCacheDir();
        }
        return appCacheDir;
    }
    private String setPicSaveFile(){
        //创建保存的路径
        File storageDir = getOwnCacheDirectory(getContext(),"MyCamera/photos");
        //返回自定义的路径
        return storageDir.getPath();
    }

    public void savePhoto(byte[] data){
        FileOutputStream fos = null;
        String timeStamp =new SimpleDateFormat("yyyy-MM-dd-HH-mm-ss").format(new Date());
        //保存路径+图片名字
        String imagePath = setPicSaveFile() + "/" + timeStamp + ".png";
        Log.i(TAG, "Saving picture in: "+ imagePath);
        try{
            fos = new FileOutputStream(imagePath);
            fos.write(data);
            //清空缓冲区数据
            fos.flush();
            //关闭
            fos.close();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            Toast.makeText(getContext(),"拍照成功!",Toast.LENGTH_SHORT).show();
        }
    }
    private class JpegPictureCallback implements Camera.PictureCallback
    {
        @Override
        public void onPictureTaken(byte[] data, Camera camera) {
            savePhoto(data);
            //停止预览
            mCamera.stopPreview();
            //重启预览
            mCamera.startPreview();
        }
    }
    /**
     * 拍照按钮事件回调
     */
    public class BtnTakePhotoListener implements View.OnClickListener {
        @Override
        public void onClick(View v) {
            Log.i("Johnson","拍照按钮事件回调");
            takePhoto();
        }
    }
    /**
     * 查找摄像头
     *
     * @param camera_facing 按要求查找，镜头是前还是后
     * @return -1表示找不到
     */
    private int findBackOrFrontCamera(int camera_facing) {
        int cameraCount = 0;
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        cameraCount = Camera.getNumberOfCameras();
        for (int camIdx = 0; camIdx < cameraCount; camIdx++) {
            Camera.getCameraInfo(camIdx, cameraInfo);
            if (cameraInfo.facing == camera_facing) {
                return camIdx;
            }
        }
        return -1;
    }
    /**
     * 按照type的类型打开相应的摄像头
     *
     * @param type 标志当前打开前还是后的摄像头
     * @return 返回当前打开摄像机的对象
     */
    private Camera openCamera(int type) {
        int cameraCount = Camera.getNumberOfCameras();

        Camera.CameraInfo info = new Camera.CameraInfo();
        for (int cameraIndex = 0; cameraIndex < cameraCount; cameraIndex++) {
            Camera.getCameraInfo(cameraIndex, info);

            if (info.facing == currentCameraType) {
                Log.i("Johnson", "we will open camera now....");
                return Camera.open(cameraIndex);
            }
        }
        return null;
    }
    public class CameraSurfaceCallBack implements SurfaceHolder.Callback {

        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            Log.i(TAG,"------surfaceCreated------");
            try {
                //这里我优先找后置摄像头,找不到再找前面的
                int cameraIndex = findBackOrFrontCamera(Camera.CameraInfo.CAMERA_FACING_BACK);
                if (cameraIndex == -1) {
                    cameraIndex = findBackOrFrontCamera(Camera.CameraInfo.CAMERA_FACING_FRONT);
                    if (cameraIndex == -1) {
                        currentCameraType = CAMERA_NOTEXIST;
                        currentCameraIndex = -1;
                        Log.i(TAG, "Find no camera");
                        return;
                    } else {
                        currentCameraType = Camera.CameraInfo.CAMERA_FACING_FRONT;
                        Log.i(TAG, "Find one front camera");
                    }
                } else {
                    currentCameraType = Camera.CameraInfo.CAMERA_FACING_BACK;
                    Log.i(TAG, "Find one black camera");
                }

                //找到想要的摄像头后，就打开
                if (mCamera == null) {
                    Log.i(TAG, "open camera type: "+ currentCameraType);
                    //mCamera = Camera.open();
                    if (ContextCompat.checkSelfPermission(getContext(), android.Manifest.permission.CAMERA)!= PackageManager.PERMISSION_GRANTED){
                        ActivityCompat.requestPermissions((Activity)getActivity(),new String[]{android.Manifest.permission.CAMERA},1);
                        mCamera = openCamera(currentCameraType);
                        Log.i(TAG, "Camera open over....");
                    }else {
                        //mCamera = Camera.open(0);
                        mCamera = openCamera(currentCameraType);
                        Log.i(TAG, "Camera open over....");
                    }
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        /**
         * 设置旋转角度
         * @param activity
         * @param cameraId
         * @param camera
         */
        private void setCameraDisplayOrientation(Activity activity,int cameraId,Camera camera){
            Camera.CameraInfo info = new Camera.CameraInfo();
            Camera.getCameraInfo(cameraId,info);
            int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();
            int degrees = 0;
            switch(rotation){
                case Surface.ROTATION_0:
                    degrees = 0;
                    break;
                case Surface.ROTATION_90:
                    degrees = 90;
                    break;
                case Surface.ROTATION_180:
                    degrees = 180;
                    break;
                case Surface.ROTATION_270:
                    degrees = 270;
                    break;
            }
            int result;
            if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT){
                result = (info.orientation + degrees) % 360;
                result = (360 - result) % 360;
            }else{
                result = (info.orientation - degrees +360) % 360;
            }
            camera.setDisplayOrientation(result);
        }

        /**
         * 初始化摄像头
         * @param holder
         */
        private void initCamera(SurfaceHolder holder){
            Log.i(TAG,"initCamera");
            if (mPreviewRunning)
                mCamera.stopPreview();

            Camera.Parameters parameters;
            try{
                //获取预览的各种分辨率
                parameters = mCamera.getParameters();
            }catch (Exception e){
                e.printStackTrace();
                return;
            }
            //这里我设为480*800的尺寸
            parameters.setPreviewSize(480,800);
            // 设置照片格式
            parameters.setPictureFormat(PixelFormat.JPEG);
            //设置图片预览的格式
            parameters.setPreviewFormat(PixelFormat.YCbCr_420_SP);
            setCameraDisplayOrientation(getActivity(),0,mCamera);
            try{
                mCamera.setPreviewDisplay(holder);
            }catch(Exception e){
                if(mCamera != null){
                    mCamera.release();
                    mCamera = null;
                }
                e.printStackTrace();
            }
            mCamera.startPreview();
            mPreviewRunning = true;
        }
        @Override
        public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
            Log.i(TAG,"------surfaceChanged------");
            initCamera(holder);
        }

        @Override
        public void surfaceDestroyed(SurfaceHolder holder) {

        }
    }
```

## Camera Demo From Ethernet
```java
package com.shizhuayuan.mycamera;

import android.app.Activity;
import android.content.Context;
import android.content.pm.ActivityInfo;
import android.graphics.PixelFormat;
import android.hardware.Camera;
import android.media.MediaPlayer;
import android.os.Bundle;
import android.os.Environment;
//import android.support.annotation.Nullable;
import android.util.Log;
import android.view.Surface;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;
import android.view.WindowManager;
import android.widget.Button;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

/**
 * Created by lzh on 2018/7/2.
 */

public class CameraActivity extends Activity {

    private static SurfaceView surfaceView;               //图像实时窗口
    private Button btn_takePhoto;                         //拍照按钮
    private SurfaceHolder surfaceHolder;                  //定义访问surfaceView的接口

    private static final int CAMERA_NOTEXIST = -1;        //无摄像头标记
    private static final int FRONT = Camera.CameraInfo.CAMERA_FACING_FRONT;                   //前置摄像头标记
    private static final int BACK = Camera.CameraInfo.CAMERA_FACING_BACK;                    //后置摄像头标记
    private int currentCameraType = CAMERA_NOTEXIST;      //当前打开摄像头标记
    private int currentCameraIndex = -1;                  //当前摄像头下标
    private boolean mPreviewRunning = false;              //预览是否启动
    private String TAG = "Johnson";
    private Camera mCamera = null;                        //Camera对象

    View olderSelectView = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);//竖屏
        setContentView(R.layout.activity_main);
        //设置屏幕常亮
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        initView();
    }

    /**
     * 初始化
     */
    private void initView() {
        surfaceView = findViewById(R.id.camera_surfaceView);
        surfaceHolder = surfaceView.getHolder();
        surfaceHolder.addCallback(new CameraSurfaceCallBack());

        btn_takePhoto = findViewById(R.id.btn_takePhoto);
        btn_takePhoto.setOnClickListener(new BtnTakePhotoListener());
    }

    /**
     * 查找摄像头
     *
     * @param camera_facing 按要求查找，镜头是前还是后
     * @return -1表示找不到
     */
    private int findBackOrFrontCamera(int camera_facing) {
        int cameraCount = 0;
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        cameraCount = Camera.getNumberOfCameras();
        for (int camIdx = 0; camIdx < cameraCount; camIdx++) {
            Camera.getCameraInfo(camIdx, cameraInfo);
            if (cameraInfo.facing == camera_facing) {
                return camIdx;
            }
        }
        return -1;
    }

    /**
     * 按照type的类型打开相应的摄像头
     *
     * @param type 标志当前打开前还是后的摄像头
     * @return 返回当前打开摄像机的对象
     */
    private Camera openCamera(int type) {
        int cameraCount = Camera.getNumberOfCameras();

        Camera.CameraInfo info = new Camera.CameraInfo();
        for (int cameraIndex = 0; cameraIndex < cameraCount; cameraIndex++) {
            Camera.getCameraInfo(cameraIndex, info);

            if (info.facing == currentCameraType) {
                return Camera.open(cameraIndex);
            }
        }
        return null;
    }

    /**
     * 初始化摄像头
     * @param holder
     */
    private void initCamera(SurfaceHolder holder){
        Log.i(TAG,"initCamera");
        if (mPreviewRunning)
            mCamera.stopPreview();

        Camera.Parameters parameters;
        try{
            //获取预览的各种分辨率
            parameters = mCamera.getParameters();
        }catch (Exception e){
            e.printStackTrace();
            return;
        }
        //这里我设为480*800的尺寸
        parameters.setPreviewSize(480,800);
        // 设置照片格式
        parameters.setPictureFormat(PixelFormat.JPEG);
        //设置图片预览的格式
        parameters.setPreviewFormat(PixelFormat.YCbCr_420_SP);
        setCameraDisplayOrientation(this,0,mCamera);
        try{
            mCamera.setPreviewDisplay(holder);
        }catch(Exception e){
            if(mCamera != null){
                mCamera.release();
                mCamera = null;
            }
            e.printStackTrace();
        }
        mCamera.startPreview();
        mPreviewRunning = true;
    }

    /**
     * 设置旋转角度
     * @param activity
     * @param cameraId
     * @param camera
     */
    private void setCameraDisplayOrientation(Activity activity,int cameraId,Camera camera){
        Camera.CameraInfo info = new Camera.CameraInfo();
        Camera.getCameraInfo(cameraId,info);
        int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();
        int degrees = 0;
        switch(rotation){
            case Surface.ROTATION_0:
                degrees = 0;
                break;
            case Surface.ROTATION_90:
                degrees = 90;
                break;
            case Surface.ROTATION_180:
                degrees = 180;
                break;
            case Surface.ROTATION_270:
                degrees = 270;
                break;
        }
        int result;
        if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT){
            result = (info.orientation + degrees) % 360;
            result = (360 - result) % 360;
        }else{
            result = (info.orientation - degrees +360) % 360;
        }
        camera.setDisplayOrientation(result);
    }

    /**
     * 实现拍照功能
     */
    public void takePhoto(){
        Camera.Parameters parameters;
        //mCamera = Camera.open();
        try{
            parameters = mCamera.getParameters();
        }catch(Exception e){
            Log.i(TAG, "We failed get Parameters?");
            e.printStackTrace();
            return;
        }
        Log.i("Johnson", "After getParameters");
        //获取摄像头支持的各种分辨率,因为摄像头数组不确定是按降序还是升序，这里的逻辑有时不是很好找得到相应的尺寸
        //可先确定是按升还是降序排列，再进对对比吧，我这里拢统地找了个，是个不精确的...
        List<Camera.Size> list = parameters.getSupportedPictureSizes();
        int size = 0;
        for (int i =0 ;i < list.size() - 1;i++){
            if (list.get(i).width >= 480){
                //完美匹配
                size = i;
                break;
            }
            else{
                //找不到就找个最接近的吧
                size = i;
            }
        }
        Log.i(TAG, "set picture size");
        //设置照片分辨率，注意要在摄像头支持的范围内选择
        parameters.setPictureSize(list.get(size).width,list.get(size).height);
        //设置照相机参数
        mCamera.setParameters(parameters);

        //使用takePicture()方法完成拍照
        mCamera.autoFocus(new myAutoFocusCallback());
    }
    private class myAutoFocusCallback implements Camera.AutoFocusCallback
    {
        @Override
        public void onAutoFocus(boolean success, Camera camera) {
            if (success && camera != null){
                //mCamera.takePicture(new ShutterCallback(), null,  new JpegPictureCallback());
                mCamera.takePicture(null, null, new JpegPictureCallback());
            }
        }
    }
    private class JpegPictureCallback implements Camera.PictureCallback
    {
        @Override
        public void onPictureTaken(byte[] data, Camera camera) {
            savePhoto(data);
            //停止预览
            mCamera.stopPreview();
            //重启预览
            mCamera.startPreview();
        }
    }

   /*
     * 快门回调接口，如果不想拍照声音，直接将new ShutterCallback()修改为null即可
     */
    // 这个地方就是调用Media Player 播放一段音频，来模拟快门按下的声音。
    private class ShutterCallback implements Camera.ShutterCallback {
        @Override
        public void onShutter() {
            MediaPlayer mPlayer = new MediaPlayer();
            mPlayer = MediaPlayer.create(getApplicationContext(), R.raw.shutter);
            try{
                mPlayer.prepare();
            }catch (IllegalStateException e){
                e.printStackTrace();
            }catch (IOException e){
                e.printStackTrace();
            }
            mPlayer.start();
        }
    }

    /**
     * 将拍照保存下来
     * @param data
     */
    public void savePhoto(byte[] data){
        FileOutputStream fos = null;
        String timeStamp =new SimpleDateFormat("yyyy-MM-dd-HH-mm-ss").format(new Date());
        //保存路径+图片名字
        String imagePath = setPicSaveFile() + "/" + timeStamp + ".png";
        Log.i(TAG, "Saving picture in: "+ imagePath);
        try{
            fos = new FileOutputStream(imagePath);
            fos.write(data);
            //清空缓冲区数据
            fos.flush();
            //关闭
            fos.close();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            Toast.makeText(this,"拍照成功!",Toast.LENGTH_SHORT).show();
        }
    }

    /**
     * 设置照片的路径，具体路径可自定义
     * @return
     */
    private String setPicSaveFile(){
        //创建保存的路径
        File storageDir = getOwnCacheDirectory(this,"MyCamera/photos");
        //返回自定义的路径
        return storageDir.getPath();
    }

    private File getOwnCacheDirectory(Context context, String cacheDir) {
        File appCacheDir = null;
        //判断SD卡正常挂载并且拥有根限的时候创建文件
        if(Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) &&
                hasExternalStoragePermission(context)){
            appCacheDir = new File(Environment.getExternalStorageDirectory(),cacheDir);
        }
        if (appCacheDir == null || !appCacheDir.exists() && !appCacheDir.mkdirs()){
            appCacheDir = context.getCacheDir();
        }
        return appCacheDir;
    }

    /**
     * 检查是否有权限
     * @param context
     * @return
     */
    private boolean hasExternalStoragePermission(Context context) {
        int permission = context.checkCallingOrSelfPermission("android.permission.WRITE_EXTERNAL_STORAGE");
        //PERMISSION_GRANTED=0
        return permission == 0;
    }


    /**
     * 拍照按钮事件回调
     */
    public class BtnTakePhotoListener implements View.OnClickListener {

        @Override
        public void onClick(View v) {
            Log.i("Johnson","拍照按钮事件回调");
            takePhoto();
        }
    }

    /*
     * surfaceView实例回调
     */
    public class CameraSurfaceCallBack implements SurfaceHolder.Callback {

        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            Log.i(TAG,"------surfaceCreated------");
            try {
                //这里我优先找后置摄像头,找不到再找前面的
                int cameraIndex = findBackOrFrontCamera(Camera.CameraInfo.CAMERA_FACING_BACK);
                if (cameraIndex == -1) {
                    cameraIndex = findBackOrFrontCamera(Camera.CameraInfo.CAMERA_FACING_FRONT);
                    if (cameraIndex == -1) {

                        currentCameraType = CAMERA_NOTEXIST;
                        currentCameraIndex = -1;
                        Log.i(TAG, "Find no camera");
                        return;
                    } else {
                        currentCameraType = Camera.CameraInfo.CAMERA_FACING_FRONT;
                        Log.i(TAG, "Find one front camera");
                    }
                } else {
                    currentCameraType = Camera.CameraInfo.CAMERA_FACING_BACK;
                    Log.i(TAG, "Find one black camera");
                }

                //找到想要的摄像头后，就打开
                if (mCamera == null) {
                    Log.i(TAG, "open camera type: "+ currentCameraType);
                    //mCamera = Camera.open();
                    mCamera = openCamera(currentCameraType);
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        @Override
        public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
            Log.i(TAG,"------surfaceChanged------");
            initCamera(holder);
        }

        @Override
        public void surfaceDestroyed(SurfaceHolder holder) {

        }
    }
}

```