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


