### NestedScrollingParentHelper
```java
public class NestedScrollingParentHelper {
//兼容NestedScrollingParent2的NestedScrollType
  private int mNestedScrollAxesTouch;
  private int mNestedScrollAxesNonTouch;


  public NestedScrollingParentHelper(@NonNull ViewGroup viewGroup) {
  }

  public void onNestedScrollAccepted(@NonNull View child, @NonNull View target,
          @ScrollAxis int axes) {
          //默认就是TYPE_TOUCH
      onNestedScrollAccepted(child, target, axes, ViewCompat.TYPE_TOUCH);
  }


  public void onNestedScrollAccepted(@NonNull View child, @NonNull View target,
          @ScrollAxis int axes, @NestedScrollType int type) {
      if (type == ViewCompat.TYPE_NON_TOUCH) {
          mNestedScrollAxesNonTouch = axes;
      } else {
          mNestedScrollAxesTouch = axes;
      }
  }


  @ScrollAxis
  public int getNestedScrollAxes() {
      return mNestedScrollAxesTouch | mNestedScrollAxesNonTouch;
  }

  public void onStopNestedScroll(@NonNull View target) {
      onStopNestedScroll(target, ViewCompat.TYPE_TOUCH);
  }


  public void onStopNestedScroll(@NonNull View target, @NestedScrollType int type) {
      if (type == ViewCompat.TYPE_NON_TOUCH) {
          mNestedScrollAxesNonTouch = ViewGroup.SCROLL_AXIS_NONE;
      } else {
          mNestedScrollAxesTouch = ViewGroup.SCROLL_AXIS_NONE;
      }
  }
}
```