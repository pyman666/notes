
- [XML语法](#xml语法)
- [文档声明](#文档声明)
- [元素](#元素)
- [属性](#属性)
- [注释](#注释)
- [CDATA节](#cdata节)
- [转义字符](#转义字符)
- [处理指令](#处理指令)
- [元素](#元素)
- [属性](#属性)
  - [实体](#实体)
- [节点](#节点)
- [谓语](#谓语)
- [未知节点](#未知节点)
- [若干路径](#若干路径)

## XML语法

## 文档声明

文档声明放在文档第一行，声明包括：version、encoding、standalone（文档定义是否独立使用，默认no）
```xml
<?xml version="1.0" encoding="gb2312" standalone="yes"?>
```
## 元素

每个XML文档必须有且只有一个**根元素**，根元素是一个完全包括文档其他所有元素的元素。

XML元素指XML文件中出现的标签，一个标签分为开始标签和结束标签，有如下几种形式：
```xml
<a>标签体</a>
<a></a>
<a/>
```
- XML中，空格和换行都作为原始内容被处理；

- 标签区分大小写，不能以数字或下划线开头，不能包含空格，中间不能有冒号；

## 属性

- 属性值用双引号或单引号分隔，属性值有单引就用双引分隔，反之同理，同时都有就用转义字符代替；

- 一个元素可以有多个属性，特定的属性名称在同一个元素中只能出现一次；
```xml
 <a name="Jack" age="18" />
```
## 注释

- 注释内容不要出现`—-`；

- 注释不能放在标记中间，不能嵌套；

- 可以在除标记以外的任何地方放注释；
```xml
<!--这是一个注释-->
```
## CDATA节

CDATA节中的所有字符都会被当作元素字符数据等常量部分。可以输入任何字符（除了`]]>`），不能嵌套。
```xml
<![CDATA[
	<sfsdfsd0<fo9wefg>werlgfjasdg>
]]>
```
## 转义字符

|  |  |
| --- | --- |
| &lt; | < |
| &gt; | > |
| &amp; | & |
| &quot; | “ |
| &apos; | ‘ |
## 处理指令

处理指令简称PI（processing instruction），用来指挥解析引擎如何解析XML文档内容。

- PI必须以`<?`开头，以`?>`结尾，声明语句就是一个处理指令；
```xml
<?xml-stylesheet type="text/css" href="my.css"?>
```
# DTD

可以编写一个文档来约束XML文档等书写规范，称之为**XML约束**，常用的约束技术包括XML DTD、XML Schema。

DTD（document type definition）文档类型定义，该文件一般和xml文件配合使用，用于约束XML文件。
```xml
<!DOCTYPE 根元素 [定义内容]>
<!DOCTYPE 根元素 SYSTEM "dtd文件路径">
<!DOCTYPE 根元素 SYSTEM "dtd文件路径" [定义内容]>
<!DOCTYPE 根元素 PUBLIC "dtd名称" "dtd文件url">

<!DOCTYPE 班级 SYSTEM "myClass.dtd">
```
## 元素
```xml
<!ELEMENT 元素名 类型>

<ELEMENT 班级 (学生+)>
<ELEMENT 学生 (姓名,年龄)>
<ELEMENT 姓名 (#PCDATA)>
<ELEMENT 年龄 (#PCDATA)>
```
类型

- `#PCDATA`：可以包含任何字符数据，但是不能在其中包含任何子元素或其它类型（组合）；

- `ANY`：可以包含任何在DTD中定义的元素内容；

- `EMPTY`：不能包含子元素和文本，但可以有属性；

修饰符

- `+`：至少出现1次；

- `*`：允许出现0次；

- `?`：只能出现0或1次；

- `,`：需要按指定顺序出现；

- `()`：用来给元素分组；

- `｜`：在列出的对象中选一个；

## 属性
```xml
<!ELEMENT 元素名 类型>

<ELEMENT 班级 (学生+)>
<ELEMENT 学生 (姓名,年龄)>
<!ATTLIST 学生
  地址 CDATA #REQUIRED
>
<ELEMENT 姓名 (#PCDATA)>
<ELEMENT 年龄 (#PCDATA)>
```
类型：

- `CDATA`：可以放入文本；

- `ID`：值不能重复，不能以数字开头；

- `IDREF`/`IDREFS`：需要引用另一个ID，或多个（空格隔开）；

- `Enumerated`：属性的值只能是列举出的；

- `ENTITY`：实体；

特点

- `#REQUIRED`：必须有；

- `#IMPLIED`：可以有；

- `#FIXED “xxx”`：如果有，必须是xxx；

- `Default “xxx”`：如果不指定，默认xxx；

### 实体

实体用于为一段内容创建一个别名，以后在XML文件中可以使用别名饮用这段内容；

**引用实体**（在xml文件中使用）：
```xml
<!ENTITY copyright "I am a programmer">
```
```xml
<学生 性别="女">好好学习，&copyright;</学生>
```
**参数实体**（在dtd文件中使用）：
```xml
<!ENTITY % name "姓名">

<!ELEMENT %myname; (#PCDATA)>
```
# XPath

## 节点

XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。 下面列出了最有用的路径表达式：

|  |  |
| --- | --- |
| **表达式** | **描述** |
| nodename | 选取此节点的所有子节点。 |
| / | 从根节点选取（取子节点）。 |
| // | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置（取子孙节点）。 |
| . | 选取当前节点。 |
| .. | 选取当前节点的父节点。 |
| @ | 选取属性。 |
在下面的表格中，我们已列出了一些路径表达式以及表达式的结果：

|  |  |
| --- | --- |
| **路径表达式** | **结果** |
| bookstore | 选取 bookstore 元素的所有子节点。 |
| /bookstore | 选取根元素 bookstore。 注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ |
| bookstore/book | 选取属于 bookstore 的子元素的所有 book 元素。 |
| //book | 选取所有 book 子元素，而不管它们在文档中的位置。 |
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| //@lang | 选取名为 lang 的所有属性。 |
## 谓语

谓语（Predicates）用来查找某个特定的节点或者包含某个指定的值的节点，谓语被嵌在方括号中

|  |  |
| --- | --- |
| **路径表达式** | **结果** |
| /bookstore/book[1] | 选取属于 bookstore 子元素的第一个 book 元素。 |
| /bookstore/book[last()] | 选取属于 bookstore 子元素的最后一个 book 元素。 |
| /bookstore/book[last()-1] | 选取属于 bookstore 子元素的倒数第二个 book 元素。 |
| /bookstore/book[position()<3] | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。 |
| //title[@lang] | 选取所有拥有名为 lang 的属性的 title 元素。 |
| //title[@lang='eng'] | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。 |
| /bookstore/book[price>35.00] | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。 |
| /bookstore/book[price>35.00]//title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |
## 未知节点

XPath 通配符可用来选取未知的 XML 元素

|  |  |
| --- | --- |
| **通配符** | **描述** |
| \* | 匹配任何元素节点。 |
| @\* | 匹配任何属性节点。 |
| node() | 匹配任何类型的节点。 |
| **路径表达式** | **结果** |
| /bookstore/\* | 选取 bookstore 元素的所有子元素。 |
| //\* | 选取文档中的所有元素。 |
| //title[@\*] | 选取所有带有属性的 title 元素。 |
## 若干路径

通过在路径表达式中使用`|`运算符，可以选取若干个路径

|  |  |
| --- | --- |
| **路径表达式** | **结果** |
| //book/title | //book/price | 选取 book 元素的所有 title 和 price 元素。 |
| //title | //price | 选取文档中的所有 title 和 price 元素。 |
| /bookstore/book/title | //price | 选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。 |
