# TextField Keyboard

## Text Field 按下后键盘会自动弹出。键盘类型的选择
```objc
    _textField.keyboardType = UIKeyboardTypeDefault;
    _textField.returnKeyType = UIReturnKeyDefault;

```

## TextField 键盘输入按下确认或者返回键，需要键盘隐藏，键盘隐藏不会自动
** 键盘的弹出和隐藏是ios的自动行为，ios中有firstResponder的概念，如果类似输入类的控件成为FirstResponder，键盘会自动弹出**
** 同理，当输入类的控件不再成为FirstResponder，键盘会自动退出。**

textField 有一个<UITextFieldDelegate> delegate.

UITextFieldDelegate有一个方法textFieldShouldReturn，当键盘按下returnkey，就会trigger这个方法，我们需要重新实现该方法，当按下returnkey，textField放弃做FirstResponder.

```objc
- (void)viewDidLoad {

    _textField.delegate = self;
}

	- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    [_textField resignFirstResponder];
    return YES;
}

```

## TextField 键盘弹出时，可能会遮挡textField，我们需要进行屏幕向上移动。
ios 官方推荐的方式是，viewControl主界面放一个scrollView，因为scrollview中只能放一个控件，放一个view，把所有需要的控件都放置到view中。

![scroll and TextField](https://stackoverflow.com/questions/37223709/using-scroll-view-with-autolayout-swift)