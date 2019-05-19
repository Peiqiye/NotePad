# 基于NotePad基本功能扩展
(1)显示NoteList时增加时间戳效果<br>
(2)查询功能（按名称查询）<br>
(3)UI美化<br>
(4)改变文本背景颜色<br>
(5)笔记导出到本地<br>
(6)对笔记进行排序<br>
   *按创建时间排序<br>
   *按修改时间排序<br>
   *按背景颜色排序<br>    
   
(7)选择字体大小与颜色<br>  
   

主要文件目录：<br>
图1图2:文件目录两张<br>
扩展功能详细介绍<br>
(1)显示NoteList时增加时间戳效果：在原有显示标题的基础下将时间戳显示出来。
在noteslist_item.xml中再新增一个显示时间的TextView。线性布局就选用垂直布局，显示在标题下方就可以。(在这里我将显示的字体设置为蓝色)<br>
代码如下：<br>

    <!--新增显示时间的TextView-->
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="50dp"

        android:gravity="center"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textColor="@android:color/holo_blue_dark"
        android:textSize="20sp" />
查看相应的数据库，发现原本定义的就有标题和时间。时间有创建时间和修改时间，显示的默认时间是创建时间，直到再次编辑文本才会更新显示修改时间。<br>
数据库定义在NotePadProvider.java:<br>

           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"    //便签id
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"     //便签标题
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"      //便签内容
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"   //创建的时间
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"   //修改的时间
                   
然后将修改时间填装到NotesList.java的PROJECTION中。  

             private static final String[] PROJECTION = new String[] {
                        NotePad.Notes._ID, // 0
                        NotePad.Notes.COLUMN_NAME_TITLE, // 1
                        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,  //加入修改时间的显示 
                };,
                
通过Cursor将数据从数据库中读出

    Cursor cursor = managedQuery(
                getIntent().getData(),            
                PROJECTION,                       
                null,                             
                null,                             
                NotePad.Notes.DEFAULT_SORT_ORDER  
            );
            
通过SimpleCursorAdapter装填数据，在dataColumns和viewIDs中加入时间。

    
    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE    //装配相应的日期
    } ;
    int[] viewIDs = { R.id.text1,R.id.text2 };
    SimpleCursorAdapter adapter
                = new MyCursorAdapter(
                          this,                             // The Context for the ListView
                          R.layout.noteslist_item,          // Points to the XML for a list item
                          cursor,                           // The cursor to get items from
                          dataColumns,
                          viewIDs
                  );

之后时间会显示为一串数字，还需进行格式转换，在NotePadProvider中的insert和NoteEditor中的updateNote加入时间格式转化代码。  

    Long now = Long.valueOf(System.currentTimeMillis());
             Date date=new Date(now);
             SimpleDateFormat format=new SimpleDateFormat("yyyy.MM.dd kk:mm:ss");
             format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
             String dateTime=format.format(date);
运行效果为：  
图片3  

(2)查询功能（按名称查询）  
首先在菜单栏增加搜索图标，点击图标进入搜索界面。在list_options_menu.xml增加一个item,搜索图标我用了自定义的图标，也可以用安卓自带的。  

    <item android:id="@+id/menu_search"
              android:icon="@drawable/search"
              android:title="搜索"
              android:showAsAction="always"/>

然后在NoteList中找到onOptionsItemSelected，添加点击搜索选项函数。  

    case R.id.menu_search:
                   Intent intent=new Intent();
                   intent.setClass(NotesList.this,NoteSearch.class);
                   NotesList.this.startActivity(intent);
                  //  startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
                    return true;
                    
接着新建布局文件note_search_list.xml：  

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="请输入搜索内容..."
        android:layout_alignParentTop="true">

    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

    </ListView>

    </LinearLayout>
    
