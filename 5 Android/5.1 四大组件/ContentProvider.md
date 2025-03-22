+ # 1 Android 权限

  以打电话为例，说明在 Android 运行时申请权限的整个流程。也就是安卓应用在第一次打开时会弹窗申请通知、位置等权限。

  ```java
  protected void onCreate(Bundle saveInstanceState) {
  	...
  		button.setOnClickListener(new View.OnClickListener() {
  			@Override
  			public void onClick(View v) {
  				if (ContextCompat.checkSelfPermission(
  					MainActivity.this, Manifest.permission.CALL_PHONE) !=
  					PackageManager.PERMISSION_GRANTED) {	//1.判断权限
  					
  					//2.申请权限
  					ActivityCompat.requestPermissions(MainActivity.this,
  													  new String[]{Manifest.permission.CALL_PHONE}, 1);
  				} else {
  					call();	//3.打电话
  				}
  			}
  		});
  }
  
  private void call() {
  	try{
  		Intent intent = new Intent(Intent.ATCION_CALL);
  		intent.setData(Uri.parse("tel:10086"));
  		startActivity(intent);
  	} catch (SecurityException e) {
  		e.printStackTrace();
  	}
  }
  
  
  @Override
  public void onRequestPermissionsResult(		//4.用户同意或拒绝后的回调
  	int requestCode, String[] permissions, int[] grantResults) {
  	
  	switch(requestCode) {
  		case 1:
  			if (grantResults.length > 0 && 
  				grantResults[0] == PackageManager.PERMISSION_GRANTED) {
  				
  				call();
  			} else {
  				Toast.makeText(this, "You denied the permission",
  							   Toast.LENGTH_SHORT).show();
  			}
  			break;
  		default:
  			break;
  	}
  }
  ```

  代码解析：程序运行过程中由用户进行授权。

  1. 判断用户是否已经给过权限，使用的是 `ContextCompat.checkSelfPermission()` 方法。该方法：

  1. 第一个参数是 Context
  2. 第二个参数是具体的权限名
  3. 使用方法返回值与 `PackageManager.PERMISSION_GRANTED` 做比较

  1. 如果没有授权，则调用 `ActivityCompat.requestPermissions()` 方法申请授权。该方法：

  1. 第一个参数是 Activity 的实例
  2. 第二个参数是一个 String 数组，将需要申请的权限放入数组中
  3. 第三个参数是请求码，只要是唯一值就可以

  1. 如果已经授权，则执行 call() 方法打电话
  2. 调用完 `requestPermissions()` 方法，系统会弹出一个对话框，用户点击同意或拒绝之后，回调到 `onRequestPermissionsResult()` 方法中，结果封装在 grantResults 参数当中

  1. 根据请求码执行不同的 case，这里执行请求码 1 的流程。
  2. 判断 `grantResult[0]` 的结果是否为需要申请的权限，是则执行 `call()` 方法，否则弹出弹窗警告。

  # 2 访问其他程序的数据

  **1>** 使用现有的内容提供器读取和操作其他程序中的数据 (**访问外部程序**)

  **2>** 创建自己的内容提供器给程序提供外部访问接口 (**被外部程序访问**)

  ## 2.1 ContentResolver

  **内访外**: 对于每个应用程序, 想要访问内容提供器中共享的数据, 就一定要借助 ContentResolver.

  - 可以通过 Context 的 `getContentResolver()` 方法获取该实例.
  - ContentResolver 提供了 CRUD 对应的方法: `insert()` `update()` `delete()` `query()`
  - ContentResolver 使用 **内容URI** 访问数据, **内容URI** 为内容提供器中的数据建立了唯一标识符, 它包含2个部分:

  - **authority**: 区分不同程序, 一般程序使用 **包名+provider** 的命名方式, 比如 `com.example.app.provider`
  - **path**: 同一应用程序区分不同表, 通常添加到 authority 后面. 比如 

  - `content://com.example.app.provider/table1` 
  - `content://com.example.app.provider/table2`

  - 使用 内容URI 之前, 需要将它解析为 Uri 对象才能用

  ```java
  Uri uri = Uri.parse("content://com.example.app.provider/table1");
  ```

  ### 2.1.1 查询数据

  **1>** 使用 Cursor 获取该 table 中的数据

  ```java
  Cursor cursor = getContentResolver().query(
      uri,
      projection,
      selection,
      selectionArgs,
      sortOrder);
  ```

  下表是 query() 方法中的参数

  | query() 参数  | 对应 SQL 语句             | 描述                 |
  | ------------- | ------------------------- | -------------------- |
  | uri           | from table_name           | 指定查询某张表       |
  | projection    | select column1, column2   | 指定列名             |
  | selection     | where column = value      | 指定 where 约束条件  |
  | selectionArgs | -                         | 给 where 提供占位符  |
  | orderBy       | order by column1, column2 | 指定查询结果排列方式 |

  **2>** 遍历 cursor 对象

  ```java
  if (cursor != null) {
      while (cursor.moveToNext()) {
          String column1 = cursor.getString(cursor.getColumnIndex("column1"));
          int column2 = cursor.getInt(cursor.getColumnIndex("column2"));
      }
  }
  ```

  ### 2.1.2 添加数据

  ```java
  ContentValues values = new ContentValues();
  values.put("column1", "text");
  values.put("column2", 1);
  getContentResolver().insert(uri, values);
  ```

  ### 2.1.3 更新数据

  ```java
  getContentResolver().update(uri,
                              values, 
                              "column1 = ? and column = ?",
                              new String[]{"text", "1"});
  ```

  ### 2.1.4 删除数据

  ```java
  getContentResolver().delete(uri,
                              "column1 = ?",
                              new String[]{ 1 });
  ```

  # 3 获取手机联系人

  ## 3.1 申请获取手机联系人的相关权限

  ```java
  if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS)
          != PackageManager.PERMISSION_GRANTED) {
      ActivityCompat.requestPermissions(this,
              new String[]{Manifest.permission.READ_CONTACTS}, 1);
  } else {
      readContacts();
  }
  ```

  ## 3.2 用户授权或者拒绝后会回调 `onRequestPermissionsResult()` 方法

  ```java
  @Override
  public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                         @NonNull int[] grantResults) {
      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
      if (requestCode == 1) {
          if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
              readContacts();
          } else {
              Toast.makeText(this, "You denied the permission",
                      Toast.LENGTH_SHORT).show();
          }
      }
  }
  ```

  ## 3.3 获取并保存手机联系人内容

  ```java
  private void readContacts() {
      Cursor cursor = null;
      try {
          cursor = getContentResolver().query(ContactsContract.CommonDataKinds.
                  Phone.CONTENT_URI, null, null, null, null);
          if (cursor != null) {
              while (cursor.moveToNext()) {
                  String displayName = cursor.getString(cursor.getColumnIndex(
                          ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                  String number = cursor.getString(cursor.getColumnIndex(
                          ContactsContract.CommonDataKinds.Phone.NUMBER));
              }
          }
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
  ```

  ## 3.4 AndroidManifest.xml 中添加权限申请

  ```xml
  <manifest ...>
    <uses-permission android:name="android.permission.READ_CONTACTS" />  
  </manifest>
  ```

  # 4 创建内容提供器

  ## 4.1 新建 MyContentProvider 类

  ```java
  public class MyContentProvider extends ContentProvider {
      @Override
      public boolean onCreate() {
          return false;
      }
  
      @Nullable
      @Override
      public Cursor query(@NonNull Uri uri, @Nullable String[] strings, @Nullable String s, @Nullable String[] strings1, @Nullable String s1) {
          return null;
      }
  
      @Nullable
      @Override
      public String getType(@NonNull Uri uri) {
          return null;
      }
  
      @Nullable
      @Override
      public Uri insert(@NonNull Uri uri, @Nullable ContentValues contentValues) {
          return null;
      }
  
      @Override
      public int delete(@NonNull Uri uri, @Nullable String s, @Nullable String[] strings) {
          return 0;
      }
  
      @Override
      public int update(@NonNull Uri uri, @Nullable ContentValues contentValues, @Nullable String s, @Nullable String[] strings) {
          return 0;
      }
  }
  ```

  - onCreate(): 初始化时调用. 通常在这里完成**数据库的创建和升级**等操作, true 初始化成功, false 失败.
  - getType(): 根据传入的 URI 返回相应的 MIME 类型.

  可以使用通配符匹配内容URI.

  - *: 匹配任意长度的任意字符
  - \#: 匹配任意长度的数字

  ## 4.2 借助 UriMatcher 类实现功能

  UriMatcher 提供 addURI() 方法, 该方法接收3个参数

  1. authority
  2. path
  3. 自定义代码

  当外部应用调用 UriMatcher 的 `match()` 方法时, 就可以传入一个 Uri 对象. 然后返回自定义代码, 利用该自定义代码就可以判断调用者期望访问什么数据.

  ```java
  public class MyContentProvider extends ContentProvider {
  
  
      public static final int TABLE1_ALL_DATA = 0;
      public static final int TABLE1_ITEM_DATA = 1;
      public static final int TABLE2_ALL_DATA = 2;
      public static final int TABLE2_ITEM_DATA = 3;
  
      private static UriMatcher uriMatcher;
      static {
          uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
          uriMatcher.addURI("com.example.zzfan.provider", "table1", TABLE1_ALL_DATA);
          uriMatcher.addURI("com.example.zzfan.provider", "table1", TABLE1_ITEM_DATA);
          uriMatcher.addURI("com.example.zzfan.provider", "table2", TABLE2_ALL_DATA);
          uriMatcher.addURI("com.example.zzfan.provider", "table2", TABLE2_ITEM_DATA);
      }
  
      @Nullable
      @Override
      public Cursor query(@NonNull Uri uri, @Nullable String[] strings, @Nullable String s,
                          @Nullable String[] strings1, @Nullable String s1) {
          switch (uriMatcher.match(uri)) {
              case TABLE1_ALL_DATA:
                  //TODO
                  break;
              case TABLE1_ITEM_DATA:
                  //TODO
                  break;
              case TABLE2_ALL_DATA:
                  //TODO
                  break;
              case TABLE2_ITEM_DATA:
                  //TODO
                  break;
              default:
                  break;
          }
          return null;
      }
  }
  ```

  ## 4.3 添加至 AndroidManifest.xml 文件中

  ```xml
  <manifest ...>
    <provider
              android:name=".MyContentProvider"
              android:authorities="com.example.zzfan.provider"
              android:enabled="true"
              android:exported="true">
    </provider>
  </manifest>
  ```

  - android:name: 指定内容提供者的类名。这里的内容提供者类名为.MyContentProvider，其中.通常表示该类位于与AndroidManifest.xml相同的应用包名下。
  - android:authorities: 定义了该内容提供者的唯一标识符，通常采用反向域名的形式。在这个例子中，标识符为com.example.zzfan.provider。
  - android:enabled="true": 表示此内容提供者是否被启用，默认值为true。如果设置为false，则该提供者将不会被系统加载。
  - android:exported="true": 控制其他应用是否可以访问这个内容提供者。设置为true意味着允许跨应用访问；如果是false，则限制只有本应用或指定权限的应用可以访问。
