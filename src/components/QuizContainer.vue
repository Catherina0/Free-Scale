<template>
  <div class="quiz-container" v-if="quiz">
    <!-- 介绍区域 -->
    <div class="intro" v-if="!submitted">
      <h1>{{ quiz.title }}</h1>
      <div class="description">{{ quiz.description }}</div>
      <div v-if="quiz.instruction" class="instruction">{{ quiz.instruction }}</div>
      <div v-if="quiz.timeLimit" class="time-limit">
        建议完成时间: {{ quiz.timeLimit }} 分钟
      </div>
    </div>

    <!-- 题目区域 -->
    <QuestionnaireForm
      v-if="!submitted"
      :quiz="quiz"
      :answers="answers"
      @answer="handleAnswer"
      @submit="handleSubmit"
    />

    <!-- 结果区域 -->
    <ResultDisplay
      v-else
      :result="result"
      :quiz="quiz"
      @reset="handleReset"
    />
  </div>

  <!-- 加载状态 -->
  <div v-else class="loading">
    <div class="spinner"></div>
    <p>正在加载量表...</p>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import QuestionnaireForm from './QuestionnaireForm.vue'
import ResultDisplay from './ResultDisplay.vue'

export default {
  name: 'QuizContainer',
  components: {
    QuestionnaireForm,
    ResultDisplay
  },
  props: {
    quizId: {
      type: String,
      required: true
    }
  },
  emits: ['back'],
  setup(props, { emit }) {
    const quiz = ref(null)
    const answers = ref({})
    const result = ref(null)
    const submitted = ref(false)
    const startTime = ref(null)

    // 加载问卷数据
    const loadQuiz = async () => {
      try {
        const response = await fetch(`/data/${props.quizId}.json`)
        if (!response.ok) {
          throw new Error(`无法加载量表: ${props.quizId}`)
        }
        quiz.value = await response.json()
        startTime.value = Date.now()
        answers.value = {}
        submitted.value = false
      } catch (error) {
        console.error('加载量表失败:', error)
        alert(`加载量表失败: ${error.message}`)
        emit('back')
      }
    }

    // 处理答案变化
    const handleAnswer = (questionId, value) => {
      answers.value[questionId] = value
    }

    // 处理提交
    const handleSubmit = () => {
      // 验证答题完整性
      if (!validateAnswers()) {
        return
      }

      // 计算结果
      const endTime = Date.now()
      const timeTaken = Math.round((endTime - startTime.value) / 1000 / 60) // 分钟

      result.value = calculateResult(timeTaken)
      submitted.value = true
    }

    // 验证答题完整性
    const validateAnswers = () => {
      const requiredQuestions = quiz.value.questions
        .filter(q => q.required !== false)
        .map((q, index) => q.id || (index + 1))

      const missingAnswers = requiredQuestions.filter(qId => {
        const answer = answers.value[qId]
        const question = quiz.value.questions.find(q => (q.id || 0) === qId)

        if (question && question.type === 'special') {
          if (!Array.isArray(answer) || answer.length === 0) {
            return true
          }
          return answer.some(a => a === undefined || a === null)
        }

        return (
          answer === undefined ||
          answer === null ||
          (Array.isArray(answer) && answer.length === 0) ||
          (typeof answer === 'string' && answer.trim() === '')
        )
      })

      if (missingAnswers.length > 0) {
        alert(`请完成所有题目！还有 ${missingAnswers.length} 题未作答。`)
        return false
      }

      return true
    }

    // 计算结果
    const calculateResult = (timeTaken) => {
      let resultData = {
        rawScore: 0,
        scaledScore: 0,
        subscores: {},
        timeTaken: timeTaken,
        answers: answers.value
      }

      // 根据计分规则计算分数
      if (quiz.value.scoring) {
        resultData = applyScoring(resultData)
      }

      return resultData
    }

    // 应用计分规则
    const applyScoring = (resultData) => {
      const scoring = quiz.value.scoring

      // 简单求和
      if (scoring.method === 'sum' || scoring.method === 'custom') {
        resultData.rawScore = Object.entries(answers.value).reduce(
          (sum, [qId, value]) => {
            const qIdNum = parseInt(qId)
            const question = quiz.value.questions.find((q, index) => {
              const questionId = q.id !== undefined ? q.id : (index + 1)
              return questionId === qIdNum
            })

            let score = Array.isArray(value) ? value.reduce((a, b) => a + b, 0) : (parseFloat(value) || 0)

            // 处理反向计分
            if (question && question.reverse) {
              const options = question.options || quiz.value.defaultOptions || []
              if (options.length > 0) {
                const maxScore = Math.max(...options.map(o => o.score || 0))
                const minScore = Math.min(...options.map(o => o.score || 0))
                score = maxScore + minScore - score
              }
            }

            return sum + score
          },
          0
        )
      }

      // 标准分数转换
      if (scoring.standardScores && scoring.standardScores[resultData.rawScore]) {
        resultData.scaledScore = scoring.standardScores[resultData.rawScore]
      } else {
        resultData.scaledScore = resultData.rawScore
      }

      return resultData
    }

    // 处理重新开始
    const handleReset = () => {
      loadQuiz()
    }

    onMounted(() => {
      loadQuiz()
    })

    return {
      quiz,
      answers,
      result,
      submitted,
      handleAnswer,
      handleSubmit,
      handleReset
    }
  }
}
</script>

<style lang="scss" scoped>
.quiz-container {
  background: white;
  border-radius: 15px;
  padding: 40px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.intro {
  text-align: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #e5e7eb;

  h1 {
    color: #1f2937;
    font-size: 2em;
    margin-bottom: 15px;
    font-weight: 600;
  }

  .description {
    color: #6b7280;
    font-size: 1em;
    margin-bottom: 12px;
    line-height: 1.6;
  }

  .instruction {
    background-color: #f0f9ff;
    padding: 15px;
    border-radius: 8px;
    border-left: 3px solid #3b82f6;
    margin: 15px 0;
    text-align: left;
    color: #374151;
    font-size: 0.95em;
  }

  .time-limit {
    color: #888;
    font-size: 0.95em;
    margin-top: 10px;
  }
}

.loading {
  text-align: center;
  padding: 60px 20px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);

  .spinner {
    width: 50px;
    height: 50px;
    margin: 0 auto 20px;
    border: 4px solid #f3f4f6;
    border-top: 4px solid #3b82f6;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }

  p {
    color: #666;
    font-size: 1.1em;
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@media (max-width: 768px) {
  .quiz-container {
    padding: 20px;
  }

  .intro h1 {
    font-size: 1.8em;
  }

  .intro .instruction {
    padding: 15px;
    margin: 15px 0;
  }
}
</style>
