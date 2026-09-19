
XML解析可以分为：DOM解析、SAX解析：

- dom（Document Object Model）是w3c组织推荐的处理xml的方式；

- sax（Sample API for XML）不是官方标准，但它是xml社区事实上的标准，几乎所有xml解析器都支持；

XML解析器：Crimson、Xerces、Aelfred2等；

XML解析API：Jaxp、jdom、dom4j等；

# DOM

从DOM看：

- dom会把xml文件看作一个dom树，并加载到**内存**；

- dom适合做crud操作，不适合操作较大的xml文件；

- dom会把xml文件中每个`Element`/`Attr`/`Text`都映射成对应的`Node`对象；
```xml
<班级>
	<学生 gender=“男”>
		<姓名>韩顺平</姓名>
	</学生>
</班级>
```
```java
import javax.xml.parsers.*;
import javax.xml.transform.*;

public static void main(String[] args) {
	// 创建dom解析器工厂
	DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
	// 得到解析器对象
	DocumentBuilder dBuilder = dbf.newDocumentBuilder();
	// 解析xml文档，得到代表整个文档的Document对象
	Document doc = dBuilder.parse("src/test.xml");

  list(doc);  // document也是node
  read(doc);
  add(doc);
  delete(doc);
  update(doc);
}

// 保存变化至文件
public static void refresh(String file) {
	TransformerFactory tff = TransformerFactory.newInstance();
  Transformer tf = tff.newTransfromer();
  tf.transform( new DOMSource(doc), new StreamResult(file) );
}

// 遍历
public static void list(Node node) {
  if (node.getNodeType() == node.ELEMENT_NODE) {
    System.out.println("node name: " + node.getNodeName());
  }
  NodeList nodeList = node.getChildNodes();
  for(int i=0; i<nodeList.getLength(); i++) {
    Node n = nodeList.item(i);
    list(n);
  }
}

// 查
public static void read(Document doc) {
  NodeList nodeList = doc.getElementsByTagName("学生");

  Element student = (Element) nodeList.item(0);
  String gender = student.getAttribute("gender");

  Element name = (Element) student.getElementsByTagName("姓名").item(0);
  String name = name.getTextContent();
}

// 增
public static void add(Document doc) {
  Element student = doc.createElement("学生");
  student.setAttribute("gender", "男");
  Element name = doc.createElement("姓名");
  name.setTextContent("Jack");

	student.appendChild(name);
  doc.getDocumentElement().appendChild(student);

	refresh("src/test.xml");
}

// 删
public static void delete(Document doc) {
  Node student = doc.getElementsByTagName("学生").item(0);
  student.getParentNode().removeChild(student);

	refresh("src/test.xml");
}

// 改
public static void update(Document doc) {
  Element student = (Element) doc.getElementsByTagName("学生").item(0);
  Element name = (Element) student.getElementsByTagName("姓名").item(0);
  name.setTextContent("Tom");

	refresh("src/test.xml");
}
```
# SAX

DOM需要读取整个xml文档，SAX允许在读取文档时就对文档进行处理，不必等整个文档加载完

- sax不能对xml文件进行添加、修改、删除操作；

- sax是推机制，把发现的内容告诉程序员，程序员可以自己决定如何处理；
```java
// 事件处理类
class MyHandler extends DefaultHandler(

  private boolean isName = false;

  // 发现文档开始
  public void startDocument(char[] ch, int start, int length) {
    super.startDocument();
  }

  // 发现xml文件中的一个元素
  public void startElement(String uri, String localName, String name, Attributes attributes) {
    if ( name.equals("姓名") ) {
      this.isName = true;
    }
  }

  // 发现xml文件中的文本
  public void characters(char[] ch, int start, int length) {
    String context = new String(ch, start, length);
    if ( context.trim().equals("")  && isName ) {
      System.out.println(context);
    }
    isName = false;
  }

  // 发现xml文件中一个元素结束</xxx>
  public void endElement(String uri, String localName, String name) {
    super.endElement(uri, localName, name);
  }

  // 发现文档结束
  public void endDocument() {
    super.endDocument();
  }
)
```
```java
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser saxParser = spf.newSAXParser();
saxParser.parser( "src/test.xml", new MyHandler() );
```
# DOM4J
```java
public static void main(String[] args) {
	SAXReader reader = new SAXReader();
	Document doc = reader.read("src/test.xml");

  list(doc.getRootElement());
  add(doc);
  delete(doc);
  update(doc);
}

public static void refresh(Document doc, String file) {
  OutputFormat fmt = OutputFormat.createPrettyPrint(); // 直接输出会中文乱码
  fmt.setEncoding("utf-8");

  XMLWriter writer  = new XMLWriter( new FileOutputStream(new File(file)), fmt );
  writer.write(doc);
  writer.close();
}

// 遍历
public static void list(Element element) {
  System.out.println( element.getName() + element.getTextTrim() );

  Iterator iter = element.element.Iterator();
  while ( iter.hasNext() ) {
    Element e = (Element) iter.next();
    list(e);
  }
}

// 查
public static void read(Document doc) {
  Element root = doc.getRootElement();
  Element student = (Element) root.elements("学生").get(0);  // 不能跨层取，可用xpath解决
  // Element student = (Element) root.element("学生");  同上

	Element name = student.element("姓名");
  String name1 = name.getText()；
  String name2 = name.attributeValue("别名");
}

// 增
public static void add(Document doc) {
  Element student = DocumentHelper.createElement("学生");
  Element name = DocumentHelper.createElement("姓名");
	name.setText("Jack");
  name.addAttribute("别名", "Jacky");
  student.add(name);
  doc.getRootElement().add(student);

  refresh(doc, "src/test.xml");
}

// 增至指定位置
public static void addByIndex(Document doc) {
  Element student = DocumentHelper.createElement("学生");
  Element name = DocumentHelper.createElement("姓名");
	name.setText("Jack");
  student.add(name);

  List<Element> students = doc.getRootElement().elements("学生");
  students.add(1, student);

  refresh(doc, "src/test.xml");
}

// 删
public static void delete(Document doc) {
  Element student = doc.getRootElement().element("学生");
  student.remove(student.element("姓名").attribute("别名"));
  student.getParent().remove(student);

  refresh(doc, "src/test.xml");
}

// 改
public static void update(Document doc) {
  List<Element> students = doc.getRootElement.elements("学生")；
  for (Element student; students) {
    Element age = student.element("年龄");
    age.setText( Integer.parseInt(age.getText()) + 3 + "" );
    Element name = student.element("姓名");
    name.addAttribute("别名", "Jacky");
  }

  refresh(doc, "src/test.xml");
}

// 结合xpath
public static void xpath(Document doc) {
  List<Element> students = doc.selectNodes("//学生");  // selectSingleNode()
}
```
