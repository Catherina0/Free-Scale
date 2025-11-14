# 心理量表测评系统 - Vue 3

这是一个使用 **Vue 3 + Vite** 构建的心理量表测评系统。这是学习Vue框架的实践项目。

## 🚀 快速开始

```bash
# 安装依赖
npm install

# 开发模式
npm run dev
# 访问 http://localhost:5173

# 生产构建
npm run build
```

## 📚 学习资源

- **[VUE_LEARNING_GUIDE.md](./VUE_LEARNING_GUIDE.md)** - 完整的Vue 3学习指南
- **[QUICK_REFERENCE.md](./QUICK_REFERENCE.md)** - Vue语法速查表

## 📁 项目结构

```
src/
├── main.js                      # 应用入口
├── App.vue                      # 根组件
├── components/
│   ├── QuizHome.vue            # 首页
│   ├── QuizContainer.vue       # 核心容器
│   ├── QuestionnaireForm.vue   # 表单
│   ├── QuestionItem.vue        # 题目
│   ├── ResultDisplay.vue       # 结果
│   └── options/                # 题型组件
└── ...

data/                            # 量表JSON配置
```

## 🎓 核心学习点

- ✅ 组件化架构
- ✅ 响应式数据 (ref, computed)
- ✅ 组件通信 (props, emits)
- ✅ 异步操作 (async/await, fetch)
- ✅ 表单处理 (v-model, 事件绑定)
- ✅ 样式处理 (SCSS, scoped)

## 💻 功能特性

- 支持多种题型（单选、多选、滑块、文本、特殊题）
- JSON配置驱动的量表系统
- 灵活的计分逻辑
- 响应式设计
- 实时答题验证

## 🛠️ 技术栈

- Vue 3 (Composition API)
- Vite
- SCSS
- ES6+

---

开始学习：打开 `VUE_LEARNING_GUIDE.md`
