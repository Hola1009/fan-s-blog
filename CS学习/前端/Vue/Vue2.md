 ## Vue 快速上手
概念/创建实例/插值表达式/响应式特性/开发者工具
### Vue 是是什么
Vue是一个用于 构建用户界面的渐进式框架
1. 构建用户界面: 基于 数据 动态 渲染 页面
2. 渐进式: 循序渐进的学习
3. 框架: 一套完整的项目解决方案

### 创建 Vue 实例, 初始化渲染
1. 准备容器
2. 引包: 开发版本
   ```java
   <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
```
3. 创建 Vue 实例
4. 指令配置项 -> 渲染数据
	1. el: 挂载点
	2. data: 提供数据

### 插值表达式
1. 作用利用表达式进行插值, 渲染到页面
```html
{{msg}}
```

### Vue 的响应式特性
数据变化, 视图自动更新

底层原理是Vue帮我们完成DOM操作, 让开发这聚焦于数据 -> 数据驱动视图

### 开发者工具

## Vue 指令
根据不同指令, 针对标签去实现不同的 功能
指令: 带有 v-前缀 的特殊 **标签属性**
1. v-html: 动态设置 innerHtml 具有解析标签的能力
2. v-show: 用来控制元素的显示隐藏
   v-show = "表达式" 表达式值为 布尔值
   底层是给标签加上了 dispaly = none 属性
   适合频繁切换显示隐藏的场景 开销小
3. v-if: 用来控制元素的显示隐藏(条件渲染)
   底层原理是控制元素的 创建 和 移除 
   适合不频繁切换的场景
4. v-else/v-else-if: 辅助 v-if 进行判断渲染
   注意: 要紧挨着 v-if 使用
   ![[Pasted image 20240825183308.png]] 
5. v-on: 注册事件 = 添加监听 + 提供处理逻辑
   v-on: 事件名 = "内敛语句"
   v-on: 事件名 = "methods中的函数名"
   `v-on:` 可以替换成 @
6. v-bind: 动态设置html的标签属性 -> src url title ...
   v-bind: 属性名= "表达式" (不能直接把表达式赋值给属性, 所以使用v-bind)
   简写: :属性名 = "表达式"
7.  v-for:  
   作用: 基于数据循环, 多次渲染整个元素 -> 数组, 对象, 数字
   v-for = "(item, index) in 数组"
   v-for的默认行为会尝试 原地修改元素 (就地复用) 所以要给元素加key, 便于Uue进行列表项的正确排序复用.
   
8. v-model: 
作用: 给 表单元素 使用, 双向数据绑定 -> 可以快速 获取 或 设置 表单元素内容
- 数据变化 -> 视图自动更新
- 视图变化 -> 数据自动更新


> [!tip]
> data中的数据可以通过 this.isShow 这种全局的方式访问

### 综合案例
功能需求
1. 列表渲染
2. 删除功能
3. 添加功能

## 指令补充
### 指令修饰符
通过 "." 指明一些指令 后缀, 不同 后缀 封装了不同的处理操作 -> 简化代码
1. 按键修饰符 
   **\@keyup.enter**

2. v-model 修饰符 
   **v-model.trim** 去除首尾空格
   **v-model.num** 转数字

3. 事件修饰符
   **@事件名.stop** 阻止冒泡
   **@事件名.prevent** 阻止默认行为

### v-bind 对于样式控制的增强
为了方便开发者进行样式控制, Vue拓展了 v-bind 的语法, 可以针对 class 类名 和 style 行内样式 进行控制

v-bind 对于样式控制的增强-操作class
语法: class = "对象/数组"
1. 对象 -> 键就是类名, 值是布尔值. 如果值是 true, 有这个类, 否则没有这个类 (适用场景, 来回切换)
2. 数组-> 数组 →数组中所有的类，都会添加到盒子上，本质就是一个class 列表 (适用场景:批量添加或删除类)

v-bind 对于样式控制的增强 - 操作上style
语法: style="样式对象"
![[Pasted image 20240826165711.png]]

### v-model 应用于其他表单元素
输入框 input:text -> value
文本框 textarca -> value
复选框 input: checkbox -> checked
单选框 input:radio -> checked
下拉菜单 select -> value

### 计算属性
概念: 基于现有的数据, 计算出来的新属性. 依赖的数据变化, 自动重新计算

```js
computed: {
	totelCount () {
		return this.a + this.b;
	}
}
```

### 计算属性完整写法
默认的计算属性, 只能读取不能修改, 修改需要完整写法
```js
computed: {
	get() {
	
	}
	set(修改的值) {
	
	}
}
```

### watch 监听器
作用: 监视数据便会, 执行一些 业务逻辑 或 异步操作
语法:
1. 简单写法 -> 简单类型数据 ,直接监视
2. 完整写法 -> 添加额外配置项
![[Pasted image 20240827092456.png|300]]

```js
watch: {
word (newValue, oldValue) {
  clearTimeout(this.timer);
  this.timer = setTimeout(async () => {
	const res = await axios({
	  url:xxx,
	  params: {
		words: newValue
	  }
	})
  }, 300);
  this.result = res.data.data;
}
}
})
```

### 监听器的完整写法
完整写法 -> 添加额外配置项
1. deep: true 对复杂类型深度监视(对对象的每个属性都进行监视)
2. immediate: true 一进页面也可以执行

![[Pasted image 20240827160743.png|400]]

