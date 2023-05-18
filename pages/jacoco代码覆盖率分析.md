# 一、什么是代码覆盖率？

代码覆盖（Code coverage）是软件测试中的一种度量，描述程序中源代码被测试的比例和程度，所得比例称为代码覆盖率。
简单来理解，就是单元测试中代码执行量与代码总量之间的比率，跑了一个测试用例，项目代码中的哪些模块、文件、类、方法 执行了。

其中行覆盖率是最细颗粒度，其他覆盖率都可从行覆盖率情况统计算出来。
# 二、常见代码覆盖率度量方式
## 1、行覆盖

又称语句覆盖(StatemenCoverage)、段覆盖(SegmentCoverage)、基本块覆盖(BasicBlockCoverage)， 这是最常用也是最常见的一种覆盖方式。测试行为，触发了这某一行代码，则这一行代码就被覆盖了
行覆盖常常被人指责为“最弱的覆盖”，它只管覆盖代码中的执行语句，很难更多地发现代码中的问题。例如
```kotlin
fun test(a:Int,b:Int):Int{
return a/b
}
```
TeseCase: a = 10, b = 5，行覆盖维度，代码覆盖率达到了100％。却未发现b = 0的 bug。
## 2、分支覆盖
  
又称为判定覆盖(DecisionCoverage)，所有边界覆盖(All-EdgesCoverage)，基本路径覆盖(BasicPathCoverage)，判定路径覆盖(Decision-Decision-Path)
if 和 switch 语句算作分支覆盖率，这个指标计算一个方法中的分支总数，并决定已执行和未执行的分支的数量。度量程序中每一个判定的分支是否都被测试到了
  
``` kotlin
fun test (a:Int,b:Int):Int{
  if (a < 10 || b < 10) // 判定
  {
      return 0 // 分支一
  } else {
      return 1 // 分支二
  }
}
```
设计分支覆盖案例时，我们只需要考虑判定结果为true和false两种情况，因此，我们设计如下的案例就能达到判定覆盖率100％：
  
TestCaes1: a = 5, b ＝ 任意数字 覆盖了分支一
TestCaes2: a = 15, b = 15 覆盖了分支二
## 3、条件覆盖

它度量判定中的每个子表达式结果true和false是否被测试到了。条件覆盖针对判断语句里面案例的取值都要去一次，不考虑条件的取值。

```kotlin
fun test (a:Int,b:Int):Int{
if (a < 10 || b < 10) // 判定
{
    return 0 // 分支一
} else {
    return 1 // 分支二
}
}
```

设计条件覆盖案例时，我们需要考虑判定中的每个条件表达式结果，为了覆盖率达到100％，我们设计了如下的案例：

TestCase1: a = 5, b = 5 true, true
TestCase4: a = 15, b = 15 false, false
## 4、路径覆盖(PathCoverage)

又称断言覆盖(PredicateCoverage)。它度量了是否函数的每一个分支都被执行了。 就是所有可能的分支都执行一遍，有多个分支嵌套时，需要对多个分支进行排列组合

```kotlin
fun test(a: Int, b: Int): Int {
var nReturn: int = 0
if (a < 10) {
    nReturn += 1
}
if (b < 10) {
    nReturn += 10
}
return nReturn
}
```

a. 语句覆盖

TestCase a = 5, b = 5 nReturn = 11   //语句覆盖率100％

b. 判定覆盖

TestCase1 a = 5, b = 5 nReturn = 11
TestCase2 a = 15, b = 15 nReturn = 0 //判定覆盖率100％

c. 条件覆盖

TestCase1 a = 5, b = 15 nReturn = 1
TestCase2 a = 15, b = 5 nReturn = 10 //条件覆盖率100％

d.路径覆盖

TestCase1 a = 5, b = 5 nReturn = 0
TestCase2 a = 15, b = 5 nReturn = 1
TestCase3 a = 5, b = 15 nReturn = 10
TestCase4 a = 15, b = 15 nReturn = 11 //路径覆盖率100％
# 三、Jacoco介绍

Jacoco是Java Code Coverage的缩写，是Java代码覆盖率统计的主流工具之一。通过ASM修改字节码，插入Jacoco插桩代码，以实现代码覆盖率的统计和分析。JaCoCo支持两种插桩模式
## On-the-fly动态插桩模式：

On-the-fly插桩指的是在**应用程序运行时**，Jacoco通过Java Agent（详解[InstrumentAPI](InstrumentAPI)）在内存中动态修改目标类的字节码，插入埋点代码。插桩时机是应用程序运行时，不需要重新编译和打包应用程序，因此比较灵活，可以随时启用和停用Jacoco插桩。但由于是在内存中修改字节码，可能会增加应用程序的运行时开销。
## Offline离线插桩模式：

