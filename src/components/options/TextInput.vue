<template>
  <div class="text-input-container">
    <input
      type="text"
      class="text-input"
      :value="answer || ''"
      :placeholder="placeholder"
      @input="handleChange"
    />
  </div>
</template>

<script>
export default {
  name: 'TextInput',
  props: {
    questionId: {
      type: [String, Number],
      required: true
    },
    answer: {
      type: String,
      default: ''
    },
    placeholder: {
      type: String,
      default: '请输入您的答案'
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const handleChange = (e) => {
      emit('answer', e.target.value)
    }

    return {
      handleChange
    }
  }
}
</script>

<style lang="scss" scoped>
.text-input-container {
  display: flex;
  align-items: center;
}

.text-input {
  width: 100%;
  padding: 12px 15px;
  font-size: 1em;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  transition: all 0.2s ease;
  font-family: inherit;

  &:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  &:hover:not(:focus) {
    border-color: #bbb;
  }
}

@media (max-width: 768px) {
  .text-input {
    padding: 10px 12px;
    font-size: 0.95em;
  }
}
</style>