创建完布局文件后，为搜索case写跳转activity，由于搜索出来的也是笔记列表，所以可以模仿NoteList的activity继承ListActivity。在安卓中有个用于搜索控件：SearchView，可以把SearchView跟ListView相结合，动态地显示搜索结果。  

    public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {
        private static final String[] PROJECTION=new String[]{
                NotePad.Notes._ID,
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    
        };
        @Override
        protected void onCreate(Bundle savedInstanceState){
            super.onCreate(savedInstanceState);
            setContentView(R.layout.note_search);
            Intent intent=getIntent();
            if(intent.getData()==null){
                intent.setData(NotePad.Notes.CONTENT_URI);
            }
            SearchView searchView=(SearchView)findViewById(R.id.search_view);
            //为查询文本框注册监听器
            searchView.setOnQueryTextListener(NoteSearch.this);
        }
        @Override
        public boolean onQueryTextSubmit(String query){
            return false;

    }
    @Override
    public boolean onQueryTextChange(String newText){
        String selection=NotePad.Notes.COLUMN_NAME_TITLE+" Like ?";
        String[] selectionArgs={"%"+newText+"%"};
        Cursor cursor=managedQuery(
                getIntent().getData(),
                PROJECTION,
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns={NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
        int[] viewIDs={R.id.text1,R.id.text2};
        SimpleCursorAdapter adapter=new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return  true;

    }
    @Override
    protected void onListItemClick(ListView l, View v, int position,long id){
        Uri uri=ContentUris.withAppendedId(getIntent().getData(),id);
        String action=getIntent().getAction();
        if(Intent.ACTION_PICK.equals(action)||Intent.ACTION_GET_CONTENT.equals(action)){
            setResult(RESULT_OK,new Intent().setData(uri));
        }else {
            startActivity(new Intent(Intent.ACTION_EDIT,uri));
        }
    }
}

onQueryTextSubmit和onQueryTextChange两个方法是实现接口必须写的方法。onListItemClick方法是点击NoteList的item跳转到对应笔记编辑界面的方法，NoteList中有这个方法，搜索出来的笔记跳转原理与NoteList中笔记一样，所以可以直接从NoteList中复制过来直接使用。  
最后要在AndroidManifest.xml中注册NoteSearch:  

    <activity android:name="NoteSearch"
                      android:theme="@android:style/Theme.Holo.Light"
                      android:label="@string/title_notes_search">
                <intent-filter>
                    <action android:name="android.intent.action.NoteSearch"/>
                    <action android:name="android.intent.action.SEARCH"/>
                    <action android:name="android.intent.action.SEARCH_LONG_PRESS"/>
                    <category android:name="android.intent.category.DELETE"/>
                    <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
                </intent-filter>

            </activity>

搜索演示：  

图片4  

(3)UI美化  
将安卓自带功能图标都换成自己的图标，将主体颜色换为白色。

(4)改变文本背景颜色<br>
创建数据库添加颜色字段  

    + NotePad.Notes.COLUMN_NAME_BACK_COLOR +" INTEGER"   //颜色*/

系统中预设颜色  

        public static final int DEFAULT_COLOR = 0; //白色
        public static final int YELLOW_COLOR = 1; //黄色
        public static final int BLUE_COLOR = 2; //蓝色
        public static final int GREEN_COLOR = 3; //绿色
        public static final int RED_COLOR = 4; //红色
   
在NotePadProvider中添加对背景颜色相应的处理，static{}中：  

    sNotesProjectionMap.put(
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR);
insert中也进行相应操作  

    if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
            }
            
新建布局文件note_color.xml  
  
      <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ImageButton
            android:id="@+id/color_white"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorWhite"
            android:onClick="white"/>
        <ImageButton
            android:id="@+id/color_yellow"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorYellow"
            android:onClick="yellow"/>
        <ImageButton
            android:id="@+id/color_blue"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorBlue"
            android:onClick="blue"/>
        <ImageButton
            android:id="@+id/color_green"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorGreen"
            android:onClick="green"/>
        <ImageButton
            android:id="@+id/color_red"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorRed"
            android:onClick="red"/>
    </LinearLayout>

自定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：  

    public class MyCursorAdapter extends SimpleCursorAdapter {
        public MyCursorAdapter(Context context, int layout, Cursor c,
                               String[] from,int[] to){
            super(context,layout,c,from,to);
        }
        @Override
        public void bindView(View view, Context context, Cursor cursor){
            super.bindView(view, context, cursor);
            //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
            int x= cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
             switch (x){
                 case NotePad.Notes.DEFAULT_COLOR:
                     view.setBackgroundColor(Color.rgb(255,255,255));
                     break;
                 case NotePad.Notes.YELLOW_COLOR:
                     view.setBackgroundColor(Color.rgb(247,216,133));
                     break;
                 case NotePad.Notes.BLUE_COLOR:
                     view.setBackgroundColor(Color.rgb(165,202,237));
                     break;
                 case NotePad.Notes.GREEN_COLOR:
                     view.setBackgroundColor(Color.rgb(161,214,174));
                     break;
                 case NotePad.Notes.RED_COLOR:
                     view.setBackgroundColor(Color.rgb(244,149,133));
                     break;
                 default:
                      view.setBackgroundColor(Color.rgb(255,255,255));
                      break;

             }
        }
    }

NoteList中的PROJECTION添加颜色项：  

    NotePad.Notes.COLUMN_NAME_BACK_COLOR,
将NoteList中用的SimpleCursorAdapter改使用MyCursorAdapter：  

    SimpleCursorAdapter adapter
            = new MyCursorAdapter(
                      this,                             // The Context for the ListView
                      R.layout.noteslist_item,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs
              );   
              
 在NoteEditor类中的onResume()方法中，将从数据库读取颜色并设置编辑界面背景色操作放入其中。  
 
     int X=mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
            switch (X){
                case NotePad.Notes.DEFAULT_COLOR:
                    mText.setBackgroundColor(Color.rgb(255,255,255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:
                    mText.setBackgroundColor(Color.rgb(247,216,133));
                    break;
                case NotePad.Notes.BLUE_COLOR:
                    mText.setBackgroundColor(Color.rgb(165,202,237));
                    break;
                case NotePad.Notes.GREEN_COLOR:
                    mText.setBackgroundColor(Color.rgb(161,214,174));
                    break;
                case NotePad.Notes.RED_COLOR:
                    mText.setBackgroundColor(Color.rgb(244,149,133));
                    break;
                default:
                    mText.setBackgroundColor(Color.rgb(255,255,255));
                    break;

            }
 
editor_options_menu.xml中增加颜色盘的图标。  

    <item android:id="@+id/menu_color"
              android:icon="@drawable/color1"
              android:title="@string/menu_color"
              android:showAsAction="always" />

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：  

    case R.id.menu_color:
                 changeColor();
                 break;

 在NoteEditor中添加函数changeColor()：  
 
     //颜色跳转函数
    private final void changeColor(){
        Intent intent=new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);

    }  
    
创建NoteColor的Acitvity，用来选择颜色。在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式：  

    public class NoteColor extends Activity {
        private Cursor mCursor;
        private Uri mUri;
        private int color;
        private static final int COLUMN_INDEX_TITLE = 1;
        private static final String[] PROJECTION = new String[]{
                NotePad.Notes._ID,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        };
        public void onCreate(Bundle savedInstanceState){
            super.onCreate(savedInstanceState);
            setContentView(R.layout.note_color);
            //从NoteEditor传入的uri
            mUri = getIntent().getData();
            mCursor = managedQuery(
                    mUri,
                    PROJECTION,
                    null,
                    null,
                    null
            );
        }
        @Override
        protected void onResume(){
            if(mCursor!=null){
                mCursor.moveToFirst();
                color=mCursor.getInt(COLUMN_INDEX_TITLE);
            }
            super.onResume();
        }
        @Override
        protected void onPause(){
            super.onPause();
            ContentValues values = new ContentValues();
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR,color);
            getContentResolver().update(mUri,values,null,null);
        }
        public void white(View view){
            color = NotePad.Notes.DEFAULT_COLOR;
            finish();
        }

    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }

    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }
    }
  
  AndroidManifest.xml:  
  
    <activity android:name="NoteColor"
                android:theme="@android:style/Theme.Holo.Light.Dialog"
                android:label="ChangeColor"
                android:windowSoftInputMode="stateVisible"/>    

