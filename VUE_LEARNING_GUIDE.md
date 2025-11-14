# Vue 3 学习指南 - 心理量表测评系统

## 概述

欢迎！这是一个完整的Vue学习项目。通过这个项目，您将学到Vue 3的核心概念和最佳实践。

## 🎯 项目目标

- 理解Vue 3的组合式API
- 学习组件化开发
- 掌握表单处理和数据绑定
- 了解组件通信方式
- 学习异步操作处理

## 📁 项目结构详解

```
quiz/
├── src/
│   ├── main.js                    # 应用入口 - Vue应用初始化
│   ├── App.vue                    # 根组件 - 应用的顶层容器
│   │
│   ├── components/
│   │   ├── QuizHome.vue           # 首页组件 - 展示所有量表
│   │   │   ├── 使用v-for循环显示量表列表
│   │   │   └── 通过emit发送事件给父组件
│   │   │
│   │   ├── QuizContainer.vue      # 主容器组件 - 核心逻辑
│   │   │   ├── 异步加载JSON数据
│   │   │   ├── 管理答题状态
│   │   │   ├── 计算分数
│   │   │   └── 动态显示不同视图
│   │   │
│   │   ├── QuestionnaireForm.vue  # 表单组件 - 管理所有题目
│   │   │   ├── v-for循环渲染题目
│   │   │   └── 收集答案数据
│   │   │
│   │   ├── QuestionItem.vue       # 单题组件 - 单个题目容器
│   │   │   ├── 计算题型（computed）
│   │   │   ├── 动态选择选项组件
│   │   │   └── emit事件到父组件
│   │   │
│   │   ├── ResultDisplay.vue      # 结果组件 - 展示答题结果
│   │   │   ├── 显示总分和标准分
│   │   │   ├── 显示解释和建议
│   │   │   └── 重新开始按钮
│   │   │
│   │   └── options/               # 选项组件 - 不同题型
│   │       ├── RadioOptions.vue    # 单选题
│   │       ├── CheckboxOptions.vue # 多选题
│   │       ├── SliderOption.vue    # 滑块题
│   │       ├── TextInput.vue       # 文本题
│   │       └── SpecialQuestion.vue # 特殊题（有子问题）
│   │
│   └── assets/                    # 资源目录
│
├── data/                          # 量表JSON数据
│   ├── GSES.json                 # 各个量表的配置文件
│   ├── SAS.json
│   ├── SCL_90.json
│   └── ...
│
├── package.json                   # 项目配置和依赖
├── vite.config.js                # Vite打包配置
└── index.html                     # HTML入口
```

## 🔑 核心概念学习

### 1. Vue 3 应用初始化 (`src/main.js`)

```javascript
import { createApp } from 'vue'
import App from './App.vue'

// 创建应用实例
const app = createApp(App)

// 挂载到DOM
app.mount('#app')
```

**学习点：**
- `createApp()` 创建应用实例
- `mount()` 挂载到DOM
- 为什么需要这样做？

### 2. 单文件组件 (SFC) - `.vue` 文件结构

每个 `.vue` 文件包含三个部分：

```vue
<!-- 模板部分 - HTML -->
<template>
  <div class="component">
    <!-- 使用数据和计算属性 -->
    <p>{{ message }}</p>
  </div>
</template>

<!-- 逻辑部分 - JavaScript -->
<script>
import { ref, computed } from 'vue'

export default {
  name: 'MyComponent',
  setup() {
    // 响应式数据
    const count = ref(0)
    
    // 计算属性
    const doubled = computed(() => count.value * 2)
    
    // 返回给模板使用
    return {
      count,
      doubled
    }
  }
}
</script>

<!-- 样式部分 - CSS/SCSS -->
<style lang="scss" scoped>
.component {
  color: blue;
}
</style>
```

**学习点：**
- `<template>` 语法和指令
- `<script setup>` vs `setup()` 函数
- `scoped` 样式隔离原理
- CSS模块化

### 3. 响应式数据 - ref 和 reactive

#### ref - 简单值的响应式

```javascript
import { ref } from 'vue'

setup() {
  // 创建响应式数据
  const count = ref(0)
  
  // 访问需要 .value
  console.log(count.value) // 0
  count.value++           // 1
  
  // 在模板中自动解包，不需要 .value
  return { count }
}
```

