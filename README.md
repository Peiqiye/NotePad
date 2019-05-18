# 基于NotePad基本功能扩展
(1)显示NoteList时增加时间戳效果<br>
(2)查询功能（按名称查询）<br>
(3)UI美化<br>
(4)改变文本背景颜色<br>
(5)改变文本字体大小与颜色<br>
(6)笔记导出到本地<br>
(7)对笔记进行排序<br>
   *按创建时间排序<br>
   *按修改时间排序<br>
   *按背景颜色排序<br>

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