### 水果购物车
需求说明:
1. 渲染功能
2. 删除功能
3. 修改个数
4. 全选反选 : 计算属性 computed 完整写法 get/set
   ```js
   isALl: {
	get() {
	  return this.fruitList.every(item => item.isChecked);
	},
	set(value) {
	  this.fruitList.forEach(item => item.isChecked = value);
	}
  },
```
5. 统计 选中的 总价 和 总输俩个
6. 持久化到本地 watch监视, localStorage, JSON.stringify, JSON.parse


## 工程化开发
### Vue的生命周期 和 生命周期的四个阶段
![[Pasted image 20240827192310.png]]

### create 应用
立刻获取焦点
![[Pasted image 20240827193549.png|600]]
初始化数据
![[Pasted image 20240827193632.png|600]]

```js
/**
   * 接口文档地址：
   * https://www.apifox.cn/apidoc/shared-24459455-ebb1-4fdc-8df8-0aff8dc317a8/api-53371058
   *
   * 功能需求：
   * 1. 基本渲染
   *    (1) 立刻发送请求获取数据 created
   *    (2) 拿到数据，存到data的响应式数据中
   *    (3) 结合数据，进行渲染 v-for
   *    (4) 消费统计 => 计算属性
   * 2. 添加功能
   *    (1) 收集表单数据 v-model
   *    (2) 给添加按钮注册点击事件，发送添加请求
   *    (3) 需要重新渲染
   * 3. 删除功能
   *    (1) 注册点击事件，传参传 id
   *    (2) 根据 id 发送删除请求
   *    (3) 需要重新渲染
   * 4. 饼图渲染
   */
```

### 小黑记账 EChart 饼图
```js
mounted () {
  this.myChart = echarts.init(document.querySelector('#main'))
  this.myChart.setOption({
	// 大标题
	title: {
	  text: '消费账单列表',
	  left: 'center'
	},
	// 提示框
	tooltip: {
	  trigger: 'item'
	},
	// 图例
	legend: {
	  orient: 'vertical',
	  left: 'left'
	},
	// 数据项
	series: [
	  {
		name: '消费账单',
		type: 'pie',
		radius: '50%', // 半径
		data: [
		  // { value: 1048, name: '球鞋' },
		  // { value: 735, name: '防晒霜' }
		],
		emphasis: {
		  itemStyle: {
			shadowBlur: 10,
			shadowOffsetX: 0,
			shadowColor: 'rgba(0, 0, 0, 0.5)'
		  }
		}
	  }
	]
  })
},
```

更新饼图
```js
// 更新图表
this.myChart.setOption({
  // 数据项
  series: [
	{
	  data: this.list.map(item => ({ value: item.price, name: item.name}))
	}
  ]
})
```

## 工程化开发 & 脚手架 Vue CLI
Vue CLl 是 Vue 官方提供的一个全局命令工具。
可以帮助我们快速创建一个开发 Vue 项目的标准化基础架子。【集成了 webpack 配置