在模板中：
```html
<!-- 直接使用，不需要 .value -->
<p>Count: {{ count }}</p>
<button @click="count++">增加</button>
```

**为什么需要 ref？**
- 让基本类型（数字、字符串等）能被Vue追踪
- `ref` 将值包装在对象中，使其可以是响应式的

#### reactive - 对象的响应式

```javascript
import { reactive } from 'vue'

setup() {
  // 用于对象
  const state = reactive({
    count: 0,
    user: { name: '张三', age: 25 }
  })
  
  // 直接访问，不需要 .value
  console.log(state.count)
  state.count++
  
  return { state }
}
```

**何时使用哪种？**
- 简单值 → `ref`
- 对象/数组 → `reactive` 或 `ref`
- 在项目中，混合使用都可以

### 4. 计算属性 - computed

```javascript
import { ref, computed } from 'vue'

setup() {
  const firstName = ref('张')
  const lastName = ref('三')
  
  // 只读计算属性
  const fullName = computed(() => {
    console.log('计算属性被调用') // 只在依赖改变时执行
    return firstName.value + lastName.value
  })
  
  // 可写计算属性
  const fullNameWritable = computed({
    get: () => firstName.value + lastName.value,
    set: (newValue) => {
      [firstName.value, lastName.value] = newValue.split('')
    }
  })
  
  return { firstName, lastName, fullName, fullNameWritable }
}
```

在模板中：
```html
<p>{{ fullName }}</p>
<input v-model="firstName" />
<input v-model="lastName" />
```

**计算属性 vs 方法：**
```javascript
// ❌ 每次访问都会执行
const getFullName = () => firstName.value + lastName.value

// ✅ 只在依赖改变时执行，有缓存
const fullName = computed(() => firstName.value + lastName.value)
```

### 5. 组件通信 - Props 和 Emits

#### 向下通信：Props

**父组件：** `QuestionItem.vue`
```vue
<template>
  <QuestionItem
    :question="myQuestion"
    :index="0"
    @answer="handleAnswer"
  />
</template>

<script>
setup() {
  const myQuestion = { text: '你好吗?', options: [...] }
  return { myQuestion }
}
</script>
```

**子组件：** `QuestionItem.vue`
```javascript
export default {
  props: {
    question: {
      type: Object,
      required: true  // 必传
    },
    index: {
      type: Number,
      default: 0      // 默认值
    }
  },
  setup(props) {
    console.log(props.question)
    console.log(props.index)
  }
}
```

#### 向上通信：Emits

**子组件发送事件：**
```javascript
export default {
  emits: ['answer'],  // 声明会发送的事件
  setup(props, { emit }) {
    const handleClick = () => {
      emit('answer', 42)  // 发送事件和数据
    }
    return { handleClick }
  }
}
```

**父组件接收事件：**
```vue
<template>
  <QuestionItem @answer="handleAnswer" />
</template>

<script>
setup() {
  const handleAnswer = (value) => {
    console.log('收到答案:', value)  // 42
  }
  return { handleAnswer }
}
</script>
```

### 6. 生命周期 - onMounted

```javascript
import { onMounted, ref } from 'vue'

setup() {
  const data = ref(null)
  
  onMounted(() => {
    // 组件挂载后执行 - 适合在这里加载数据
    console.log('组件已挂载到DOM')
    loadData()
  })
  
  const loadData = async () => {
    const response = await fetch('/api/data')
    data.value = await response.json()
  }
  
  return { data }
}
```

**常用生命周期钩子：**
- `onMounted` - 组件挂载到DOM
- `onUpdated` - 组件更新后
- `onBeforeUnmount` - 组件卸载前
- `onUnmounted` - 组件卸载后

### 7. 条件渲染 - v-if 和 v-show

```html
<!-- v-if - 完全移除/添加DOM元素（切换成本高） -->
<QuizHome v-if="!quizStarted" />
<QuizContainer v-else />

<!-- v-show - 使用CSS display隐藏/显示（初始渲染成本高） -->
<div v-show="isVisible">内容</div>
```

