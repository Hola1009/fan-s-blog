## 为什么用 Vue3

### Vue3 的优势
1. 更易维护
2. 更快速度
3. 更小体积
4. 更优的数据响应式

### Vue2 选项式 API   vs   Vue3 组合式 API
 
 ![[Pasted image 20240920175544.png|400]]
vue2: 同一个功能要用到的属性和方法散落于页面各个位置

vue3: 代码量变少了, 分散式维护转为集中式维护, 更易封装复用


## create-vue 搭建Vue3项目

### 认识 create-vue
create-vue 是Vue官方新的脚手架工具, 底层切换到了 vite (下一代构建工具), 为了提高响应速度

### 使用 create-vue 创建项目
1. 前提条件
   Node.js 版本16以上
2. 创建
```bash
npm init vue@latest
```

初学接下了的一些选项全选 No 即可

关键文件
1. vite.config.js
2. package.json - 项目包文件 核心依赖项编程了 Vue3.x 和 vite
3. main.js - 入口文件 createApp 函数创建应用实例
4. app.vue - 根组件 SFC单文件组件
   变化一:脚本script和模板template顺序调整 ->**更易维护**
   变化二:模板template不再要求唯一根元素
   变化三:脚本script添加setup标识支持组合式API
 5. index.html-单页入口 提供id为app的挂载点

## 组合式API - setup 选项

### setup 选项的写法和执行时机
setup 是一个声明周期钩子
![[Pasted image 20240920184343.png]]
```js
export default {
    setup () {
      const message = 'hello world'
      const logMessage = () => {
        console.log(message)
      }
      return {
        message,
        logMessage
      }
    }
  }
```

## 组合式 API - reactive 和 ref 函数
### reactive()
作用: 接收对象类型的数据作为参数传入并返回响应式的对象
```js
const state = reactive({
	count:1
})
```

### ref()
作用: 接收简单类型或者对象类型的数据传入并返回一个响应式数据
使用:
```js
import { ref } from 'vue'
// 在原有传入的数据的基础上吗, 外层包了一层对象, 
const count = ref(0)
console(count.value)
```
注意:
在 template 中, .value 不用加 (帮我们扒了一层), 但在 js 中需要使用 .value


## 组合式API - computed
基本思想和 Vue2 完全一致, 组合式 API 下的计算属性 只是修改了写法

核心步骤: 
1. 导入 computed 函数
2. 执行函数: 在回调函数中 return 基于响应式数据做计算的值, 用于变量接收
```js
const value = computed(() => {
  return 计算函数
})
```

将计算属性改成可以修改的做法
```js
const plusOne = computed({
	get: () => count.value + 1,
	set: (val) => { count.value = val - 1 } 
})
```

### 最佳实践
1. 计算属性不应该有副作用
   **比如异步请求 / 修改 dom**   
2. 避免直接修改计算属性的值


## 组合式API - watch
作用: 侦听一个或多个数据变化, 数据变化时执行回调函数
两个额外参数: 1. immediate (一进页面立即执行) 2. deep (深度侦听)
```js
watch(
  [count, name],
  ([newCount, newName], [oldCount, oldName]) => {
    业务逻辑代码
  },
  {
  immediate: true,
  deep: true // 任何一个属性变化都会触发
  }
)
```

deep 深度监视, 默认 watch 进行的是 浅层监视(监视不到复杂类型内部数据的变化)

### 精确侦听对象的某个属性

需求: 在不开始 deep 的前提下, 侦听 age 变化, 只是 age 变化时才执行回调
```js
const info = ref({
  name: 'cp,'
  age: 18
})

watch(
  () => info.value.age,
  (newValue, oldValue) => console.log('age发生变化了')
)
```

## 组合式API - 生命周期函数
![[Pasted image 20240921191504.png]]

这些函数都可以写多个

## 组合式API - 父子通信
基本思想
1. 父组件中给 子组件绑定属性
2. 子组件内部通过 props 选项接收

```js
<script setup>
  const props = defineProps({
    message: String
  })
</script>
```

## 组合式API下的子传父
基本思想
1. 父组件中给子组件标签通过@绑定事件
```vue
<SonComVue @get-message="getMessage"></SonComVue>
```
2. 子组件内部通过 emit 方法触发事件
```vue
<script setup>
  // 先显示声明后再到下面触发
  const emit = defineEmits(['get-message'])
  const sendMsg = () => {
    emit('get-message', 'this is a message from son')
  }
</script>
```


