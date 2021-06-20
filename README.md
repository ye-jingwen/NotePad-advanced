# NotePad_timestamp
This is an AndroidStudio rebuild of google SDK sample NotePad
 
## 1.在layout包下的noteslist_item中添加显示时间的TextView：
        <TextView
                android:id="@+id/times"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:paddingLeft="5dip"
                android:textAppearance="?android:attr/textAppearanceSmall"
                android:textColor="@color/OrangeRed"
                android:textSize="15dp" />
![github](https://github.com/ye-jingwen/NotePad_timestamp/blob/master/Image/noteslist_item.png "github")
## 2.修改java文件中的NotesList
### （1）加上时间的显示
        private static final String[] PROJECTION = new String[] {
                    NotePad.Notes._ID, // 0
                    NotePad.Notes.COLUMN_NAME_TITLE, // 1
                    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 2 时间
            };
### （2）修改onCreate中的dataColumns和viewIDs为private
        private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        private int[] viewIDs = { android.R.id.text1 , R.id.times };
![github](https://github.com/ye-jingwen/NotePad_timestamp/blob/master/Image/NotesList.png "github")
## 3.在java文件中的NoteEditor中updateNote函数中添加获取时间的功能
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());

        //修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
![github](https://github.com/ye-jingwen/NotePad_timestamp/blob/master/Image/NoteEditor.png "github")
## 4.在java文件中的NotePadProvider中insert函数中添加与NoteEditor相同的获取时间的功能
        ContentValues values = new ContentValues();
        
        //修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
![github](https://github.com/ye-jingwen/NotePad_timestamp/blob/master/Image/NotePadProvider.png "github")
## 5.运行截图
![github](https://github.com/ye-jingwen/NotePad_timestamp/blob/master/Image/NotePad_timestamp.png "github")

========================================================================================================================================================
# 笔记颜色排序
        参考 https://blog.csdn.net/douzajun/article/details/77669658
# 一.笔记背景颜色
## 1.先给NoteList换个主题，把黑色换成白色，在AndroidManifest.xml中NotesList的Activity中添加：
        android:theme="@android:style/Theme.Holo.Light"
## 2.UI美化主要是让NoteList和NoteSearch每条笔记都有背景色，并且能保存。要做到保存颜色的数据，最直接的办法就是在数据库中添加一个颜色的字段，在这之前在NotePad契约类中添加：
        public static final String COLUMN_NAME_BACK_COLOR = "color";
## 3.创建数据库表地方添加颜色的字段：
         @Override
            public void onCreate(SQLiteDatabase db) {
                db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
                + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
                + ");");
               }
## 4.把颜色定义为INTEGER的主要原因是，在系统中预定于好五种颜色，根据颜色对应不int值选择要显示的颜色，契约类中的定义：
         public static final int DEFAULT_COLOR = 0; //白
         public static final int YELLOW_COLOR = 1; //黄
         public static final int BLUE_COLOR = 2; //蓝
         public static final int GREEN_COLOR = 3; //绿
         public static final int RED_COLOR = 4; //红
## 5.由于数据库中多了一个字段，所以要在NotePadProvider中添加对其相应的处理，
### static{}中：
         sNotesProjectionMap.put(
                 NotePad.Notes.COLUMN_NAME_BACK_COLOR,
                 NotePad.Notes.COLUMN_NAME_BACK_COLOR);
### insert中：
           // 新建笔记，背景默认为白色
          if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
                  values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
                  }
## 6.将颜色填充到ListView，可以用SimpleCursorAdapter中的getView，bindView，newView方法来实现，我选择了bindView。自定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：
          public class MyCursorAdapter extends SimpleCursorAdapter {
              public MyCursorAdapter(Context context, int layout, Cursor c,
                                     String[] from, int[] to) {
                  super(context, layout, c, from, to);
              }
              @Override
              public void bindView(View view, Context context, Cursor cursor){
                  super.bindView(view, context, cursor);
                  //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
                  int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
                  /**
                   * 白 255 255 255
                   * 黄 247 216 133
                   * 蓝 165 202 237
                   * 绿 161 214 174
                   * 红 244 149 133
                   */
                  switch (x){
                      case NotePad.Notes.DEFAULT_COLOR:
                          view.setBackgroundColor(Color.rgb(255, 255, 255));
                          break;
                      case NotePad.Notes.YELLOW_COLOR:
                          view.setBackgroundColor(Color.rgb(247, 216, 133));
                          break;
                      case NotePad.Notes.BLUE_COLOR:
                          view.setBackgroundColor(Color.rgb(165, 202, 237));
                          break;
                      case NotePad.Notes.GREEN_COLOR:
                          view.setBackgroundColor(Color.rgb(161, 214, 174));
                          break;
                      case NotePad.Notes.RED_COLOR:
                          view.setBackgroundColor(Color.rgb(244, 149, 133));
                          break;
                      default:
                          view.setBackgroundColor(Color.rgb(255, 255, 255));
                          break;
                  }
              }
          }
## 7.NoteList中的PROJECTION添加颜色项：
        private static final String[] PROJECTION = new String[] {
                    NotePad.Notes._ID, // 0
                    NotePad.Notes.COLUMN_NAME_TITLE, // 1
                    //扩展 显示时间 颜色
                    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
                    NotePad.Notes.COLUMN_NAME_BACK_COLOR,
            };
## 8.并且将NoteList中用的SimpleCursorAdapter改使用MyCursorAdapter：
        //修改为可以填充颜色的自定义的adapter，自定义的代码在MyCursorAdapter.java中
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
            );
## 9.背景更换指的是编辑笔记时的背景色更换。编辑笔记的Activity为NoteEditor。同样的，在PROJECTION中添加颜色项：
         private static final String[] PROJECTION =
                new String[] {
                    NotePad.Notes._ID,
                    NotePad.Notes.COLUMN_NAME_TITLE,
                    NotePad.Notes.COLUMN_NAME_NOTE,
                    NotePad.Notes.COLUMN_NAME_BACK_COLOR
            };
## 10.可以注意到，在NoteEditor类中有onResume()方法，onResume()方法在正常启动时会被调用，一般是onStart()后会执行onResume()，在Acitivity从Pause状态转化到Active状态也会被调用，利用这个特点，将从数据库读取颜色并设置编辑界面背景色操作放入其中，好处除了从笔记列表点进来时可以被执行到，跳到改变颜色Activity（接下来会提到）回来时也会被执行到。
          //读取颜色数据
          int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
              /**
              * 白 255 255 255
              * 黄 247 216 133
              * 蓝 165 202 237
              * 绿 161 214 174
              * 红 244 149 133
              */
              switch (x){
                  case NotePad.Notes.DEFAULT_COLOR:
                      mText.setBackgroundColor(Color.rgb(255, 255, 255));
                      break;
                  case NotePad.Notes.YELLOW_COLOR:
                      mText.setBackgroundColor(Color.rgb(247, 216, 133));
                      break;
                  case NotePad.Notes.BLUE_COLOR:
                      mText.setBackgroundColor(Color.rgb(165, 202, 237));
                      break;
                  case NotePad.Notes.GREEN_COLOR:
                      mText.setBackgroundColor(Color.rgb(161, 214, 174));
                      break;
                  case NotePad.Notes.RED_COLOR:
                      mText.setBackgroundColor(Color.rgb(244, 149, 133));
                      break;
                  default:
                      mText.setBackgroundColor(Color.rgb(255, 255, 255));
                      break;
              }     
## 11.先在菜单文件中添加一个更改背景的选项，editor_options_menu.xml，图标自己添加，item总是显示：
        <item android:id="@+id/menu_color"
                android:title="@string/menu_color"
                android:icon="@drawable/ic_menu_color"
                android:showAsAction="always"/>
## 12.在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：
        //换背景颜色选项
            case R.id.menu_color:
                changeColor();
                break;
## 13.在NoteEditor中添加函数changeColor()：
        //跳转改变颜色的activity，将uri信息传到新的activity
            private final void changeColor() {
                Intent intent = new Intent(null,mUri);
                intent.setClass(NoteEditor.this,NoteColor.class);
                NoteEditor.this.startActivity(intent);
            }
## 14.在此之前，要对选择颜色界面进行布局，新建布局note_color.xml，垂直线性布局放置5个ImageButton，并创建NoteColor的Acitvity，用来选择颜色。在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式：
        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:orientation="horizontal" android:layout_width="match_parent"
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
       
       public class NoteColor extends Activity {
           private Cursor mCursor;
           private Uri mUri;
           private int color;
           private static final int COLUMN_INDEX_TITLE = 1;
           private static final String[] PROJECTION = new String[] {
                   NotePad.Notes._ID, // 0
                   NotePad.Notes.COLUMN_NAME_BACK_COLOR,
           };
           public void onCreate(Bundle savedInstanceState) {
               super.onCreate(savedInstanceState);
               setContentView(R.layout.note_color);
               //从NoteEditor传入的uri
               mUri = getIntent().getData();
               mCursor = managedQuery(
                       mUri,        // The URI for the note that is to be retrieved.
                       PROJECTION,  // The columns to retrieve
                       null,        // No selection criteria are used, so no where columns are needed.
                       null,        // No where columns are used, so no where values are needed.
                       null         // No sort order is needed.
               );
           }
           @Override
           protected void onResume(){
           //执行顺序在onCreate之后
               if (mCursor != null) {
                   mCursor.moveToFirst();
                   color = mCursor.getInt(COLUMN_INDEX_TITLE);
               }
               super.onResume();
           }
           @Override
           protected void onPause() {
           //执行顺序在finish()之后，将选择的颜色存入数据库
               super.onPause();
               ContentValues values = new ContentValues();
               values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
               getContentResolver().update(mUri, values, null, null);
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
       
       <!--换背景色-->
       <activity android:name="NoteColor"
           android:theme="@android:style/Theme.Holo.Light.Dialog"
           android:label="ChangeColor"
           android:windowSoftInputMode="stateVisible"/>
## 15.在NotesList中增加静待变量
        private MyCursorAdapter adapter;
        private Cursor cursor;
# 二.颜色排序
## 1.笔记排序相对来说简单，只要把Cursor的排序参数变换下就可以了。在菜单文件list_options_menu.xml中添加：
        <item
            android:id="@+id/menu_sort"
            android:title="@string/menu_sort"
            android:icon="@android:drawable/ic_menu_sort_by_size"
            android:showAsAction="always" >
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
## 2.在NoteList菜单switch下添加case：
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
## 3.因为排序会多次使用到cursor，adapter，所以将adapter,cursor,dataColumns,viewIDs定义在函数外类内：
        private MyCursorAdapter adapter;
        private Cursor cursor;
        private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
## 4.
## 5.
## 6.
## 7.
## 8.
## 9.
## 10.
