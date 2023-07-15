- 再看其他方法的实现就很简单了，无非就是构造一个 Op，设置对应的状态值。doAddOp：[[doAddOp]]
- ```java
  @Override
  public FragmentTransaction remove(Fragment fragment) {
      Op op = new Op();
      op.cmd = OP_REMOVE;
      op.fragment = fragment;
      addOp(op);
      return this;
  }
  @Override
  public FragmentTransaction hide(Fragment fragment) {
      Op op = new Op();
      op.cmd = OP_HIDE;
      op.fragment = fragment;
      addOp(op);
      return this;
  }
  @Override
  public FragmentTransaction show(Fragment fragment) {
      Op op = new Op();
      op.cmd = OP_SHOW;
      op.fragment = fragment;
      addOp(op);
      return this;
  }
  ```