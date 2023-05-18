- ## 背景
  collapsed:: true
	- 作为 Android 开发者，我们对 Activity、Service 等系统组件的生命周期非常熟悉，在日常开发中也会利用组件的生命周期特点开发业务逻辑。
	- Flutter 中有“一切皆Widget”的说法，Widget 是 Flutter 功能的抽象描述，是视图的配置信息，也是数据的映射。Android 开发中的 View、Layout、Activity、Application等，对应到 Flutter 中都是 Widget。
	- Flutter 的 Widget 有 StatelessWidget 和 StatefulWidget 两种类型，StatelessWidget 用于处理静态的、无状态变化的视图展示，而 StatefulWidget 应对有交互、需要动态变化视觉效果的场景。
	- StatelessWidget 通过父 Widget 初始化时传入的静态配置就能完全控制其静态展示，而 StatefulWidget 需要借助于 State 对象，在特定阶段来处理用户交互或内部数据的变化，然后更新到 UI 上。这些特定的阶段涵盖了一个 Widget 从初始化到卸载的全过程，即生命周期。Flutter 中的生命周期即指的 StatefulWidget 的生命周期，并通过 State 对象来体现，StatelessWidget 的生命周期只有 build 过程，且只会执行一次。
- ## 生命周期方法
	- Flutter中的生命周期方法主要包括以下内容：
	-