<template>
  <div class="slider-container">
    <div class="slider-wrapper">
      <input
        type="range"
        class="slider"
        :min="min"
        :max="max"
        :value="answer !== undefined ? answer : (min + max) / 2"
        @input="handleChange"
      />
    </div>
    <div class="slider-display">
      <span class="slider-min">{{ min }}</span>
      <span class="slider-value">{{ answer !== undefined ? answer : (min + max) / 2 }}</span>
      <span class="slider-max">{{ max }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SliderOption',
  props: {
    questionId: {
      type: [String, Number],
      required: true
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    answer: {
      type: Number,
      default: undefined
    }
  },
  emits: ['answer'],
  setup(props, { emit }) {
    const handleChange = (e) => {
      const value = parseFloat(e.target.value)
      emit('answer', value)
    }

    return {
      handleChange
    }
  }
}
</script>

<style lang="scss" scoped>
.slider-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 15px 0;
}

.slider-wrapper {
  display: flex;
  align-items: center;
}

.slider {
  width: 100%;
  height: 8px;
  border-radius: 5px;
  background: linear-gradient(to right, #e0e0e0 0%, #667eea 50%, #764ba2 100%);
  outline: none;
  appearance: none;
  cursor: pointer;

  &::-webkit-slider-thumb {
    appearance: none;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
    transition: all 0.2s ease;

    &:hover {
      width: 28px;
      height: 28px;
      box-shadow: 0 4px 12px rgba(102, 126, 234, 0.6);
    }
  }

  &::-moz-range-thumb {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
    border: none;
    transition: all 0.2s ease;

    &:hover {
      width: 28px;
      height: 28px;
      box-shadow: 0 4px 12px rgba(102, 126, 234, 0.6);
    }
  }
}

.slider-display {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 5px;
}

.slider-min,
.slider-max {
  color: #888;
  font-size: 0.9em;
}

.slider-value {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 6px 14px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 1em;
  min-width: 50px;
  text-align: center;
}

@media (max-width: 768px) {
  .slider-container {
    gap: 10px;
  }

  .slider {
    height: 6px;

    &::-webkit-slider-thumb {
      width: 20px;
      height: 20px;

      &:hover {
        width: 22px;
        height: 22px;
      }
    }

    &::-moz-range-thumb {
      width: 20px;
      height: 20px;

      &:hover {
        width: 22px;
        height: 22px;
      }
    }
  }

  .slider-value {
    padding: 4px 12px;
    font-size: 0.9em;
  }
}
</style>
