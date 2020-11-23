# Navigation

## Navigation steps

### step 1
选中一个ViewController，Editor--> Embed in -> Navigation Bar, Xcode 会生成一个Navigation Controller的viewController，并跟起初的viewController
自动形成关联关系。

### step 2
为Navigation ViewController 新定义一个类。

### step 3
从Login 界面，设置调转到Navigation View Controller。

### step 4

起初生成NavigationViewController的viewcontroller作为Dashboard。从Dashboard通过正常方式指向新的viewcontroller，新的viewcontroller就会自动添加navigation bar。

### step 5

如果想从Dashboard页面切换回原来的login页面，需要添加一个navigation bar item，logout 
```objc
[self.presentingViewController dismissViewControllerAnimated: TRUE completiom:nil];
```

