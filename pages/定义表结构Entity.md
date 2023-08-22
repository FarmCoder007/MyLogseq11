## 使用
	- ```kotlin
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
- # bean上加注解@Entity
- ## @PrimaryKey声明主键，autoGenerate=true  自增长
- ## @ColumnInfo 声明字段 name 加别名优先级更高
- ## @Ignore 表示一个属性不加入生成表的字段，只是临时使用