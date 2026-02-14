<template>
  <div class="testing-mode">
    <div v-if="!currentTest" class="test-setup">
      <h2>
        <img src="/study-svgrepo-com.svg" class="testing-icon" alt="Test" />
        Generate a Test
      </h2>
      <p>AI will create personalized questions based on "{{ book.customName || book.name }}"</p>
      
      <div class="test-options">
        <div class="option-group">
          <label>Number of Questions:</label>
          <select v-model="testOptions.questionCount">
            <option :value="5">5 questions</option>
            <option :value="10">10 questions</option>
            <option :value="15">15 questions</option>
            <option :value="20">20 questions</option>
          </select>
        </div>
        
        <div class="option-group">
          <label>Difficulty Level:</label>
          <select v-model="testOptions.difficulty">
            <option value="easy">Easy</option>
            <option value="medium">Medium</option>
            <option value="hard">Hard</option>
            <option value="mixed">Mixed</option>
          </select>
        </div>
        
        <div class="option-group">
          <label>Focus Topic (Optional):</label>
          <input 
            v-model="testOptions.topic" 
            type="text" 
            placeholder="e.g., Chapter 3, Linear Algebra, etc."
          />
        </div>
      </div>
      
      <button @click="generateTest" class="btn-primary btn-large" :disabled="isLoading">
        {{ isLoading ? 'Generating...' : 'Generate Test' }}
      </button>
    </div>

    <!-- Active Test -->
    <div v-else class="active-test">
      <div class="test-header">
        <h2>Test in Progress</h2>
        <div class="test-progress">
          Question {{ currentQuestionIndex + 1 }} of {{ currentTest.questions.length }}
        </div>
      </div>

      <div class="question-card">
        <h3>Question {{ currentQuestionIndex + 1 }}</h3>
        <p class="question-text">{{ currentTest.questions[currentQuestionIndex].question }}</p>
        
        <div class="answer-options" v-if="currentTest.questions[currentQuestionIndex].type === 'multiple-choice'">
          <div 
            v-for="(option, idx) in currentTest.questions[currentQuestionIndex].options" 
            :key="idx"
            :class="['option', { selected: currentAnswer === option }]"
            @click="currentAnswer = option"
          >
            <span class="option-letter">{{ String.fromCharCode(65 + idx) }}</span>
            <span>{{ option }}</span>
          </div>
        </div>
        
        <div v-else class="answer-input">
          <textarea 
            v-model="currentAnswer"
            placeholder="Type your answer here..."
            rows="5"
          ></textarea>
        </div>
      </div>

      <div class="test-navigation">
        <button 
          @click="previousQuestion" 
          class="btn-secondary"
          :disabled="currentQuestionIndex === 0"
        >
          ← Previous
        </button>
        
        <button 
          v-if="currentQuestionIndex < currentTest.questions.length - 1"
          @click="nextQuestion" 
          class="btn-primary"
          :disabled="!currentAnswer"
        >
          Next →
        </button>
        
        <button 
          v-else
          @click="submitTest" 
          class="btn-primary"
          :disabled="!currentAnswer"
        >
          Submit Test
        </button>
      </div>
    </div>

    <!-- Test Results -->
    <div v-if="testResults" class="test-results">
      <div class="results-header">
        <h2>🎉 Test Completed!</h2>
        <div class="score-circle">
          <span class="score">{{ testResults.score }}%</span>
        </div>
      </div>
      
      <div class="results-summary">
        <div class="stat">
          <span class="stat-label">Correct Answers:</span>
          <span class="stat-value">{{ testResults.correct }} / {{ testResults.total }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Readiness Level:</span>
          <span class="stat-value">{{ testResults.readiness }}</span>
        </div>
      </div>

      <div class="detailed-feedback">
        <h3>Detailed Feedback</h3>
        <div 
          v-for="(result, idx) in testResults.details" 
          :key="idx"
          class="feedback-item"
        >
          <div class="feedback-header">
            <span class="question-num">Q{{ idx + 1 }}</span>
            <span :class="['result-badge', result.correct ? 'correct' : 'incorrect']">
              {{ result.correct ? '✓ Correct' : '✗ Incorrect' }}
            </span>
          </div>
          <p class="feedback-text">{{ result.feedback }}</p>
        </div>
      </div>

      <button @click="resetTest" class="btn-primary">Take Another Test</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  book: {
    type: Object,
    required: true
  }
})

