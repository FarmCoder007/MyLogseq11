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
- ## @Dao标识 Dao
	- ```java
	  @Dao
	  public interface StudentDao {
	  .....
	  }
	  ```
- ## @Query 标识查询操作
  collapsed:: true
	- ```java
	  @Query("select * from Student")
	      List<Student> getAll();
	  ```
	- ## 可以把参数加入查询语句
		- ```java
		  	//查询一条记录
		      @Query("select * from Student where name like:name")
		      Student findByName(String name);
		      //查找部份ID号的记录
		      @Query("select * from Student where uid in (:userIds)")
		      List<Student> getAllId(int[] userIds);
		  ```
- ## @Insert 插入
  collapsed:: true
	- ```java
	  @Insert
	      void insert(Student... students);
	  ```
- ## @Delete 删除
  collapsed:: true
	- ```java
	   @Delete
	      void delete(Student student);
	  ```
- ## @Update 更新
  collapsed:: true
	- ```java
	  @Update
	      void update(Student student);
	  
	  ```