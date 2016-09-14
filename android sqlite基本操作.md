sqlite是android内置的关系数据库。

网上的基础教程比较多，此处主要讲如何在项目中搭建数据库代码基础架构，以API的形式给APP使用

我们知道，APP中使用数据库其实主要就那么几个操作
1. 建立数据表
2. 存储数据（包括插入数据，修改数据）
3. 删除数据
4. 查询数据（包括全部查询和条件查询）

因此我们使用方法也可以对应：
##### 1. 初始化数据库管理类 
我们知道android提供了一个SQLiteOpenHelper来完成对sqlite的读取，初始化，以及更新。
我们一般使用一个独立类DBManger来管理sqlite，DBManger其实也就是通过SQLiteOpenHelper管理数据库。一般情况下，我们这样来读取并初始化SQLiteOpenHelper。
```
public class DBHelper extends SQLiteOpenHelper {
}
```
```
public class DBManager {
    private DBHelper helper;
    private SQLiteDatabase db;
    public DBManager(Context context) {
        helper = new DBHelper(context);
        db = helper.getWritableDatabase();
    }
   
    public void closeDB() {
        db.close();
    }
}
```
##### 2. 数据表的建立和更新
数据表只在第一次使用即创建数据库的时候（SQLiteOpenHelper的onCreate）会建立。而数据表结构的更新（如加列删列）会在SQLiteOpenHelper的onUpdate中调用。
```
public class DBHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "test.db";
    private static final int DATABASE_VERSION = 1;

    public DBHelper(Context context) {
        //CursorFactory设置为null,使用默认值
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    //数据库第一次被创建时onCreate会被调用
    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE IF NOT EXISTS facedb" +
                "(_id INTEGER PRIMARY KEY AUTOINCREMENT, name VARCHAR, iscurrent INTEGER)");
    }

    //如果DATABASE_VERSION值被改为2,系统发现现有数据库版本不同,即会调用onUpgrade
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("ALTER TABLE facedb ADD COLUMN other STRING");
    }
}
```

##### 3. 各种数据操作的API
###### 3.1. add 操作
```
    public void add(String name) {
        db.beginTransaction();    //开始事务
        try {
            db.execSQL("INSERT INTO facedb VALUES(null, ?,?)", new Object[]{name, false});
            db.setTransactionSuccessful();    //设置事务成功完成
        } finally {
            db.endTransaction();    //结束事务
        }
    }
```
###### 3.2. update操作
```
    public void updateIscurrent(String oriName) {
        String sql1 = "update facedb set iscurrent = 0";//所有元素的iscurrent都置为0
        db.execSQL(sql1);//执行修改
        String sql2 = "update facedb set iscurrent = 1 where name = '" + oriName + "'";// name=oriName的元素iccurrent置为1
        db.execSQL(sql2);//执行修改
    }
```
###### 3.3. delete操作
```
    public void delete(String name) {
        db.beginTransaction();    //开始事务
        try {
            db.delete("facedb", "name = ?", new String[]{name});
            db.setTransactionSuccessful();    //设置事务成功完成
            } 
        finally {
            db.endTransaction();    //结束事务
        }
    }
```
###### 3.4. select操作 android提供一个Cursor类来保存查询结果。
1. 查询所有数据
```
    public Cursor queryTheCursor() {
        Cursor c = db.rawQuery("SELECT * FROM facedb", null);
        return c;
    }
```
2. 查询所有数据的某个元素的集合
```
    public List<String> queryNameList() {
        Cursor cursor = queryTheCursor();
        List<String> list = new ArrayList<>();
        if (cursor.getCount() > 0) {
            do {
                String name = cursor.getString(cursor.getColumnIndex("name"));
                list.add(name);
            } while (cursor.moveToNext());
        }
        return list;
    }
```
3. 条件查询
```
    public List<String> queryisCurrent() {
        List<String> list = new ArrayList<>();
        Cursor cursor = db.rawQuery("SELECT * FROM facedb where iscurrent = 1", null);
        cursor.moveToFirst();
        if (cursor.getCount() > 0) {
            while (cursor.moveToNext()){
                int index = cursor.getColumnIndex("name");
                String name = cursor.getString(index);
                list.add(name);
           }
        }
        return list;
    }
```