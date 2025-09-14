# Java 网页抓取
<h2>使用 Java 进行网页抓取的速查指南（含代码示例）</h2>

![Java 网页抓取头图](https://github.com/bright-cn/java-web-scraping/blob/main/Web%20scraping%20with%20Java%20-%20Ultimate%20guide.png "Java 抓取指南横幅")

尽管很多人更喜欢使用 Python，另一种同样流行的选择是使用 Java 进行网页抓取。下面是一份循序渐进的指南，帮助你轻松完成这一过程。

在开始之前，请确保你的电脑已完成以下环境配置，以便更好地进行网页抓取：

[Java 11](https://www.java.com/en/download/help/download_options.html) —— 虽然有更高版本，但 Java 11 仍是开发者中应用最广泛的版本。

[Maven](https://maven.apache.org/) —— 构建自动化与依赖管理工具。

[IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=windows) —— 用于开发 Java 软件的集成开发环境（IDE）。

[HtmlUnit](https://htmlunit.sourceforge.io/) —— 浏览器行为模拟器（如表单提交模拟）。

你可以用以下命令检查安装情况：

* ```java -version```
* ```mvn -v```

## 替代方案

[Bright Data 的 Web Scraping API](https://www.bright.cn/products/web-scraper) 提供全自动的数据采集解决方案。无需处理搭建与维护爬虫的复杂性——只需定义目标网站、所需数据集与输出格式即可。无论你需要实时结构化数据还是定时交付，Bright Data 强大的工具都能确保准确性、可扩展性与易用性。非常适合重视效率与可靠性的专业团队。

现在，让我们继续编写 Java 爬虫。

<h3>第一步：检查目标页面</h3>
进入你想采集数据的目标网站，在页面任意位置右键并点击“检查元素”（或“检查”）以打开“开发者工具”，从而访问网页的 HTML。

<h3>第二步：开始抓取 HTML</h3>
打开 IntelliJ IDEA，创建一个 Maven 项目：

![IntelliJ IDEA Maven 项目](https://github.com/bright-cn/java-web-scraping/blob/main/Java2%20intellij.png "IntelliJ Maven 项目")

Maven 项目包含一个 pom.xml 文件。打开 pom.xml，首先设置项目的 JDK 版本：

```
<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>11</maven.compiler.source>
		<maven.compiler.target>11</maven.compiler.target>
	</properties>
```

然后在 pom.xml 中添加 HtmlUnit 依赖，如下所示：
  
```
<dependencies>
		<dependency>
			<groupId>net.sourceforge.htmlunit</groupId>
			<artifactId>htmlunit</artifactId>
			<version>2.63.0</version>
		</dependency>
	</dependencies>
```

现在一切就绪，可以开始编写第一个 Java 类。先像这样创建一个新的 Java 源文件：
  
![新建 Java 源文件](https://github.com/bright-cn/java-web-scraping/blob/main/Java3%20intellij.png "新建 Java 类")
  
我们需要创建一个 main 方法作为应用程序的入口。如下所示：
  
```
public static void main(String[] args) throws IOException {
}
``` 

应用程序将从该方法启动，它是程序的入口点。现在可以按如下方式引入 HtmlUnit 并发送 HTTP 请求：
  
```   
import com.gargoylesoftware.htmlunit.*;
import com.gargoylesoftware.htmlunit.html.*;
import java.io.IOException;
import java.util.List;
```  

接着按如下方式创建一个 WebClient 并设置选项：
  
``` 
private static WebClient createWebClient() {
		WebClient webClient = new WebClient(BrowserVersion.CHROME);
		webClient.getOptions().setThrowExceptionOnScriptError(false);
		webClient.getOptions().setCssEnabled(false);
	  webClient.getOptions().setJavaScriptEnabled(false);
		return webClient;
}
``` 
  
<h3>第三步：从 HTML 提取/解析数据</h3>
现在我们来提取感兴趣的目标价格数据。我们将使用 HtmlUnit 的内置方法来实现。以下示例展示了如何针对“商品价格”这一数据点进行操作：

```
WebClient webClient = createWebClient();
	    
		try {
			String link = "https://www.ebay.com/itm/332852436920?epid=108867251&hash=item4d7f8d1fb8:g:cvYAAOSwOIlb0NGY";
			HtmlPage page = webClient.getPage(link);
			
			System.out.println(page.getTitleText());
			
			String xpath = "//*[@id=\"mm-saleDscPrc\"]";			
			HtmlSpan priceDiv = (HtmlSpan) page.getByXPath(xpath).get(0);			
			System.out.println(priceDiv.asNormalizedText());
			
			CsvWriter.writeCsvFile(link, priceDiv.asNormalizedText());
			
		} catch (FailingHttpStatusCodeException | IOException e) {
			e.printStackTrace();
		} finally {
			webClient.close();
		}	
```

要获取目标元素的 XPath，可使用开发者工具。在开发者工具中右键单击选中的区域，点击“Copy XPath（复制 XPath）”。该操作会将所选区域复制为一个 XPath 表达式：

![元素的 XPath](https://github.com/bright-cn/java-web-scraping/blob/main/Java4.png "获取 XPath")

网页包含链接、文本、图形和表格。如果你选择的是某个表格的 XPath，就可以将其导出为 CSV，并在 Microsoft Excel 等程序中进行进一步计算与分析。下一步，我们将演示如何将表格导出为 CSV 文件。

<h3>第四步：导出数据</h3>
现在数据已经解析完成，我们可以将其导出为 CSV 格式以便进一步分析。这种格式有些专业人士更偏好，因为可以在 Microsoft Excel 中方便地打开/查看。以下是实现该功能的示例代码：

```
public static void writeCsvFile(String link, String price) throws IOException {
		
		FileWriter recipesFile = new FileWriter("export.csv", true);

		recipesFile.write("link, price\n");

		recipesFile.write(link + ", " + price);

		recipesFile.close();
}
```

----

## 结论

尽管 Java 可以帮助各类从业者提取所需数据，但网页抓取过程可能相当耗时。若希望全面自动化数据采集流程，你可以使用 Bright Data 的 [Web Scraping API](https://brightdata.com/products/web-scraper)。你只需选择目标网站和输出数据集，然后设定所需的计划、文件格式与交付方式即可。
