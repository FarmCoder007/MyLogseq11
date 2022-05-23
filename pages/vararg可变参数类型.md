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
  ```