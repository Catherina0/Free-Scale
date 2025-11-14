# 🚀 Vue 快速参考卡片

> 打印这个文档或收藏到浏览器，随时查阅

## 快速启动

```bash
npm install      # 安装依赖（仅需一次）
npm run dev      # 启动开发服务器
npm run build    # 构建生产版本
```

## Vue 3 核心语法速查

### 1. 响应式数据

```javascript
import { ref, reactive, computed } from 'vue'

// 简单值
const count = ref(0)
count.value++  // 需要 .value

// 对象/数组
const state = reactive({ name: 'Vue' })
state.name = 'React'  // 不需要 .value

// 计算属性
const doubled = computed(() => count.value * 2)
```

### 2. 模板语法

```html
<!-- 插值 -->
<p>{{ message }}</p>
<p>{{ count + 1 }}</p>
<p>{{ ok ? 'YES' : 'NO' }}</p>

<!-- 绑定 -->
<a :href="url">链接</a>
<img :src="imageSrc" />
<div :class="{ active: isActive }"></div>
<div :style="{ color: activeColor }"></div>

<!-- 事件 -->
<button @click="count++">按钮</button>
<input @input="message = $event.target.value" />
<form @submit.prevent="submit">表单</form>

<!-- 表单绑定 -->
<input v-model="message" />
<input type="checkbox" v-model="checked" />
<select v-model="selected">
  <option value="a">A</option>
</select>

<!-- 条件渲染 -->
<p v-if="visible">可见</p>
<p v-else>隐藏</p>
<p v-show="visible">仅CSS隐藏</p>

<!-- 列表渲染 -->
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>
```

### 3. 组件结构

```vue
<template>
  <div class="component">
    <h1>{{ title }}</h1>
    <button @click="handleClick">{{ count }}</button>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'MyComponent',
  props: {
    title: String,
    count: { type: Number, default: 0 }
  },
  emits: ['update'],
  setup(props, { emit }) {
    const localCount = ref(props.count)
    
    const handleClick = () => {
      localCount.value++
      emit('update', localCount.value)
    }
    
    return { localCount, handleClick }
  }
}
</script>

<style scoped>
.component {
  padding: 20px;
}
</style>
```

### 4. 生命周期

```javascript
import { 
  onBeforeMount, 
  onMounted, 
  onUpdated, 
  onUnmounted 
} from 'vue'

export default {
  setup() {
    onMounted(() => console.log('已挂载'))
    onUpdated(() => console.log('已更新'))
    onUnmounted(() => console.log('已卸载'))
  }
}
```

### 5. 异步操作

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data')
    if (!response.ok) throw new Error('请求失败')
    const data = await response.json()
    return data
  } catch (error) {
    console.error('错误:', error)
  }
}

// 在 onMounted 中调用
onMounted(async () => {
  const data = await fetchData()
  items.value = data
})
```

## 项目文件速查

```
src/
├── main.js               # 入口：创建应用
├── App.vue              # 根组件：全局状态
├── components/
│   ├── QuizHome.vue     # 首页：列表
│   ├── QuizContainer.vue # 核心：数据管理
│   ├── QuestionnaireForm.vue # 表单
│   ├── QuestionItem.vue # 题目
│   ├── ResultDisplay.vue # 结果
│   └── options/         # 选项组件
data/                     # JSON 配置
```

## 常见任务速查

### 任务1：添加新数据

```javascript
const users = ref([])

// 添加
users.value.push({ id: 1, name: '张三' })

// 删除
users.value = users.value.filter(u => u.id !== 1)

// 修改
const user = users.value.find(u => u.id === 1)
if (user) user.name = '李四'

// 排序
users.value.sort((a, b) => a.id - b.id)
```

### 任务2：父子通信

```javascript
// 父组件 - 传递 props 和监听事件
<ChildComponent 
  :message="msg"
  @child-event="handleChildEvent"
/>

// 子组件 - 接收 props 和发送事件
export default {
  props: ['message'],
  emits: ['child-event'],
  setup(props, { emit }) {
    emit('child-event', data)
  }
}
```

### 任务3：条件显示

```html
<!-- 完全移除/添加 DOM（初始化成本）-->
<div v-if="show">内容</div>

