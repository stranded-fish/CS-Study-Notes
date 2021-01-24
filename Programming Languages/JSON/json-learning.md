# JSON 学习

* JSON: JavaScript Object Notation（JavaScript 对象表示法）。
* 轻量级的文本数据交换格式。
* 对象存储和交换文本信息的语法，类似于XML。

该教程主要针对：**JSON 语法** 与 **使用场景** 等学习。

目录：

- [JSON 学习](#json-学习)
  - [语法](#语法)
  - [JSON 与 XML](#json-与-xml)
  - [JSON 使用场景](#json-使用场景)
    - [JSON.parse()](#jsonparse)
    - [JSON.stringify()](#jsonstringify)
  - [参考链接](#参考链接)

## 语法

JSON 语法是 JavaScript 对象表示语法的子集。

* 数据以 **键/值对** 形式表示
  * 数据的书写格式：
    > key : value
    > "name" : "JSON学习"
    > 等价于 JavaScript: name = "JSON学习"
  * **键**：需要包含在 **双引号** 中
  * **值**：
    * 数字（整数或浮点数）
    * 字符串（在双引号中）
    * 逻辑值（true 或 false）
    * 数组（在中括号中）
    * 对象（在大括号中）
    * null
* 数据由 **逗号分隔**
  * 示例：
    > {key1 : value1, key2 : value2, ... keyN : valueN }  
* 大括号 `{}` 保存对象，对象可以包含 **多条数据**（键值对，以逗号分隔）
  * JSON对象
    > { "name":"JSON学习", "url":"www.runoob.com" }
    > 等价于 JavaScript:
    > name = "JSON学习"
    > url = "www.runoob.com"
* 中括号 `[]` 保存数组，数组可以包含 **多个对象**
  * JSON数组
  
    ```JSON
    [
        { key1 : value1-1 , key2:value1-2 },
        { key1 : value2-1 , key2:value2-2 },
        { key1 : value3-1 , key2:value3-2 },
        ...
        { keyN : valueN-1 , keyN:valueN-2 },
    ]
    ```

  * 数组访问

    ```JSON
    {
      "name":"网站",
      "num":3,
      "sites":[ "Google", "Runoob", "Taobao" ]
    }
    x = myObj.sites[0];
    ```

## JSON 与 XML

实例对比：

JSON实例：

```JSON
{
    "sites": [
    { "name":"菜鸟教程" , "url":"www.runoob.com" },
    { "name":"google" , "url":"www.google.com" },
    { "name":"微博" , "url":"www.weibo.com" }
    ]
}
```

XML实例：

```XML
<sites>
  <site>
    <name>菜鸟教程</name> <url>www.runoob.com</url>
  </site>
  <site>
    <name>google</name> <url>www.google.com</url>
  </site>
  <site>
    <name>微博</name> <url>www.weibo.com</url>
  </site>
</sites>
```

* **相同之处**：
  * JSON 和 XML 数据都是『自我描述』，都易于理解。
  * JSON 和 XML 数据都是有层次的结构。
  * JSON 和 XML 数据可以被大多数编程语言使用。

* **不同之处**：
  * JSON 不需要结束标签。
  * JSON 更加简短。
  * JSON 读写速度更快。
  * JSON 可以使用数组。

* **JSON优势**：
  * JSON 更易解析。
  * JSON 可以直接使用现有的 JavaScript 对象解析。
  * 针对 AJAX 应用，JSON 比 XML 数据加载更快，更简单。

* **解析步骤**：

  * XML：
    1. 获取 XML 文档。
    2. 使用 XML DOM 迭代循环文档。
    3. 接数据解析出来复制给变量。

  * JSON：
    1. 获取 JSON 字符串。
    2. JSON.Parse 解析 JSON 字符串。

## JSON 使用场景

通常用于与服务端交换数据（以字符串形式交互）。

### JSON.parse()

接受服务器端数据时，接受数据一般为字符串类型，通过 JSON.parse() 方法将数据转换为 JavaScript 对象。

```js
JSON.parse(text[, reviver])
```

**参数说明：**

* text: 必需，一个有效的 JSON 字符串。
* reviver: 可选，一个转换结果的函数，将为对象的每个成员调用此函数。

> 注：确保你的数据是标准的 JSON 格式，否则会解析出错

解析后，即可在网页上使用JSON数据。

**使用实例：**

```html
<p id="demo"></p>
<script>
var obj = JSON.parse('{ "name":"runoob", "alexa":10000, "site":"www.runoob.com" }');
document.getElementById("demo").innerHTML = obj.name + "：" + obj.site;
</script>
```

### JSON.stringify()

向服务器端发送数据时，数据一般为字符串类型，通过 JSON.stringify() 方法将 JavaScript 对象转换为字符串。

```js
JSON.stringify(value[, replacer[, space]])
```

**参数说明：**

* value:
必需， 要转换的 JavaScript 值（通常为对象或数组）。
* replacer:
  * 可选。用于转换结果的函数或数组。
  * 如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值。使用返回值而不是原始值。如果此函数返回 undefined，则排除成员。根对象的键是一个空字符串：""。
  * 如果 replacer 是一个数组，则仅转换该数组中具有键值的成员。成员的转换顺序与键在数组中的顺序一样。当 value 参数也为数组时，将忽略 replacer 数组。
* space:
可选，文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 也可以使用非数字，如：\t。

**使用实例：**

```js
var obj = { "name":"runoob", "alexa":10000, "site":"www.runoob.com"};
var myJSON = JSON.stringify(obj);
```

## 参考链接

* [JSON 教程 | 菜鸟教程](https://www.runoob.com/json/json-tutorial.html)
