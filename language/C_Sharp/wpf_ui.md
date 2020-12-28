# WPF Note
## Column/Row Definition.
```c

```

## 如何查看当前使用C#的版本。
```
C# 1.0 with Visual Studio.NET
C# 2.0 with Visual Studio 2005
C# 3.0 with Visual Studio 2008，2010（bai.net 2.0, 3.0, 3.5）du
C# 4.0 with Visual Studio 2010（.net 4）
C# 5.0 with Visual Studio 2012（.net 4.5），2013（.net 4.5.1, 4.5.2）
C# 6.0 with Visual Studio 2015（.net 4.6, 4.6.1, 4.6.2）
C# 7.0 with Visual Studio 2017（.net 4.6.2, 4.7）
```

## Layout
```c#
<Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="20" />
            <ColumnDefinition Width="auto" />
            <ColumnDefinition Width="auto" />
            <ColumnDefinition Width="auto" />
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="20" />
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="20"/>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="20"/>
        </Grid.RowDefinitions>    
        <TextBlock Grid.Column="1"  Grid.Row="1" Grid.ColumnSpan ="3" FontSize="18" Text="WPF Super Demo" Margin="0,0,0,10"/>
        <TextBlock Grid.Column="1" Grid.Row="2" FontWeight="Bold"
                   Text="username" />
        <TextBox x:Name="FirstName" Grid.Column="2" Grid.Row="2" Width="100" />
        <Button x:Name="SubmitButton" Grid.Column="1" Grid.Row="3" Grid.ColumnSpan ="2" Content="Run Me" Margin="10" Click="SubmitButton_Click"/>
        <ComboBox x:Name="myCombox" Grid.Column="1" Grid.Row="4" Grid.ColumnSpan="2" SelectionChanged="myCombox_SelectionChanged">
            <ComboBox.ItemTemplate>
                <DataTemplate>
                    <!--<TextBlock Text="{Binding FullNmae}" />-->
                    <StackPanel Orientation="Horizontal">
                        <Image Height="20" Width="20" Source="D:\Johnson.jpg" />
                        <TextBlock Text="{Binding FirstName}" />
                        <TextBlock Text=" " />
                        <TextBlock Text="{Binding LastName}" />

                    </StackPanel>

                </DataTemplate>
            </ComboBox.ItemTemplate>
        </ComboBox>
        
        <Image Grid.Row="1" Grid.Column="4" Grid.RowSpan="6" Source="D:\Johnson.jpg" />
    </Grid>
```

## 如何创建一个新的Win
Add--> Window

## Win show/showDialog
[Difference](https://www.cnblogs.com/uniquefrog/archive/2013/01/07/2849676.html)

## 如何完成窗口之间的切换。
**First Window, pass pointer to new Window**
```c#
Window1 _newWin = new Window1(this);
this.Hide();
_newWin.Show();
```

**Add one parameter in structure, **

```c#
        public Window1(Window parent)
        {
            InitializeComponent();
            this.parent_win = parent;
        }
        private void SubmitButton_Click(object sender, RoutedEventArgs e)
        {
            parent_win.Show();
            this.Close();
        }
```

## 如何最大化窗口
```c#
this.WindowState = WindowState.Maximized;
```

## 在CS文件中如何引用xaml中控件的Value
```c#
        private void LoginButton_Click(object sender, RoutedEventArgs e)
        {
            String s = String.Format("Username is {0}, and password is {1}", UserName.Text, this.Passwd.Text);
            MessageBox.Show(s);
        }
```

## 将code value to UI Control

### Example 1
```c#
        public User my_user = new User{ UserName = "Johnson" };
        public MainWindow()
        {
            InitializeComponent();
            
            //this.myBindData();
            this.WindowState = WindowState.Maximized;
            this.UserName.DataContext = my_user;
   
        }
        public class User : Class_Notify
        {
            private string username;
            public string UserName {
                get { return username;  }
                set{
                    username = value;
                    this.NotifyPropertyChanged("UserName");
                } 
            }
        }
        
```
```xml
<TextBox x:Name="UserName" Text="{Binding UserName}" Grid.Column="2" Grid.Row="2" MinWidth="100" Margin="15,10,10,10" HorizontalAlignment="Left" VerticalAlignment="Center"/>
```

### Example2
```c#

   public partial class MainWindow : Window
    {
        public User my_user = new User();// { UserName = "Johnson" };
        public void BindData()
        {
            my_user.UserName = "Johnson";
            Binding binding = new Binding();
            binding.Source = my_user;
            binding.Path = new PropertyPath("UserName");
            BindingOperations.SetBinding(this.UserName, TextBox.TextProperty, binding);
        }
        //public 
        public MainWindow()
        {
            InitializeComponent();
            this.BindData();
        }
    }
    public class User : INotifyPropertyChanged
    {
        private string username;
        public string UserName {
            get { return username;  }
            set{
                username = value;
                NotifyPropertyChanged("UserName");
            } 
        }
    
        public event PropertyChangedEventHandler PropertyChanged;

        private void NotifyPropertyChanged(string propertyName)
        {
            if(PropertyChanged != null)
            {
                if (PropertyChanged != null)
                {
                    PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
                }
            }
        }
    }
```


## Control UI

### PasswordBox