**何时用哪个？**
- 切换频繁 → `v-show`
- 初始条件可能为false → `v-if`

### 8. 列表渲染 - v-for

```html
<!-- 基本用法 -->
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>

<!-- 获取索引 -->
<div v-for="(item, index) in items" :key="index">
  {{ index }}: {{ item.name }}
</div>

<!-- 对象循环 -->
<div v-for="(value, key) in obj" :key="key">
  {{ key }}: {{ value }}
</div>
```

**关键点：**
- 始终使用 `:key` 绑定唯一值
- `:key` 帮助Vue跟踪每个元素
- 不要使用索引作为 `:key`（会导致bug）

### 9. 表单绑定 - v-model

```html
<!-- 文本输入 -->
<input v-model="message" />

<!-- 复选框 -->
<input type="checkbox" v-model="isChecked" />

<!-- 单选框 -->
<input type="radio" value="a" v-model="selected" />
<input type="radio" value="b" v-model="selected" />

<!-- 选择框 -->
<select v-model="selected">
  <option value="">--选择--</option>
  <option value="a">A</option>
  <option value="b">B</option>
</select>
```

**v-model 的原理：**
```html
<!-- 这行代码：-->
<input v-model="message" />

<!-- 等同于：-->
<input :value="message" @input="message = $event.target.value" />
```

### 10. 事件处理 - @click 和其他事件

```html
<!-- 基本点击 -->
<button @click="count++">增加</button>

<!-- 传递参数 -->
<button @click="greet('你好')">问候</button>

<!-- 访问事件对象 -->
<input @input="handleInput" />

<!-- 事件修饰符 -->
<form @submit.prevent="handleSubmit">
  <input type="text" />
</form>

<!-- 键盘修饰符 -->
<input @keyup.enter="search" />
```

**常见修饰符：**
- `.prevent` - 阻止默认行为
- `.stop` - 停止事件传播
- `.enter` - 仅回车键
- `.left`/`.right` - 鼠标按钮

## 📚 项目实战学习

### 学习路线

#### 第1步：理解数据流 (30分钟)
1. 打开 `src/App.vue`
   - 看这个根组件如何管理 `quizStarted` 状态
   - 看它如何通过props给子组件传递数据
   - 看它如何通过@事件接收子组件的消息

2. 画一个数据流图：
   ```
   App
   ├─ QuizHome (发送: start-quiz)
   └─ QuizContainer (发送: back)
   ```

#### 第2步：理解组件通信 (30分钟)
1. 打开 `src/components/QuizHome.vue`
   - 看 `availableQuizzes` 是如何定义的
   - 看 `@click="selectQuiz"` 如何触发事件
   - 看 `emit('start-quiz', quizId)` 如何发送数据

2. 打开 `src/components/QuizContainer.vue`
   - 看它如何接收 `quizId` props
   - 看它如何使用 `onMounted` 加载数据
   - 看它如何处理异步操作

#### 第3步：理解表单处理 (30分钟)
1. 打开 `src/components/QuestionnaireForm.vue`
   - 看它如何循环渲染题目
   - 看它如何通过 `@answer` 事件收集答案

2. 打开 `src/components/options/RadioOptions.vue`
   - 看单选题如何处理用户输入
   - 看 `@change` 事件如何更新数据
   - 看 `:value` 和 `:checked` 如何绑定

#### 第4步：理解计算属性 (20分钟)
1. 打开 `src/components/QuestionItem.vue`
   - 看 `questionType` computed 如何工作
   - 看 `effectiveOptions` 如何选择选项

2. 打开 `src/components/ResultDisplay.vue`
   - 看 `hasSubscores` computed 如何判断
   - 看 `conclusion` computed 如何查找解释

#### 第5步：改进和扩展 (自由练习)
尝试实现以下功能：
- [ ] 添加页码显示
- [ ] 添加进度条
- [ ] 添加时间限制倒计时
- [ ] 保存答题进度到localStorage
- [ ] 添加分享结果功能
- [ ] 实现答题历史记录

## 🔍 代码示例分析

### 示例1：答题状态管理

看这段代码如何工作：

