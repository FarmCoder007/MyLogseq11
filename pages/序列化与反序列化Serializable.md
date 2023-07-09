- Serializable 的序列化与反序列化分别通过 ObjectOutputStream 和 ObjectInputStream 进行
- ```java
  /**
  * 序列化对象
  *
  * @param obj
  * @param path
  * @return
  */
  synchronized public static boolean saveObject(Object obj, String path) {
      if (obj == null) {
      	return false;
      }
      ObjectOutputStream oos = null;
      try {
          // 创建序列化流对象
          oos = new ObjectOutputStream(new FileOutputStream(path));
          //序列化
          oos.writeObject(obj);
          oos.close();
          return true;
      } catch (IOException e) {
      	e.printStackTrace();
      } finally {
          if (oos != null) {
              try {
                  // 释放资源
                  oos.close();
              } catch (IOException e) {
          		e.printStackTrace();
          	}
      	}
      }
      return false;
  }
  /**
  * 反序列化对象
  *
  * @param path
  * @param <T>
  * @return
  */
  @SuppressWarnings("unchecked ")
  synchronized public static <T> T readObject(String path) {
      ObjectInputStream ojs = null;
      try {
          // 创建反序列化对象
          ojs = new ObjectInputStream(new FileInputStream(path));
          // 还原对象
          return (T) ojs.readObject();
      } catch (IOException | ClassNotFoundException e) {
      	e.printStackTrace();
      } finally {
          if(ojs!=null){
              try {
                  // 释放资源
                  ojs.close();
              } catch (IOException e) {
              	e.printStackTrace();
              }
          }
      }
      return null;
  }
  ```