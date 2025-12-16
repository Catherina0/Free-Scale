<template>
  <div class="result-display">
    <div class="result-header">
      <h2>测评结果</h2>
      <p class="completion-time">完成时间: {{ result.timeTaken }} 分钟</p>
    </div>

    <!-- MMPI详细得分显示 -->
    <div v-if="isMMPI" class="mmpi-scores-section">
      <div class="gender-info">
        <span class="label">使用常模:</span>
        <span class="value">{{ result.mmpiScores.gender === 'male' ? '男性' : '女性' }}</span>
      </div>

      <!-- 效度量表 -->
      <div class="scale-category">
        <h3 class="category-title">效度量表</h3>
        <div class="scales-grid">
          <div v-for="(score, key) in result.mmpiScores.validity" :key="key" class="scale-card">
            <div class="scale-header">
              <span class="scale-name">{{ key }} - {{ score.name }}</span>
            </div>
            <div class="scale-scores">
              <div class="score-item">
                <span class="score-label">原始分:</span>
                <span class="score-val">{{ score.rawScore }}</span>
              </div>
              <div v-if="score.tScore !== undefined" class="score-item">
                <span class="score-label">T分:</span>
                <span class="score-val" :class="getTScoreClass(score.tScore)">{{ score.tScore }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 临床量表 -->
      <div class="scale-category">
        <h3 class="category-title">临床量表</h3>
        <div class="scales-grid">
          <div v-for="(score, key) in result.mmpiScores.clinical" :key="key" class="scale-card">
            <div class="scale-header">
              <span class="scale-name">{{ key }} - {{ score.name }}</span>
            </div>
            <div class="scale-scores">
              <div class="score-item">
                <span class="score-label">原始分:</span>
                <span class="score-val">{{ score.rawScore }}</span>
              </div>
              <div class="score-item">
                <span class="score-label">T分:</span>
                <span class="score-val" :class="getTScoreClass(score.tScore)">{{ score.tScore }}</span>
              </div>
              <div v-if="score.kCorrected" class="score-item k-corrected">
                <span class="score-label">K校正(×{{ score.kCorrected.coefficient }}):</span>
                <span class="score-val">{{ score.kCorrected.rawScore }} 分</span>
              </div>
              <div v-if="score.kCorrected" class="score-item k-corrected">
                <span class="score-label">K校正T分:</span>
                <span class="score-val" :class="getTScoreClass(score.kCorrected.tScore)">{{ score.kCorrected.tScore }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 附加量表 -->
      <div v-if="hasAdditionalScales" class="scale-category">
        <h3 class="category-title">附加量表</h3>
        <div class="scales-grid">
          <div v-for="(score, key) in result.mmpiScores.additional" :key="key" class="scale-card">
            <div class="scale-header">
              <span class="scale-name">{{ key }} - {{ score.name }}</span>
            </div>
            <div class="scale-scores">
              <div class="score-item">
                <span class="score-label">原始分:</span>
                <span class="score-val">{{ score.rawScore }}</span>
              </div>
              <div class="score-item">
                <span class="score-label">T分:</span>
                <span class="score-val" :class="getTScoreClass(score.tScore)">{{ score.tScore }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- T分说明 -->
      <div class="tscore-legend">
        <h4>T分解读</h4>
        <div class="legend-items">
          <div class="legend-item">
            <span class="legend-color normal"></span>
            <span>正常范围 (40-60)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color mild"></span>
            <span>轻度异常 (60-70 或 30-40)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color severe"></span>
            <span>显著异常 (>70 或 <30)</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 普通得分显示 -->
    <div v-else class="score-section">
      <div class="score-card">
        <h3>原始分</h3>
        <div class="score-value">{{ result.rawScore }}</div>
        <p v-if="quiz.scoring && quiz.scoring.range" class="score-range">
          量表范围: {{ quiz.scoring.range.min }} - {{ quiz.scoring.range.max }}
        </p>
      </div>

      <div v-if="result.scaledScore !== undefined" class="score-card">
        <h3>标准分</h3>
        <div class="score-value">{{ result.scaledScore }}</div>
      </div>
    </div>

    <!-- 结论解释 -->
    <div v-if="conclusion" class="conclusion-section">
      <h3>结论解释</h3>
      <div class="conclusion-box">
        <div class="interpretation">{{ conclusion }}</div>
      </div>
    </div>

    <!-- 详细分析 -->
    <div v-if="hasSubscores" class="subscores-section">
      <h3>各维度得分</h3>
      <div class="subscores-grid">
        <div v-for="(score, name) in result.subscores" :key="name" class="subscore-card">
          <div class="subscore-name">{{ name }}</div>
          <div class="subscore-value">{{ score }}</div>
        </div>
      </div>
    </div>

    <!-- 建议 -->
    <div class="recommendation-section">
      <h3>温馨提示</h3>
      <ul class="recommendation-list">
        <li>本测评仅供参考，不能替代专业的心理诊断</li>
        <li>如果您的测评结果显示可能存在心理健康问题，建议咨询专业的心理咨询师或医生</li>
        <li>定期进行自我检查和评估有助于维护心理健康</li>
      </ul>
    </div>

    <!-- 按钮区域 -->
    <div class="button-container">
      <button class="save-btn" @click="handleSave">
        <span class="icon">💾</span> 保存结果
      </button>
      <button class="reset-btn" @click="handleReset">
        <span class="icon">↻</span> 重新开始
      </button>
      <button class="back-btn" @click="handleBack">
        <span class="icon">←</span> 返回首页
      </button>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'ResultDisplay',
  props: {
    result: {
      type: Object,
      required: true
    },
    quiz: {
      type: Object,
      required: true
    }
  },
  emits: ['reset'],
  setup(props, { emit }) {
    const isMMPI = computed(() => {
      return props.result.mmpiScores !== undefined
    })

    const hasAdditionalScales = computed(() => {
      return props.result.mmpiScores?.additional && 
        Object.keys(props.result.mmpiScores.additional).length > 0
    })

    const hasSubscores = computed(() => {
      return props.result.subscores && Object.keys(props.result.subscores).length > 0
    })

    const getTScoreClass = (tScore) => {
      if (tScore >= 70 || tScore <= 30) return 'severe'
      if ((tScore >= 60 && tScore < 70) || (tScore > 30 && tScore <= 40)) return 'mild'
      return 'normal'
    }

    const handleSave = () => {
      // 准备保存的数据
      const timestamp = new Date().toISOString()
      const saveData = {
        timestamp: timestamp,
        quiz: {
          id: props.quiz.id,
          title: props.quiz.title,
          description: props.quiz.description
        },
        answers: props.result.answers,
        result: {
          rawScore: props.result.rawScore,
          scaledScore: props.result.scaledScore,
          subscores: props.result.subscores,
          timeTaken: props.result.timeTaken
        },
        conclusion: conclusion.value
      }

      // 创建Blob对象
      const blob = new Blob([JSON.stringify(saveData, null, 2)], {
        type: 'application/json'
      })

      // 创建下载链接
      const url = URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = url
      
      // 使用时间戳创建文件名
      const dateStr = new Date().toISOString().replace(/[:.]/g, '-').slice(0, -5)
      link.download = `quiz_result_${props.quiz.id}_${dateStr}.json`
      
      // 触发下载
      document.body.appendChild(link)
      link.click()
      
      // 清理
      document.body.removeChild(link)
      URL.revokeObjectURL(url)
    }

    const conclusion = computed(() => {
      // 支持两种格式：quiz.scoring.interpretations 或 quiz.interpretation
      const interpretations = props.quiz.scoring?.interpretations || props.quiz.interpretation
      
      if (!interpretations) {
        return null
      }

      // 查找对应的区间
      for (const range of interpretations) {
        if (
          props.result.rawScore >= range.min &&
          props.result.rawScore <= range.max
        ) {
          return range.description || range.conclusion
        }
      }

      return null
    })

    const handleReset = () => {
      emit('reset')
    }

    const handleBack = () => {
      window.location.href = '/'
    }

    return {
      isMMPI,
      hasAdditionalScales,
      hasSubscores,
      conclusion,
      getTScoreClass,
      handleSave,
      handleReset,
      handleBack
    }
  }
}
</script>

<style lang="scss" scoped>
.result-display {
  animation: fadeIn 0.5s ease-out;

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
}

.result-header {
  text-align: center;
  margin-bottom: 40px;
  padding-bottom: 20px;
  border-bottom: 2px solid #eee;

  h2 {
    font-size: 2.2em;
    color: #333;
    margin-bottom: 10px;
  }

  .completion-time {
    color: #888;
    font-size: 0.95em;
  }
}

.score-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.score-card {
  background: #3b82f6;
  color: white;
  padding: 30px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.2);

  h3 {
    font-size: 1em;
    font-weight: 500;
    margin-bottom: 15px;
    opacity: 0.9;
  }

  .score-value {
    font-size: 3em;
    font-weight: 700;
    margin-bottom: 10px;
  }

  .score-range {
    font-size: 0.9em;
    opacity: 0.85;
  }
}

.conclusion-section {
  background-color: #f0f9ff;
  padding: 25px;
  border-radius: 10px;
  border-left: 4px solid #3b82f6;
  margin-bottom: 30px;

  h3 {
    color: #333;
    margin-bottom: 15px;
    font-size: 1.15em;
  }

  .conclusion-box {
    background: white;
    padding: 20px;
    border-radius: 8px;
    border: 1px solid #e0e0e0;
  }

  .interpretation {
    color: #333;
    line-height: 1.8;
    font-size: 1em;
  }
}

.subscores-section {
  margin-bottom: 30px;

  h3 {
    color: #333;
    margin-bottom: 20px;
    font-size: 1.15em;
  }

  .subscores-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 15px;
  }

  .subscore-card {
    background: white;
    border: 2px solid #e0e0e0;
    padding: 20px;
    border-radius: 10px;
    text-align: center;
    transition: all 0.3s ease;

    &:hover {
      border-color: #3b82f6;
      box-shadow: 0 2px 8px rgba(59, 130, 246, 0.1);
    }

    .subscore-name {
      font-size: 0.9em;
      color: #6b7280;
      margin-bottom: 10px;
    }

    .subscore-value {
      font-size: 1.8em;
      font-weight: 700;
      color: #3b82f6;
    }
  }
}