Offline插桩指的是在**应用程序编译时**，Jacoco将插桩代码注入到目标类的字节码文件中，以实现代码覆盖率的统计和分析。插桩时机是应用程序编译时，需要重新编译和打包应用程序，因此相对于On-the-fly插桩比较麻烦，需要在每次编译打包前执行。但由于插桩是在编译时完成的，插桩后的目标类文件中已经包含了插桩代码和原始代码，因此不会增加应用程序的运行时开销。

On-the-fly 和 offline比较：On-the-fly模式更方便简单，但是如下情况不适合：
（1）不支持 Java 代理的运行时环境。
（2）无法配置 JVM 选项的部署。
（3）需要为另一个 VM（如 Android Dalvik VM）转换字节码。
（4）与执行动态类文件转换的其他代理冲突。
## Jacoco插桩模式在Android上的使用
Android采用Jacoco统计代码覆盖率时，需要依据使用场景来决定采用哪种插桩模式。

1、如果需要实时监控应用程序的代码覆盖率，可以选择使用On-the-fly插桩：
（1）在应用程序运行时启动Java Agent（反射获取），在内存中动态修改目标类的字节码，插入埋点代码；
（2）测试完成后会将代码覆盖率数据输出到文件中；
（3）将文件从手机copy到AS指定目录，通过Jacoco提供的gradle插件进行分析，产出报告
2、编译时插桩，采用Offline离线插桩：
（1）编译时修改字节码，插入埋点代码
（2）借助Junit 和 robolectric 自动化测试框架，测试代码，输出覆盖率文件
（3）通过Jacoco提供的gradle插件分析文件 产出报告。

为了脱离真机，方便自动化测试，产出分析报告。Metax采用的第二种方式。
# 四、Jacoco离线插桩原理浅析
举个例子一个Test.java代码如下：
```java
public class Test {
  void testMethod(boolean result) {
      if (result) {
          System.out.println("分支1");
          return;
      }
      System.out.println("分支2");
  }
}
```
1、首先经过javac编译成字节码Test.class。
```language
public class com.metax.demo.Test {
public com.metax.demo.Test();
  Code:
     0: aload_0
     1: invokespecial #1                  // Method java/lang/Object."<init>":()V
     4: return

void testMethod(boolean);
  Code:
     0: iload_1
     1: ifeq          13
     4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
     7: ldc           #3                  // String 分支1
     9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    12: return
    13: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
    16: ldc           #5                  // String 分支2
    18: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    21: return
}

```

2、借助jacococli.jar执行jacoco离线插桩命令
```language
java -jar jacococli.jar instrument Test.class --dest out

```
3、javap -c Test.class 查看插桩后的字节码
```language
public class com.metax.demo.Test {
public com.metax.demo.Test();
  Code:
     0: invokestatic  #35                 // Method $jacocoInit:()[Z
     3: astore_1
     4: aload_0
     5: invokespecial #1                  // Method java/lang/Object."<init>":()V
     8: aload_1
     9: iconst_0
    10: iconst_1
    11: bastore
    12: return

void testMethod(boolean);
  Code:
     0: invokestatic  #35                 // Method $jacocoInit:()[Z
     3: astore_2
     4: iload_1
     5: ifeq          25
     8: aload_2
     9: iconst_1
    10: iconst_1
    11: bastore
    12: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
    15: ldc           #3                  // String 分支1
    17: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    20: aload_2
    21: iconst_2
    22: iconst_1
    23: bastore
    24: return
    25: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
    28: ldc           #5                  // String 分支2
    30: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    33: aload_2
    34: iconst_3
    35: iconst_1
    36: bastore
    37: return
}

```
4、为了方便对比前后代码逻辑，将插桩后的class，反编译如下
```java
import org.jacoco.agent.rt.internal_f3994fa.Offline;

public class Test {
  private static transient /* synthetic */ boolean[] $jacocoData;

  public Test() {
      boolean[] arrbl = Test.$jacocoInit();
      arrbl[0] = true;
  }

  void testMethod(boolean result) {
      boolean[] arrbl = Test.$jacocoInit();
      if (result) {
          arrbl[1] = true;
          System.out.println("分支1");
          arrbl[2] = true;
          return;
      }
      System.out.println("分支2");
      arrbl[3] = true;
  }

  private static /* synthetic */ boolean[] $jacocoInit() {
      boolean[] arrbl = $jacocoData;
      if ($jacocoData == null) {
          arrbl = $jacocoData = Offline.getProbes((long)-2561759577465524258L, (String)"com/metax/demo/Test", (int)4);
      }
      return arrbl;
  }
}
```
对比插桩前后代码逻辑：

![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/5029d519-e9b0-480b-ba20-7701f2a9c0ccimage.png)
### 通过对比可见Jacoco插桩做了如下操作：
1、每个类都会插入一个名为$jacocoData的boolean[]成员数组，用来记录该类的代码分支执行情况。

2、每一个类都会插入$jacocoInit()的初始化数组方法，$jacocoInit()的作用是在任意方法执行时，从本地已经保存的记录文件中获取当前类的分支执行情况，

