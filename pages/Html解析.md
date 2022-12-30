- implementation 'org.jsoup:jsoup:1.8.3'
- [java读取.html文件并获取数据](https://blog.csdn.net/weixin_43407520/article/details/123043024)
- ## jsoup读取表格table
  collapsed:: true
	- 原数据
	  collapsed:: true
		- ```
		  
		  <?xml version="1.0" encoding="UTF-8"?><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
		          "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
		  <html xmlns="http://www.w3.org/1999/xhtml" lang="zh">
		  <head>
		      <meta http-equiv="Content-Type" content="text/html;charset=UTF-8"/>
		      <link rel="stylesheet" href="jacoco-resources/report.css" type="text/css"/>
		      <link rel="shortcut icon" href="jacoco-resources/report.gif" type="image/gif"/>
		      <title>demo-sample</title>
		      <script type="text/javascript" src="jacoco-resources/sort.js"></script>
		  </head>
		  <body onload="initialSort(['breadcrumb', 'coveragetable'])">
		  <div class="breadcrumb" id="breadcrumb"><span class="info"><a href="jacoco-sessions.html"
		                                                                class="el_session">Sessions</a></span><span
		          class="el_report">demo-sample</span></div>
		  <h1>demo-sample</h1>
		  <table class="coverage" cellspacing="0" id="coveragetable">
		      <thead>
		      <tr>
		          <td class="sortable" id="a" onclick="toggleSort(this)">Element</td>
		          <td class="down sortable bar" id="b" onclick="toggleSort(this)">Missed Instructions</td>
		          <td class="sortable ctr2" id="c" onclick="toggleSort(this)">Cov.</td>
		          <td class="sortable bar" id="d" onclick="toggleSort(this)">Missed Branches</td>
		          <td class="sortable ctr2" id="e" onclick="toggleSort(this)">Cov.</td>
		          <td class="sortable ctr1" id="f" onclick="toggleSort(this)">Missed</td>
		          <td class="sortable ctr2" id="g" onclick="toggleSort(this)">Cxty</td>
		          <td class="sortable ctr1" id="h" onclick="toggleSort(this)">Missed</td>
		          <td class="sortable ctr2" id="i" onclick="toggleSort(this)">Lines</td>
		          <td class="sortable ctr1" id="j" onclick="toggleSort(this)">Missed</td>
		          <td class="sortable ctr2" id="k" onclick="toggleSort(this)">Methods</td>
		          <td class="sortable ctr1" id="l" onclick="toggleSort(this)">Missed</td>
		          <td class="sortable ctr2" id="m" onclick="toggleSort(this)">Classes</td>
		      </tr>
		      </thead>
		      <tfoot>
		      <tr>
		          <td>Total</td>
		          <td class="bar">73 of 73</td>
		          <td class="ctr2">0%</td>
		          <td class="bar">0 of 0</td>
		          <td class="ctr2">n/a</td>
		          <td class="ctr1">56</td>
		          <td class="ctr2">56</td>
		          <td class="ctr1">64</td>
		          <td class="ctr2">64</td>
		          <td class="ctr1">56</td>
		          <td class="ctr2">56</td>
		          <td class="ctr1">1</td>
		          <td class="ctr2">1</td>
		      </tr>
		      </tfoot>
		      <tbody>
		      <tr>
		          <td id="a0"><a href="com.metax/index.html" class="el_package">com.metax</a></td>
		          <td class="bar" id="b0"><img src="jacoco-resources/redbar.gif" width="120" height="10"
		                                       title="73" alt="73"/></td>
		          <td class="ctr2" id="c0">0%</td>
		          <td class="bar" id="d0"/>
		          <td class="ctr2" id="e0">n/a</td>
		          <td class="ctr1" id="f0">56</td>
		          <td class="ctr2" id="g0">56</td>
		          <td class="ctr1" id="h0">64</td>
		          <td class="ctr2" id="i0">64</td>
		          <td class="ctr1" id="j0">56</td>
		          <td class="ctr2" id="k0">56</td>
		          <td class="ctr1" id="l0">1</td>
		          <td class="ctr2" id="m0">1</td>
		      </tr>
		      </tbody>
		  </table>
		  <div class="footer"><span class="right">Created with <a
		          href="http://www.jacoco.org/jacoco">JaCoCo</a> 0.8.5.201910111838</span></div>
		  </body>
		  </html>
		  ```
	- 解析table
		- ```kotlin
		  fun parserJacocoReport(filePath: String): JacocoReportBean {
		          // 获取所有的行
		          val tableElements: Elements = parserHtml(filePath).body().select("table").select("tr")
		          // 表格数值
		          val bodyRowElements = tableElements.get(1).select("td")
		          val jacocoReportBean: JacocoReportBean = JacocoReportBean()
		          val bodyCount = bodyRowElements.size
		          // 指令覆盖率
		          if (bodyCount < 3) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.InstructionsCoverage = bodyRowElements[2].text()
		  
		          // 分支覆盖
		          if (bodyCount < 5) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.branchCoverage = bodyRowElements[4].text()
		  
		          // 圈复杂覆盖率
		          if (bodyCount < 7) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.cxtyCoverage = getCoverage(bodyRowElements, 6)
		  
		          // 行覆盖率
		          if (bodyCount < 9) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.linesCoverage = getCoverage(bodyRowElements, 8)
		  
		          // 方法覆盖率
		          if (bodyCount < 11) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.methodCoverage = getCoverage(bodyRowElements, 10)
		  
		          // 类覆盖率
		          if (bodyCount < 13) {
		              return jacocoReportBean
		          }
		          jacocoReportBean.classCoverage = getCoverage(bodyRowElements, 12)
		          return jacocoReportBean
		      }
		  
		      private fun getCoverage(bodyRowElements: Elements, index: Int): Float {
		          val missedCount = bodyRowElements[index - 1].text().toFloat()
		          val allCount = bodyRowElements[index].text().toFloat()
		          if (allCount == 0.0f) {
		              return 0.0f
		          }
		          return (allCount - missedCount) / allCount
		      }
		  
		  ```