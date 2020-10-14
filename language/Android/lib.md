# Android Lib

## Source Resource
[resource](https://developer.android.com/studio/projects/android-library?hl=zh-cn)

## Create one Android Lib
- File -- New -- New Module
- Create New Module -- Android Library -- Next
- Finish

Note: 我们在OUTPUT文件夹中会找到编译之后生成的aar文件。

## Import one Android Lib

[Source](https://jingyan.baidu.com/article/2a13832890d08f074a134ff0.html)

- Copy aar to Lib dir
- Add in build.gradle
```
repositories   {
	flatDir {    
		dirs 'libs'    
			}
  }
```
- Modify denpendency
```
compile(name:'usericonchooserutil', ext:'aar')
```

Note: Android Studio 中找不到lib文件夹
```
sourceSets{

    main {
        //jni库的调用会到资源文件夹下libs里面找so文件
        jniLibs.srcDirs = ['libs']
    }

}
```