.recommendation-section {
  background-color: #fff3cd;
  border: 1px solid #ffc107;
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 30px;

  h3 {
    color: #333;
    margin-bottom: 15px;
    font-size: 1.1em;
  }

  .recommendation-list {
    list-style: none;
    padding: 0;

    li {
      color: #333;
      padding: 8px 0;
      padding-left: 25px;
      position: relative;

      &:before {
        content: '✓';
        position: absolute;
        left: 0;
        color: #ffc107;
        font-weight: bold;
      }
    }
  }
}

.button-container {
  display: flex;
  gap: 15px;
  margin-top: 40px;
  padding-top: 20px;
  border-top: 2px solid #eee;
}

button {
  flex: 1;
  padding: 14px 24px;
  font-size: 1em;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;

  &:hover {
    transform: translateY(-2px);
  }

  &:active {
    transform: scale(0.98);
  }

  .icon {
    font-size: 1.2em;
  }
}

.save-btn {
  background: #10b981;
  color: white;
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);

  &:hover {
    background: #059669;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
  }
}

.reset-btn {
  background: #3b82f6;
  color: white;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);

  &:hover {
    background: #2563eb;
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4);
  }
}

.back-btn {
  background-color: #10b981;
  color: white;
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);

  &:hover {
    background-color: #059669;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
  }
}

