- # 概念
	- Room是对SQLite的封装，流畅的访问数据库
- # 优点
	- 1、内部做优化，性能提升
	- 2、使用简单
- # 一、Room数据库结构
	- 如下图，Room数据库包含三个对象：
	  1. Entity : 对应数据库中的表，可以使用Entity注解将一个类变成数据库中的一张表结构。
	  2. DAO : 全称Database Access Object,定义了对数据库中数据的读写等操作，DAO中可以使用SQL语句来操作数据库。
	  3. RoomDatabase : 数据库持有类，用于创建数据库或者连接到数据库。内部包含DAO和Entity。
	- ![image.png](../assets/image_1684224407502_0.png)
# 二、示例：
	- ## 定义表结构Entity
	  ```kotlin
	  @Entity
	  class Person(
	              @PrimaryKey(autoGenerate = true) var id: Int,
	              @ColumnInfo(name = "name") // 加别名优先级更高
	     			var name: String,
	              var value: String,
	              var age : Int
	      ) {
	  
	   override fun toString(): String {
	   	 return "Person(id=$id,name=$name,value=$value)"
	   }
	  }
	  ```
	- ## 定义数据库操作 Dao
		- 查询：返回 LiveData 或 Flowable，无需放在工作线程，此类查询Room会会根据需要在后台线程上异步运行查询。 
		  插入、删除、更改等操作需要借助协程、rxjava等切换线程操作
		  ```kotlin
		  @Dao
		  public interface PersonDao{
		      @Query("SELECT * FROM Person")
		      fun loadAllPersons(): LiveData<List<Person>>
		  
		      @Insert(onConflict = OnConflictStrategy.REPLACE)
		      suspend fun insertAll(products: List<Person>)
		  
		      @Query("select * from Person where id = :id")
		      fun loadPerson(id: Int): LiveData<Person>
		  
		      @Insert(onConflict = OnConflictStrategy.IGNORE)
		      suspend fun insert(person: Person)
		  
		      @Query("DELETE FROM Person")
		      suspend fun deleteAll()
		  }
		  ```
	- ## 定义RoomDatabase
		- 1. 单例提供 数据库的实例
		  2. 设置数据库 的名字
		  3. 提供各个表的Dao
		  4. 数据库的升级
		  5. entities定义表
		- ```kotlin
		  @Database(entities = [TestEntity::class, Person::class], version = 2,exportSchema = false)
		  abstract class MasonDatabase : RoomDatabase() {
		  
		  abstract fun getTestDao(): TestDao
		  
		  abstract fun getPersonDao():PersonDao
		  
		  companion object {
		      /**
		       *  数据库从版本1 -> 迁移到版本2 ：Person表添加1列last_update
		       */
		      private val MIGRATION_1_2: Migration = object : Migration(1, 2) {
		          override fun migrate(database: SupportSQLiteDatabase) {
		              database.execSQL(
		                  "ALTER TABLE Person ADD COLUMN age INTEGER"
		              )
		          }
		      }
		  
		      private const val DB_NAME = "MasonDatabase.db"
		  
		      @Volatile
		      private var masonDatabase: MasonDatabase? = null
		  
		      /**
		       *  创建数据库实例
		       */
		      @Synchronized
		      fun getInstance(context: Context): MasonDatabase {
		          if (masonDatabase == null) {
		              masonDatabase = create(context)
		          }
		          return masonDatabase!!
		      }
		  
		  
		      fun create(context: Context): MasonDatabase {
		          return Room.databaseBuilder(context, MasonDatabase::class.java, DB_NAME)
		              .addMigrations(MIGRATION_1_2).build()
		      }
		  }
		  }
		  ```
# 三、踩坑指南
	- ## 问题一：
		- 按照官方文档集成
		  
		  ```
		  dependencies {
		  def room_version = "2.3.0"
		  implementation "androidx.room:room-runtime:$room_version"
		  annotationProcessor "androidx.room:room-compiler:$room_version"
		  }
		  ```
		  
		  编译不会报错  运行报错：
		  
		  ```
		  Caused by: java.lang.RuntimeException: cannot find implementation for com.wuba.wuxian.wubamason.room.MasonDatabase. MasonDatabase_Impl does not exist
		  ```
		- ## 解决方案
		  collapsed:: true
			- ### 第一步：检查注释是否添加
			  确保注释是否都已经添加，并且确保注释内容是否正确.
			  @Database：表示数据库.
			  @Entity：表示数据库中的表。
			  @DAO：包含用于访问数据库的方法。
			  如果注释添加错误也会有以上错误。
			- ### 第二步：检查依赖是否添加
			  
			  ```
			  //Android官网依赖是这样的，java开发人员使用
			  compile "android.arch.persistence.room:runtime:$room_version"
			  annotationProcessor 'android.arch.persistence.room:compiler:$room_version'
			  
			  //对于那些使用Kotlin的人，请尝试在应用中更改annotationProcessor为kapt
			  compile "android.arch.persistence.room:runtime:$room_version"
			  kapt "android.arch.persistence.room:compiler:$room_version"
			  
			  //如果您已迁移到androidx
			  implementation "androidx.room:room-runtime:$room_version"
			  implementation "androidx.room:room-ktx:$room_version"
			  kapt "androidx.room:room-compiler:$room_version"
			  ```
			  如果使用了kotlin项目，不要忘记在顶部引用kotlin-kapt插件
			  
			  ```
			  apply plugin: 'kotlin-kapt'
			  ```
			- ### 三是否是多模块项目
			  如果项目包含多个模块，在使用RoomDataBase的那个模块中，同样需要添加kapt依赖。
			  
			  ```
			  apply plugin: 'kotlin-kapt'
			  dependencies {
			        kapt  "androidx.room:room-compiler:$room_version"
			  }
			  ```
	- ## 问题二：
		- AS： M1预览版不兼容
		  
		  room:根目录gradle:
		  ```
		   ext.roomVersion = "2.3.0"
		  ```
		  app：
		  
		  ```
		  implementation "androidx.room:room-runtime:$roomVersion"
		  implementation "androidx.room:room-ktx:$roomVersion"
		  kapt "androidx.room:room-compiler:$roomVersion"
		  ```
		  编译报错：Caused by: java.lang.Exception: No native library is found for os.name=Mac and os.arch=aarch64.
		- ### 原因：SQLite原生库没有对Apple M1 支持
		- ### 解决方案一：添加依赖
		  kapt'org.xerial:sqlite-jdbc:3.34.0'
		- ### 解决方案二：
		  现在在Room最新版 2.4.0-alpha03修复了这个问题。
		  
		  所以依赖 Room 2.4.0-alpha03或以上版本：
		  
		  ```
		  implementation "androidx.room:room-runtime:2.4.0-alpha03"
		  kapt "androidx.room:room-compiler:2.4.0-alpha03"
		  ```