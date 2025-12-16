<template>
  <div class="quiz-home">
    <div class="quiz-content">
      <div class="header-container">
        <h1 class="title">心理量表测评系统</h1>
        <a href="https://github.com/Catherina0/quiz" target="_blank" rel="noopener" class="github-link" title="如果觉得有用，请给我们一个 Star ⭐">
          <img src="https://github.githubassets.com/assets/GitHub-Mark-ea2971cee799.png" alt="GitHub Repository">
        </a>
      </div>
      <p class="subtitle">选择您要进行的心理测评量表</p>
      <p class="star-hint">💡 如果觉得有用，请到 <a href="https://github.com/Catherina0/quiz" target="_blank" rel="noopener">GitHub</a> 给我们一个 Star ⭐</p>

      <div class="categories">
        <div 
          v-for="category in categories" 
          :key="category.name"
          class="category-section"
        >
          <div class="category-header" @click="toggleCategory(category.name)">
            <span class="category-icon">{{ category.expanded ? '📂' : '📁' }}</span>
            <h2 class="category-name">{{ category.name }}</h2>
            <span class="category-count">({{ category.quizzes.length }})</span>
          </div>
          
          <div v-show="category.expanded" class="quiz-list">
            <div
              v-for="quiz in category.quizzes"
              :key="quiz.id"
              class="quiz-item"
              @click="selectQuiz(quiz.id)"
            >
              <div class="quiz-info">
                <h3 class="quiz-name">{{ quiz.name }}</h3>
                <p class="quiz-desc">{{ quiz.description }}</p>
              </div>
              <div class="quiz-meta">
                <span v-if="quiz.timeLimit" class="quiz-time">⏱ {{ quiz.timeLimit }}分钟</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'QuizHome',
  emits: ['start-quiz'],
  setup(props, { emit }) {
    const categories = ref([
      {
        name: '情绪与焦虑',
        expanded: true,
        quizzes: [
          {
            id: 'sas',
            name: '焦虑自评量表 (SAS)',
            description: '用于检测焦虑症状的严重程度',
            timeLimit: 5
          },
          {
            id: 'PASS',
            name: '领悟社会支持量表 (PASS)',
            description: '评估感知到的社会支持水平',
            timeLimit: 5
          },
          {
            id: 'SCL_90',
            name: '症状自评量表 (SCL-90)',
            description: '评估心理健康和症状程度的综合量表',
            timeLimit: 10
          }
        ]
      },
      {
        name: '人格评估',
        expanded: true,
        quizzes: [
          {
            id: 'MMPI',
            name: 'MMPI-2 明尼苏达多项人格调查表',
            description: '广泛应用于临床心理诊断的人格测试',
            timeLimit: 30
          },
          {
            id: 'PDQ4',
            name: '人格障碍问卷 (PDQ-4)',
            description: '人格障碍筛查工具',
            timeLimit: 15
          }
        ]
      },
      {
        name: '自我效能与解离',
        expanded: true,
        quizzes: [
          {
            id: 'GSES',
            name: '通用自我效能感量表 (GSES)',
            description: '评估个人对应对生活中各种挑战的能力的信心',
            timeLimit: 5
          },
          {
            id: 'DES_II',
            name: '解离体验量表第二版 (DES-II)',
            description: '评估解离症状的严重程度',
            timeLimit: 10
          }
        ]
      },
      {
        name: '躁狂症状',
        expanded: true,
        quizzes: [
          {
            id: 'HCL_32',
            name: '轻躁狂症状检查清单 (HCL-32)',
            description: '躁狂症症状筛查量表',
            timeLimit: 5
          }
        ]
      }
    ])

    const toggleCategory = (categoryName) => {
      const category = categories.value.find(c => c.name === categoryName)
      if (category) {
        category.expanded = !category.expanded
      }
    }

    const selectQuiz = (quizId) => {
      emit('start-quiz', quizId)
    }

    return {
      categories,
      toggleCategory,
      selectQuiz
    }
  }
}
</script>

<style lang="scss" scoped>
.quiz-home {
  background: white;
  border-radius: 8px;
  padding: 30px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.quiz-content {
  max-width: 100%;
}

.header-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
  margin-bottom: 8px;
}

.title {
  text-align: center;
  font-size: 2em;
  color: #1f2937;
  margin: 0;
  font-weight: 600;
}

.github-link {
  display: inline-flex;
  align-items: center;
  
  img {
    width: 32px;
    height: 32px;
    transition: opacity 0.3s, transform 0.3s;
    
    &:hover {
      opacity: 0.8;
      transform: scale(1.1);
    }
  }
}

.subtitle {
  text-align: center;
  color: #6b7280;
  font-size: 1em;
  margin-bottom: 10px;
}

.star-hint {
  text-align: center;
  color: #6b7280;
  font-size: 0.9em;
  margin-bottom: 30px;
  
  a {
    color: #3b82f6;
    text-decoration: none;
    font-weight: 500;
    
    &:hover {
      text-decoration: underline;
    }
  }
}

.categories {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.category-section {
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
}

.category-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 15px 20px;
  background: #f9fafb;
  cursor: pointer;
  transition: background 0.2s;
  user-select: none;

  &:hover {
    background: #f3f4f6;
  }
}

.category-icon {
  font-size: 1.2em;
}

.category-name {
  font-size: 1.1em;
  font-weight: 600;
  color: #374151;
  margin: 0;
  flex: 1;
}

.category-count {
  color: #9ca3af;
  font-size: 0.9em;
}

.quiz-list {
  display: flex;
  flex-direction: column;
}

.quiz-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-top: 1px solid #e5e7eb;
  cursor: pointer;
  transition: all 0.2s;

  &:hover {
    background: #f9fafb;
    padding-left: 25px;
  }
}

.quiz-info {
  flex: 1;
}

.quiz-name {
  font-size: 1em;
  font-weight: 500;
  color: #1f2937;
  margin: 0 0 5px 0;
}

.quiz-desc {
  font-size: 0.85em;
  color: #6b7280;
  margin: 0;
  line-height: 1.4;
}

.quiz-meta {
  display: flex;
  align-items: center;
  margin-left: 15px;
}

.quiz-time {
  font-size: 0.85em;
  color: #9ca3af;
  white-space: nowrap;
}

@media (max-width: 768px) {
  .quiz-home {
    padding: 20px 15px;
  }

  .header-container {
    gap: 10px;
  }

  .title {
    font-size: 1.5em;
  }

  .github-link img {
    width: 28px;
    height: 28px;
  }

  .subtitle,
  .star-hint {
    font-size: 0.85em;
    margin-bottom: 15px;
  }

  .category-header {
    padding: 12px 15px;
  }

  .quiz-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .quiz-meta {
    margin-left: 0;
  }
}
</style>
