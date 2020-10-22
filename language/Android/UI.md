# 基本的UI控件
Android的SDK定义了一个View类，他是所有Android控件和容器类的父类。
- View: Android所有控件的顶层基类。
- ViewGroup: View的子类，代表一个view的容器，可以存放其他View对象。
- TextView：是View的子类，用于展示基本的文本。

## Button Listener

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private Button reg_btn;
    private Button login_btn;
    public void getUIInterface()
    {
        reg_btn = (Button)findViewById(R.id.register_btn);
        reg_btn.setOnClickListener(this);
        login_btn = (Button)findViewById(R.id.login_btn);
        login_btn.setOnClickListener(this);
	}
	public void onClick(View v) {
        switch(v.getId())
        {
            case R.id.register_btn:
                Intent intent = new Intent(this, Register.class);
                startActivity(intent);
                break;
            case R.id.login_btn:
                Intent nav_intent = new Intent(this, NavActivity.class);
                startActivity(nav_intent);
                break;
        }
	}
}
```