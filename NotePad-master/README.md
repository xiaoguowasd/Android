# NotePad Android期中实验

### 1. NoteList界面中笔记条目增加时间戳显示

- #### 思路

  ①（1）修改NotesList.java中PROJECTION的内容，添加modif字段，使其在后面的搜索中才能从SQLite中读取修改时间的字段。

  （2）实验代码：

  ```java
  private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
    };
  ```
  ②（1）修改适配器内容，增加dataColumns中装配到ListView的内容，同时增加一个文本框来存放时间。
  
  （2）实验代码：
  
  ```java
  final String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
  int[] viewIDs = { android.R.id.text1, R.id.text2};
  ```
  ③（1）修改layout文件夹中noteslist_item.xml的内容，增加一个textview组件，为他们添加一个布局。
  
  （2）实验代码：

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
 
 
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
        />
  </LinearLayout>


  ```
  ④（1）修改NoteEditor.java中updateNote方法中的时间类型。
  
  （2）实验代码：
  
  ```java
  private final void updateNote(String text, String title) {
 
        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);


  ```
  ⑤实验结果如下图：

- <img src="https://github.com/17515424731/Android/blob/main/image/n1.png" alt="avatar" style="zoom:50%; width:750px" />


### 2. 添加笔记查询功能
- #### 思路

  ①（1）搜索组件在主页面的菜单选项中，在list_options_menu.xml布局文件中添加搜索功能。

  （2）实验代码：

  ```xml
      <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        android:title="@string/menu_search"
        android:showAsAction="always" />
  ```
  ②（1）新建一个查找笔记内容的布局文件note_search.xml。
  
  （2）实验代码：
  
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        />
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
    </LinearLayout>
  ```
  ③（1）在NotesList.java中的onOptionsItemSelected方法中添加search查询的处理。
  
  （2）实验代码：

  ```java
          case R.id.menu_search:
        //Find function
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
          Intent intent = new Intent(this, NoteSearch.class);
          this.startActivity(intent);
          return true;
  ```
  ④（1）新建一个NoteSearch.java用于search功能的功能实现。
  
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
  ⑤（1）在清单文件AndroidManifest.xml里面注册NoteSearch。
  
  （2）实验代码：
  
  ```xml
            <activity android:name=".NoteSearch" android:label="@string/search_note" />
  ```
  
   ⑥实验结果如下图：

- <img src="https://github.com/17515424731/Android/blob/main/image/n2.png" alt="avatar" style="zoom:50%; width:750px" />