演示效果：  
图5  

(5)笔记导出到本地<br>
在菜单栏editor_options_menu.xml中添加导出选项  

    <item android:id="@+id/menu_output"
              android:title="@string/menu_output"/>

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：  

    case R.id.menu_output:
            outputNote();
            break;
在NoteEditor中添加函数outputNote()：  
 
      private final void outputNote() {
            Intent intent = new Intent(null,mUri);
            intent.setClass(NoteEditor.this,OutputText.class);
            NoteEditor.this.startActivity(intent);
        }

对导出文件界面进行布局，命名为output_text.xml  

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingLeft="6dip"
        android:paddingRight="6dip"
        android:paddingBottom="3dip">

    <EditText
        android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="25"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true"/>

    <Button
        android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk"/>

    </LinearLayout>
创建OutputText的Acitvity，用来选择颜色。  

      public class OutputText extends Activity {
          private static final String[] PROJECTION=new String[]{
                  NotePad.Notes._ID,
                  NotePad.Notes.COLUMN_NAME_TITLE,
                  NotePad.Notes.COLUMN_NAME_NOTE,
                  NotePad.Notes.COLUMN_NAME_CREATE_DATE,
                  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
          };
          //读取出的值放入这些变量
          private String TITLE;
          private String NOTE;
          private String CREATE_DATE;
          private String MODIFICATION_DATE;
          //读取该笔记信息
          private Cursor mCursor;
          //导出文件的名字
          private EditText mName;
          //NoteEditor传入的uri,用于从数据库查出该笔记
          private Uri mUri;
          //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
          private boolean flag = false;
          private static final int COLUMN_INDEX_TITLE = 1;
          public void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.output_text);
              mUri = getIntent().getData();
              mCursor = managedQuery(
                      mUri,        // The URI for the note that is to be retrieved.
                      PROJECTION,  // The columns to retrieve
                      null,        // No selection criteria are used, so no where columns are needed.
                      null,        // No where columns are used, so no where values are needed.
                      null         // No sort order is needed.
              );
              mName = (EditText) findViewById(R.id.output_name);
          }
          @Override
          protected void onResume(){
              super.onResume();
              if (mCursor != null) {
                  // The Cursor was just retrieved, so its index is set to one record *before* the first
                  // record retrieved. This moves it to the first record.
                  mCursor.moveToFirst();
                  //编辑框默认的文件名为标题，可自行更改
                  mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
              }
          }
          @Override
          protected void onPause() {
              super.onPause();
              if (mCursor != null) {
                  //从mCursor读取对应值
                  TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
                  NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
                  CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
                  MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
                  //flag在点击导出按钮时会设置为true，执行写文件
                  if (flag == true) {
                      write();
                  }
                  flag = false;
              }
          }
          public void OutputOk(View v){
              flag = true;
              finish();
          }
          private void write()
          {
              try
              {
                  // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
                  if (Environment.getExternalStorageState().equals(
                          Environment.MEDIA_MOUNTED)) {
                      // 获取SD卡的目录
                      File sdCardDir = Environment.getExternalStorageDirectory();
                      //创建文件目录
                      File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                      //写文件
                      PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                      ps.println(TITLE);
                      ps.println(NOTE);
                      ps.println("创建时间：" + CREATE_DATE);
                      ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                      ps.close();
                      Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
                  }
              }
              catch (Exception e)
              {
                  e.printStackTrace();
              }
          }
      }

