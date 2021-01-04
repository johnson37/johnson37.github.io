# ios alert

```objc
UIAlertController* alert = [UIAlertController alertControllerWithTitle:@"My Alert"
                               message:@"This is an alert."
                               preferredStyle:UIAlertControllerStyleAlert];
 
UIAlertAction* defaultAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault
   handler:^(UIAlertAction * action) {}];
 
[alert addAction:defaultAction];
[self presentViewController:alert animated:YES completion:nil];
```

## Note, alert 不能工作在viewDidLoad, 可以工作在viewDidAppear.
