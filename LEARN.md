XML DOM
# 1 DOM
DOM是文档对象化模型（Document Object Model）的简称，DOM 定义了访问诸如 XML 和 XHTML 文档的标准。
DOM 将 XML 文档作为一个树形结构，而树叶被定义为节点。
# 2 XML DOM
XML DOM用于 XML 文档的标准模型，定义访问和操作XML文档的标准方法。XML DOM 定义了所有 XML 元素的对象和属性，以及访问它们的方法（接口）。换句话说：XML DOM 是用于获取、更改、添加或删除 XML 元素的标准。

## 2.1 XML DOM 节点
在 DOM 中，XML 文档中的每个成分都是一个节点。
DOM 是这样规定的：

* 整个文档是一个文档节点
* 每个 XML 元素是一个元素节点
* 包含在 XML 元素中的文本是文本节点
* 每一个 XML 属性是一个属性节点
* 注释是注释节点

XML DOM 把 XML 文档视为一棵节点树。可通过这棵树访问所有节点。可以修改或删除它们的内容，也可以创建新的元素。

## 2.2 XML DOM 解析器(parse)
XML DOM 包含了遍历 XML 树，访问、插入及删除节点的方法（函数）。然而，在访问和操作 XML 文档之前，它必须加载到 XML DOM 对象。XML 解析器读取 XML，并把它转换为 XML DOM 对象，这样才可以使用 JavaScript 访问它。大多数浏览器有一个内建的 XML 解析器。

## 2.3 XML DOM 属性和方法
###　2.3.1 XML DOM 属性
一些典型的 DOM 属性：

x.nodeName - x 的名称
x.nodeValue - x 的值
x.parentNode - x 的父节点
x.childNodes - x 的子节点
x.attributes - x 的属性节点
注释：在上面的列表中，x 是一个节点对象。
### 2.3.2 XML DOM 方法
x.getElementsByTagName(name) - 获取带有指定标签名称的所有元素
x.appendChild(node) - 向 x 插入子节点
x.removeChild(node) - 从 x 删除子节点
注释：在上面的列表中，x 是一个节点对象。

## 2.4 XML DOM 节点信息
三个重要的节点属性是：
* nodeName
* nodeValue
* nodeType

### 2.4.1 nodeName 属性
nodeName 属性规定节点的名称，nodeName 是只读的。
* 元素节点的 nodeName 与标签名相同
* 属性节点的 nodeName 是属性的名称
* 文本节点的 nodeName 永远是 #text
* 文档节点的 nodeName 永远是 #document

### 2.4.2 nodeValue 属性
nodeValue 属性规定节点的值。

* 元素节点的 nodeValue 是 undefined
* 文本节点的 nodeValue 是文本本身
* 属性节点的 nodeValue 是属性的值

### 2.4.3 nodeType 属性
nodeType 属性规定节点的类型，nodeType 是只读的。
最重要的节点类型是：
* 元素	1
* 属性	2
* 文本	3
* 注释	8 (注意这也是个节点！！！)
* 文档	9
##　3.RapidXml
RapidXml是指 XML DOM解析工具包。


## struct
是由自己定义的数据类型，可以容纳许多不同的数据值。和class很像，但是struct的成员默认是public的，而class的成员默认是private的。一般不包含成员函数，只声明成员变量。

http://c.biancheng.net/view/1407.html

## 容器
vector, list, map等都是容器，可以放各种数据类型进去（比如：struct），但是有时候要进行操作符重载。