
> [!tip] 为什么叫 IO `流`
> "流"这个词形象地反映了数据在程序间的流动过程。
> 1. 连续性: 流表示数据的连续传输, 像水流一样从一个地方流向另一个地方
> 2. 顺序性: 数据通常是按顺序流动的

## 常用文件操作
### 创建文件对象
即是 File 对象, 不多介绍
### 获取文件的相关信息 API
- getName
- getAbsolutePath
- getParent
- length: 字节数
- exists
- isFile
- isDirectory
### 目录操作 API
- mkdir: 创建一级目录
- mkdirs: 创建多级目录 `aaa\bbb\ccc\ddd`
- delete: 删除空目录或文件

## IO 流原理和分类
文件在程序中是以流的形式来操作的
按数据单位分为字节流和字符流, 按传输方向又有输入流和输出流, 故 java io 流有4个大的抽象基类
- InputStream
- OutputStream
- Reader
- Writer

### InputSteam
- FileInputStream: 文件输入
- BufferInputStream: 缓存字节输入
- ObjectInputStream: 对象字节输入

#### FileInputStream
有  3 个构造参数, 入参分别为
- File 文件
- FielDescriptor 文件描述符
- String 路径名

##### api
read 读一个字节
read(byte\[\] b) 读取最多 b.length 个自己

### 节点流
- 可以重特定数据源读取数据 (FileReader)
### 处理流
(包装流) `连接` 已经存在的流, 为程序提供更强大的读写功能 (BufferedReader)

处理流包装节点流, 既可以消除不同节点流的实现一查, 也可以提供更方便的方法来完成输入输出

优点:
- 性能的提高: 主要以增加缓冲的方式来提高输入输出的效率
- 操作的便捷: 处理流可能提供了一系列便捷的方法来一次输入输出大量的数据, 

> [!tip] 设计模式
> 为装饰器模式, 不直接与数据源相连, 有点像代理模式, 装饰器模式与代理模式的区别在于装饰器模式用于动态拓展功能, 而代理模式用于控制对对像的访问

### BufferReader
关闭流的时候, 直接关闭外层流即可, 内层流被外层流维护着所以也会被关闭






