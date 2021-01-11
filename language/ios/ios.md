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

## alert
[alert](./alert.md)

## Different between viewDidLoad and viewDidAppear
viewDidLoad is called exactly once, when the view controller is first loaded into memory. This is where you want to instantiate any instance variables and build any views that live for the entire lifecycle of this view controller. However, the view is usually not yet visible at this point.
viewDidAppear is called when the view is actually visible, and can be called multiple times during the lifecycle of a View Controller (for instance, when a Modal View Controller is dismissed and the view becomes visible again). This is where you want to perform any layout actions or do any drawing in the UI - for example, presenting a modal view controller. However, anything you do here should be repeatable. It's best not to retain things here, or else you'll get memory leaks if you don't release them when the view disappears.

## variable property
!(variable property)[https://medium.com/bitmountn/attributes-of-property-nonatomic-retain-strong-weak-etc-b7ea93a0f772]

## selector
@selector是指查找本类以及本类子类中的方法
@selector(method)时，这个method只有在该方法存在参数时需要 ":"，如果该方法不需要参数就不需要加这个冒号。

```objc
@implementation ClassForSelectors
- (void) fooNoInputs {
    NSLog(@"Does nothing");
}
- (void) fooOneIput:(NSString*) first {
    NSLog(@"Logs %@", first);
}
- (void) fooFirstInput:(NSString*) first secondInput:(NSString*) second {
    NSLog(@"Logs %@ then %@", first, second);
}
- (void) performMethodsViaSelectors {
    [self performSelector:@selector(fooNoInputs)];
    [self performSelector:@selector(fooOneInput:) withObject:@"first"];
    [self performSelector:@selector(fooFirstInput:secondInput:) withObject:@"first" withObject:@"second"];
}
@end

```

## UIControl state
[UIControl state](./uiControlState.md)