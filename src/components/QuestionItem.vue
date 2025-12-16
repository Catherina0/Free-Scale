<template>
  <div class="question" :data-question-id="questionId">
    <div class="question-header">
      <span class="question-number">第 {{ questionNumber }} 题</span>
      <span v-if="question.required !== false" class="required-mark">*</span>
    </div>
    <div class="question-text">{{ question.text }}</div>
    <div v-if="question.subtext" class="question-subtext">{{ question.subtext }}</div>

    <div class="options-wrapper">
      <!-- 单选题 -->
      <RadioOptions
        v-if="questionType === 'radio'"
        :question-id="questionId"
        :options="effectiveOptions"
        :answer="answer"
        @answer="handleAnswer"
      />

      <!-- 多选题 -->
      <CheckboxOptions
        v-else-if="questionType === 'checkbox'"
        :question-id="questionId"
        :options="effectiveOptions"
        :answer="answer"
        @answer="handleAnswer"
      />

      <!-- 滑块题 -->
      <SliderOption
        v-else-if="questionType === 'slider'"
        :question-id="questionId"
        :min="question.min || 0"
        :max="question.max || 100"
        :answer="answer"
        @answer="handleAnswer"
      />

      <!-- 文本输入题 -->
      <TextInput
        v-else-if="questionType === 'text'"
        :question-id="questionId"
        :answer="answer"
        @answer="handleAnswer"
      />

      <!-- 特殊问题 -->
      <SpecialQuestion
        v-else-if="questionType === 'special'"
        :question-id="questionId"
        :question="question"
        :answer="answer"
        @answer="handleAnswer"
      />
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'
import RadioOptions from './options/RadioOptions.vue'
import CheckboxOptions from './options/CheckboxOptions.vue'
import SliderOption from './options/SliderOption.vue'
import TextInput from './options/TextInput.vue'
import SpecialQuestion from './options/SpecialQuestion.vue'

export default {
  name: 'QuestionItem',
  components: {
    RadioOptions,
    CheckboxOptions,
    SliderOption,
    TextInput,
    SpecialQuestion
  },
  props: {
    question: {
      type: Object,
      required: true
    },
    questionNumber: {
      type: Number,
      required: true
    },
    answer: {
      type: [String, Number, Array],
      default: undefined
    },
    defaultOptions: {
      type: Array,
      default: () => []
    },
    defaultQuestionType: {
      type: String,
      default: 'radio'
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const questionId = computed(() => props.question.id !== undefined ? props.question.id : props.questionNumber)

    const questionType = computed(() => {
      return props.question.type || props.defaultQuestionType || 'radio'
    })

    const effectiveOptions = computed(() => {
      return props.question.options || props.defaultOptions || []
    })

    const handleAnswer = (value) => {
      emit('answer', value)
    }

    return {
      questionId,
      questionType,
      effectiveOptions,
      handleAnswer
    }
  }
}
</script>

<style lang="scss" scoped>
.question {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  border-left: 3px solid #3b82f6;
  transition: all 0.2s ease;

  &:hover {
    border-left-color: #2563eb;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  &.highlight-missing {
    border-color: #ff6b6b;
    background-color: #fff5f5;
    animation: pulse 0.6s ease-out;
  }
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(255, 107, 107, 0.4);
  }
  70% {
    box-shadow: 0 0 0 10px rgba(255, 107, 107, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(255, 107, 107, 0);
  }
}

.question-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.question-number {
  font-weight: 600;
  color: #3b82f6;
  font-size: 0.9em;
}

.required-mark {
  color: #ef4444;
  font-weight: bold;
  font-size: 1.1em;
}

.question-text {
  font-size: 1em;
  color: #1f2937;
  margin-bottom: 8px;
  font-weight: 500;
  line-height: 1.5;
}

.question-subtext {
  font-size: 0.95em;
  color: #888;
  margin-bottom: 15px;
  font-style: italic;
}

.options-wrapper {
  margin-top: 15px;
}

@media (max-width: 768px) {
  .question {
    padding: 18px;
  }

  .question-text {
    font-size: 1em;
  }

  .question-subtext {
    font-size: 0.9em;
  }
}
</style>
