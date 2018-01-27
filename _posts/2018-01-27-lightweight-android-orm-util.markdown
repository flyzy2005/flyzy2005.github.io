---
layout:     post
title:      一个轻量级的Android数据库操作工具
subtitle:   封装了增删改查的轻量级ORM工具
date:       2018-01-27
author:     flyzy
header-img: "img/post-bg-2018.jpg"
tags:
    - Android
    - SQLite
    - ORM
    - 数据库操作工具
---

> 原文地址：[一个轻量级的Android数据库操作工具](https://www.flyzy2005.cn/tech/programmers/lightweight-orm-dao-util-android)

写了一个轻量级的Android操作数据库的ORM工具。方便Android定义数据库，操作数据库（增删改查），数据库更新，实现了Android对象与数据库对象之间的映射。源码地址：[轻量级Android操作数据库ORM工具](https://github.com/Flyzy2005/daoutils)。可以直接gradle依赖~

## Android数据库操作工具使用步骤 ##

 1. 用Navicat（或其他工具）新建一个SQLite数据库文件，放在工程res文件下的raw文件夹里；
 2. 继承AbstractSQLiteManger创建一个database；
 3. 为每一个表创建一个Java对象，以实现数据库表对象与数据库表对象的一一映射； 为每一个表创建一个Dao类，继承AbstractDao；
 4. 抽象类中已经实现了基本的增删改查功能，只需要传递一个表名即可使用所有方法，如果你需要自定义SQL操作，也可以获得database后自己处理。


##Android数据库操作工具实例##
1.新建一个SQLiteHelper（这里用到的test.db可以直接用工具生成）：

```
public class SQLiteHelper extends AbstractSQLiteManger {
    /**
     * 构造函数
     *
     * @param databaseName    保存的数据库文件名，如test.db
     * @param packageName     工程包名，如cn.flyzy2005.fzthelper，在工程的build.gradle文件可以看到
     * @param databaseVersion 当前的数据库版本
     * @param databaseRawId   需要写入的database文件所对应的R.raw的id，如R.id.test（把test.db文件拷贝到res下的raw下面即可）
     * @param context         ApplicationContext
     */
    public SQLiteHelper(String databaseName, String packageName, int databaseVersion, int databaseRawId, Context context) {
        super(databaseName, packageName, databaseVersion, databaseRawId, context);
    }

    @Override
    protected void updateDatabase(int oldVersion, int newVersion) {
        if(oldVersion >= newVersion)
            return;

        //根据数据库版本依次更新
        for(int i = oldVersion; i < newVersion; ++i){
            switch (i){
                case 0:
                    //更新操作包括4个步骤
                    //1.将所有表重命名成temp表 String TEMP_TABLE = "ALTER TABLE \"routeline\" RENAME TO \"_temp_routeline\"";
                    //2.建立一个新表
                    //String NEW_TABLE = "CREATE TABLE \"routeline\" (\n" +
                    //"\"ID\"  INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,\n" +
                    //        "\"userName\"  TEXT,\n" +
                    //        "\"RouteLine\"  BLOB NOT NULL,\n" +
                    //        "\"beginDate\"  TEXT NOT NULL,\n" +
                    //        "\"stopDate\"  TEXT NOT NULL,\n" +
                    //        "\"upload\"  INTEGER NOT NULL DEFAULT 0\n" +
                    //        ")";
                    //3.转移数据 String INSERT_DATA = "INSERT INTO \"routeline\" (\"ID\", \"userName\", \"RouteLine\", \"beginDate\", \"stopDate\") SELECT \"ID\", \"userName\", \"RouteLine\", \"beginDate\", \"stopDate\" FROM \"_temp_routeline\"";
                    //4.删除temp表 String DROP_TEMP = "drop table _temp_routeline";
                    //依次调用getDatabase().execSQL(sql)即可
                    break;
                case 1:

                    break;
                default:
                    break;
            }
        }
    }
}
```
2.为每一个表创建一个对象：

```
public class BaseBook {
    @PrimaryKey
    private int id;
    @Ignore
    private String test;
    @Ignore
    private String test1;
    @Ignore
    private String test2;
    //... get set
}

public class Book extends BaseBook{
    @ColumnAlias(columnName = "name")
    private String name1;
    private String author;
    private String publisher;
    //... get set
}
```
提供了注解的方式，表明该属性是否为主键，是否需要参与映射等。

3.为每一个对象创建一个数据库操作对象：

```
public class BookDao extends AbstractDao<Book> {
    public BookDao() {
        setDatabase(BaseApplication.getInstance().getDatabase());
        setTableName("book");
    }

    public void myOpera(){
        SQLiteDatabase myDatabase =  getDatabase();
        //...do anything you want with SQLiteDatabase
    }
}
```
4.此时，你就能使用所有在IBaseDao中实现的方法了：

```
interface IBaseDao<T> {
    /**
     * 根据primaryKey进行查询
     * 需要设置{@link cn.flyzy2005.daoutils.anno.PrimaryKey}
     *
     * @param primaryKey 主键的值
     * @return entity实例，没有则为null
     */
    T getByPrimaryKey(Object primaryKey);

    /**
     * 根据条件条件进行查询
     *
     * @param condition 查询条件{"name":"fly"}(封装在JSONObject中)
     * @return 满足条件的第一个实例，没有则为null
     */
    T getByParams(JSONObject condition);

    /**
     * 根据条件条件进行查询
     *
     * @param condition 查询条件{"name":"fly"}(封装在JSONObject中)
     * @return 所有满足条件的entity实例，没有则为空ArrayList
     */
    List<T> listByParams(JSONObject condition);

    /**
     * 查询表中所有数据
     *
     * @return 所有entity实例，没有则为空ArrayList
     */
    List<T> listAll();

    /**
     * 根据sql语句进行查询
     *
     * @param sql sql语句
     * @return 满足条件的第一个实例，没有则为null
     */
    T getBySql(String sql);

    /**
     * 根据sql语句进行查询
     *
     * @param sql sql语句
     * @return 满足查询条件的entity实例，没有则为空ArrayList
     */
    List<T> listBySql(String sql);

    /**
     * 添加一条记录
     *
     * @param model  entity实例
     * @param withId 是否需要插入PrimaryKey，不插入的话就采用SQLite自带的自增策略（不插入需要设置{@link cn.flyzy2005.daoutils.anno.PrimaryKey}）
     * @return 是否插入成功
     */
    boolean insert(T model, boolean withId);

    /**
     * 根据primaryKey删除一条记录
     *
     * @param model entity实例
     * @return 是否删除成功
     */
    boolean delete(T model);

    /**
     * 根据Id删除一条记录
     *
     * @param primaryKey primaryKey
     * @return 是否删除成功
     */
    boolean deleteByPrimaryKey(Object primaryKey);

    /**
     * 根据条件删除一条（多条）记录
     *
     * @param condition 删除条件
     * @return 是否删除成功
     */
    boolean deleteByParams(JSONObject condition);

    /**
     * 删除表中所有数据
     *
     * @return 是否删除成功
     */
    boolean deleteAll();

    /**
     * 修改一条记录，会根据传入的entity的primaryKey进行匹配修改，并且会用除primaryKey之外的所有其他属性的新值更新数据库
     *
     * @param model 修改的entity实例
     * @return 是否修改成功
     */
    boolean update(T model);

    /**
     * 根据条件修改一条记录，并且会用除primaryKey的所有其他属性的新值更新数据库
     *
     * @param model     修改的entity实例
     * @param condition 条件
     * @return 是否修改成功
     */
    boolean updateByParams(T model, JSONObject condition);

    /**
     * 所有行数
     *
     * @return int
     */
    int count();

    /**
     * 满足条件的所有行数
     *
     * @param condition 条件
     * @return int
     */
    int countByParams(JSONObject condition);
}
```
##获取方法##
直接在gradle中配置：
```
compile 'cn.flyzy2005:daoutils:1.0.1'
```

##总结##
一个简单的Android操作数据库的轻量级ORM对象~对一些简单的Android项目还是挺有帮助的，欢迎star，一起学习~