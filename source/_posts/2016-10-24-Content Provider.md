---
title: API Guides - Content Provider
tags:
  - android
date: 2016-10-24 15:37:22
categories: 笔记
---

#  Content Providers  #
一些参考：
[What is a Contract Class and how is it used](http://stackoverflow.com/questions/9243361/what-is-a-contract-class-and-how-is-it-used)
## API Guides ##
[官网](https://developer.android.com/guide/topics/providers/content-providers.html)

​	内容提供器用来管理对结构化数据集的访问，它们把数据封装起来，并提供用于定义数据安全性的机制。数据提供器是连接一个进程中的数据与运行在其他进程中的代码的标准接口。

​	当想要访问content provider中数据时，可以使用应用Context中的ContentResolver对象作为客户端来与provider进行通信。

​	android本身包含了content providers来管理比如音频、视频、images和联系人信息等的数据。[android.provider](https://developer.android.com/reference/android/provider/package-summary.html)中可以查看对它们的一些引用。

### Content Provider Basics ###
> 如何访问content provider中表格形式的数据？

​	provider是android应用的一部分，应用通常提供自己的UI来处理数据。但是，content providers主要是被其他应用通过provider客户端对象来访问provider使用。provider和provider client共同提供了一个便利、标准的数据接口来处理进程间通信和安全数据的访问。

​	本节将描述以下基本内容：
- content provider是如何工作的
- 获取content provider数据的api
- 对content provider进行insert、update、delete操作的api
- 其他便于使用provider的api

#### 概述 ####
content provider以一个或多个表格（类似于关系数据库中表格）的方式来	向外部应用展示数据

| word        | app id | frequency | locale | _ID  |
| ----------- | ------ | --------- | ------ | ---- |
| mapreduce   | user1  | 100       | en_US  | 1    |
| precompiler | user14 | 200       | fr_FR  | 2    |
| applet      | user2  | 225       | fr_CA  | 3    |
| const       | user1  | 255       | pt_BR  | 4    |
| int         | user5  | 100       | en_UK  | 5    |

> 注意：provider并不要求要有主键，并且不要求使用_ID作为一个主键的列名。但是，**如果需要将provider中的数据绑定到一个ListView，其中的一个列名必须为_ID**.

##### 访问provider #####
​	使用ContentResolver客户端对象。该对象拥有provider对象（Content Provider的一个具体子类）中的同名方法。ContentResolver方法提供了持久化存储的基本GUID（create、update、insert、delete）功能。
> 注意：访问provider通常需要在Manifest文件中生命权限。

​	如：获取User Dictionary Provider中一个word和locale的列表，需要调用ContentResolver.query()。query()方法将调用定义在User Dictionary Provider中的ContentProvider.query()方法。

    // Queries the user dictionary and returns results
    mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause                    // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows

| query() argument | SELECT keyword/parameter                 | Notes                                    |
| ---------------- | ---------------------------------------- | ---------------------------------------- |
| Uri              | FROM table_name                          | Uri maps to the table in the provider named table_name. |
| projection       | col,col,col,...                          | projection is an array of columns that should be included for each row retrieved. |
| selection        | WHERE col = value                        | selection specifies the criteria for selecting rows. |
| selectionArgs    | (No exact equivalent. Selection arguments replace ? placeholders in the selection clause.) | &nbsp;                                   |
| sortOrder        | ORDER BY col,col,...                     | sortOrder specifies the order in which rows appear in the returned Cursor. |

##### Content URIs #####
​	content URI是标识provider中数据的一种uri。Content URI包含标识整个provider的名称（authority）和指向该表的一个名字（path）。

​	ContentResolver对象解析出URI的authority，并据此来与指向一个已知providers的系统表格的authority进行比较来确定provider。ContentResolver使用Content URI的path部分来选择访问的表格。

> content://user_dictionary/words 
> user_dictionary是provider的authority，words是表格路径

​	很多provider允许通过在URI末尾附加一个id值来访问一个表格的一行。
> Uri singleUri = ContentUris.withAppendedId(UserDictionary.Words.CONTENT_URI,4);

​	通常在需要获取一个"行"的集合来进行修改或者删除时使用id值。
> 注意：[ Uri](https://developer.android.com/reference/android/net/Uri.html)和[ Uri.Builder](https://developer.android.com/reference/android/net/Uri.Builder.html)类提供了便利方法来根据String构造符合语法规则的URI对象。[ContentUris](https://developer.android.com/reference/android/content/ContentUris.html)类则包含了将id值附加到URI的便利方法。

#### Retrieving Data from the Provider ####
从一个provider中获取数据，使用以下步骤 
1. 请求provider的访问权限。
   获取数据需要得到provider的读权限，该权限必须在manifest文件的<uses-permission>中使用provider定义的权限名来进行声明。
2. 确定向provider发送query的代码。
   定义访问User Dictionary Provider的一些变量：
```
// A "projection" defines the columns that will be returned for each row
String[] mProjection =
{
    UserDictionary.Words._ID,    // Contract class constant for the _ID column name
    UserDictionary.Words.WORD,   // Contract class constant for the word column name
    UserDictionary.Words.LOCALE  // Contract class constant for the locale column name
};
// Defines a string to contain the selection clause
String mSelectionClause = null;
// Initializes an array to contain selection arguments
String[] mSelectionArgs = {""};
```
使用query方法：
```
/*
 * This defines a one-element String array to contain the selection argument.
 */
String[] mSelectionArgs = {""};
// Gets a word from the UI
mSearchString = mSearchWord.getText().toString();
// Remember to insert code here to check for invalid or malicious input.
// If the word is the empty string, gets everything
if (TextUtils.isEmpty(mSearchString)) {
    // Setting the selection clause to null will return all words
    mSelectionClause = null;
    mSelectionArgs[0] = "";
} else {
    // Constructs a selection clause that matches the word that the user entered.
    mSelectionClause = UserDictionary.Words.WORD + " = ?";
    // Moves the user's input string to the selection arguments.
    mSelectionArgs[0] = mSearchString;
}
// Does a query against the table and returns a Cursor object
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,  // The content URI of the words table
    mProjection,                       // The columns to return for each row
    mSelectionClause                   // Either null, or the word the user entered
    mSelectionArgs,                    // Either empty, or the string the user entered
    mSortOrder);                       // The sort order for the returned rows
// Some providers return null if an error occurs, others throw an exception
if (null == mCursor) {
    /*
     * Insert code here to handle the error. Be sure not to use the cursor! You may want to
     * call android.util.Log.e() to log this error.
     *
     */
// If the Cursor is empty, the provider found no matches
} else if (mCursor.getCount() < 1) {
    /*
     * Insert code here to notify the user that the search was unsuccessful. This isn't necessarily
     * an error. You may want to offer the user the option to insert a new row, or re-type the
     * search term.
     */
} else {
    // Insert code here to do something with the results
}
```

##### Protecting against malicious input #####
​	假如被content provider管理的数据使用的是SQL数据库，将外部不受信任的数据包含进原生的SQL语句中将会产生SQL注入。如：
```
// Constructs a selection clause by concatenating the user's input to the column name
String mSelectionClause =  "var = " + mUserInput;
```
​	如果使用以上语句，将允许用户将一些恶意的SQL串联到SQL语句中，如，用户输入"nothing; DROP TABLE *;"。这样将可能导致底层的所有SQLite数据库被抹去。
​	为避免这种情况，在selection clause中使用？作为可替代参数和一个分离的数组作为selection argument。这样用户的输入将直接绑定到query，而不是被解读为SQL语句的一部分。
```
// Constructs a selection clause with a replaceable parameter
String mSelectionClause =  "var = ?";
```
```
// Defines an array to contain the selection arguments
String[] selectionArgs = {""};
```
```
// Sets the selection argument to the user's input
selectionArgs[0] = mUserInput;
```

###### Displaying query results #####
​	ContentResolver.query()通常返回一个包含由query的projection指定的列的[Cursor](https://developer.android.com/reference/android/database/Cursor.html " Cursor")。Cursor对象提供了对其所包含的行和列的随机读权限。一些Cursor实现会在provider的数据发生变化时自动更新其对象或者触发观察者对象中的方法。


​	如果没有行匹配selection规则，provider返回一个Cursor.getCount()为0的Cursor对象。

​	以下代码段创建了一个包含通过query获取的Cursor的[SimpleCursorAdapter](https://developer.android.com/reference/android/widget/SimpleCursorAdapter.html)对象，并将该对象设为ListView的adapter。
```
// Defines a list of columns to retrieve from the Cursor and load into an output row
String[] mWordListColumns =
{
    UserDictionary.Words.WORD,   // Contract class constant containing the word column name
    UserDictionary.Words.LOCALE  // Contract class constant containing the locale column name
};

// Defines a list of View IDs that will receive the Cursor columns for each row
int[] mWordListItems = { R.id.dictWord, R.id.locale};

// Creates a new SimpleCursorAdapter
mCursorAdapter = new SimpleCursorAdapter(
    getApplicationContext(),               // The application's Context object
    R.layout.wordlistrow,                  // A layout in XML for one row in the ListView
    mCursor,                               // The result from the query
    mWordListColumns,                      // A string array of column names in the cursor
    mWordListItems,                        // An integer array of view IDs in the row layout
    0);                                    // Flags (usually none are needed)

// Sets the adapter for the ListView
mWordList.setAdapter(mCursorAdapter);
```
> 注意：在ListView中使用Cursor，cursor必须包含_ID列。

##### Getting data from query results ####
```
// Determine the column index of the column named "word"
int index = mCursor.getColumnIndex(UserDictionary.Words.WORD);

/*
 * Only executes if the cursor is valid. The User Dictionary Provider returns null if
 * an internal error occurs. Other providers may throw an Exception instead of returning null.
 */

if (mCursor != null) {
    /*
     * Moves to the next row in the cursor. Before the first movement in the cursor, the
     * "row pointer" is -1, and if you try to retrieve data at that position you will get an
     * exception.
     */
    while (mCursor.moveToNext()) {

        // Gets the value from the column.
        newWord = mCursor.getString(index);

        // Insert code here to process the retrieved word.

        ...

        // end of while loop
    }
} else {

    // Insert code here to report an error if the cursor is null or the provider threw an exception.
}
```

#### Content Provider Permissions ####
​	一个provider的应用可以指定其他应用访问provider数据所需要的权限。如果应用未指定provider的任何权限，其他应用将无法访问该provider数据。

	<uses-permission android:name="android.permission.READ_USER_DICTIONARY">
#### Inserting, Updating, and Deleting Data ####
##### Inserting data #####
​	调用[ContentResolver.insert()](https://developer.android.com/reference/android/content/ContentResolver.html#insert(android.net.Uri, android.content.ContentValues))方法。该方法向provider插入一条新行并返回该行的content uri。
```
// Defines a new Uri object that receives the result of the insertion
Uri mNewUri;

...

// Defines an object to contain the new values to insert
ContentValues mNewValues = new ContentValues();

/*
 * Sets the values of each column and inserts the word. The arguments to the "put"
 * method are "column name" and "value"
 */
mNewValues.put(UserDictionary.Words.APP_ID, "example.user");
mNewValues.put(UserDictionary.Words.LOCALE, "en_US");
mNewValues.put(UserDictionary.Words.WORD, "insert");
mNewValues.put(UserDictionary.Words.FREQUENCY, "100");

mNewUri = getContentResolver().insert(
    UserDictionary.Word.CONTENT_URI,   // the user dictionary content URI
    mNewValues                          // the values to insert
);
```
​	如果不希望指定值，可以调用[ContentValues.putNull()](https://developer.android.com/reference/android/content/ContentValues.html#putNull(java.lang.String)来将该列设为null。
​	新添加行返回的uri将使用以下形式：
> content://user_dictionary/words/<id_value>

​	可以通过调用[ContentUris.parseId()](https://developer.android.com/reference/android/content/ContentUris.html#parseId(android.net.Uri))来获取返回uri的_ID。

##### Updating data ####
```
// Defines an object to contain the updated values
ContentValues mUpdateValues = new ContentValues();

// Defines selection criteria for the rows you want to update
String mSelectionClause = UserDictionary.Words.LOCALE +  "LIKE ?";
String[] mSelectionArgs = {"en_%"};

// Defines a variable to contain the number of updated rows
int mRowsUpdated = 0;

...

/*
 * Sets the updated value and updates the selected words.
 */
mUpdateValues.putNull(UserDictionary.Words.LOCALE);

mRowsUpdated = getContentResolver().update(
    UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI
    mUpdateValues                       // the columns to update
    mSelectionClause                    // the column to select on
    mSelectionArgs                      // the value to compare to
);
```
##### Deleting data ####
```
// Defines selection criteria for the rows you want to delete
String mSelectionClause = UserDictionary.Words.APP_ID + " LIKE ?";
String[] mSelectionArgs = {"user"};

// Defines a variable to contain the number of rows deleted
int mRowsDeleted = 0;

...

// Deletes the words that match the selection criteria
mRowsDeleted = getContentResolver().delete(
    UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI
    mSelectionClause                    // the column to select on
    mSelectionArgs                      // the value to compare to
);
```

#### Provider Data Types ####
​	provider提供了以下几中数据类型：(可通过[Cursor](https://developer.android.com/reference/android/database/Cursor.html)的get方法查看有效类型)
- integer
- long integer(long)
- floating point
- long floating point(double)
- text
- Binary Large OBject (BLOB)

每一列的数据类型通常会在文档中说明，也可以通过[Cursor.getType()](https://developer.android.com/reference/android/database/Cursor.html#getType(int))来确定数据类型。
provider为所有定义的content uri维持了MIME数据类型信息。通过使用该类型来确定自己的应用是否支持处理该provider提供的数据或者选择一种处理该MIME类型的方式。

#### Alternative Forms of Provider Access ####
​	对开发者而言，有三种重要的provider访问方式。
1. **Batch access**。在[ContentProviderOperation](https://developer.android.com/reference/android/content/ContentProviderOperation.html)类中创建批量访问所调用的方法，然后使用[ContentResolver.applyBatch()](https://developer.android.com/reference/android/content/ContentResolver.html#applyBatch(java.lang.String, java.util.ArrayList<android.content.ContentProviderOperation>))应用。
2. **Asynchronous queries**。应当在一个单独的线程中进行查询操作，一种方式是使用一个[CursorLoader](https://developer.android.com/reference/android/content/CursorLoader.html)对象。
3. **Data access via intents**。尽管无法直接操作一个provider，但可以通过发送intent给provider的应用，这通常是修改provider数据的最好方式。intent提供对content provider的间接访问。
   （1）Getting access with temporary permissions
   　　即使没有合适的访问权限，也可以通过向一个具有权限并返回包含"URI"权限的result intent的应用发送intent来访问content provider的数据。此类由指定content URI指定的权限持续到activity销毁。具有权限的应用通过设置result intent的以下标志位来授予权限：FLAG_GRANT_READ_URI_PERMISSION和FLAG_GRANT_WRITE_URI_PERMISSION。
   　　provider在manifest中定义content URIs的URI权限：在<provider>元素中使用 android:grantUriPermission属性，也可以添加<grant-uri-permission>子元素。

　　（2）Using another application
　　　　一种修改数据的简单方式是启动一个具有权限的应用。

#### MIME Type Reference ####
​	Content Provider可以返回标准的MIME媒体类型或者自定义的MIME类型字符串，或两者。
MIME类型格式：
> type/subtype

​	自定义MIME类型字符串，也称作"vendor-specific"MIME类型。type的值通常使用多行

> vnd.android.cursor.dir

​	单行
> vnd.android.cursor.item

​	而subtype使用provider指定的。android内置provider通常使用简易的subtype。如


> vnd.android.cursor.item/phone_v2

​	其他的provider开发者则可能基于provider的authority和表名来创建他们自己的subtype参数。如content uri为content://com.example.trains/Line1，则provider可能为表Line1返回以下MIME类型
> vnd.android.cursor.dir/vnd.example.line1

​	相应的content URI为content://com.example.trains/Line2/5则返回
> vnd.android.cursor.item/vnd.example.line2

### Creating a Content Provider ###
创建provider步骤：
1. 为数据设计原始数据存储（raw storage）。content provider以两种方式提供数据：（1）文件数据：数据通常会转为诸如图片、音频、视频等的文件。（2）“结构化”数据：数据通常会转为数据库、数组或者相似的结构，该方式将数据存储为与表格的行列兼容的形式。
2. 定义ContentProvider类的具体实现及其方法。该类是数据及android系统其他部分间的接口。
3. 定义provider的authority字符串、content URIs及列名。如果希望由provider的应用来处理intent的话，还需要定义intent action、extra data和flags。同时，也需要定义访问数据的应用请求时需要的权限。我们应考虑将这些作为常量的值定义在一个分离的合约类（[contract class](http://developer.android.com/guide/topics/providers/content-provider-basics.html#ContractClasses)）中，然后可以将该类提供给开发者。
4. 添加可选部分，如数据样本或者实现一个可以同步provider和云数据间数据的[AbstractThreadedSyncAdapter](https://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html)。

#### Designing Data Storage ####
一个content provider是以结构化形式存储数据的接口。在Android中可用的数据存储方式有：
- SQLite数据库。[SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)可用于方便地创建数据库，[SQLiteDatabase](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html)类是是访问数据库的基本类。
- 为存储文件数据，android提供了各种面向file的API。
- 为使用基于网络的数据，可以使用[java.net](https://developer.android.com/reference/java/net/package-summary.html) 和 [android.net](https://developer.android.com/reference/android/net/package-summary.html)类。

注意事项：











[UriMatcher](https://developer.android.com/reference/android/content/UriMatcher.html)
## 使用总结 ##


限制插入table的数目（行数）：
(1) 执行SQL语句进行删除
```
public static final String SQL_LIMIT_COUNT = "DELETE FROM "
		+ MusicPlayedProvider.TABLE_NAME
		+ " where _id NOT IN (SELECT _id from "
		+ MusicPlayedProvider.TABLE_NAME
		+ " ORDER BY "
		+ MusicPlayedColumns.PLAYED_TIME
		+ " DESC LIMIT 20)";
```
(2) 创建触发器（此方法未成功，待验证）
> CREATE TRIGGER delete_till_50 INSERT ON _table WHEN (select count(*) from _table)>50 
> BEGIN
>     DELETE FROM _table WHERE _table._id IN  (SELECT _table._id FROM _table ORDER BY _table._id limit (select count(*) -50 from _table ));
> END;


```
private static final String TRIGGER_LIMIT_COUNT = "CREATE TRIGGER delete_till_20 INSERT ON "
			+ MusicPlayedProvider.TABLE_NAME
			+ " WHEN (select count(*) from "
			+ MusicPlayedProvider.TABLE_NAME
			+ ")>20 BEGIN DELETE FROM "
			+ MusicPlayedProvider.TABLE_NAME
			+ " WHERE "
			+ MusicPlayedProvider.TABLE_NAME
			+ "._id IN  (SELECT "
			+ MusicPlayedProvider.TABLE_NAME
			+ "._id FROM "
			+ MusicPlayedProvider.TABLE_NAME
			+ " ORDER BY "
			+ MusicPlayedProvider.TABLE_NAME
			+ "._id limit (select count(*) -20 from "
			+ MusicPlayedProvider.TABLE_NAME
			+ " )); END;
```