- [官网](https://github.com/dom4j/dom4j)
- [Dom4j完整教程](https://blog.csdn.net/qq_41860497/article/details/84339091?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-84339091-blog-123076975.pc_relevant_recovery_v2&spm=1001.2101.3001.4242.2&utm_relevant_index=4)
- # 一、最新依赖
  collapsed:: true
	- implementation 'org.dom4j:dom4j:2.1.3'
-
- # 二、dom解析
	- ```kotlin
	  package com.metax.utils
	  
	  import org.w3c.dom.Document
	  import org.w3c.dom.NamedNodeMap
	  import org.w3c.dom.Node
	  import org.w3c.dom.NodeList
	  import org.xml.sax.SAXException
	  import java.io.IOException
	  import javax.xml.parsers.DocumentBuilder
	  import javax.xml.parsers.DocumentBuilderFactory
	  import javax.xml.parsers.ParserConfigurationException
	  
	  /**
	   * @author:xuwenbin
	   * @time:2022/12/26 3:33 下午
	   * @description:
	   */
	  class XmlParserUtils {
	      fun parser() {
	          val documentBuilderFactory: DocumentBuilderFactory = DocumentBuilderFactory.newInstance()
	  //创建一个DocumentBuilder的对象
	          try {
	              //创建DocumentBuilder对象
	              val db: DocumentBuilder = documentBuilderFactory.newDocumentBuilder()
	              //通过DocumentBuilder对象的parser方法加载books.xml文件到当前项目下
	              val document: Document =
	                  db.parse("../demo-sample/build/reports/tests/testDebugUnitTest/index.xml")
	              //获取所有book节点的集合
	              val bookList: NodeList = document.getElementsByTagName("book")
	              //通过nodelist的getLength()方法可以获取bookList的长度
	              System.out.println("一共有" + bookList.getLength().toString() + "本书")
	              //遍历每一个book节点
	              for (i in 0 until bookList.getLength()) {
	                  println("=================下面开始遍历第" + (i + 1) + "本书的内容=================")
	                  //通过 item(i)方法 获取一个book节点，nodelist的索引值从0开始
	                  val book: Node = bookList.item(i)
	                  //获取book节点的所有属性集合
	                  val attrs: NamedNodeMap = book.getAttributes()
	                  println("第 " + (i + 1) + "本书共有" + attrs.getLength() + "个属性")
	                  //遍历book的属性
	                  for (j in 0 until attrs.getLength()) {
	                      //通过item(index)方法获取book节点的某一个属性
	                      val attr: Node = attrs.item(j)
	                      //获取属性名
	                      System.out.print("属性名：" + attr.getNodeName())
	                      //获取属性值
	                      System.out.println("--属性值" + attr.getNodeValue())
	                  }
	                  //解析book节点的子节点
	                  val childNodes: NodeList = book.getChildNodes()
	                  //遍历childNodes获取每个节点的节点名和节点值
	                  println(
	                      "第" + (i + 1) + "本书共有" +
	                              childNodes.getLength() + "个子节点"
	                  )
	                  for (k in 0 until childNodes.getLength()) {
	                      //区分出text类型的node以及element类型的node
	                      if (childNodes.item(k).getNodeType() === Node.ELEMENT_NODE) {
	                          //获取了element类型节点的节点名
	                          print(
	                              ("第" + (k + 1) + "个节点的节点名："
	                                      + childNodes.item(k).getNodeName())
	                          )
	                          //获取了element类型节点的节点值
	                          System.out.println(
	                              "--节点值是：" + childNodes.item(k).getFirstChild().getNodeValue()
	                          )
	                          //System.out.println("--节点值是：" + childNodes.item(k).getTextContent());
	                      }
	                  }
	                  println("======================结束遍历第" + (i + 1) + "本书的内容=================")
	              }
	          } catch (e: ParserConfigurationException) {
	              e.printStackTrace()
	          } catch (e: SAXException) {
	              e.printStackTrace()
	          } catch (e: IOException) {
	              e.printStackTrace()
	          }
	  
	      }
	  }
	  
	  
	  
	  
	  ```