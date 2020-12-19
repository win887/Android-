# FJNU_test
Android期中实验，修改NotePad的源代码完成基本功能并做出扩展

# 流程说明

**基本功能一：NoteList中显示条目增加时间戳显示**

1.1在显示笔记列表的nolteslist_item.xml中用于显示标题的TextView上方新增一个 TextView 用于显示时间戳

```
<TextView    android:id="@+id/edit_time"    
android:layout_width="match_parent"    
android:layout_height="match_parent"    
android:paddingLeft="3dip"    
android:singleLine="true"    
android:gravity="center_vertical" />
```

1.2在NotesList类游标适配器所需的列中拿出笔记的更新时间

```
      NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//添加投影更新时间
```

在需要显示的内容中增添显示更新时间并把要显示的内容与布局文件中的组件对应

```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
int[] viewIDs = { android.R.id.text1,R.id.edit_time };
```

1.3修改在 NotePadProvider中的新增笔记方法中的时间保存格式让时间戳能够以日期的样式显示

```
Long now = Long.valueOf(System.currentTimeMillis());//将插入方法中的时间替换为日期格式
Date date = new Date(now);//获取系统日期时间
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String exactdate = simpleDateFormat.format(date);
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, exactdate);//将值设为当前日期时间}
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, exactdate);//将值设为当前日期时间}
```

 1.4修改在NoteEditor中的编辑方法，同样是改变系统时间的显示样式

```
Date date = new Date(System.currentTimeMillis());
//将系统获得的时间转为日期格式再次赋予
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String exactdate = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, exactdate);
```

运行效果

![QQ图片20201216184724.png](https://i.loli.net/2020/12/16/wXiYBE9tbZRhNMj.png)

**基本功能二：添加笔记查询功能（根据标题查询）**

2.1新建note_search.xml布局文件以及NoteSearch类用于在新界面实现笔记查询功能，note_search.xml包含一个用于输入关键词的SearchView和用于显示检索结果的ListView

```
<SearchView   
android:id="@+id/search_view"    
android:layout_width="match_parent"    
android:layout_height="wrap_content"    
android:iconifiedByDefault="false"    />
<ListView    
android:id="@+id/list_view"    
android:layout_width="match_parent"    
android:layout_height="wrap_content"    
android:layout_weight="1"    />
```

2.2在list_options_menu菜单文件中新增一个搜素图标用于实现NotesList和NoteSearch之间的跳转

```
//新增搜索图标
<item    
android:id="@+id/menu_search"   
android:icon="@drawable/search"    
android:showAsAction="always"    
android:title="search"></item>
```

在NotesList为该图标添加相应点击事件

```
//新增点击搜索图标的事件
case R.id.menu_search:      
	Intent intent = new Intent();      
	intent.setClass(this, NoteSearch.class);    
	System.out.println("图标事件");       
	this.startActivity(intent);        
return true;
```

2.3监听搜索框改变做出响应事件

```
protected void onCreate(Bundle savedInstanceState) 
{    super.onCreate(savedInstanceState);    
setContentView(R.layout.note_search);    
Intent intent = getIntent();    
if (intent.getData()  == null) { intent.setData(NotePad.Notes.CONTENT_URI);    }    SearchView searchview =  (SearchView)findViewById(R.id.search_view);   
//为查询文本框注册监听器    
searchview.setOnQueryTextListener(NoteSearch.this);}
```

文本改变时进行模糊查询并把搜索结果装入适配器中在列表实现显示

```
public boolean onQueryTextChange(String newText) {       
String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";       
String[] selectionArgs = { "%"+newText+"%" };        
Cursor cursor = managedQuery(  getIntent().getData(),PROJECTION,            selection, selectionArgs, NotePad.Notes.DEFAULT_SORT_ORDER        );        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE  ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };        
int[]  viewIDs = { android.R.id.text1   , R.id.list_view };        SimpleCursorAdapter adapter = new SimpleCursorAdapter(this,R.layout.noteslist_item, cursor, dataColumns, viewIDs);
new String[]{"Animalname","Animalhead"},new int[]{R.id.name,R.id.head});           
ListView list=(ListView)findViewById(R.id.list_view);       list.setAdapter(adapter);       
System.out.printf("jieshu\n");      
return true;    }
```

![QQ图片2020121619354.jpg](https://i.loli.net/2020/12/16/5rGIc369BQ4ftVg.jpg)

**扩展功能一：使用 XML定义菜单增加字体大小变化和字体颜色改变效果**

在NoteEditor类的开头定义所需要使用的字体大小规格和颜色

```
private static final int FONT_10 = 0x111;
private static final int FONT_16 = 0x114;
private static final int FONT_20= 0x115;
private static final int FONT_RED = 0x116;
private static final int FONT_DARK = 0x117;
```

在XML菜单中添加选择项

```
SubMenu fontMenu = menu.addSubMenu("字体大小");
fontMenu.setHeaderTitle("选择字体大小");
fontMenu.add(0, FONT_10, 0, "小");
fontMenu.add(0, FONT_16, 0, "中");
fontMenu.add(0, FONT_20, 0, "大");
SubMenu colorMenu = menu.addSubMenu("字体颜色");
colorMenu.setHeaderTitle("选择文字颜色");
colorMenu.add(0, FONT_RED, 0, "红色");
colorMenu.add(0, FONT_DARK, 0, "黑色");
```

在onOptionsItemSelected方法中添加选择项的响应事件

```
case FONT_10: text.setTextSize(10 * 2); break;
case FONT_16: text.setTextSize(16 * 2); break;
case FONT_20: text.setTextSize(20 * 2); break;
case FONT_RED: text.setTextColor(getResources().getColor(R.color.colorRed));break;
case FONT_DARK: text.setTextColor(getResources().getColor(R.color.colorDark)); break;
```

普通运行效果

![QQ图片20201218162915.png](https://i.loli.net/2020/12/18/uKjqmdYhts2vl14.png)

将字体大小改为大，字体颜色改为红色后的显示效果

![QQ图片20201218163422.png](https://i.loli.net/2020/12/18/9BOjQFaImidz2gM.png)

**扩展功能二：为主页面添加窗口边框和背景**

在values中建立文件styles.xml来增加一个主题

```
<resources>    
<style name="AppTheme" parent="android:Theme.Material.Light.DarkActionBar"></style>    <style name="CrazyTheme">        
<item name="android:windowNoTitle">false</item>        
<item name="android:windowFullscreen">true</item>        
<item name="android:windowFrame">@drawable/window_border</item>        
<item name="android:windowBackground">@drawable/time1</item>    </style></resources>
```

在drawable中建立window_border.xml文件定义窗口边框

```
<shape xmlns:android="http://schemas.android.com/apk/res/android"
android:shape="rectangle">
<solid android:color="#0fff"/>
<padding android:bottom="7dp"    
android:left="7dp"    
android:right="7dp"    
android:top="7dp"/>
<stroke android:width="10dp"    
android:color="#eff7bd"/></shape>
```

指定NotesList使用该主题

```
android:theme="@style/CrazyTheme"
```

导入背景图片后运行效果

![QQ图片20201218232519.png](https://i.loli.net/2020/12/18/AGL63IsD1rZv9xP.png)
