<template>
  <div class="app-container">
    <!-- 返回首页按钮 -->
    <button class="home-link" v-if="showHomeLink" @click="handleBack">
      返回首页
    </button>

    <div class="container">
      <QuizHome v-if="!quizStarted" @start-quiz="handleStartQuiz" />
      <QuizContainer v-else :quiz-id="currentQuizId" @back="handleBack" />
    </div>

    <footer class="footer">
      <p>免责声明：本测评仅供参考，不能替代专业的心理诊断。如有需要，请咨询专业心理健康工作者。</p>
    </footer>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import QuizHome from './components/QuizHome.vue'
import QuizContainer from './components/QuizContainer.vue'

export default {
  name: 'App',
  components: {
    QuizHome,
    QuizContainer
  },
  setup() {
    const quizStarted = ref(false)
    const currentQuizId = ref(null)

    const showHomeLink = computed(() => {
      return quizStarted.value
    })

    const handleStartQuiz = (quizId, updateUrl = true) => {
      currentQuizId.value = quizId
      quizStarted.value = true
      // 仅在需要时更新URL
      if (updateUrl) {
        window.history.pushState({}, '', `?id=${quizId}`)
      }
    }

    const handleBack = () => {
      quizStarted.value = false
      currentQuizId.value = null
      // 回到首页URL
      window.history.pushState({}, '', '/')
    }

    // 检查URL参数，支持旧链接格式
    onMounted(() => {
      const params = new URLSearchParams(window.location.search)
      const quizIdFromUrl = params.get('id')
      
      if (quizIdFromUrl) {
        // 从URL加载，不需要再次更新URL
        handleStartQuiz(quizIdFromUrl, false)
      }
    })

    return {
      quizStarted,
      currentQuizId,
      showHomeLink,
      handleStartQuiz,
      handleBack
    }
  }
}
</script>

<style lang="scss">
/* 全局样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Microsoft YaHei', 'PingFang SC', 'Helvetica Neue', Arial, sans-serif;
  background: #f5f7fa;
  min-height: 100vh;
  line-height: 1.6;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.container {
  max-width: 900px;
  margin: 0 auto;
  flex: 1;
  width: 100%;
}

/* 返回首页按钮 */
.home-link {
  position: fixed;
  top: 20px;
  left: 20px;
  background-color: #3b82f6;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 500;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  z-index: 1000;
  border: none;
  cursor: pointer;

  &:hover {
    background-color: #2563eb;
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  }
}

/* 页脚 */
.footer {
  text-align: center;
  margin-top: 40px;
  padding: 20px;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9em;
}

@media (max-width: 768px) {
  .container {
    padding: 20px;
  }

  .home-link {
    top: 10px;
    left: 10px;
    padding: 10px 16px;
    font-size: 0.9em;
  }
}
</style>