AndroidManifest.xml中加入定义  

     <activity android:name=".OutputText"
              android:label="@string/output_name"
              android:theme="@android:style/Theme.Holo.Light.Dialog"
              android:windowSoftInputMode="stateVisible">

          </activity>

加上权限  

     <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS">

     </uses-permission>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"></uses-permission>

设置了一个flag用于判断是否是点击导出按钮，是点击按钮，则flag设置为true，可以进行文件导出，否则flag为默认的false，判断语句为假，不能执行到导出文件的操作。  
效果演示：  
图6  

(6)对笔记进行排序<br>
   *按创建时间排序<br>
   *按修改时间排序<br>
   *按背景颜色排序<br>   
   
list_options_menu.xml中添加：  

    <item
            android:id="@+id/menu_sort"
            android:title="@string/menu_sort"
            android:icon="@drawable/ic_menu_sort"
            android:showAsAction="always">
            <menu>
                <item
                    android:id="@+id/menu_sort1"
                    android:title="@string/menu_sort1"/>
                <item
                    android:id="@+id/menu_sort2"
                    android:title="@string/menu_sort2"/>
                <item
                    android:id="@+id/menu_sort3"
                    android:title="@string/menu_sort3"/>
            </menu>


        </item>  
        
在NoteList菜单switch下添加case：  

    //创建时间排序
        case R.id.menu_sort1:
            cursor = managedQuery(
                    getIntent().getData(),            
                    PROJECTION,                      
                    null,                          
                    null,                          
                    NotePad.Notes._ID 
                    );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
     //修改时间排序
        case R.id.menu_sort2:
            cursor = managedQuery(
                    getIntent().getData(),          
                    PROJECTION,                      
                    null,                            
                    null,                       
                    NotePad.Notes.DEFAULT_SORT_ORDER 
            );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
        //颜色排序
        case R.id.menu_sort3:
            cursor = managedQuery(
                    getIntent().getData(),
                    PROJECTION,      
                    null,       
                    null,       
                    NotePad.Notes.COLUMN_NAME_BACK_COLOR
                    );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
                    );
            setListAdapter(adapter);
            return true;  
            
