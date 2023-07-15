- Fragment 后退栈中的一个元素
- ```java
  //后退栈中的一个元素
  public interface BackStackEntry {
    //栈中该元素的唯一标识
    public int getId();
    //获取 FragmentTransaction # addToBackStack(String) 设置的名称
    public String getName();
    @StringRes
    public int getBreadCrumbTitleRes();
    @StringRes
    public int getBreadCrumbShortTitleRes();
    public CharSequence getBreadCrumbTitle();
    public CharSequence getBreadCrumbShortTitle();
  }
  ```
-