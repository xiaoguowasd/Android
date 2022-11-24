# NotePad Android期中实验

### 1. 时间戳功能

- #### 思路

  1.在主页面的每个列表项中添加时间戳的位置，即在notelist_item.xml布局文件中添加一个


  ```xml
  <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textSize="12dp"
        android:gravity="center_vertical"
        android:paddingLeft="10dip"
        android:singleLine="true"
        android:layout_weight="1"
        android:layout_margin="0dp"
        />  

  ```
  2.需要修改这个方法中的时间戳格式NotePadProvider中的insert方法:
  
  ```java
    Long now = Long.valueOf(System.currentTimeMillis());
   //修改 需要将毫秒数转换为时间的形式yy.MM.dd HH:mm:ss

   Date date = new Date(now);

   SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");

    String dateFormat = simpleDateFormat.format(date);
    //转换为yy.MM.dd HH:mm:ss形式的时间

     if(values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
        values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
     }

     if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
         values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
    } 

  ```
  3.NoteEditor中的updateNote方法:

  ```java
   long now = System.currentTimeMillis();
   Date date = new Date(now);
   SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
    String dateFormat = simpleDateFormat.format(date);
   values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
   
  ```
  4.NoteList的修改  如果我们要加入时间戳，必须要将修改时间的列也投影出来. 所以我们将PROJECTION添加上修改时间：
  
  ```java
  private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
    };

  ```
  
  PROJECTION只是定义了需要被取出来的数据列，而之后用Cursor进行数据库查询，再之后用Adapter进行装填。我们需要将显示列dataColumns和他们的viewIDs加入修改时间这一属性:
  
  ```java
  String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE }//加入修改时间;
  int[] viewIDs = { android.R.id.text1, R.id.text2 }//加入修改时间;

  ```
  5.实验结果如下图：

- <img src="https://github.com/xiaoguowasd/Android/blob/main/image/%E6%97%B6%E9%97%B4%E6%88%B3.png" alt="avatar" style="zoom:50%; width:750px" />


### 2. 搜索功能
- #### 思路

 1.搜索组件在主页面的菜单选项中，所以应在list_options_menu.xml布局文件中添加搜索功能

 
  ```xml
      <item
      android:id="@+id/menu_search"
      android:icon="@android:drawable/ic_menu_search"
      android:title="@string/menu_search"
      android:showAsAction="always" />

  ```
  2.新建一个查找笔记内容的布局文件note_search.xml

  
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <SearchView
        android:id="@+id/search"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false" />

    <ListView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

   </LinearLayout>

  ```
  3.在NoteList类中的onOptionsItemSelected方法中添加search查询的处理(跳转)
  

  ```java
          
        case R.id.menu_search:  
        //查找功能  
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));  
          Intent intent = new Intent(this, NoteSearch.class);  
          this.startActivity(intent);  
          return true;  

  ```
  4.新建一个NoteSearch类用于search功能的功能实现
  
  （2）实验代码：
  
  ```java
  package com.example.android.notepad;
  import android.app.Activity;
  import android.content.Intent;
  import android.database.Cursor;
  import android.database.sqlite.SQLiteDatabase;
  import android.os.Bundle;
  import android.widget.ListView;
  import android.widget.SearchView;
  import android.widget.SimpleCursorAdapter;
  import android.widget.Toast;
 
  public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
  {
    ListView listView;
    SQLiteDatabase sqLiteDatabase;
    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };
 
    public boolean onQueryTextSubmit(String query) {
        Toast.makeText(this, "you choose:"+query, Toast.LENGTH_SHORT).show();
        return false;
    }
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //Set the searchview to display the search button
        searchView.setSubmitButtonEnabled(true);
 
        //Set the prompt text displayed by default in this searchview
        searchView.setQueryHint("search");
        searchView.setOnQueryTextListener(this);
 
    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
      }
  }


  ```
  5.最后要在清单文件AndroidManifest.xml里面注册NoteSearch,否则无法实现界面的跳转
  
  
  ```xml
            <activity android:name=".NoteSearch" android:label="@string/menu_search" />
  ```
  
   ⑥实验结果如下图：

- <img src="https://github.com/xiaoguowasd/Android/blob/main/image/%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD.png" alt="avatar" style="zoom:50%; width:750px" />
