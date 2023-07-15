- FragmentContainer 就是一个接口，定义了关于布局的两个方法：
- ```java
  public abstract class FragmentContainer {
      @Nullable
      public abstract View onFindViewById(@IdRes int id);
      public abstract boolean onHasView();
  }
  ```