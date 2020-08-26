# Android Study Note
## Android Environment

## Android Usage

## Android Debug LOG
- Log.v("Johnson", "log content")
- Log.d()
- Log.i()
- Log.w()
- Log.e()

## Layout
- ConstraintLayout
 
## Activity

## Main Class

- Integer.toString(position)

### Set the applicaiton Name

<string name="app_name">EEducationOL</string>

### Modify the label

- Put the png pic to the related document directory:  app/src/main/res/
- AndroidManfest.xml: android:icon="@mipmap/theme"

### Rotate the Screen

<activity android:name=".VideoActivity" android:screenOrientation="sensor">

### Get the permission

```c
      public int REQUESTCODE =10;
      private String[] permissions = {Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.INTERNET};
        int i = ContextCompat.checkSelfPermission(this, permissions[0]);
        if (i != PackageManager.PERMISSION_GRANTED)
        {
            Log.i("Johnson","We don't have the possision");
            //showDialogTipUserRequestPermission();
            Toast.makeText(this, "We don't have permission", Toast.LENGTH_SHORT).show();
            ActivityCompat.requestPermissions(this, permissions, 10);
        }
    public void onRequestPermissionsResult(int requestCode,String permissions[], int[] grantResults) {
        switch (requestCode)
        {
            case 10:
            {
                if (grantResults.length > 0 )
                {
                    Toast.makeText(this, "We get the permission", Toast.LENGTH_SHORT).show();
                    Toast.makeText(this, "Start the video", Toast.LENGTH_SHORT).show();
                    Button play_button = (Button)findViewById(R.id.play_button);
                    StartView();
                }

            }
        }
    }
```

### Toast

```c
Toast.makeText(this, "O, we are given the permission", Toast.LENGTH_SHORT).show();
```
### VideoView

```c
        videoview = (VideoView)findViewById(R.id.videoView);
        //String path = Environment.getExternalStorageDirectory().getPath()+"/demo.mp4";
        Uri uri = Uri.parse("https://johnson37.github.io./demo.mp4");
        //Uri uri = Uri.parse(path);
        videoview.setMediaController(new MediaController(this));
        videoview.setVideoURI(uri);
        videoview.requestFocus();
        videoview.start();
```
### RadioGroup

```c
        RadioGroup mRadioGroup = (RadioGroup)findViewById(R.id.radiogroup);
        mRadioGroup.setOnCheckedChangeListener(new OnCheckedChangeListener(){
            public void onCheckedChanged(RadioGroup arg0, int arg1) {
                int radioButtonId = arg0.getCheckedRadioButtonId();
                RadioButton mRadioButton = (RadioButton)findViewById(radioButtonId);
                Log.i("Johnson", "Johnson test "+  mRadioButton.getText());
            }
        });
```

### ListView

```c
        ArrayAdapter<String> teacherAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, Teacher.getAllTeachers());
        ListView listView = (ListView) findViewById(R.id.teacher_listView);
        listView.setAdapter(teacherAdapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener(){
            public void onItemClick(AdapterView<?> parent, View view, int position, long id){
                String tmp_str = Integer.toString(position);
                List<String> temp_list  = Teacher.getAllTeachers();
                Log.i("Johnson", temp_list.get(position));
            }

        });
```

### Button

- Button set click event:

```c
        retButton.setOnClickListener(new View.OnClickListener(){
            public void onClick(View v)
            {
            }

        });
```
### Intent

```c
                Intent intent = new Intent();
                intent.putExtra("teacher", "Johnson");
                intent.setClass(MainActivity.this,TestActivity.class);
                startActivity(intent);
```
```c
                String teacher_name = getIntent().getStringExtra("teacher");
                TextView textview = (TextView)findViewById(R.id.textView);
                textview.setText(teacher_name);
```