<!-- CSS 显示/隐藏（初始化一次）-->
<div v-show="show">内容</div>

<!-- 多条件 -->
<div v-if="type === 'a'">A</div>
<div v-else-if="type === 'b'">B</div>
<div v-else>C</div>
```

### 任务4：列表循环

```html
<!-- 基础循环 -->
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

### 任务5：表单处理

```html
<!-- 文本 -->
<input v-model="text" />

<!-- 复选框 -->
<input type="checkbox" v-model="checked" />

<!-- 单选 -->
<input type="radio" value="a" v-model="selected" />

<!-- 选择 -->
<select v-model="selected">
  <option value="">请选择</option>
  <option value="a">选项A</option>
</select>

<!-- 多选 -->
<input type="checkbox" value="a" v-model="checkedList" />
```

### 任务6：样式绑定

```html
<!-- 单个类 -->
<div :class="{ active: isActive }"></div>

<!-- 多个类 -->
<div :class="{ active: isActive, 'text-danger': hasError }"></div>

<!-- 类数组 -->
<div :class="[baseClass, { active: isActive }]"></div>

<!-- 行内样式 -->
<div :style="{ color: activeColor, fontSize: size + 'px' }"></div>

<!-- 样式对象 -->
<div :style="styleObject"></div>
```

## 性能优化技巧

### ✅ 推荐做法

```javascript
// 使用 computed 缓存计算结果
const filtered = computed(() => items.value.filter(...))

// 使用 v-key 优化列表
<div v-for="item in items" :key="item.id"></div>

// 使用 v-show 频繁切换
<div v-show="isVisible">内容</div>

// 使用 v-if 不频繁切换
<div v-if="isVisible">内容</div>
```

### ❌ 避免做法

```javascript
// 避免：在模板中计算
{{ items.filter(...) }}  // ❌

// 避免：使用索引作为 :key
<div v-for="(item, index) in items" :key="index"></div> // ❌

// 避免：在模板中定义方法
@click="() => console.log('hello')"  // ❌

// 应该是：
@click="handleClick"  // ✅
```

## 调试技巧

### 浏览器控制台

```javascript
// 检查响应式数据
vm.$data  // 查看所有响应式数据

// 检查组件
vm.$options  // 查看组件定义

// 手动调用方法
vm.handleClick()  // 测试方法

// 监听数据变化
vm.$watch('count', (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})
```

### console.log 技巧

```javascript
// 在 setup 中调试
console.log('props:', props)
console.log('answers:', answers.value)

// 在模板中调试
<p>{{ console.log('render') || message }}</p>

// 条件调试
if (debug) console.log('状态:', state.value)
```

## 常见错误排查

| 症状 | 原因 | 解决方案 |
|-----|-----|--------|
| 页面空白 | 组件未加载 | 检查 import 路径 |
| 数据不更新 | 未使用 ref | 用 ref 包装 |
| 事件不触发 | 事件名错误 | 检查 emits 声明 |
| 样式不生效 | 未用 scoped | 添加 scoped |
| 列表不更新 | 直接修改数组 | 使用 .value |

## 快速命令参考

```bash
# 项目管理
npm install         # 安装依赖
npm run dev        # 开发模式
npm run build      # 生产构建

# Git 命令
git add .          # 添加文件
git commit -m "信息"  # 提交
git push           # 推送

# 文件操作
ls -la             # 列出文件
cd folder          # 进入文件夹
touch file.txt     # 新建文件
rm file.txt        # 删除文件
```

## 文档位置速查

| 文档 | 位置 | 内容 |
|-----|-----|-----|
| 主README | /README.md | 项目简介 |
| Vue指南 | /VUE_LEARNING_GUIDE.md | 详细教程 |
| 项目完成 | /PROJECT_COMPLETE.md | 学习路线 |
| 快速参考 | /QUICK_REFERENCE.md | 本文档 |

---

💡 **提示**: 收藏本页面，随时查阅！

🎓 **下一步**: 打开 `VUE_LEARNING_GUIDE.md` 开始学习
