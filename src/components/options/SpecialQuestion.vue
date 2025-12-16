<template>
  <div class="special-question-container">
    <div
      v-for="(subQuestion, index) in question.subQuestions"
      :key="index"
      class="sub-question"
    >
      <span class="sub-question-letter">{{ String.fromCharCode(97 + index) }})&nbsp;</span>
      <span class="sub-question-text">{{ subQuestion }}</span>
      <div class="sub-question-options">
        <label class="option-label">
          <input
            type="radio"
            :name="`q${questionId}_${index}`"
            value="1"
            :checked="getSubAnswer(index) == 1"
            @change="handleSubChange(index, 1)"
          />
          <span class="option-text">是</span>
        </label>
        <label class="option-label">
          <input
            type="radio"
            :name="`q${questionId}_${index}`"
            value="0"
            :checked="getSubAnswer(index) == 0"
            @change="handleSubChange(index, 0)"
          />
          <span class="option-text">否</span>
        </label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SpecialQuestion',
  props: {
    questionId: {
      type: [String, Number],
      required: true
    },
    question: {
      type: Object,
      required: true
    },
    answer: {
      type: Array,
      default: () => []
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const getSubAnswer = (index) => {
      return props.answer && props.answer[index]
    }

    const handleSubChange = (index, value) => {
      const newAnswer = [...(props.answer || [])]
      newAnswer[index] = parseFloat(value)
      emit('answer', newAnswer)
    }

    return {
      getSubAnswer,
      handleSubChange
    }
  }
}
</script>

<style lang="scss" scoped>
.special-question-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.sub-question {
  background-color: #fff;
  padding: 15px;
  border-radius: 8px;
  border: 1px solid #e5e5e5;
  display: flex;
  gap: 10px;
  align-items: flex-start;

  &:hover {
    border-color: #667eea;
    background-color: #f8f8ff;
  }
}

.sub-question-letter {
  font-weight: 600;
  color: #667eea;
  min-width: 20px;
  margin-top: 2px;
}

.sub-question-text {
  flex: 1;
  color: #333;
  font-size: 0.95em;
  line-height: 1.5;
  margin-right: 15px;
}

.sub-question-options {
  display: flex;
  gap: 15px;
  min-width: 100px;
}

.option-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  white-space: nowrap;
  gap: 6px;

  input[type='radio'] {
    cursor: pointer;
    accent-color: #667eea;
  }

  .option-text {
    cursor: pointer;
    font-size: 0.95em;
    color: #333;
  }
}

@media (max-width: 768px) {
  .sub-question {
    flex-direction: column;
    gap: 8px;
  }

  .sub-question-text {
    margin-right: 0;
  }

  .sub-question-options {
    justify-content: flex-start;
  }
}
</style>
