# 声明变量
var let const 优先使用 const, const声明的对象可以修改里面的属性
一个变量经常变得时候用let

# DOM 和 BOM
## DOM
什么时DOM : 文档对象模型, 用来操作标签
### DOM树
: 将标签之间得关系以树得形式表现出来, 直观得将标签与标签之间得关系体现出来
![[Pasted image 20240722114653.png]]
### DOM对象
: 浏览器根据html标签生成的js对象
- 所有的标签属性都可以在这个对象上找到
- 修改这个对象的属性会自动映射到标签上

DOM的核心思想
- 网页内容当作对象处理

document对象
- 是DOM提供的一个对象
- 所以它提供的属性和方法都是用来访问和操作网页内容的
	- 例如: document.write()
- 网页所有内容都在document里面

### 获取DOM对象
根据css选择器来获取 : 选择匹配的第一个元素 可以直接做修改操作
```js
document.querySelector('css选择器');
```
选多个 得到的是伪数组 不能做增删改查
```js
document.querySelectorAll('css选择器');
```

### 其他获取DOM元素的方法
```js
document.getElementById('')
document.getElementByTagName('')
document.getElementByClassName('')
```

### 操作元素内容
对象.innerText  属性
对象.innerHTML 属性

### 操作元素属性
  \
### 操作样式属性

1. 通过style属性来操作 对象.style.样式属性 = 值
2. 通过类名操作 CSS: 在style 标签内将类样式写好, 通过js代码修改目标标签类名
3. 通过 classList 操作类控制css
```js
box.classList.add('active');
box.classList.remove('active');
box.classList.toggle('active');// 切换 有就删掉, 没有就加上
```
### 操作表单元素属性


### 定时器-间歇函数
目标: 能使用定时器函数重复执行代码 
定时器函数可以开启和关闭定时器

1. 开启定时器
```js
setInterval(函数, 间隔时间(ms));
```

2. 关闭定时器 ```
```js
clearInterval(变量名);
```

### 事件监听
- 语法
```js
元素对象.addEventListener('事件类型', 要执行的函数);
```

- 事件监听三要素
事件源: 那个dom元素被事件触发了, 要获取dom元素
事件类型: 用什么方式触发, 比如鼠标点击 click, 鼠标经过 mouseover
事件要调用的函数

#### 事件类型
##### 鼠标事件
click 点击;
mouseenter 鼠标经过;
mouseleave 鼠标离开;

---
##### 焦点事件
foucs 获得焦点;
blur 失去焦点;

---

##### 键盘事件
keydown 键盘按下;
keyiup 键盘抬起触发;

---

##### 文本事件
input 用户输入事件;


#### 事件对象
1. 什么是事件对象?
也是个对象, 这个对象里有事件触发时的相关信息
2. 事件对象在哪里
在事件绑定的回调函数的第一个参数就是

#### 评论回车发布
需求: 按下回车键发布信息
1. 用到按下键盘事件 keydown 和 keyup 
```js
tx.addEventListener('keyup', function(e) {
	if(e.keu === 'Enter') {
		
	}
})
```

#### 环境对象
this 当前函数运行时所处的环境
普通函数 指向的时window
谁调用 this 就是谁

#### 回调函数
函数A作为参数传给函数B, A就是回调函数

#### TAB栏切换

#### 全选
专选 checked 的伪类选择器
```css
ss: checked {

}
```

### 事件流
捕获: 从大到小
冒泡: 从小到大

#### 事件捕获
简单了解事件捕获执行过程
- 事件捕获概念
从dom的根元素开始去执行对应的事件
- 事件捕获需要写对应代码才能看到效果

```js
DOM.addEventListener('事件类型', 函数, 是否用捕获机制(默认是冒泡))
```
#### 事件冒泡
默认存在, 一次向上调用父级 同名元素

#### 阻止冒泡
防止影响父级元素, 限制事件在当前元素内
```js
// 也可以用来阻止捕获
事件对象.stopPropagatin()
son.addEventListener('click', function (e) {

  alert('我是儿子')

  // 组织流动传播  事件对象.stopPropagation()

  e.stopPropagation()

})
```

> [!tip] 小芝士
> 事件event对象是指在浏览器中触发事件时，浏览器会自动创建一个event对象，其中存储了本次事件相关的信息，包括事件类型、事件目标、触发元素等等。浏览器创建完event对象之后，会自动将该对象作为参数传递给绑定的事件处理函数，我们可以在事件处理函数中通过访问event对象的属性和方法，来获取事件的相关信息，并做出后续的逻辑处理。
#### 解绑事件
```js
//1
btn.onclick = null
//2
btn.removeEventListener('click', 函数名)// 所以匿名函数无法这样解绑
```
mouseover 支持冒泡;
mouseenter 不支持

#### 事件委托
事件委托是利用事件流的特征解决一些开发需求的知识技巧
- 优点: 减少注册次数, 可以提高程序性能
- 原理: 事件委托其实是利用事件冒泡的特点
e.target 获得当前点击对象
e.target.tagName 获取点击对象标签名

#### 组织默认行为
```js
//比如阻止提交, 跳转
e.preventDefault()
```

#### 页面加载事件
```js
// 等待页面全部资源加载完毕再执行回调函数
window.adEventListener('load', function () {
	const btn = document.querySelector('button');
	...
})
```

#### 页面加载事件
- 当初始的html文档被完全加载和解析完成之后, DOMContentLoaded事件被触发, 而无需等待样式表, 图像等全加载
- 事件名: DOMContentLoaded
- 监听页面DOM加载完毕
	- 给document添加DOMContentLoaded事件

```js
document.addEventListener('DOMContentLoaded', function() {
	
});
```

#### 页面滚动事件
scroll
```js
window.addEventListener('scorll', function() {
	
})
```
