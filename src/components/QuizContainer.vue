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
        // 使用 Vite 动态导入 JSON 文件
        let quizData
        switch(props.quizId) {
          case 'sas':
            quizData = (await import('../../data/sas.json')).default
            break
          case 'PASS':
            quizData = (await import('../../data/PASS.json')).default
            break
          case 'SCL_90':
            quizData = (await import('../../data/SCL_90.json')).default
            break
          case 'MMPI':
            quizData = (await import('../../data/MMPI.json')).default
            break
          case 'PDQ4':
            quizData = (await import('../../data/PDQ4.json')).default
            break
          case 'DES_II':
            quizData = (await import('../../data/DES_II.json')).default
            break
          case 'GSES':
            quizData = (await import('../../data/GSES.json')).default
            break
          case 'HCL_32':
            quizData = (await import('../../data/HCL_32.json')).default
            break
          default:
            throw new Error(`未知的量表ID: ${props.quizId}`)
        }
        quiz.value = quizData
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

      // 检查是否是MMPI量表
      if (quiz.value.title && quiz.value.title.includes('MMPI')) {
        return applyMMPIScoring(resultData)
      }

      // 检查是否是PDQ4量表
      if (quiz.value.title && quiz.value.title.includes('PDQ')) {
        return applyPDQ4Scoring(resultData)
      }

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

      // 计算分项得分（如果有subscales配置）
      if (quiz.value.subscales && Array.isArray(quiz.value.subscales)) {
        quiz.value.subscales.forEach(subscale => {
          let subscaleScore = 0
          subscale.questions.forEach(qId => {
            const answer = answers.value[qId]
            const question = quiz.value.questions.find((q, index) => {
              const questionId = q.id !== undefined ? q.id : (index + 1)
              return questionId === qId
            })

            let score = Array.isArray(answer) ? answer.reduce((a, b) => a + b, 0) : (parseFloat(answer) || 0)

            // 处理反向计分
            if (question && question.reverse) {
              const options = question.options || quiz.value.defaultOptions || []
              if (options.length > 0) {
                const maxScore = Math.max(...options.map(o => o.score || 0))
                const minScore = Math.min(...options.map(o => o.score || 0))
                score = maxScore + minScore - score
              }
            }

            subscaleScore += score
          })
          
          resultData.subscores[subscale.name] = subscaleScore
        })
      }

      // 标准分数转换
      if (scoring.standardScores && scoring.standardScores[resultData.rawScore]) {
        resultData.scaledScore = scoring.standardScores[resultData.rawScore]
      } else {
        resultData.scaledScore = resultData.rawScore
      }

      return resultData
    }

    // PDQ4专用计分函数
    const applyPDQ4Scoring = (resultData) => {
      const scoringRules = quiz.value.scoring.scoringRules
      const userAnswers = answers.value
      
      // 计算每个人格障碍类型的得分
      Object.keys(scoringRules).forEach(typeKey => {
        if (typeKey === 'concealmentScale') return // 跳过掩饰量表，暂不处理
        
        const type = scoringRules[typeKey]
        let typeScore = 0
        
        type.items.forEach(itemId => {
          const answer = userAnswers[itemId]
          if (answer === 1) { // 回答"是"计1分
            typeScore++
          }
        })
        
        // 保存得分到subscores
        resultData.subscores[type.name] = typeScore
      })
      
      // 计算总分（所有类型得分之和）
      resultData.rawScore = Object.values(resultData.subscores).reduce((sum, score) => sum + score, 0)
      resultData.scaledScore = resultData.rawScore
      
      return resultData
    }

    // MMPI专用计分函数
    const applyMMPIScoring = (resultData) => {
      const rules = quiz.value.scoringRules
      const userAnswers = answers.value
      
      // 假设性别，实际应用中应该由用户选择
      const gender = 'male' // 可以添加性别选择功能
      const norms = rules.norms[gender]
      
      resultData.mmpiScores = {
        validity: {},
        clinical: {},
        additional: {},
        gender: gender
      }

      // 计算效度量表
      if (rules.validity) {
        // Q量表 - 疑问分数
        if (rules.validity.Q) {
          let qScore = 0
          rules.validity.Q.pairs.forEach(pair => {
            const ans1 = userAnswers[pair[0]]
            const ans2 = userAnswers[pair[1]]
            if (ans1 !== undefined && ans2 !== undefined && ans1 === ans2) {
              qScore++
            }
          })
          resultData.mmpiScores.validity.Q = {
            name: '疑问分数',
            rawScore: qScore
          }
        }

        // L量表 - 说谎分数
        if (rules.validity.L) {
          let lScore = 0
          rules.validity.L.falseAnswers.forEach(qid => {
            if (userAnswers[qid] === 0) lScore++
          })
          const lNorm = norms.L
          const lTScore = calculateTScore(lScore, lNorm.mean, lNorm.sd)
          resultData.mmpiScores.validity.L = {
            name: '说谎分数',
            rawScore: lScore,
            tScore: lTScore
          }
        }

        // F量表 - 诈病分数
        if (rules.validity.F) {
          let fScore = 0
          rules.validity.F.trueAnswers.forEach(qid => {
            if (userAnswers[qid] === 1) fScore++
          })
          rules.validity.F.falseAnswers.forEach(qid => {
            if (userAnswers[qid] === 0) fScore++
          })
          const fNorm = norms.F
          const fTScore = calculateTScore(fScore, fNorm.mean, fNorm.sd)
          resultData.mmpiScores.validity.F = {
            name: '诈病分数',
            rawScore: fScore,
            tScore: fTScore
          }
        }

        // K量表 - 校正分数
        if (rules.validity.K) {
          let kScore = 0
          if (rules.validity.K.trueAnswers) {
            rules.validity.K.trueAnswers.forEach(qid => {
              if (userAnswers[qid] === 1) kScore++
            })
          }
          rules.validity.K.falseAnswers.forEach(qid => {
            if (userAnswers[qid] === 0) kScore++
          })
          const kNorm = norms.K
          const kTScore = calculateTScore(kScore, kNorm.mean, kNorm.sd)
          resultData.mmpiScores.validity.K = {
            name: '校正分数',
            rawScore: kScore,
            tScore: kTScore
          }
          // 保存K分数供后续校正使用
          resultData.mmpiScores.kScore = kScore
        }
      }

      // 计算临床量表
      if (rules.clinical) {
        Object.keys(rules.clinical).forEach(scaleKey => {
          const scale = rules.clinical[scaleKey]
          let rawScore = 0
          
          if (scale.trueAnswers) {
            scale.trueAnswers.forEach(qid => {
              if (userAnswers[qid] === 1) rawScore++
            })
          }
          if (scale.falseAnswers) {
            scale.falseAnswers.forEach(qid => {
              if (userAnswers[qid] === 0) rawScore++
            })
          }

          const norm = norms[scaleKey]
          const tScore = calculateTScore(rawScore, norm.mean, norm.sd)
          
          resultData.mmpiScores.clinical[scaleKey] = {
            name: scale.name,
            rawScore: rawScore,
            tScore: tScore
          }

          // 如果有K校正
          if (scale.kCorrection && resultData.mmpiScores.kScore !== undefined) {
            const kCorrectedRaw = rawScore + Math.round(resultData.mmpiScores.kScore * scale.kCorrection)
            const kCorrectedKey = `${scaleKey}+${scale.kCorrection}K`
            const kCorrectedNorm = norms[kCorrectedKey]
            if (kCorrectedNorm) {
              const kCorrectedTScore = calculateTScore(kCorrectedRaw, kCorrectedNorm.mean, kCorrectedNorm.sd)
              resultData.mmpiScores.clinical[scaleKey].kCorrected = {
                rawScore: kCorrectedRaw,
                tScore: kCorrectedTScore,
                coefficient: scale.kCorrection
              }
            }
          }
        })
      }

      // 计算附加量表
      if (rules.additional) {
        Object.keys(rules.additional).forEach(scaleKey => {
          const scale = rules.additional[scaleKey]
          let rawScore = 0
          
          if (scale.trueAnswers) {
            scale.trueAnswers.forEach(qid => {
              if (userAnswers[qid] === 1) rawScore++
            })
          }
          if (scale.falseAnswers) {
            scale.falseAnswers.forEach(qid => {
              if (userAnswers[qid] === 0) rawScore++
            })
          }

          const norm = norms[scaleKey]
          if (norm) {
            const tScore = calculateTScore(rawScore, norm.mean, norm.sd)
            resultData.mmpiScores.additional[scaleKey] = {
              name: scale.name,
              rawScore: rawScore,
              tScore: tScore
            }
          }
        })
      }

      // 设置总分为临床量表的平均T分
      const clinicalTScores = Object.values(resultData.mmpiScores.clinical)
        .map(s => s.tScore)
        .filter(t => t !== undefined)
      resultData.rawScore = Math.round(
        clinicalTScores.reduce((sum, t) => sum + t, 0) / clinicalTScores.length
      )
      resultData.scaledScore = resultData.rawScore

      return resultData
    }

    // 计算T分数
    const calculateTScore = (rawScore, mean, sd) => {
      return Math.round(50 + 10 * (rawScore - mean) / sd)
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
