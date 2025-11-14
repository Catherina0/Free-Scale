# 使用指南

## 启动项目

```bash
npm run dev
```

然后打开浏览器访问 `http://localhost:5173`

## 访问方式

### 方式1：首页选择（推荐）

直接访问 `http://localhost:5173`，在首页选择要进行的量表测评

### 方式2：直接访问特定量表（支持旧链接格式）

使用URL查询参数 `?id=<量表ID>` 直接打开指定量表：

```
http://localhost:5173?id=DES_II
http://localhost:5173?id=GSES
http://localhost:5173?id=SAS
http://localhost:5173?id=SCL_90
http://localhost:5173?id=MMPI
http://localhost:5173?id=HCL_32
http://localhost:5173?id=PDQ4
http://localhost:5173?id=PASS
```

**注意**：旧的格式 `http://localhost:5173/quiz.html?id=DES_II` 已经自动重定向到新格式。

## 可用的量表列表

| 量表ID | 量表名称 | 预计时间 |
|--------|--------|--------|
| GSES | 通用自我效能感量表 | 5分钟 |
| SAS | 焦虑自评量表 | 5分钟 |
| SCL_90 | 症状自评量表 | 10分钟 |
| MMPI | MMPI-2 明尼苏达多项人格调查表 | 30分钟 |
| HCL_32 | Hypomanic Checklist (HCL-32) | 5分钟 |
| PDQ4 | 人格障碍问卷 (PDQ-4) | 15分钟 |
| PASS | PASS | 5分钟 |
| DES_II | 解离体验量表第二版 | 10分钟 |

## 功能说明

### 答题
1. 阅读题目和题目说明
2. 选择相应的答案
3. 所有必填题都完成后，点击"提交评估"

### 查看结果
1. 提交答题后，自动显示测评结果
2. 结果包括：原始分数、标准分数、分项得分（如有）、解释
3. 点击"重新开始"可以重新答题
4. 点击"返回首页"回到首页

### 重新开始
- 在任何时刻点击左上角"返回首页"按钮可以回到首页重新选择量表
- 已有的答案会被清除

## 浏览器支持

- 支持所有现代浏览器（Chrome、Firefox、Safari、Edge等）
- 建议使用最新版本的浏览器

## 常见问题

**Q: 提交时显示"还有X题未作答"**
A: 请检查是否有题目没有选择答案，带有红色*号的题目是必填的。

**Q: 为什么有些题目没有答案选项显示？**
A: 这可能是因为页面加载不完整，请刷新页面。

**Q: 如何分享测评链接？**
A: 可以复制URL分享给他人，对方访问时会自动进入相应的量表。例如：
```
http://localhost:5173?id=GSES
```

## 开发相关

更多技术细节请参考：
- [README.md](./README.md) - 项目概况
- [VUE_LEARNING_GUIDE.md](./VUE_LEARNING_GUIDE.md) - Vue学习指南
