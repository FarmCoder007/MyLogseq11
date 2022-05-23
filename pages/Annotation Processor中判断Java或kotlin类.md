- 在Android Studio中，项目编译build之后kapt会在项目的build/tmp/kapt3/stubs目录下会生成kotlin编写的类的Java“存根类”，在这些类的顶部我们可以看到有这样一个注解@kotlin.Metadata(...)
  @Metadata是 Kotlin 里比较特殊的一个注解。它记录了 Kotlin 代码元素的一些信息，比如 class 的可见性，function 的返回值，参数类型，property 的 lateinit，nullable 的属性等等。这些 Metadata 的信息由 kotlinc 生成，最终会以注解的形式存于 .class 文件。
  所以要在注解处理器判断一个类是kotlin语言或者Java语言编写的我们可以通过判断该类是否有@Metadata注解来区分：
- ```
      /**
       * if true mean this class is java class
       */
      private fun isJavaFile(element: TypeElement): Boolean {
          val tmMetadata = mElements.getTypeElement("kotlin.Metadata").asType()
          return element.annotationMirrors.find { it.annotationType == tmMetadata } == null
      }
  ```