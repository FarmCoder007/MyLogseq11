- 1、是view 和 wms的桥梁
- 2、一个窗口只对应一个ViewRootImpl
- 3、window操作view实际是通过ViewRootImpl实现
-
- # 其他
	- 1、addView时将view相关参数（并不是实例） 通过binder-> wms
	- 2、创建了个surface(底层的也可以称作画布)-》上层是canvas