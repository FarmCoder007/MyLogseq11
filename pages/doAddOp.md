- 将Fragment 和命令 添加到操作列表
- ```java
  private void doAddOp(int containerViewId, Fragment fragment, String tag, int opcmd) {
      final Class fragmentClass = fragment.getClass();
      final int modifiers = fragmentClass.getModifiers();
      if (fragmentClass.isAnonymousClass() || !Modifier.isPublic(modifiers)
                          || (fragmentClass.isMemberClass() && !Modifier.isStatic(modifiers))) {
          throw new IllegalStateException("Fragment " + fragmentClass.getCanonicalName()
          + " must be a public static class to be properly recreated from"+ " instance state.");
      }
      //1.修改添加的 fragmentManager 为当前栈的 manager
      fragment.mFragmentManager = mManager;
      if (tag != null) {
          if (fragment.mTag != null && !tag.equals(fragment.mTag)) {
              throw new IllegalStateException("Can't change tag of fragment "
              + fragment + ": was " + fragment.mTag + " now " + tag);
          }
          fragment.mTag = tag;
      }
      if (containerViewId != 0) {
          if (containerViewId == View.NO_ID) {
              throw new IllegalArgumentException("Can't add fragment "
                              + fragment + " with tag " + tag + " to container view with no id");
          }
      if (fragment.mFragmentId != 0 && fragment.mFragmentId != containerViewId) {
          throw new IllegalStateException("Can't change container ID of
          fragment " + fragment + ": was " + fragment.mFragmentId + " now " + containerViewId);
      }
      //2.设置宿主 ID 为布局 ID
      fragment.mContainerId = fragment.mFragmentId = containerViewId;
      }
      //3.构造 Op
      Op op = new Op();
      op.cmd = opcmd; //状态
      op.fragment = fragment;
      //4.添加到数组列表中
      addOp(op);
  }
  ArrayList   mOps = new ArrayList()    
   
  // 将操作命令 和 Fragment 放入 操作列表mOps
  void addOp(Op op) {
      mOps.add(op);
      op.enterAnim = mEnterAnim;
      op.exitAnim = mExitAnim;
      op.popEnterAnim = mPopEnterAnim;
      op.popExitAnim = mPopExitAnim;
  }
  ```