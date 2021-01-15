# TextField Keyboard

## Text Field 按下后键盘会自动弹出。键盘类型的选择
```objc
    _textField.keyboardType = UIKeyboardTypeDefault;
    _textField.returnKeyType = UIReturnKeyDefault;

```

## TextField 键盘输入按下确认或者返回键，需要键盘隐藏，键盘隐藏不会自动
** 键盘的弹出和隐藏是ios的自动行为，ios中有firstResponder的概念，如果类似输入类的控件成为FirstResponder，键盘会自动弹出 **

** 同理，当输入类的控件不再成为FirstResponder，键盘会自动退出。 **

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

[scrollandTextField](https://stackoverflow.com/questions/37223709/using-scroll-view-with-autolayout-swift)

### 向上移动以及向下移动的时机
需要检测键盘的弹出事件以及回收事件。
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(keyBoardWillShow:) name:UIKeyboardWillShowNotification object:nil];
    [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(keyBoardWillHide:) name:UIKeyboardWillHideNotification object:nil];    // Do any additional setup after loading the view.
}

- (void)keyBoardWillShow:(NSNotification *)notification{
	//Move position
}

- (void)keyBoardWillHide:(NSNotification *)notification{
	//Move position
}
```


### 向上移动的距离
考虑两种比较极端的情况，textField比较靠上，当我们向上移动过多，textField可能会移出屏幕，textField比较靠下，当我们向上移动太少，textField可能依然被遮挡。

两种策略：1. 始终把textField移动到最上面。 2. 默认移动keyboard Height，需要判断是否textField比较靠上，太靠上则需要调整移动位置。

- contentInset: 把屏幕总尺寸调大
- contentOffset: 移动当前frame到屏幕的偏移量。

```objc

- (void)keyBoardWillShow:(NSNotification *)notification{
    //Step 1: get keyboard information.
    NSDictionary * info = [notification userInfo];
    CGSize kbSize = [[info objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue].size;

    //Step 2: get textField frame information.
    float textField_pos = _textField.frame.origin.y;

    //Step 3: We have two choice,
    //Choice 1: move textField to top, keyboard will always show.
    //Choice 2: move up for keyboard height, but if textField is top originally, it's possible move out of the screen.
    //Step 3.1

    //Step 3.2
    float offset = 0.0f;
    if (textField_pos > kbSize.height)
        offset = kbSize.height;
    else
        offset = textField_pos;

    UIEdgeInsets contentInsets = UIEdgeInsetsMake(0.0, 0.0, offset, 0.0);
    self.scrollview.contentInset = contentInsets;
    self.scrollview.scrollIndicatorInsets = contentInsets;
    self.scrollview.contentOffset = CGPointMake(self.scrollview.contentOffset.x, self.scrollview.contentOffset.y + offset);

}

- (void)keyBoardWillHide:(NSNotification *)notification{
    //UIEdgeInsets contentInsets = UIEdgeInsetsZero;
    self.scrollview.contentInset = UIEdgeInsetsZero;
    self.scrollview.scrollIndicatorInsets = UIEdgeInsetsZero;
}

```

### 适配多个textField的case。
拿到firstResponder，判断一下是不是textField，然后得到对应的位置。
```objc
#pragma mark setup textField
- (void)textFieldSetup:(UITextField *) textfield {
    textfield.keyboardType = UIKeyboardTypeDefault;
    textfield.returnKeyType = UIReturnKeyDefault;
    textfield.delegate = self;
}


#pragma mark Find First Responder
- (id)findFirstResponder{
    if (_viewInScroll.isFirstResponder)
        return _viewInScroll;

    for (UIView * subView in _viewInScroll.subviews){
        //NSLog(@"view is %@", subView);
        if([subView isFirstResponder]){
            return subView;
        }
    }
    return nil;
}



#pragma mark keyBoard Notification
- (void)keyBoardWillShow:(NSNotification *)notification{
    //Step 1: get keyboard information.
    NSDictionary * info = [notification userInfo];
    CGSize kbSize = [[info objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue].size;

    //Step 2: get textField frame information.
    UIView * view = [self findFirstResponder];
    if ([view class] != [UITextField class])
    {
        return;
    }
    float textField_pos = view.frame.origin.y;
    //float textField_pos = _textField.frame.origin.y;

    //Step 3: We have two choice,
    //Choice 1: move textField to top, keyboard will always show.
    //Choice 2: move up for keyboard height, but if textField is top originally, it's possible move out of the screen.
    //Step 3.1

    //Step 3.2
    float offset = 0.0f;
    if (textField_pos > kbSize.height)
        offset = kbSize.height;
    else
        offset = textField_pos;

    UIEdgeInsets contentInsets = UIEdgeInsetsMake(0.0, 0.0, offset, 0.0);
    self.scrollview.contentInset = contentInsets;
    self.scrollview.scrollIndicatorInsets = contentInsets;
    self.scrollview.contentOffset = CGPointMake(self.scrollview.contentOffset.x, self.scrollview.contentOffset.y + offset);

    //NSLog(@"keyboard will show");
}

```
