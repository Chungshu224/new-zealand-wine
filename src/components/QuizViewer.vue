<template>
  <div class="quiz-section">
    <h3 class="quiz-title">💡 知識檢測</h3>
    
    <img 
      v-if="quiz.image" 
      :src="quiz.image" 
      alt="quiz image" 
      class="quiz-image"
    />
    
    <p class="quiz-question">{{ quiz.question }}</p>
    
    <div class="quiz-options">
      <button 
        v-for="(option, index) in quiz.options"
        :key="index"
        @click="selectAnswer(index)"
        :class="getQuizOptionClass(index)"
        :disabled="answered"
      >
        {{ option }}
      </button>
    </div>

    <div v-if="answered" class="quiz-feedback">
      <p :class="['feedback-text', isCorrect ? 'correct' : 'incorrect']">
        {{ isCorrect ? '✅ 答對了！' : '❌ 再想想看' }}
      </p>
      <p class="quiz-explanation">{{ quiz.explanation }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  quiz: {
    type: Object,
    required: true,
    validator: (value) => {
      return value.question && 
             Array.isArray(value.options) && 
             typeof value.correct === 'number' &&
             value.explanation
    }
  }
})

const answered = ref(false)
const selectedAnswer = ref(null)

const isCorrect = computed(() => {
  return answered.value && selectedAnswer.value === props.quiz.correct
})

const selectAnswer = (index) => {
  if (!answered.value) {
    selectedAnswer.value = index
    answered.value = true
  }
}

const getQuizOptionClass = (index) => {
  const classes = ['quiz-option']
  
  if (!answered.value) {
    if (selectedAnswer.value === index) {
      classes.push('selected')
    }
  } else {
    if (index === props.quiz.correct) {
      classes.push('correct')
    } else if (index === selectedAnswer.value) {
      classes.push('incorrect')
    }
  }
  
  return classes.join(' ')
}
</script>

<style scoped>
.quiz-section {
  background: #f8f9fa;
  padding: 2rem;
  border-radius: 15px;
  margin: 2rem 0;
}

.quiz-title {
  color: #333;
  margin-bottom: 1.5rem;
  font-size: 1.3rem;
  font-weight: 600;
}

.quiz-image {
  display: block;
  max-width: 520px;
  width: 100%;
  height: auto;
  margin: 0 0 1.5rem 0;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

.quiz-question {
  font-size: 1.1rem;
  color: #555;
  margin-bottom: 1.5rem;
  line-height: 1.6;
  font-weight: 500;
}

.quiz-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.quiz-option {
  padding: 1rem 1.5rem;
  border: 2px solid #dee2e6;
  border-radius: 10px;
  background: white;
  color: #495057;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  text-align: left;
  font-weight: 500;
}

.quiz-option:hover:not(:disabled) {
  border-color: #4a90e2;
  background: #f0f7ff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.2);
}

.quiz-option.selected {
  border-color: #4a90e2;
  background: #e3f2fd;
}

.quiz-option.correct {
  border-color: #28a745;
  background: #d4edda;
  color: #155724;
}

.quiz-option.incorrect {
  border-color: #dc3545;
  background: #f8d7da;
  color: #721c24;
}

.quiz-option:disabled {
  cursor: not-allowed;
}

.quiz-feedback {
  background: white;
  padding: 1.5rem;
  border-radius: 10px;
  border-left: 4px solid #4a90e2;
  margin-top: 1.5rem;
}

.feedback-text {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 1rem;
}

.feedback-text.correct {
  color: #28a745;
}

.feedback-text.incorrect {
  color: #dc3545;
}

.quiz-explanation {
  color: #666;
  line-height: 1.6;
  font-size: 1rem;
}

@media (max-width: 768px) {
  .quiz-section {
    padding: 1.5rem 1rem;
  }

  .quiz-options {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .quiz-option {
    padding: 14px 16px;
    font-size: 16px;
    min-height: 52px;
    text-align: left;
    line-height: 1.5;
  }

  .quiz-question {
    font-size: 18px;
    line-height: 1.6;
    margin-bottom: 24px;
  }

  .quiz-title {
    font-size: 20px;
  }

  .quiz-feedback {
    padding: 16px;
  }

  .feedback-text {
    font-size: 17px;
  }
}

@media (max-width: 480px) {
  .quiz-section {
    padding: 1.25rem 0.75rem;
  }

  .quiz-question {
    font-size: 17px;
  }

  .quiz-option {
    padding: 12px 14px;
    font-size: 15px;
    min-height: 48px;
  }
}
</style>