const isLoading = ref(false)
const testOptions = ref({
  questionCount: 10,
  difficulty: 'medium',
  topic: ''
})
const currentTest = ref(null)
const currentQuestionIndex = ref(0)
const currentAnswer = ref('')
const userAnswers = ref([])
const testResults = ref(null)

const generateTest = () => {
  isLoading.value = true
  
  setTimeout(() => {
    const questions = []
    for (let i = 0; i < testOptions.value.questionCount; i++) {
      questions.push({
        question: `Question ${i + 1}: Based on "${props.book.name}", ${generateQuestionText()}`,
        type: Math.random() > 0.5 ? 'multiple-choice' : 'open-ended',
        options: Math.random() > 0.5 ? [
          'Option A - First possible answer',
          'Option B - Second possible answer',
          'Option C - Third possible answer',
          'Option D - Fourth possible answer'
        ] : null,
        correctAnswer: 'Option B - Second possible answer'
      })
    }
    
    currentTest.value = { questions }
    currentQuestionIndex.value = 0
    currentAnswer.value = ''
    userAnswers.value = new Array(questions.length).fill('')
    isLoading.value = false
  }, 2000)
}

const generateQuestionText = () => {
  const questions = [
    'what is the main principle discussed in this chapter?',
    'explain the relationship between these key concepts.',
    'how would you apply this theory in a practical scenario?',
    'what are the three main components of this system?',
    'describe the process outlined in the text.'
  ]
  return questions[Math.floor(Math.random() * questions.length)]
}

const nextQuestion = () => {
  userAnswers.value[currentQuestionIndex.value] = currentAnswer.value
  currentQuestionIndex.value++
  currentAnswer.value = userAnswers.value[currentQuestionIndex.value] || ''
}

const previousQuestion = () => {
  userAnswers.value[currentQuestionIndex.value] = currentAnswer.value
  currentQuestionIndex.value--
  currentAnswer.value = userAnswers.value[currentQuestionIndex.value] || ''
}

const submitTest = () => {
  userAnswers.value[currentQuestionIndex.value] = currentAnswer.value
  
  const correct = Math.floor(Math.random() * 3) + Math.floor(testOptions.value.questionCount * 0.6)
  const total = testOptions.value.questionCount
  const score = Math.round((correct / total) * 100)
  
  let readiness = 'Needs Improvement'
  if (score >= 90) readiness = 'Excellent - Ready for Exam!'
  else if (score >= 75) readiness = 'Good - Almost Ready'
  else if (score >= 60) readiness = 'Fair - More Practice Needed'
  
  const details = currentTest.value.questions.map((q, idx) => ({
    correct: Math.random() > 0.3,
    feedback: `Your answer demonstrates ${Math.random() > 0.5 ? 'a good' : 'some'} understanding. ${Math.random() > 0.5 ? 'Consider reviewing the related concepts.' : 'Great job on this question!'}`
  }))
  
  testResults.value = {
    score,
    correct,
    total,
    readiness,
    details
  }
}

const resetTest = () => {
  currentTest.value = null
  currentQuestionIndex.value = 0
  currentAnswer.value = ''
  userAnswers.value = []
  testResults.value = null
  testOptions.value = {
    questionCount: 10,
    difficulty: 'medium',
    topic: ''
  }
}

defineExpose({ resetTest })
</script>

<style scoped>
.testing-mode {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  min-height: 0;
}

.test-setup {
  background: rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  text-align: center;
  backdrop-filter: blur(10px);
}

.test-setup h2 {
  color: #ffffff;
  margin-bottom: 0.75rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.testing-icon {
  width: 30px;
  height: 30px;
  filter: brightness(0) saturate(100%) invert(100%) sepia(0%) saturate(0%) hue-rotate(0deg) brightness(100%) contrast(100%);
}

.test-setup p {
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.95rem;
}

.test-options {
  max-width: 500px;
  margin: 1rem auto;
  text-align: left;
}

.option-group {
  margin-bottom: 1rem;
}

.option-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #ffffff;
  font-weight: 600;
}

.option-group select,
.option-group input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.9);
  color: #2c3e50;
  outline: none;
}

