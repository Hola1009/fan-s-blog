> [!tip] 
> 这篇笔记记载的是我不熟悉的内容

# let 和 var 的 区别
let 是为了解决var不合理的地方
var不合理的地方
- 先使用再声明
- 可以重复声明
- 变量提升, 全局变量, 没有块级作用域等等

# 数据类型
为什么要有数据分类:
- 更加充分和高效的利用内存
- 更加方便程序员使用数据

基本: Number, boolean, string undefined, null

# 模板字符串
使用场景: 拼接字符串和变量
语法: 外面是反引号
```js
let age = 18;
document.write(`刚满${age}`);
```

# 类型转换
## 隐式转换
- \- 号 有隐式转换
- + 号作为正号解析可以转换称字符型
## 显示转换
Number(xxx);
parseInt(xxx);
parseFloat(xxx);

# 断点调试
![[Pasted image 20240718083533.png]]

# 操作数组
arr.push(): 添加到数组的末位,返回数组的长度  push(元素1, 元素2,...);

arr.unshift(): 将一个或多个元素添加到数组开头, 返回数组长度

arr.pop(): 删除最有一个元素, 返回删除元素的值

arr.shift(): 删除第一个元素

arr.splice(): arr.splice(起始位置, 删除几个元素/\*默认删到最后\*\/)

# 函数
## 函数传参设置默认值
![[Pasted image 20240718095656.png]]
## 函数返回值
## 作用域

## 变量的一个坑点
在函数内部一个变量没有声明直接赋值也当全局变量来看

## 匿名函数
![[Pasted image 20240718101936.png]]

1. 函数表达式: 将匿名函数赋值给一个变量, 并且通过变量名来调用, 我们称之为函数表达式
```js
let fn = function () {
	// 函数体
}
```

具名函数可以在任意位置调用;
函数表达式必须在声明后再使用(变量的特点);

2. 立即执行函数
场景介绍: 避免全局变量之间的污染
```js
(function(){
	let num;
})();
(function(){
	let num;
})();
```


# 布尔类型
 '', 0, undefined, null, false, NaN 转成 布尔值都为 false, 其余为true


# 内置对象
## Math
Math.floor() 向下取整;
Math.ceil() 向下取整;
Math.round 四舍五入;
Math.max(1,2,3,4,...) 找最大数;
Math.min(1,2,3,4,...) 找最小值;
Math.abs() 绝对值;
Math.pow() 幂运算;
### 随机数函数
如何生成N-M之间的随机数 左闭右开
```js
N + Math.floor(Math.random() \* (M - N + 1));
```

































