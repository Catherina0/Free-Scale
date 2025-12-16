<template>
  <div class="questionnaire-form">
    <!-- 题目列表 -->
    <div class="questions-container">
      <QuestionItem
        v-for="(question, index) in quiz.questions"
        :key="question.id !== undefined ? question.id : index"
        :question="question"
        :question-number="index + 1"
        :answer="answers[question.id !== undefined ? question.id : (index + 1)]"
        :default-options="quiz.defaultOptions"
        :default-question-type="quiz.defaultQuestionType"
        @answer="(value) => $emit('answer', question.id !== undefined ? question.id : (index + 1), value)"
      />
    </div>

    <!-- 按钮区域 -->
    <div class="button-container">
      <button class="reset-btn" @click="handleReset">
        <span class="icon">↻</span> 重新开始
      </button>
      <button class="submit-btn" @click="handleSubmit">
        <span class="icon">✓</span> 提交评估
      </button>
    </div>
  </div>
</template>

<script>
import QuestionItem from './QuestionItem.vue'

export default {
  name: 'QuestionnaireForm',
  components: {
    QuestionItem
  },
  props: {
    quiz: {
      type: Object,
      required: true
    },
    answers: {
      type: Object,
      required: true
    }
  },
  emits: ['answer', 'submit'],
  setup(props, { emit }) {
    const handleSubmit = () => {
      emit('submit')
    }

    const handleReset = () => {
      // 通过发送空答案来重置所有答案
      props.quiz.questions.forEach((q, index) => {
        emit('answer', q.id || index, undefined)
      })
    }

    return {
      handleSubmit,
      handleReset
    }
  }
}
</script>

<style lang="scss" scoped>
.questionnaire-form {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.questions-container {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.button-container {
  display: flex;
  gap: 15px;
  justify-content: center;
  padding: 30px 0 0 0;
  border-top: 2px solid #eee;
}

button {
  padding: 14px 32px;
  font-size: 1.1em;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;

  .icon {
    font-size: 1.2em;
  }

  &:active {
    transform: scale(0.98);
  }
}

.submit-btn {
  background: #3b82f6;
  color: white;
  flex: 1;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);

  &:hover {
    background: #2563eb;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4);
  }
}

.reset-btn {
  background-color: #f0f0f0;
  color: #333;
  flex: 1;

  &:hover {
    background-color: #e0e0e0;
  }
}

@media (max-width: 768px) {
  .button-container {
    flex-direction: column;
    gap: 10px;
  }

  button {
    width: 100%;
  }
}
</style>