```javascript
// QuizContainer.vue
setup(props, { emit }) {
  const answers = ref({})  // 存储所有答案
  
  const handleAnswer = (questionId, value) => {
    answers.value[questionId] = value  // 更新特定题目的答案
  }
  
  const handleSubmit = () => {
    if (!validateAnswers()) return      // 验证所有题目已作答
    
    const result = calculateResult()    // 计算分数
    submitted.value = true              // 切换到结果页面
  }
  
  return {
    answers,
    handleAnswer,
    handleSubmit
  }
}
```

**关键点分析：**
- `answers` 是一个对象，key是题目ID，value是答案
- `handleAnswer` 每次用户改变答案时被调用
- 当点击提交时，验证和计算结果

### 示例2：动态组件选择

看这段代码如何根据题型选择不同的组件：

```javascript
// QuestionItem.vue
const questionType = computed(() => {
  return props.question.type || props.defaultQuestionType || 'radio'
})
```

在模板中：
```vue
<RadioOptions v-if="questionType === 'radio'" ... />
<CheckboxOptions v-else-if="questionType === 'checkbox'" ... />
<SliderOption v-else-if="questionType === 'slider'" ... />
```

**学习点：**
- 使用 `computed` 计算题型
- 使用 `v-if` 条件渲染
- 这叫做"根据状态选择组件"的模式

### 示例3：异步加载数据

看这段代码如何加载JSON数据：

```javascript
// QuizContainer.vue
onMounted(async () => {
  try {
    const response = await fetch(\`/data/\${props.quizId}.json\`)
    if (!response.ok) throw new Error('加载失败')
    quiz.value = await response.json()
  } catch (error) {
    console.error('错误:', error)
    alert('加载量表失败')
  }
})
```

**关键点：**
- `onMounted` 是正确的加载数据的地方
- 使用 `async/await` 处理异步
- 使用 `try/catch` 处理错误
- 组件加载时显示loading状态

## 🎨 样式和SCSS

### scoped 样式隔离

```vue
<style lang="scss" scoped>
/* 这里的样式只会应用到这个组件 */
.question {
  color: blue;
}

/* 嵌套语法 */
.question {
  &:hover {
    color: red;
  }
}
</style>
```

**原理：**
```html
<!-- Vue会自动添加属性 -->
<div class="question" data-v-12345678></div>
```

```css
/* 然后编译成 */
.question[data-v-12345678] {
  color: blue;
}
```

### SCSS变量和混入

```scss
// 定义变量
$primary-color: #667eea;
$border-radius: 8px;

// 使用变量
.button {
  background: $primary-color;
  border-radius: $border-radius;
}

// 混入（重用样式块）
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.container {
  @include flex-center;
}
```

## ⚙️ 项目配置说明

### vite.config.js

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],  // 启用Vue 3支持
  server: {
    port: 5173,      // 开发服务器端口
    open: true       // 自动打开浏览器
  },
  build: {
    outDir: 'dist',  // 输出目录
  }
})
```

### package.json 脚本

```json
{
  "scripts": {
    "dev": "vite",              // 启动开发服务器
    "build": "vite build",      // 构建生产版本
    "preview": "vite preview"   // 预览生产构建
  }
}
```

## 🐛 常见错误和解决方案

### 错误1：页面不显示
```
症状：页面空白
原因：组件没有正确导入
解决：检查 import 语句，确保路径正确
```

### 错误2：数据不更新
```javascript
// ❌ 错误 - 直接修改基本值不会响应
count = 1

// ✅ 正确 - 使用 ref
const count = ref(0)
count.value = 1
```

### 错误3：事件没有触发
```vue
<!-- ❌ 错误 - 父组件忘记监听 -->
<ChildComponent />

<!-- ✅ 正确 -->
<ChildComponent @my-event="handleEvent" />
```

## 📖 推荐阅读

- [Vue 3 官方文档](https://vuejs.org/)
- [Vue 3 API 参考](https://vuejs.org/api/)
- [Vite 文档](https://vitejs.dev/)

## 🚀 下一步学习建议

1. **Vue Router** - 多页面应用
2. **Pinia** - 状态管理库
3. **TypeScript** - 类型系统
4. **Vite 构建优化** - 性能提升

---

祝您学习愉快！有问题欢迎提问。🎉
