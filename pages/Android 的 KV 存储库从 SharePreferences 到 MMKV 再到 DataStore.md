- # 概述
	- 在我们日常开发的项目中，存在大量的KV（Key-Value）数据读写，这些数据都会以某种文件格式保存来实现数据的持久化。读写文件是一种耗时的操作，特别是非常频繁地写入或者要写入的数据非常大时容易发生IO阻塞，从而引起App的卡顿甚至ANR异常。
	- Android SDK 从最初的版本就提供了 SharedPreferences API （通常简称为SP）来保存键值对数据，在大量开发者的应用实践中发现 SP 容易产生 ANR 问题；微信团队为了解决微信中特定的数据保存问题开发并在 2018 年开源了 MMKV，很多应用也接入了 MMKV 并提升了 KV 数据存储的效率；Android 官方在 2020 年又推出了一个 Jetpack 组件 DataStore，是改进版本的数据存储解决方案，它的目标是取代 SharedPreferences。
	- 那么 KV 存储库从 SP 到 MMKV 再到 DataStore，他们分别有什么特点和价值？这些存储方案一路升级有什么收益？
- # SharedPreferences
  collapsed:: true
	- SharedPreferences 是 Android 平台上轻量级的存储类，用来保存 App 的各种配置信息，其本质是一个以键值对（key-value）的方式保存数据的 xml 文件，被保存在 /data/data/shared_prefs 目录下。将数据以xml文件的键值对形式保存，每次读取数据的时候，解析xml文件得到指定 key 对应的 value，每次更新数据也通过文件中的 key 更新对应的 value。
	- 每次读写数据都是操作文件，其性能势必有很大的问题，在SharedPreferences的实现上对读写操作也有一些优化。
	- 当调用Context.getSharedPreferences()对SP进行初始化时对xml文件进行一次读取，并将文件中的所有内容缓存到内存中，接下来的所有读操作只需要从这个缓存Map中取就可以了。这其实是一种空间换时间的策略，当xml文件很大时，这种内存缓存机制就会产生内存占用过高的问题。
	- 所以SP不适合在一个文件中存储过多内容，在实际开发中可以根据业务范围定义一些轻量的SP文件。
	- 通常我们对SP进行更新是通过 mSharedPreferences.edit().putString().commit() 进行操作的。为什么首先需要通过SP对象获取edit，然后再更新键值对，最后提交更新呢？SP设计者希望，在复杂业务中将多次更新文件合并到一次写操作中，以达到性能的优化。
	- 对SP的写操作抽象了一个Editor类，在一次写操作中不管调用多少次putXXX()方法，更新了xml文件中的多少键值对，只有调用了commit方法后才会真正写入xml文件。
	  collapsed:: true
		- ```
		  // 简单的业务，一次更新一个键值对
		  sharedPreferences.edit().putString().commit();
		  
		  // 复杂的业务，一次更新多个键值对，仍然只进行一次IO操作（文件的写入）
		  Editor editor = sharedPreferences.edit();
		  editor.putString();
		  editor.putBoolean().putInt();
		  editor.commit(); // commit()才会更新文件
		  
		  ```
	- 直接在主线程中执行的commit()方法当要更新的数据量过大时会导致ANR，因此考虑将数据更新放在子线程执行，SP在Editor中提供了一个apply()方法，用于异步执行文件数据的同步。
- # MMKV
  collapsed:: true
	- MMKV 是基于 mmap 内存映射的 key-value 组件，底层采用 protobuf 格式实现数据序列化/反序列化，具有性能高、稳定性强并且支持跨平台的特点，据说在微信上从 2015 年开始使用至今。MMKV 最初开发出来是为了解决微信客户端中遇到特殊文字引起系统的 crash 的问题，现在已经成为了提供高性能的 KV 数据持久化的解决方案。
	- MMKV 库对外公开的接口类 MMKV 实现了接口 SharedPreferencs 和 Editor，所以在接入了 MMKV 的应用中可以像使用 SP API 那样进行数据的写入和读取，只是在调用 put 方法后不需要再调用 commit() 或者 apply() 方法；也可以使用 encode 系列重载方法写数据，然后用对应的 decode 方法读数据。MMKV 的使用非常简单，所有变更立马生效，无需调用 sync、apply。
	- ```
	  MMKV.initialize(context)
	  
	  val mmkv = MMKV.defaultMMKV()
	  mmkv.putInt("key_my_mmkv_1", 10)
	  val putVal = mmkv.getInt("key_my_mmkv_1", 0)
	  
	  mmkv.encode("key_mmkv_encode", "abcd")
	  val encodeVal = mmkv.decodeString("key_mmkv_encode")
	  ```
- # Jetpack DataStore
  collapsed:: true
	- DataStore 是 Android 团队在2020年9月份发布的一个提供数据存储解决方案的 Jetpack 组件，它使用 Kotlin 协程和 Flow 通过异步、保持一致性和事务方式实现数据存储。DataStore 提供了两种不同的实现：Preferences DataStore 和 Proto DataStore，前者使用键值对的方式存储数据，后者将数据作为自定义数据类型的实例进行存储。
	- 这里只介绍 KV 数据的存储，即 Preferences DataStore 的使用。首先需要通过 preferencesDataStore 创建的属性委托来创建 Datastore<Preferences> 实例，在 Kotlin 文件顶层调用一次该实例，然后就可以在其他地方轻松访问该 DataStore 的单例。使用 Preferences DataStore 存储数据需要使用与存储值的类型对应的键类型函数定义一个 key，比如 int 值对应的 key 通过 intPreferencesKey() 创建，String 类型对应的 key 由 stringPreferencesKey() 创建。写入数据通过 DataStore 实例的 edit() 函数以事务方式更新其中的数据，读取数据使用 DataStore.data 属性，通过 Flow 提供相应的存储值。
	- ```
	  // 在Kotlin文件顶层定义dataStore实例
	  val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
	  
	  // 写入数据
	  suspend fun editDataStore() {
	      dataStore.edit { settings ->
	          settings[intPreferencesKey("key_datastore_int")] = 256
	          settings[stringPreferencesKey("key_datastore_str")] = "data"
	      }
	  }
	  
	  // 读取数据
	  val dataInt: Flow<Int> = context.dataStore.data.map { value ->
	      value[intPreferencesKey("key_datastore_int")] ?: 0
	  }
	  
	  val dataStr: Flow<String> = context.dataStore.data.map { value ->
	      value[stringPreferencesKey("key_datastore_str")] ?: ""
	  }
	  ```