.option-group select:focus,
.option-group input:focus {
  border-color: rgba(255, 255, 255, 0.6);
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
}

.active-test {
  background: rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
  height: 100%;
}

.test-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
  flex-shrink: 0;
}

.test-header h2 {
  color: #ffffff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 1.3rem;
}

.test-progress {
  color: rgba(255, 255, 255, 0.9);
  font-weight: 600;
  font-size: 0.9rem;
}

.question-card {
  background: rgba(255, 255, 255, 0.25);
  padding: 1.5rem;
  border-radius: 12px;
  margin-bottom: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.question-card h3 {
  color: #ffffff;
  margin-bottom: 0.75rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 1.1rem;
  flex-shrink: 0;
}

.question-text {
  font-size: 1.05rem;
  color: #ffffff;
  margin-bottom: 1rem;
  line-height: 1.5;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  flex-shrink: 0;
}

.answer-options {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  flex: 1;
  overflow-y: auto;
}

.option {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1rem;
  background: rgba(255, 255, 255, 0.8);
  border: 2px solid rgba(255, 255, 255, 0.5);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  color: #2c3e50;
  font-size: 0.95rem;
  flex-shrink: 0;
}

.option:hover {
  border-color: rgba(255, 255, 255, 0.9);
  transform: translateX(5px);
  box-shadow: 0 4px 20px rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.95);
}

.option.selected {
  background: rgba(255, 255, 255, 0.95);
  border-color: #6d28d9;
  font-weight: 600;
  box-shadow: 0 0 25px rgba(109, 40, 217, 0.3);
}

.option-letter {
  width: 32px;
  height: 32px;
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  flex-shrink: 0;
}

.answer-input textarea {
  width: 100%;
  padding: 1rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  font-family: inherit;
  font-size: 1rem;
  resize: vertical;
  background: rgba(255, 255, 255, 0.9);
  color: #2c3e50;
  outline: none;
}

.answer-input textarea:focus {
  border-color: rgba(255, 255, 255, 0.6);
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
}

.test-navigation {
  display: flex;
  gap: 0.75rem;
  justify-content: space-between;
  flex-shrink: 0;
  margin-top: 1rem;
}

/* Test Results */
.test-results {
  background: rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  overflow-y: auto;
  height: 100%;
}

.results-header {
  text-align: center;
  margin-bottom: 1rem;
}

.results-header h2 {
  color: #ffffff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 1.5rem;
}

.score-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: linear-gradient(135deg, #8b5cf6, #06b6d4);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 1rem auto;
  box-shadow: 0 10px 40px rgba(139, 92, 246, 0.5);
}

.score {
  font-size: 2.5rem;
  color: white;
  font-weight: bold;
}

.results-summary {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.stat {
  background: rgba(255, 255, 255, 0.25);
  padding: 1rem;
  border-radius: 8px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.stat-label {
  display: block;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 0.3rem;
  font-size: 0.9rem;
}

.stat-value {
  display: block;
  font-size: 1.3rem;
  font-weight: bold;
  color: #ffffff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.detailed-feedback {
  margin-top: 1rem;
}

.detailed-feedback h3 {
  margin-bottom: 0.75rem;
  color: #ffffff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 1.1rem;
}

.feedback-item {
  background: rgba(255, 255, 255, 0.25);
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.feedback-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.question-num {
  font-weight: bold;
  color: #ffffff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
}

.result-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
}

.result-badge.correct {
  background: rgba(16, 185, 129, 0.9);
  color: white;
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);
}

.result-badge.incorrect {
  background: rgba(239, 68, 68, 0.9);
  color: white;
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
}

.feedback-text {
  color: rgba(255, 255, 255, 0.9);
  line-height: 1.5;
  font-size: 0.9rem;
}

/* Buttons */
button {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 600;
  transition: all 0.3s;
}

button:hover:not(:disabled) {
  transform: translateY(-2px);
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.4);
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.8);
  color: #6d28d9;
  border: 2px solid rgba(255, 255, 255, 0.5);
}

.btn-large {
  padding: 0.75rem 2rem;
  font-size: 1.05rem;
}

@media (max-width: 768px) {
  .test-navigation {
    flex-direction: column;
  }
}
</style>
