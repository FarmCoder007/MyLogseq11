- 1. 单例提供 数据库的实例
  2. 设置数据库 的名字
  3. 提供各个表的Dao
  4. 数据库的升级
  5. entities定义表
- # 示例
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
- # 注解@Database标识
	- ## entities={Student.class} 定义数据库中包含的表
	- ## version=1 数据库版本号