效果演示：  

图7  

(7)选择字体大小与颜色<br>   
在editor_options_menu.xml中添加：  

     <item
            android:id="@+id/font_size"
            android:title="@string/font_size">
            <!--子菜单-->
            <menu>
                <!--定义一组单选菜单项-->
                <group>
                    <!--定义多个菜单项-->
                    <item
                        android:id="@+id/font_10"
                        android:title="@string/font10"
                        />

                <item
                    android:id="@+id/font_16"
                    android:title="@string/font16" />
                <item
                    android:id="@+id/font_20"
                    android:title="@string/font20" />
            </group>
        </menu>
    </item>

    <item
        android:title="@string/font_color"
        android:id="@+id/font_color"
        >
        <menu>
            <!--定义一组普通菜单项-->
            <group>
                <!--定义两个菜单项-->
                <item
                    android:id="@+id/red_font"
                    android:title="@string/red_title" />
                <item
                    android:title="@string/black_title"
                    android:id="@+id/black_font"/>
            </group>
        </menu>
    </item>

NteEditor.java中添加case:  

    case R.id.font_10:

                mText.setTextSize(20);
                Toast toast =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast.show();
                /*finish();*/
                break;

        case R.id.font_16:
                mText.setTextSize(32);
                Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast2.show();
               /* finish();*/
                break;
        case R.id.font_20:
                mText.setTextSize(40);
                Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast3.show();
                /*finish();*/
                break;
        case R.id.red_font:
                mText.setTextColor(Color.RED);
                Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast4.show();
                /*finish();*/
                break;
        case R.id.black_font:
                mText.setTextColor(Color.BLACK);
                Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast5.show();
               /* finish();*/
                break;
其他步骤与背景颜色类似，代码省略。
效果演示：  

图8  



















