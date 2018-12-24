# Android Study Note
## Android Environment
## Android Usage
## Android Debug LOG
- Log.v("Johnson", "log content")
- Log.d()
- Log.i()
- Log.w()
- Log.e()

### Layout
- ConstraintLayout
- 
### Activity

### Main Class
- Integer.toString(position)

#### Set the applicaiton Name
<string name="app_name">EEducationOL</string>

#### Modify the label
- Put the png pic to the related document directory:  app/src/main/res/
- AndroidManfest.xml: android:icon="@mipmap/theme"

### Get the permission
```java
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

#### RadioGroup
```java
        RadioGroup mRadioGroup = (RadioGroup)findViewById(R.id.radiogroup);
        mRadioGroup.setOnCheckedChangeListener(new OnCheckedChangeListener(){
            public void onCheckedChanged(RadioGroup arg0, int arg1) {
                int radioButtonId = arg0.getCheckedRadioButtonId();
                RadioButton mRadioButton = (RadioButton)findViewById(radioButtonId);
                Log.i("Johnson", "Johnson test "+  mRadioButton.getText());
            }
        });
```
#### ListView
```java
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

#### Button
- Button set click event:
```java
        retButton.setOnClickListener(new View.OnClickListener(){
            public void onClick(View v)
            {
            }

        });
```
#### Intent
```java
                Intent intent = new Intent();
                intent.putExtra("teacher", "Johnson");
                intent.setClass(MainActivity.this,TestActivity.class);
                startActivity(intent);
```
```java
                String teacher_name = getIntent().getStringExtra("teacher");
                TextView textview = (TextView)findViewById(R.id.textView);
                textview.setText(teacher_name);
```

### Broadcast Receiver

### Service