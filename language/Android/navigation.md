# Navigation

使用BottomNavigationView 

## Basic Navigation

### Step 1
xml 中添加BottomNavigationView 控件。

```
    <TextView
        android:id="@+id/message"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="@dimen/activity_horizontal_margin"
        android:layout_marginLeft="@dimen/activity_horizontal_margin"
        android:layout_marginTop="@dimen/activity_vertical_margin"
        android:text="@string/title_home"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
		
    <com.google.android.material.bottomnavigation.BottomNavigationView

        android:id="@+id/navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="0dp"
        android:layout_marginEnd="0dp"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/navigation" />
```

### Step 2 BottomNavigationView 控件中指定了menu信息。
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/navigation_home"
        android:icon="@drawable/home"
        android:title="@string/title_home"
        />
    <item
        android:id="@+id/navigation_dashboard"
        android:icon="@drawable/help"
        android:title="@string/Camera" />

    <item
        android:id="@+id/navigation_notifications"
        android:icon="@drawable/help"
        android:title="@string/title_notifications" />
</menu>
```

### Step 3:
```java
public class Style1Activity extends AppCompatActivity {

    private TextView mTextMessage;

    private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
            = new BottomNavigationView.OnNavigationItemSelectedListener() {

        @Override
        public boolean onNavigationItemSelected(@NonNull MenuItem item) {
            switch (item.getItemId()) {
                case R.id.navigation_home:
                    mTextMessage.setText(R.string.title_home);
                    return true;
                case R.id.navigation_dashboard:
                    mTextMessage.setText(R.string.title_dashboard);
                    return true;
                case R.id.navigation_notifications:
                    mTextMessage.setText(R.string.title_notifications);
                    return true;
            }
            return false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_style1);

        mTextMessage = (TextView) findViewById(R.id.message);
        BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);
        navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
    }

}
```

该例子可实现导航栏，该导航栏的问题是每个导航页里面都是一个text，没有定制化，如果想定制化，最好分成几个fragment。

## Navigation + Fragment

### Step 1
```
    <include
        layout="@layout/content_main"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/navigation"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        />

    <!-- Bottom Navigation View -->
    <com.google.android.material.bottomnavigation.BottomNavigationView

        android:id="@+id/navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="0dp"
        android:layout_marginEnd="0dp"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/navigation" />
```
### Step 2
contain_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:context="MainActivity"
    tools:showIn="@layout/activity_main"
    android:padding="1dp"
    android:id="@+id/main_container">

</FrameLayout>
```

### Step 3
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/navigation_home"
        android:icon="@drawable/home"
        android:title="@string/title_home"
        />
    <item
        android:id="@+id/navigation_dashboard"
        android:icon="@drawable/help"
        android:title="@string/Camera" />

    <item
        android:id="@+id/navigation_notifications"
        android:icon="@drawable/help"
        android:title="@string/title_notifications" />
</menu>
```

### Step 4
Add some fragment.

### Step 5
```java
public class NavActivity extends AppCompatActivity {
    private TextView mTextMessage;

    private Fragment homefrag = new HomeFragment();
    private Fragment dashfrag = new Dashfrag();
    private Fragment Notifyfrag = new NotifyFrag();
    private Fragment active = homefrag;
    final FragmentManager fm = getSupportFragmentManager();
    private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
            = new BottomNavigationView.OnNavigationItemSelectedListener() {

        @Override
        public boolean onNavigationItemSelected(@NonNull MenuItem item) {
            switch (item.getItemId()) {
                case R.id.navigation_home:
                     fm.beginTransaction().hide(active).show(homefrag).commit();
                    active = homefrag;
                    Log.i("Johnson", "Enter title home");
                    //replaceFragment(new HomeFragment());
                    return true;
                case R.id.navigation_dashboard:
                    fm.beginTransaction().hide(active).show(dashfrag).commit();
                    active = dashfrag;
                    Log.i("Johnson", "Enter DashBoard");
                    return true;
                case R.id.navigation_notifications:
                    fm.beginTransaction().hide(active).show(Notifyfrag).commit();
                    Log.i("Johnson", "Enter Notification");
                    active = Notifyfrag;
                    return true;
            }
            return false;
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_nav);
        fm.beginTransaction().add(R.id.main_container, Notifyfrag, "3").hide(Notifyfrag).commit();
        fm.beginTransaction().add(R.id.main_container, dashfrag, "2").hide(dashfrag).commit();
        fm.beginTransaction().add(R.id.main_container,homefrag, "1").commit();
        mTextMessage = (TextView) findViewById(R.id.message);
        BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);
        navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
    }
}

```