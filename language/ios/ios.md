# IOS study

## Misc
https://github.com.cnpmjs.org
### textField 密码加密
textField has one parameter for secureTextEntry
```objc
UItextField * test = [ UItextField alloc] init ];
test.secureTextEntry = YES;
```

### textField 透明色
```objc
[_keyTextField setBackgroundColor:[UIColor clearColor]];//透明效果
```
### textFiled 边框颜色
```objc
_keyTextField.layer.borderColor = [UIColor whiteColor].CGColor;//边框颜色
```
### textField hint
ios storyboard中有一个textField有一个placeHolder字段。

### textField 键盘的选择
textField 根据输入的内容的不同，可以在attribute中选择不同的键盘类型。

### textField 弹出键盘的同时，如何将整个界面往上移动

### 获得控件相对屏幕的位置
```objc
UIWindow *window = [[[UIApplication sharedApplication] delegate] window];
CGRect rect=[_searchTextField convertRect: self.searchTextField.bounds toView:window];
```

### tableview的用法
[tableview](./tableview.md)

### ViewController change
```swift
        let vc = storyboard?.instantiateViewController(withIdentifier: "second")
        print(type(of: vc))
        if let myvc = vc {
            print("myvc type is : ")
            print(type(of: myvc))
            let secondvc = myvc as! SecondViewController
            secondvc.parameter = "hello, this is one parameter"
            present(secondvc, animated: true)
        }
        else{
            let temp_vc = SecondViewController()
            present(temp_vc, animated: true)
        }
```
### cocoapods 添加库
[cocoapods](./cocoapods.md)

### Navigation Bar
[navigation bar](./navigation.md)

### 如何根据不同情况选择不同的启动页面
AppDelegate.m 
#### step 1. storyboard
```objc
UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
```

#### step2 . choose ViewController and set ViewController to rootViewController

```objc
	FavorViewController * favorBoardViewController = [storyboard instantiateViewControllerWithIdentifier:@"FavorViewController"];
	self.window.rootViewController = favorBoardViewController;
```

## protocol and delegate
protocol 等同于java中的interface

我们一个新的类，可以继承不同的protocol，与此同时，我们需要在我们的类中添加对应的implementation，implementation就是delegate。

一个protocol可能会定义很多方法，protocol的定义中设置了optional和required，optional的方法，我们可以根据情况不实现，但是required的方法是必须要实现的。


## set Icon
![Logo Template](https://www.canva.cn)
![Enlarger](https://www.photoenlarger.com/)
![Generate App Icon](https://appicon.co/)

