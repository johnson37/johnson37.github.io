# tableview Usage

## storyboard usage

### step1 add tableview in viewcontrller

### step2 
在tableview中按住左键，拖动到viewController，选中datasource和delegate

### step3 编码显示

#### ViewController.h 文件
```objc
#import <UIKit/UIKit.h>

@interface ViewController : UIViewController<UITableViewDelegate, UITableViewDataSource>

@end
```

#### ViewController.m

##### 定义全局变量arrayData，用于装我们要显示在列表上的内容文字

##### 在viewDidLoad中，对数组变量进行初始化

##### 实现代理方法
```objc
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong) NSArray *arrayData;
@end

@implementation ViewController
@synthesize arrayData;

- (void)viewDidLoad {
[super viewDidLoad];
// Do any additional setup after loading the view, typically from a nib.
arrayData = [NSArray arrayWithObjects:@"王小虎",@"郭二牛",@"宋小六",@"耿老三",@"曹大将军", nil];
}

#pragma mark -- delegate方法
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
return arrayData.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
static NSString *indentifier = @"cell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:indentifier];

if (!cell) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:indentifier];
}

cell.textLabel.text = [arrayData objectAtIndex:indexPath.row];

return cell;
}

- (void)didReceiveMemoryWarning {
[super didReceiveMemoryWarning];
// Dispose of any resources that can be recreated.
}

@end
```
