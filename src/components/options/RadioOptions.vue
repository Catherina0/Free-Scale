<template>
  <div class="radio-options">
    <label
      v-for="(option, index) in options"
      :key="index"
      class="option-label"
    >
      <input
        type="radio"
        :name="`q${questionId}`"
        :value="option.value || option.score || index"
        :checked="answer == (option.value || option.score || index)"
        @change="handleChange"
      />
      <span class="option-text">{{ option.text || option.label }}</span>
    </label>
  </div>
</template>

<script>
export default {
  name: 'RadioOptions',
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
      type: [String, Number],
      default: undefined
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const handleChange = (e) => {
      const value = isNaN(e.target.value) ? e.target.value : parseFloat(e.target.value)
      emit('answer', value)
    }

    return {
      handleChange
    }
  }
}
</script>

<style lang="scss" scoped>
.radio-options {
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

  input[type='radio'] {
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
  .radio-options {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }
  
  .option-label {
    padding: 8px 10px;
  }

  input[type='radio'] {
    width: 16px;
    height: 16px;
  }

  .option-text {
    font-size: 0.95em;
  }
}
</style>
