- 1、PKM[[#red]]==**S构造函数中**==调用scanDirTracedLi[[#red]]==**开启APK扫描**==
- 2、扫描==**收集手机中的所有APK,交给PackageParser 解析**==
- 3、[[#red]]==**对APK的AndroidManifest.xml进行解析， 把所有的标签信**==息"application"、"permission"、"package"、"manifest" [[#red]]==**解析后放在 Package对象**==
- 4、==**解析Application标签中的四大组件，放在Package对象中**==，由于一个 APK 可声明多个组件，因此activites 和 receivers等均声明为 ArrayList