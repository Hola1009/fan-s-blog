xml 文件有
1. 声明
2. 元素: 有且只有一个根元素, 区分大小写, 
3. 属性: 
4. 注释
5. cdata 节: 用来包含普通文本

## 转义
- `&lt;` : `<`
- `&gt;` : `>`
- `&a,p;` : `&`
- `&quot;` : `"`
- `&apos;` : `'`

## dom4j
用来解析 xml 文档

### 开发流程
1. 加载 xml 文件
```java
SAXReader reader = new SAXReader();  
 
Document document = reader.read(new File("stu.xml"));
```

2. 获取元素 `\n也算一个元素`
```java
List<Element> elements = rootElement.elements("student");

Element name =  element.element("name");
```

