# 概念
	- Room是对SQLite的封装，流畅的访问数据库
- # 优点
	- 1、内部做优化，性能提升
	- 2、使用简单
- # 一、[[Room数据库结构]]
- # 二、示例：
	- ## [[定义表结构Entity]]
	- ## [[定义数据库操作 Dao]]
	- ## [[定义RoomDatabase抽象类]]
	- ## [[使用上边定义的数据库]]
- # 三、基本使用
  collapsed:: true
	- ## 返回例的子集
	  collapsed:: true
		- ```java
		  public class NameTuple {
		      @ColumnInfo(name="first_name")
		      public String firstName;
		  
		      @ColumnInfo(name="last_name")
		      public String lastName;
		  }
		  
		  @Dao
		  public interface MyDao {
		      @Query("SELECT first_name, last_name FROM user")
		      public List<NameTuple> loadFullName();
		  }
		  ```
	- ## 表与表之间的实体联系
	  collapsed:: true
		- ```java
		  设置外键
		  @Entity(foreignKeys = @ForeignKey(entity = User.class,
		                                    parentColumns = "id",
		                                    childColumns = "user_id"))
		  public class Book {
		      @PrimaryKey
		      public int bookId;
		  
		      public String title;
		  
		      @ColumnInfo(name = "user_id")
		      public int userId;
		  }
		  ```
	- ## 创建嵌套对象@ Embedded
		- ```java
		  public class Address {
		      public String street;
		      public String state;
		      public String city;
		  
		      @ColumnInfo(name = "post_code")
		      public int postCode;
		  }
		  
		  @Entity
		  public class User {
		      @PrimaryKey
		      public int id;
		  
		      public String firstName;
		  
		      @Embedded
		      public Address address;
		  }
		  ```
	- ## 传递参数集合
	  collapsed:: true
		- ```java
		  @Dao
		  public interface MyDao {
		      @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
		      public List<NameTuple> loadUsersFromRegions(List<String> regions);
		  }
		  ```
	- ## 可观察的查询
	  collapsed:: true
		- ```java
		  @Dao
		  public interface MyDao {
		      @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
		      public LiveData<List<User>> loadUsersFromRegionsSync(List<String> regions);
		  }
		  ```
	- ## 可观察的查询
	  collapsed:: true
		- ```java
		  @Dao
		  public interface MyDao {
		     @Query("SELECT user.name AS userName, pet.name AS petName "
		            + "FROM user, pet "
		            + "WHERE user.id = pet.user_id")
		     public LiveData<List<UserPet>> loadUserAndPetNames();
		  
		  
		     // You can also define this class in a separate file, as long as you add the
		     // "public" access modifier.
		     static class UserPet {
		         public String userName;
		         public String petName;
		     }
		  }
		  ```
	- ## 支持Rxjava
	  id:: 64d5f4c6-ed9e-4d1a-acfa-8e90a7ffc962
	  collapsed:: true
		- ```java
		  @Dao
		  public interface MyDao {
		      @Query("SELECT * from user where id = :id LIMIT 1")
		      public Flowable<User> loadUserById(int id);
		  }
		  ```
	- ## 返回Cursor
	  collapsed:: true
		- ```java
		  @Dao
		  public interface MyDao {
		      @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
		      public Cursor loadRawUsersOlderThan(int minAge);
		  }
		  ```
	- ## [[Room数据库迁移]]
	  collapsed:: true
	- ## [[ViewModel+livedata+database使用]]
- # 四、踩坑指南
	- ## 问题一：
	  collapsed:: true
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
	  collapsed:: true
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
- # [[Room-面试题]]