<template>
  <div class="checkbox-options">
    <label
      v-for="(option, index) in options"
      :key="index"
      class="option-label"
    >
      <input
        type="checkbox"
        :value="option.value || option.score || index"
        :checked="isChecked(index)"
        @change="handleChange"
      />
      <span class="option-text">{{ option.text || option.label }}</span>
    </label>
  </div>
</template>

<script>
export default {
  name: 'CheckboxOptions',
  props: {
    questionId: {
      type: [String, Number],
      required: true
    },
    options: {
      type: Array,
      required: true
    },
    answer: {
      type: Array,
      default: () => []
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const isChecked = (index) => {
      const value = props.options[index].value || props.options[index].score || index
      return props.answer && props.answer.includes(value)
    }

    const handleChange = (e) => {
      const value = isNaN(e.target.value) ? e.target.value : parseFloat(e.target.value)
      let newAnswer = [...(props.answer || [])]

      if (e.target.checked) {
        if (!newAnswer.includes(value)) {
          newAnswer.push(value)
        }
      } else {
        newAnswer = newAnswer.filter(v => v !== value)
      }

      emit('answer', newAnswer)
    }

    return {
      isChecked,
      handleChange
    }
  }
}
</script>

<style lang="scss" scoped>
.checkbox-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 8px;
  max-width: 100%;
}

.option-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 8px 12px;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  transition: all 0.2s ease;
  background: white;

  &:hover {
    border-color: #3b82f6;
    background-color: #eff6ff;
  }

  input[type='checkbox'] {
    margin-right: 8px;
    cursor: pointer;
    width: 16px;
    height: 16px;
    accent-color: #3b82f6;
    flex-shrink: 0;
  }

  .option-text {
    color: #374151;
    font-size: 0.9em;
    user-select: none;
    line-height: 1.3;
  }

  &:has(input:checked) {
    background-color: #dbeafe;
    border-color: #3b82f6;
    font-weight: 500;
  }
}

@media (max-width: 768px) {
  .checkbox-options {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }
  
  .option-label {
    padding: 8px 10px;
  }

  input[type='checkbox'] {
    width: 16px;
    height: 16px;
  }

  .option-text {
    font-size: 0.95em;
  }
}
</style>
