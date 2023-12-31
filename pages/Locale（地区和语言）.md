- `Locale` 是 Java 中用于表示地区和语言信息的类。它包含了一组用于标识特定地区和语言的信息。`Locale` 对象通常包含以下信息：
  title:: Locale（地区和语言）
- 1、**语言（Language）：** 表示使用的语言，由 ISO 639 标准定义的两个字母语言代码表示，例如 "en" 表示英语，"fr" 表示法语。
- 2、**国家/地区（Country/Region）：** 表示地区，由 ISO 3166 标准定义的两个字母或三个字母的国家/地区代码表示，例如 "US" 表示美国，"FR" 表示法国。
- 3、**变体（Variant）：** 表示一些特定变体或定制的信息，通常是一个字符串。
- 4、**脚本（Script）：** 表示使用的写作系统，由 ISO 15924 标准定义的四个字母脚本代码表示。
- `Locale` 对象的实例可以通过以下方式创建：
	- ```kotlin
	  // 创建一个代表美国英语的 Locale
	  Locale usLocale = new Locale("en", "US");
	  
	  // 获取当前系统的默认 Locale
	  Locale defaultLocale = Locale.getDefault();
	  
	  ```