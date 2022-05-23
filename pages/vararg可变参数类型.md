- 调用含有可变参数类型的方法时，需要传*
- ```
  fun main(args: Array<String>) {
      invokMsg("AAA", "BBB", "CCC", "DDD")
  }
  
  fun invokMsg(vararg value: Any?){
      printlnMsg(value)
      printlnMsg(*value)
  }
  
  fun printlnMsg(vararg msg: Any?){
      println(msg.joinToString(" "))
  }
  ```
- ```
  //输出了[Ljava.lang.Object;@a09ee92（类的对象名称）
  [Ljava.lang.Object;@a09ee92
  // 带*输出了值
  AAA BBB CCC DDD
  
  Process finished with exit code 0
  ```
-