1. 全局安装(一次):yarn globaladd @vue/cli 或 npmi(@vue/cli -g
2. 查看 Vue 版本:vue --version
3. 创建项目架子:vue create project-name(项目名-不能用中文)
4. 启动项目:yarn serve 或 npm run serve(找package.json)
### 组件化开发 & 根组件
组件化:一个页面可以拆分成一个个组件，每个组件有着自己独立的结构、样式、行为。好处:便于维护，利于复用提升开发效率
组件分类: 普通组件, 根组件

根组件: 整个应用最上层的组件

### 普通组件的注册使用
1. 局部注册: 只能在注册的组件内使用
   创建 .vue 文件
   在使用的组件内导入并注册
   使用:
   当成 html标签使用 `<组件名></组件名>` 
   注意:
   使用大驼峰命名法
   
2. 全局注册: 所有组件内都能使用
   创建.vue文件
   main.js中进行全局注册
```js
import xxx form xxx
Vue.conponent('HmButton', HmButton)
```

### 小兔鲜首页 - 组件拆分
页面开发思路:
1. 分析页面, 按模块拆分组件, 搭架子(局部或全局注册)

### 组件的三大组成部分

结构 样式 逻辑

### 组件冲突 scoped
sciped 原理:
1. 给当前组件模板的所有元素, 都会被加上应给自定义属性 data-v-hash 值

默认情况下: 写在组件中的样式会全局生效 -> 


### data 是一个函数
保证每个组件实例, 维护独立的一份数据对象, 每次创建新的组件实例, 都会执行新的一次data函数, 得到一个新对象

### 什么是组件通信
组件通信, 是指 组件与组件 之间的数据传递
- 组件的数据是独立的, 无法直接访问其他组件的数据
- 想用其他组件的数据 -> 组件推荐哦你更新

组件关系分类:
1. 父子关系
2. 非父子关系

### 组件通信方案
父传子 props
```vue
<template>
  <div>
    <Son :title="myTitle"></Son>
  </div>
</template>

<script>
import Son from './VSon.vue';
export default {
  data () {
    return {
      myTitle: "呼唤! 呼唤!"
    }
  },
  components: {
    Son
  }
}
</script>

//子中
export default {
  props: ["title"]
};

```

子传父 $emit
![[Pasted image 20240829172022.png|400]]
![[Pasted image 20240829172040.png|400]]


### 什么是 props
组件上 注册的一些自定义属性
作用: 向子组件传递数据

### props 校验
思考: 组件的 prop 可以乱传吗?
作用: 给组件的prop指定验证要求, 不符就错误提示

语法:
1. 类型校验
2. 非空检验
3. 默认值
4. 自定义校验
```js
props {
	属性名: 类型 
}
props {
	属性名: {
		type: 类型,
		requried: ture,
		default: 0
		validator () {
			return false;
		}
	}
}
```

### prop & data
   data 中的数据随便改
   但是 prop 中的数据不可以, 外部数据是不能直接改的 (谁的数据谁负责)
只需要通知老爹修改 即遵守单向数据流


### 小黑记事本组件版本

### 非父子通信 event bus 事件总线
作用: 非父子至今啊, 进行简易消息传递 (复杂场景 -> Veux)
1. 创建一个都能访问到的事件总线
```vue
import vue from 'vue'
const Bus = new Vue()
export default Bus
```
2. A 组件接收方 , 监听 Bus 实例 事件
```js
created () {
	Bus.$on('sendMsg', (msg) => {
		this.msg = msg
	})
}
```

3. B组件发送方, 触发 Bus 实例 事件
```js
Bus.$emit('sendMsg', 'xxxx')
```

### 跨层级共享数据
provide & inject 作用: 跨层级共享数据
1. 父组件 provide 提供数据
```js
provide () {
	color: this.color
}
```

孙子组件
```js
inject:[color]
```
基本数据类型是非响应式的
而引用类型是响应式的

### v-model 原理
本质 语法汤, value属性 和input事件的组合
作用: 提供双向数据绑定
```html
<input :value="msg" @input="msg = $event.target.value" type="text">
```

注意: $event 用于在模板中, 获取事件的形参

### 表单类组件封装 v-model 简化代码
1. 表单类组件的封装
父传子: 数据 应该是父组件props传递过来的, v-model 拆解绑定数据
子传父: 监听输入, 子传父值给父亲修改

2. v-model 简化代码
![[Pasted image 20240829195529.png]]
### .sync 修饰符
作用: 可以实现 子组件 与 父组件数据 的 双向绑定, 简化代码
特点: prop 属性名, 可以自定义, 非固定 value
场景: 封装弹框类的基础组件, visible属性 true显示false隐藏
本质: 就是 :属性名 和 @update:属性名 合写 

![[Pasted image 20240829202545.png]]

### ref 和 $refs
作用:利用 ref 和 $refs 可以获取 dom 元素 或组件实例
作用范围为 当前组件

1. 目标组件 - 添加ref属性
![[Pasted image 20240829205220.png]]
2. 恰当时机, 通过this.$refs
![[Pasted image 20240829205235.png]]

### Vue 异步更新
需求: 编辑标题, 编辑框自动聚焦
1. 点击编辑, 显示编辑框
2. 让编辑框, 立刻获取焦点
```js
// 显示输入框
this.isShowEdit = true
this.$refs.inp.focus()
```

问题: 显示之后, 立即获取焦点是不成功的

```js
// 等dom更新完立即执行
this.$nextTick(() => {
	this.$refs.inp.focus()
})
```



### 自定义指令
![[Pasted image 20240902155343.png]]

自定义指令: 自己定义的指令, 可以封装一些 dom 操作, 拓展额外功能
需求: 当页面加载时, 让元素将获得焦点

![[Pasted image 20240902155652.png]]
当前元素被插入后执行
```js
// 局部注册
Vue.directive('指令名', {
	"inserted" (el) { // 会在元素插入后执行
	// el 指令所绑定的元素
	el.focus();
	}  
})

// 局部注册
export default {
	directives: {
		// 指令名: 指令配置项
		focus: {
			inserted (el) { // 会在元素插入后执行
				el.focus();
			}
		}
	}
}
```

### 自定义指令 - 指令的值
需求: 实现一个 color 指令 - 传入不同的颜色, 给标签设置文字颜色
- 语法: 在绑定指令时, 可以通过"等号"的形式为指令 绑定 具体的参考值
```html
<div  v-color="red">我是内容</div>
```
- 通过binding.value 可以拿到指令值, 指令值修改会 触发 update 函数
```js
export default {
	directives: {
		color: {
			// 指令名: 指令配置项
			inserted (el, binding) { // 会在元素插入后执行
			binding.value = blue;
				el.focus();
			},
			update (el, binding) { // 指令的值 修改的时候触发
				
			}
		}
	}
}
```


 1. 通过指令的相关语法, 可以应对更复杂指令封装场景
 2. 指令值得语法: 


### 自定义指令 - v-loading 指令封装
场景: 实际开过程中, 发送请求需要时间, 在请求得数据未回来时, 页面处于空白状态 => 用户体验不好

核心思路:
1. 准备类型 loading, 通过伪元素(:before)提供遮罩层
2. 添加或移除类名, 实现loading蒙层得添加移除
3. 利用指令语法, 封装 v-loading 通用指令
   inserted 钩子中, binding.value 判断指令的值, 设置默认状态
   update 钩子中, binding.value 判断指令的值, 设置默认状态

### 插槽-默认插槽
作用: 让组件内部的一些结构支持自定义
插槽基本语法:
1. 组件内需要定制的结构部分, 改用\<slot\>\<\\slot\>占位
2. 使用组件时, \<MyDialog\>\<\\MyDialog\> 标签内部, 传入结构替换slot

### 插槽 - 后备内容
通过插槽完成了内容的定制, 传什么显示什么, 但是不传, 则是空白 -> 需要默认内容 (默认值)
加入slot标签内的内容就是默认内容

### 插槽 - 具名插槽
具名插槽语法:
多个插槽时需要给slot插槽取名
```html
<!-- 出处 -->
<div name="xxxName"><\div>

<!-- 用处 template 标签是必须的 -->
<template v-slot:head> <\template>

``` 


### 插槽 - 作用域插槽
定义slot插槽的同时, 是可以传值的, 给插槽上绑定数据, 将来使用组件时可以使用
场景: 封装表格组件
1. 父传子, 动态渲染表格内容
2. 利用默认插槽, 定制操作列
3. 删除或查看都需要用到 当前项目的id, 属于 组件内部数据 需要传给父组件

基本使用:
1. 给slot标签, 以添加属性的方式传值
```html
<slot :id="" msg="" ><\slot>
```
2. 所有添加的属性, 都会被收集到一个对象中
```json
{
	id: 3,
	msg: 'ceshi',
	data: {xxx: 999, sss: 666}
}
```
3. 在template 中, 通过 `#插槽名="obj"`接收, 默认插槽名 未 default

作用域插槽的作用:
可以给插槽上绑定数据, 供将来使用组件时使用

### 综合案例 - 商品列表
#vue/靓靓组件/双击聚焦显示输入框
  标签组件
  实现功能:
  1. 双击显示, 并且自动聚焦: 双击事件 dblclick  vue是异步操作, 得dom加载完了才能获取焦点
```js
// 封装全局 插入选然后 获取焦点指令
Vue.directive('fucos', {
	inserted (el, binding) {
		el.focus()
	}
})
```
2. 失去焦点隐藏输入框 失焦事件 @blur
3. 回显标签信息 : 回显标签信息是父组件传递过来的, 在子组件内部可以对其进行修改 子传父 v-model 简化代码, 组件内部通过props接收, :value 设置给输入框
4. 内容修改了, 回车 => 修改标签内容 @keyup.enter, 触发事件 $emit('input', e.target.value)

表格组件
1. 动态传递表格数据渲染
2. 表头支持用户自定义
3. 主题支持用户自定义

### 单页应用程序
页面按需更新

### 路由的介绍
生活中的路由: 设备和ip的映射关系

vue中的路由: 路径和组件的映射关系


### VueRouter 的介绍
目的: 认识插件 VueRouter, 掌握 VueRouter 的基本使用步骤
作用: 修改地址栏路劲时, 切换显示匹配的组件
说明: Vue 官方的一个路由插件, 是一个第三方报


### VueRouter 的使用 (5 + 2)
1. 下载VueRouter模块到当前工程 Vue2对应3.6.5
```shell
yarn add vue-router@3.6.5
```
2. 引入
```js
import VueRouter form 'vue-router'
```
3. 安装注册
```js
Vue.use(VueRouter)
```
4. 创建路由对象
```js
const router = new VueRouter()
```
5. 注入, 将路由对象注入到new Vue实例中, 建立关联
```js
new Vue({
	render: h => h(App),
	router
}).$mount('#app')
```

```js
import VueRouter from 'vue-router'

Vue.use(VueRouter) // VueRouter插件初始化

const router = new VueRouter({
  // routes 路由规则们
  // route  一条路由规则 { path: 路径, component: 组件 }
  routes: [
    { path: '/find', component: Find },
    { path: '/my', component: My },
    { path: '/friend', component: Friend },
  ]
})
```


两个核心步骤
1. 创建需要的组件 (views目录), 配置路由规则
```js
const router = new VueRouter({
  // routes 路由规则们
  // route  一条路由规则 { path: 路径, component: 组件 }
  routes: [
    { path: '/find', component: Find },
    { path: '/my', component: My },
    { path: '/friend', component: Friend },
  ]
})
```
2. 配置导航, 配置路由出口 (路劲配置的组件显示的位置)
```html
<div class="footer_wrap">
  <a href="#/find">发现音乐</a>
  <a href="#/my">我的音乐</a>
  <a href="#/friend">朋友</a>
</div>

<div class="top">
  <!-- 路由出口 → 匹配的组件所展示的位置 -->
  <router-view></router-view>
</div>
```

### 组件目录存放文帝

注意: .vue 文件本质无区别
路由相关的组件, 为什么要放在 views 目录? 

组件分类: .vue文件分为两类; 页面组件 & 复用组件

分类开来 更易维护

views 配合路由用
component 常用于复用

## 路由进阶
### 路由模块封装抽离
问题: 所有的路由配置都堆在main.js中合适么?
目标: 将路由模块抽离出来. 好处: 拆分模块, 利于维护
/src/router/index.js

允许使用 `@` 来代表src的绝对路径
```js
import Find from '@/views/Find'
import My from '@/views/My'
import Friend from '@/views/Friend'

import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter) // VueRouter插件初始化

// 创建了一个路由对象
const router = new VueRouter({
  // routes 路由规则们
  // route  一条路由规则 { path: 路径, component: 组件 }
  routes: [
    { path: '/find', component: Find },
    { path: '/my', component: My },
    { path: '/friend', component: Friend },
  ]
})

export default router
```
路由模块的封装抽离的好处
拆分模块, 利于维护

如何快速引入组件
用 `@` 指代 src 的绝对路径
### 声明式导航 & 导航高亮 
需求: 实现导航高亮效果
vue-router 提供了一个全局组件 router-link (取代 a 标签)
```html
<router-link to="/find">发现音乐<\router-link>
```
1. 能跳转, 配置 to 属性指定路径. 本质还是 a 标签, `to 无序 #`
2. 能高亮, 默认机会提供高亮类名, 可以直接设置高亮样式

### 声明式导航 - 两个类名
说明: 我们发现 router-link 自动给当前导航添加了 两个高亮类
![[Pasted image 20240903181827.png]]
一个是精确, 一个是模糊
 模糊: /find/one 有效
 精确: /fing 有效

说明: 两个类名太长, 我们希望能定制怎么办
```js
const router = new VueRouter({
	routes: [...],
	linkActiceClass: "类名1",
	linkExactActiceClass: "类名2"
})
```

### 声明式导航 - 跳转传参
目标: 在跳转路由时, 进行传参
1. 查询参数传参
 语法格式如下:
 to="/path?参数名=值"
 
 对应页面组件接收传递过来的值
 $route.query.参数名

2. 动态路由传参
配置动态路由
```js
const router = new VueRouter({
	routes: [
		{
			path: '/search/:words',
			component: Search
		}
	]
})
```
配置导航链接
to="/search/参数值"
对应页面组件接收传递过来的值
$route.params.参数名

两种方式的区别
1. 查询参数传参 (比较适合多个参数)
2. 动态路由传参(优雅简洁, 传单个参数比较方便)



### 路由重定向
问题: 网页打开, url 默认是 / 路径, 未匹配到组件时, 会出现空白
说明: 重定向 -> 匹配path后, 强制添砖到path路径
语法: {path: 匹配路径, redirect: 重定向到的路径}


### Vue路由 404
作用: 当路径找不到匹配时, 给个提示页面
位置: 配置在路由最后
语法: path: "\*" (任意路径) - 前面不匹配就命中最后这个

### Vue路由 - 模式设置
问题: 路由的路径看起来不自然, 有#, 能否切成真正路径形式
- hash路由(默认)
- history路由(常用) 上线后需要服务断支持
![[Pasted image 20240903195942.png]]



### 编程式导航 - 基本跳转
问题: 点击按钮跳转如何实现?
编程式导航: 用JS代码来进行跳转

两种语法:
path路径跳转 简易方便
![[Pasted image 20240903200945.png]]

name命名路由跳转 (适合 path 路径长的场景)
![[Pasted image 20240903201138.png]]
![[Pasted image 20240904090859.png]]

编程式导航有几种跳转方式
1. 通过路径跳转
2. 通过路由名字跳转

### 路由传参
问题: 点击搜索按钮, 跳转需要传参如何实现?
两种传参方式: 查询参数 + 动态路由传参
两种跳转方式, 对于两种传参方式都支持:
1. path 路径跳转参数
2. name 命名路由跳转传参(query传参)

```js
this.$router.push({
	name: '路由名'
	query: {参数名: 参数值} // 命名路由传参
	params: {参数名: 参数值} // 动态路由传参
})
```

### 综合案例
1. 配路由
	- 首页 和 面经详情, 两个一级路由
	- 首页内嵌四个可切换页面(嵌套二级路由)
![[Pasted image 20240904093206.png|400]]
2. 实现功能
	- 首页请求渲染
	- 跳转传参 到 详情页, 详情页渲染
	- 组件缓存, 性能优化


二级路由配置
```js
routes: [
  {
    path: '/'
    component: Layout,
    children: [
	    
    ]
  }
]
```

### 首页请求动态渲染
1. 安装 axios
2. 看接口文档, 确认请求方式, 请求地址, 请求参数


### 组件缓存 keep-alive
问题: 从面经 点到 详情页, 又点回来, 数据重新加载了 -> 希望回到原来的位置
原因: 路由跳转后, 组件被销毁了, 返回回来组件又被重建了, 所以数据重新被加载了

1. keep-alive是什么
keep-alive 是Vue的内置组件, 当他包裹动态组件时, 会缓存不活动的组件实例, 而不是销毁他们
keep-alive 是一个抽象组件: 它本身不会渲染成一个 DOM 元素, 也不会出现在父组件链内
2. 优点
在组件切换过程中 把切换出去的组件保留在内存中，防止重复渲染DOM减少加载时间及性能消耗，提高用户体验性。

![[Pasted image 20240904193339.png]]

3. keep-alive 的三个属性
include: 组件名数据, 只有匹配的组件会被缓存
exclude: 组件名数组, 任何匹配的组件都不会被缓存
max: 最多可以缓存多少组件实例

被缓存的组件多了两个生命周期
actived 激活时
deactived 失活时

### 自定义创建项目
 目标: 基于VueCli 自定义创建项目架子

   创建项目的时候选择 manually select features
![[Pasted image 20240904200712.png]]

### 代码规范错误
如果你的代码不符合standard的要求, ESlint会跳出来提示

手动修正
自动修正: 装ESLint插件
```json
// 当保存的时候, eslint自动帮我们修复错误
"editor.codeActionsOnSave": {
	"source.fixAll": true
},
// 保存代码不自动格式化
"editor.formatOnSave": false
```

### vuex概述
目标: 明确vuex是什么, 应用场景, 优势
1. 是什么:
vuex是一个vue的状态管理工具, 状态就是数据
vuex 是一个插件, 可以帮我们管理vue通用的数据(多组件共享的数据)

2. 场景:
某个状态 在 很多个组件 来使用
多个组件 共同维护 一份数据

![[Pasted image 20240904204525.png]]

3. 优势
共同维护了同一份数据, 数据集中化管理
响应式变化
操作简洁(vuex提供了一些辅助函数)

### 基于vuex

### 核心概念 - state 状态

目标明确如何给仓库 **提供** 数据, 如何 **使用** 仓库数据
1. 提供数据
State 提供唯一的公共数据源, 所有共享的数据都要统一放到 Store 中的 State 中存储. 在 state 对象中可以添加我们要的共享数据

```js
const store = new Vuex.Store({
	state: {
		count: 101
	}
})
```

2. 使用数据
通过 store 直接访问
```js
this.$store
impore 导入 store

模板中: {{$store.state.xxx}}
组件逻辑中: this.$store.state.xxx
js模板中: store.state.xxx
```
通过辅助函数(简化)
mapState是辅助函数, 帮助我们把 store 中的数据 自动应色号到组件的计算属性中

导入mapState =>  数组方式引用 =>  展开运算符映射
```js
import { mapState } from 'vuex'
```

```js
mapState(['count'])
```

```js
computed: {
	...mapState(['count', 'title'])
}

在模板中就可以直接是使用了
{{count}}
{{title}}
```

### 核心概念 - mutations
目标: 明确 vuex 同样遵循`单向数据流`, 组件中不能直接修改仓库中的数据
通过: strict: true 可以开启严格模式, 防止错误方式修改
方法: 通过 mutations 提交 给仓库, 让仓库去改


```html
<button @click="handleAdd(1)">值 + 1</button>
```

```js
handleAdd (n) {
  this.$store.commit('addCount', {
	count: n,        
	msg: '哈哈'
  })
},
```

目标掌握 mutations的传参语法
提交载荷 **有且只能有一个参数** ，如果需要多个参数，包装成一个对象
```js
// 2. 通过 mutations 可以提供修改数据的方法
const store = new Vuex.store({
	state: {
		count: 0
	}
	mutations: {
		// 所有mutation函数，第一个参数，都是 state
		// 注意点：mutation参数有且只能有一个，如果需要多个参数，包装成一个对象
		addCount (state, obj) {
		 console.log(obj)
		 // 修改数据
		 state.count += obj.count
		},
	}
})
```

### 辅助函数: mapMutations
目标: 掌握辅助函数 mapMutations, 映射方法
mapMutations 是把位于 mutations中的方法提取出来, 映射到 **组件methods** 中

```js
mutations: {
	subCount (state, n) {
		state.count -= n;
	}
}
```

```js
import { mapMutations } from 'vuex'

methods: {
	...mapMutations(['subCount'])
}

```
等价于
```js
methods: {
	subCount (n) {
		this.$store.commit('subCount', n)
	}
}
```

### 核心概念 - actions
目标: 明确 actions 的基本语法,  专门 **处理异步** 操作
需求: 一秒钟后, 修改 state 的 count 成 666
说明: mutations 必须是同步的

1. 提供 action 方法
```js
// action 不能直接操作 state , 需要commit mutation
actions: {
	changeConutAction (context, num) { // 此处未分上下文, context 可当作 state
		// setTimeOut 模拟异步, 以后大部分场景是发送请求
		setTimeOut(() => {
			context.commit('changeCount', num)
		}, 1000)
	}
}
```

2. 使用action 方法
```js
hndleChange () {
	this.$store.dispatch('changeCountAction', 666)
}
```
 
###  辅助函数 - mapActions
目标: 掌握辅助函数 mapActions, 映射方法
mapActions 是把位于 actions中的方法提取出来, 映射到组件methods中

```js
import { mapActions } from 'vuex'

methods: {
	...mapActions(['changeCountAction'])
}
```
等价于
```js
methods: {
	hndleChange () {
		this.$store.dispatch('changeCountAction', 666)
	}
}
```


### 核心概念 - getters
目标: 掌握核心概念 getters 的基本语法 (类似于计算属性)
说明: 除了 state 之外, 优势我们还需要从 state 中派生出一些状态, 这些状态是依赖 state 的, 此时会用到 getters
例如: state中定义了list, 为 1 - 10 的数组, 组件中, 需要显示所有大于5的数据
```js
state: {
	list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ]
}
```
1. 定义 getters
```js
getters: {
	filterList (state) {
		return state.list.filter(item => item > 5)
	}
}
```

2. 访问getters
通过 store 访问
```js
{{$store.getters.filterList}}
```
通过辅助函数 mapGetters 映射
```js
computed: {
	...mapGetters(['filterList'])
}

{{ filterList }}
```

### 核心概念 - 模块 module (进阶语法)
目标: 掌握核心概念 module 模块的创建
由于 vuex 使用单一状态树, 应用的所有状态会集中到一个比较大的对象. 当应用变得非常复杂时, store 对象就会变得臃肿, 难以维护

![[Pasted image 20240906181059.png]]

模块拆分:
user模块: store/modules/user.js

```js
// user模块
const state = {  
 userInfo: {
   name: 'zs',
   age: 18
 },
 score: 80
}
const mutations = {
 setUser (state, newUserInfo) {
   state.userInfo = newUserInfo
 }

const actions = {
 setUserSecond (context, newUserInfo) {
   // 将异步在action中进行封装
   setTimeout(() => {
    // 调用mutation   context上下文，默认提交的就是自己模块的action和mutation
    context.commit('setUser', newUserInfo)
  }, 1000)
}

const getters = {
	// 分模块后，state指代子模块的state
	UpperCaseName (state) {
	  return state.userInfo.name.toUpperCase()  
	}
}  
export default {  
	namespaced: true,
	state,  
	mutations,  
	actions,  
	getters
}
```

写好模块后需要挂载, 导入到 index.js文件中

```js
modules: {
	user,
}
```


尽管已经分模块了, 但其实子模块的状态, 还是会挂到根级别的 state 中, 属性名 就是 模块名

直接通过模块名访问
```js
$store.state.模块名.xxx
```
通过 mapState 映射
默认根级别的映射 mapState(['xxx'])
子模块的应色号 mapState('模块名', ['xxx']) - 需要开启命名空间
```js
export default {  
	namespaced: true, // 开启命名空间
	state,  
	mutations,  
	actions,  
	getters
}
```

访问getters
```js
{{ $store.getters['user/UpperCaseName'] }}
```

辅助函数获取
```js
computed: {
	...mapGetters('user', ['UpperCaseName'])
}
```

注意: 默认模块中的 mutation 和 actions 会被挂载到全局, 需要开启命名空间, 才会被挂载到子模块

### 综合案例 - 购物车
目标: 构建 cart 购物车模板
说明: 既然明确数据要存 vuex, 建议分模块存, 购物车数据存 cart 模块, 将来还会有 user 模块, article 模块



目标: 基于 json-server工具, 准备后端接口服务环境
1. 安装全局工具 json-server 
2. 代码根目录新建应给 db 目录
3. 将资料 index.json 一如 db 目录
4. 进入 db 目录, 执行
```shell
json-server index.json
```
5. 访问接口测试


目标: 请求获取数据存入 vuex, 映射渲染

**安装 axios** => **准备 actions 和 mutations** => **调用 actions 获取数据**


## 实战项目 - 电商购物


### 调整初始化目录
![[Pasted image 20240907201201.png]]

1. 删除 多余的文件
2. 修改 路由配置 和 App.vue
3. 新增 两个目录 api/utils
   api 接口模块: 发送 ajax 请求的接口模块
   utils 工具模块: 自己疯涨的一些工具方法模块

### vant 组件库
目标: 认识第三方 Vue 组件库 vant-ui
组件库: 第三方 封装 好了很多很多的 组件, 整合到一起就是一个组件库

### vant 全部导入 和 按需导入

目标: 明确 全部导入 和 按需导入 的区别
![[Pasted image 20240907202934.png]]

![[Pasted image 20240907203111.png]]

全部导入:
1. 安装 vant-ui
```shell
yarn add vant@latest-v2
```
2. 在 main.js 中注册
```js
import Vant from 'vant'
import 'vant/lib/index.css'

// 把vant中多有的组件都导入
Vue.use(Vant) 
```
3. 使用测试
```html
<van-button type="primary"> 按钮 </van-button>
```



目标: 阅读文档, 掌握 **按需导入** 的基本使用

1. 安装 vant-ui
2. 安装插件
```shell
npm i babel-plugin-import -D
```
3. babel.config.js 中配置
```js
// 对于使用 babel7 的用户，可以在 babel.config.js 中配置
module.exports = {
  plugins: [
    ['import', {
      libraryName: 'vant',
      libraryDirectory: 'es',
      style: true
    }, 'vant']
  ]
}
```
4. main.js 按需导入注册 (通常会 抽离到 util/vant-ui.js 中)
```js
import { Button } from 'vant'
Vue.use(Button)
// =======================
// 提取到 utils 文件夹下 的 vant-ui.js 文件中
import '@/utils/vant-ui'
```
5. 测试使用
```html
<van-button type="info"></van-button> 
```

### 项目中的 vw 适配

目标: 基于 postcss 插件 实现项目于 vw 适配

1. 安装插件
```shell
yarn add postcss-px-to-viewport@1.1.1
```
2. 根目录新建 postcss.config.js 文件, 填入配置
```js
module.exports = {
  plugins: {
    'postcss-px-to-viewport': {
      // 标准平缪宽高
      viewportWidth: 375
    }
  }
}
```

### 路由设计配置
目标: 分析项目页面, 设计路由, 配置一级路由
但凡是单个页面独立展示的都是一级路由
![[Pasted image 20240909134830.png]]


目标: 阅读vant组件库文档, 实现 底部导航 tabber 

tabbar 标签页
1. vant-ui.js 按需引入
```js
import Vue from 'vue'
import { Tabbar, TabbarItem } from 'vant'
Vue.use(Tabbar)
Vue.use(TabbarItem)
```

2. layout.vue 粘贴官方代码测试
```html
<van-tabbar v-model="active"
  active-color="#07c160">
  <van-tabbar-item icon="home-o">首页</van-tabbar-item>
  <van-tabbar-item icon="apps-o" dot>分类</van-tabbar-item>
  <van-tabbar-item icon="cart-o" info="5">购物车</van-tabbar-item>
  <van-tabbar-item icon="user-o" info="20">我的</van-tabbar-item>
</van-tabbar>
```

目标: 基于底部导航, 完成二级路由配置

![[Pasted image 20240909142938.png]]



目标: 基于笔记, 快速实现登录静态布局

### request 模块 - axios 封装
目标: 将 axios 请求方法, 封装到 request 模块
使用 axios 来**请求后端接口**, 一般都会对 axios 进行 **一些配置**(比如: 配置基础地址, 请求响应拦截器) 所以在项目开发中, 都会对 axios 进行基本的**二次封装**, 单独封装到一个 request 模块中, **方便维护使用**


```js
import axios from 'axios'  
// 创建 axios 实例 将实例进行自定义配置  
// 好处不会污染原始的 axios 实例  
const instance = axios.create({  
	baseURL: 'http://cba.itlike.com/public/index.php?s=/api/',  
	timeout: 1000,  
	headers: { 'X-Custom-Header': 'foobar' }  
})  
  
// 自定义配置 - 请求响应拦截器  
// 添加请求拦截器  
instance.interceptors.request.use(function (config) {  
	// 在发送请求之前做些什么  
	return config  
	}, function (error) {  
	// 对请求错误做些什么  
	return Promise.reject(error)  
})  
  
// 添加响应拦截器  
instance.interceptors.response.use(function (response) {  
	// 2xx 范围内的状态码都会触发该函数。  
	// 对响应数据做点什么 (默认axios会多包装一层data, 需要响应拦截器多扒一层外衣)  
	return response  
	}, function (error) {  
	// 超出 2xx 范围的状态码都会触发该函数。  
	// 对响应错误做点什么  
	return Promise.reject(error)  
})
```

### 图形验证码功能完成
目标: 基于请求回来的 base64 图片, 实现图形验证功能



### api 接口模块 - 封装图片验证码接口
目标: 将请求封装成方法, 统一存放导 api 模块, 与页面分离

api 模块 用来存放封装好的请求函数

好处
1. 请求与页面分离
2. 相同的请求可以复用
3. 请求进行统一管理

![[Pasted image 20240909190956.png]]

### Toast 请提示
注册安装
```js
import { Toast } from 'vant'
Vue.use(Toast)
```

两种方式调用
1. 导入调用
```js
import { Toast } from 'vant'
Toast{'提示内容'}
```
2. 通过this调用 (必须在组件内)
```js
this.$toast('提示内容')
```

### 验证码倒计时
目标: 事先短信验证倒计时功能

步骤分析:
1. 点击按钮, 实现 倒计时 效果
2. 倒计时之前的 校验处理 (手机号, 验证码)
3. 封装短信验证请求接口, 发送请求添加提示

### 封装 API 接口实现登录
目标: 封装 api 登录接口, 实现登录功能

步骤分析:
1. **封装登录接口**
2. 登录前的校验 ( 手机号, 图形验证码, 短信验证码)
3. **调用方法,** 发送请求, 成功添加提示并跳转

### 响应拦截器统一处理错误提示

目标: 通过响应拦截器, 统一处理接口的错误提示
问题: 每次请求, 都有可能会错误, 就都需要错误提示
说明: 响应来解其是咱们拿到数据, 第一个 数据流转站, 可以在里面统一处理

### 登录权证信息存储
目标: vuex 构建 user 模块 存储登录权证(token & userId)
说明:
1. token 存入 vuex 的好处, 易拉取, 响应式
2. vuex 需要分模块 => user 模块
![[Pasted image 20240910171340.png]]

### storage 存储模块 - vuex 持久化管理

目标: 封装 storage 存储模块, 利用本地存储, 进行 vuex 持久化处理
问题1: vuex 刷新会丢失, 怎么办?
```js
localStorage.setItem('xxx', JSON.stringfy(xxx))
```
问题2: 每次存取操作太长, 太麻烦
将存储操作封装成一个模块
![[Pasted image 20240910183139.png]]
```js
import { setInfo, getInfo } from '@/utils/storage'
  
export default {
  namespaced: true,
  state () { // 提供属性
    return {
      // 个人权证相关
      userInfo: getInfo()
    }
  },
  mutations: { // 提供修改属性方法
    // mutation 的第一个参数一定是 state
    setUserInfo (state, obj) {
      state.userInfo = obj
      setInfo(obj)
    }
  },
  actions: { // 提供异步操作
  
  },
  getters: { // 提供计算属性

  }
}


const INFO_KEY = 'hm_shopping_info'

// 获取个人信息
export const getInfo = () => {
  const defaultObj = { token: '', userId: '' }
  const result = localStorage.getItem(INFO_KEY)

  return result ? JSON.parse(result) : defaultObj
}
```




### 添加请求 loading 效果
目标: 统一在每次请求后台时, 添加 loading 效果
背景: 有时候因为网络原因, 一次请求的结果可以需要一段时间后才能回来, 此时, 需要给用户 添加loading 提示

添加loading提示的好处
1. 节流处理: 防止用户在一次请求还没回来之前, 多次进行点击, 发送无效请求
2. 友好提示: 告知用户, 目前是在加载中, 请耐心等待, 用户体验会更好

实现步骤:
1. 请求拦截器中, 每次请求, 打开 loading
```js
Toast.loading({
	message: '加载中',
	forbidClick: true,
	duration: 0 // 不会自动消失
})
```
2. 响应拦截器中, 每次请求, 关闭 loading
```js
Toast.clear()
```

### 页面访问拦截
目标: 基于全局前置守卫, 进行页面访问拦截处理
说明: 智慧商城项目, 大部分页面, 游客都可以直接访问, 如遇到需要登录才能进行的操作, 提示并跳转登录
但是: 对于支付页, 订单页, 必须是登录的用户才能访问的, 游客不能进入该页面, 需要拦截处理

路由导航首位 - 全局前置首位
1. 所有路由一旦被匹配到, 都会经过全局前置首位
2. 只有全局前置首位放行, 才会真正解析渲染组件, 才能看到页面内容


访问权限页面时, 拦截或放行的关键点 -> 用户是否登录

跳转路由 -> 全局守卫 -> 是否访问全权限页面

实现: 定义一个数组专门用户存放 需要鉴权的url
![[Pasted image 20240911174606.png]]


```js
router.beforeEach((to, from, next) => {
  if (authUrls.includes(to.path)) {
    const token = store.getters.token
    if (token) {
      next()
    } else {
      next('/login')
      return
    }
    next('/')
  }
  next()
})
```

### 搜索 - 历史记录管理
需求:
1. 搜索历史基本渲染
2. 点击搜索(添加历史)
   点击 搜索按钮 或 底下历史记录, 都能进行搜索
   若 之前 没有 相同搜索关键字, 则直接追加到最前面
   若之前 已有 相同, 则源有移除, 再追加
3. 清空历史: 添加清空图标, 可以清空历史记录
4. 持久化: 搜索历史需要持久化, 刷新历史不会丢失