3、在每个代码块（如方法、分支等）的进入点和出口点插入相应的boolean标记，JaCoCo把这个标记叫做探针（Probes），代码执行过的地方，改变标记为true，以记录代码块是否被执行过。具体字节码探针插入策略见[官方文档](https://www.jacoco.org/jacoco/trunk/doc/flow.html)
### 总结
1、插桩：通过ASM做静态代码插桩，提前给期望覆盖的类都添加上代码探针
2、覆盖率统计：运行时，把对应的类执行过的探针记录存储。
3、报告计算：对探针记录，进行合并转换，转为期望的结果显示，比如html等
# 五、Jacoco集成使用
已在Metax中接入，使用可以直接使用App插件
```language
apply plugin: 'metax-app-plugin'
```
# 六、踩坑点
jacoco 0.75以上 需要开启JacocoTaskExtension中的includeNoLocationClasses，否则报告覆盖率为0
```language
  tasks.withType(Test) {
      jacoco.includeNoLocationClasses = true
      jacoco.excludes = ['jdk.internal.*']
  }
```
# 七、Jacoco报告分析
报告示例：
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/68fe0301-b128-4df8-b63c-5b0afb8b50b5image.png)

Jacoco是从代码指令（Instructions， Coverage），分支（Branches， Coverage），圈复杂度（Cyclomatic Complexity），行（Lines），方法（Methods），类（Classes）等维度进行分析的。

每个维度具体含义如下：
## 维度一：字节码指令覆盖（Instruction Coverage）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/32da5714-0d89-44e6-9219-a0c71574b63cimage.png)
含义：指令覆盖是衡量执行了多少条代码指令，包括Java字节码中的所有指令，例如if语句、while循环、try-catch语句等等，都算作一条指令。Jacoco 计算的最小单位就是字节码指令。

1、红色进度条表未覆盖
2、绿色进度条表示已覆盖
3、Cov 为总体覆盖率。

Total：780表示没有覆盖的分支,1483表示总的分支，47%表示总体覆盖率。
## 维度二：分支覆盖（Branch Coverage）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/bf2c4613-7169-496e-8f5e-2e420f5e6475image.png)
含义：指被测试用例覆盖的分支（例如 if 语句的两个分支）占总分支数的比例。这个指标反映了测试用例对代码分支的覆盖程度。

1、红色进度条表未覆盖
2、绿色进度条表示已覆盖
3、Cov为总体覆盖率。

Total：104表示没有覆盖的分支,158表示总的分支，34%表示总体覆盖率。
## 维度三：圈复杂度（Cyclomatic Complexity）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/3a7a846f-9e23-40a6-983a-cb0e1f12c02dimage.png)
含义：圈复杂度是衡量程序复杂度的一个指标。也可以理解为覆盖所有可能情况，最少使用测试用例数。圈复杂度越高，说明程序的复杂度也越高。

1、Missed  15表示该模块圈复杂度缺失数 
2、Cxty    21表示该模块圈复杂度总数

Total：表示整个项目的缺失数，和总数
## 维度四：行覆盖（Line Coverage）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/9eecbd2e-f4ca-4451-96af-7301a3ff2468image.png)
含义：指被测试用例覆盖的代码行占总代码行数（指的是源代码的行）的比例。这个指标可以帮助我们了解测试用例对代码行的覆盖情况。

1、Missed 54 表示该模块行缺失数 
2、Lines 71  表示该模块行总数
Total：表示整个项目的缺失数，和总数
## 维度五：方法覆盖（Method Coverage）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/cce550d8-2938-4e22-b3c8-97534ff65df1image.png)
含义：指被测试用例覆盖的方法占总方法数的比例。这个指标可以帮助我们了解测试用例对方法的覆盖情况。

1、Missed 4    表示该模块方法缺失数 
2、Methods 10  表示该模块方法总数
Total：表示整个项目的缺失数，和总数
## 维度六：类覆盖（Class Coverage）
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/78ca997a-dbd9-475f-94a1-70e20152e6f9image.png)
含义：指被测试用例覆盖的类占总类数的比例。这个指标可以帮助我们了解测试用例对类的覆盖情况。

1、Missed 1    表示该模块类缺失数 
2、Classes 3  表示该模块类总数
Total：表示整个项目的缺失数，和总数

实际使用中，根据需要关注对应的维度，提升代码覆盖率，提高代码质量
## 报告中的覆盖率标识
### 行覆盖：
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/d616d30b-cc02-4498-bd2e-a0c91cc1438dimage.png)
- 绿色代表已执行
- 红色代表未执行
- 黄色代表执行了一部分
### 分支覆盖标识
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/f4c7f84d-dd60-4a3c-ae57-b4c2c9c4e3dcimage.png)
![image.png](https://wos.58cdn.com.cn/IjGfEdCbIlr/ishare/84ddecd8-68ac-4599-8ca5-54e2de818f1bimage.png)
- 红钻表示未覆盖
- 黄钻表示部分覆盖
- 绿钻表示全部覆盖