// MMPI专用样式
.mmpi-scores-section {
  margin-bottom: 30px;

  .gender-info {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 15px 25px;
    border-radius: 10px;
    margin-bottom: 25px;
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 1.1em;

    .label {
      font-weight: 600;
    }

    .value {
      font-weight: 700;
      font-size: 1.2em;
    }
  }

  .scale-category {
    margin-bottom: 35px;
    background: #f8f9fa;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.05);

    .category-title {
      font-size: 1.4em;
      color: #1e293b;
      margin-bottom: 20px;
      padding-bottom: 10px;
      border-bottom: 3px solid #3b82f6;
    }

    .scales-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
      gap: 15px;

      .scale-card {
        background: white;
        border: 2px solid #e5e7eb;
        border-radius: 10px;
        padding: 18px;
        transition: all 0.3s ease;

        &:hover {
          border-color: #3b82f6;
          box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15);
          transform: translateY(-2px);
        }

        .scale-header {
          margin-bottom: 12px;
          padding-bottom: 10px;
          border-bottom: 2px solid #f3f4f6;

          .scale-name {
            font-weight: 700;
            color: #334155;
            font-size: 0.95em;
          }
        }

        .scale-scores {
          display: flex;
          flex-direction: column;
          gap: 8px;

          .score-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 6px 10px;
            background: #f9fafb;
            border-radius: 6px;

            &.k-corrected {
              background: #fef3c7;
              border-left: 3px solid #f59e0b;
            }

            .score-label {
              font-size: 0.85em;
              color: #64748b;
              font-weight: 500;
            }

            .score-val {
              font-weight: 700;
              font-size: 1em;
              padding: 2px 8px;
              border-radius: 4px;

              &.normal {
                color: #10b981;
                background: #d1fae5;
              }

              &.mild {
                color: #f59e0b;
                background: #fef3c7;
              }

              &.severe {
                color: #ef4444;
                background: #fee2e2;
              }
            }
          }
        }
      }
    }
  }

  .tscore-legend {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px 25px;
    border-radius: 10px;
    margin-top: 25px;

    h4 {
      margin: 0 0 15px 0;
      font-size: 1.1em;
      font-weight: 600;
    }

    .legend-items {
      display: flex;
      flex-wrap: wrap;
      gap: 20px;

      .legend-item {
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 0.9em;

        .legend-color {
          width: 20px;
          height: 20px;
          border-radius: 4px;
          display: inline-block;

          &.normal {
            background: #10b981;
          }

          &.mild {
            background: #f59e0b;
          }

          &.severe {
            background: #ef4444;
          }
        }
      }
    }
  }
}

@media (max-width: 768px) {
  .result-header h2 {
    font-size: 1.6em;
  }

  .score-section {
    grid-template-columns: 1fr;
    gap: 15px;
  }

  .score-card {
    padding: 20px;

    .score-value {
      font-size: 2.2em;
    }
  }

  .button-container {
    flex-direction: column;
    gap: 10px;

    button {
      width: 100%;
    }
  }

  .subscores-grid {
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)) !important;
    gap: 10px;
  }

  .mmpi-scores-section {
    .scale-category .scales-grid {
      grid-template-columns: 1fr;
    }

    .tscore-legend .legend-items {
      flex-direction: column;
      gap: 10px;
    }
  }
}
</style>