## 组合式API - 模板引用
### 模板引用的概念
通过 **ref** 标识获取真实的 dom 对象或者组件实例对象

### 如何使用 (以获取dom为例 组件同理)
1. 通过调用ref函数生成一个 ref 对象
2. 通过 ref 标识绑定 ref 对象到标签
```vue
<script setup>
import { ref  } from 'vue';

const h1Ref = ref(null)
</script>

<template>

  <h1 ref="h1Ref"></h1>
  
</template>
```

拿到组件dom 主要是为了拿到其中属性和方法
### defineExpose()
默认情况下在 \<script setup\> 语法糖下组件内部的属性和方法是不开放给父组件访问呢的
可以通过 defineExpose 编译宏指定哪些属性和方法允许访问

```vue
<script setup>
  const message = "我是子组件"
  defineExpose({
    message
  })
</script>
```

## 组合式API - provide 和 inject
### 作用和场景
顶层组件向任意的底层组件传递数据和方法, 实现跨层级组件通信

### 跨层传递普通数据
1. 顶层组件通过 **provide 函数提供**数据
2. 底层组件通过 **inject 函数获取**数据

```js
// 顶层组件
provide('key', 顶层组件数据)

// 底层组件
const message = inject('key')
```

如果要修改顶层数据, 可以让接收顶层传过来的修改数据的方法

## Vue3.3 新特性 - defineOptions
背景说明:
有\<script setup\>之前，如果要定义 props，emits 可以轻而易举地添加一个与 setup 平级的属性但是用了\<script setup\>后，就没法这么干了 setup 属性已经没有了，自然无法添加与其平级的属性
```vue
<script setup>
defineOptions({
  name: 'sonComponentVue'
})
</script>
```

## Vue 3.3 新特性 - defineModel
在 Vue3 中, 自定义组件上使用 v-model, 相当于传递了一个 modleValue 属性, 同时触发 update: modelValue 事件
```js
<Child v-model="isVisible">
// 等同于
<Child :modelValue="isVisible" @update:modelValue=""isVisible=$event>
```

```js
<script setup>

defineOptions({
  modelValue: String
})

const emit = defineEmits(['update:modelValue'])
</script>
  
<template>
  <div>
    <input type="text"
      :value="modelValue"
      @input="e => emit('update:modelValue', e.target.value)"
    >
  </div>
</template>
```

这些操作可以用 defineModel 来代替
这是一个试验性质的特性, 需要在配置文件中做出配置
```js
export default defineConfig({
  plugins: [
  // 配置处
    vue({
      script: {
        defineModel: true
      }
    }),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```
使用 defineModel 后
```vue
<script setup>
import { defineModel } from 'vue'
const modelValue = defineModel()
</script>
  
<template>
  <div>
    <input type="text"
      :value="modelValue"
      @input="e => modelValue = e.target.value"
    >
  </div>
</template>
```

对比
```js
defineOptions({
  modelValue: String
})

const emit = defineEmits(['update:modelValue'])

// 变成了
<script setup>
import { defineModel } from 'vue'
const modelValue = defineModel()
</script>

// =========================================

@input="e => emit('update:modelValue', e.target.value)"

// 变成了

@input="e => modelValue = e.target.value"
```

## 什么是 Pinia
Pinia 是 Vue 的最新 **状态管理工具**, 是 Vuex 的**代理品**
1. 提供更加简单的API (去掉了 mutation)
2. 提供符合组合式风格的 API (和 Vue2 新语法统一)
3. 去掉了 modules 的概念, 每一个 store 都是一个独立的模块
4. 配合 TypeScript 更加友好, 提供可靠的类型推断

#### 创建应给 pinia 实例
```js
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia'
const pinna = createPinia()

const app = createApp(App)
app.use(pinna).mount('#app')
```

#### 定义 store
```js
import { defineStore } from "pinia";
import { ref } from "vue";

// 定义 store
// defineStore (仓库的唯一标识, () => { ... })
  
export const useCounterStore = defineStore('counter', () => {
  // 声明数据 state
  const count = ref(0)
  // 需要把声明的数据全部导出, 不然会报错
  return {
    count
  }
})
```

#### 计算属性的实现
跟再 setup 中的写法是一样的
```js
const double = computed(() => {
	// 逻辑代码
})
```

### action 的异步实现
编写方式: 异步 action 函数的写法和**组件中获取异步数据的写法完全一致**


