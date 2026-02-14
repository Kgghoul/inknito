<template>
  <div class="section-content">
    <div class="section-header">
      <h2>Interview Practice</h2>
    </div>

    <div v-if="!interviewStarted" class="interview-setup">
      <h3>Prepare for Your Interview</h3>
      <p>Practice with our AI interviewer tailored to your field</p>
      
      <div class="form-grid">
        <div class="form-group">
          <label>Job Role</label>
          <input 
            v-model="interviewSetup.role" 
            type="text" 
            class="form-input"
            placeholder="e.g., Software Engineer, Marketing Manager"
          />
        </div>
        <div class="form-group">
          <label>Experience Level</label>
          <select v-model="interviewSetup.level" class="form-input">
            <option value="entry">Entry Level</option>
            <option value="mid">Mid Level</option>
            <option value="senior">Senior Level</option>
          </select>
        </div>
        <div class="form-group full-width">
          <label>Company/Industry (Optional)</label>
          <input 
            v-model="interviewSetup.company" 
            type="text" 
            class="form-input"
            placeholder="e.g., Tech Startup, Finance, Healthcare"
          />
        </div>
      </div>

      <button @click="startInterview" class="btn-primary btn-large">
        Start Interview Practice
      </button>
    </div>

    <div v-else class="interview-session">
      <div class="interview-progress">
        <span>Question {{ currentQuestion + 1 }} of {{ interviewQuestions.length }}</span>
        <button @click="endInterview" class="btn-end">End Interview</button>
      </div>

      <div class="interview-content">
        <div class="interviewer-section">
          <div class="interviewer-avatar">👔</div>
          <div class="question-bubble">
            <p><strong>AI Interviewer:</strong></p>
            <p>{{ interviewQuestions[currentQuestion]?.question }}</p>
          </div>
        </div>

        <div class="answer-section">
          <label>Your Answer:</label>
          <textarea 
            v-model="currentAnswer"
            class="form-input"
            rows="6"
            placeholder="Type your answer here..."
            :disabled="isProcessingAnswer"
          ></textarea>
          <button 
            @click="submitAnswer" 
            class="btn-primary"
            :disabled="!currentAnswer.trim() || isProcessingAnswer"
          >
            {{ currentQuestion < interviewQuestions.length - 1 ? 'Submit & Next' : 'Submit & Finish' }}
          </button>
        </div>

        <div v-if="currentFeedback" class="answer-feedback">
          <h4>Feedback on Your Answer:</h4>
          <p>{{ currentFeedback }}</p>
        </div>
      </div>
    </div>

    <!-- Interview Results -->
    <div v-if="interviewResults" class="interview-results">
      <h3>Interview Performance</h3>
      
      <div class="results-overview">
        <div class="result-stat">
          <span class="stat-label">Overall Score</span>
          <span class="stat-value">{{ interviewResults.overallScore }}/100</span>
        </div>
        <div class="result-stat">
          <span class="stat-label">Communication</span>
          <span class="stat-value">{{ interviewResults.communication }}/10</span>
        </div>
        <div class="result-stat">
          <span class="stat-label">Technical Knowledge</span>
          <span class="stat-value">{{ interviewResults.technical }}/10</span>
        </div>
        <div class="result-stat">
          <span class="stat-label">Confidence</span>
          <span class="stat-value">{{ interviewResults.confidence }}/10</span>
        </div>
      </div>

      <div class="results-details">
        <h4>Detailed Feedback</h4>
        <div 
          v-for="(result, index) in interviewResults.answers" 
          :key="index"
          class="result-item"
        >
          <h5>Q{{ index + 1 }}: {{ result.question }}</h5>
          <p class="answer-summary">{{ result.feedback }}</p>
        </div>
      </div>

      <button @click="resetInterview" class="btn-primary">
        Practice Again
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const interviewStarted = ref(false)
const interviewSetup = ref({
  role: '',
  level: 'entry',
  company: ''
})

const interviewQuestions = ref([])
const currentQuestion = ref(0)
const currentAnswer = ref('')
const currentFeedback = ref('')
const isProcessingAnswer = ref(false)
const interviewResults = ref(null)

const mockQuestions = [
  { question: "Tell me about yourself and your background." },
  { question: "Why are you interested in this position?" },
  { question: "What are your greatest strengths?" },
  { question: "Describe a challenging project you've worked on." },
  { question: "Where do you see yourself in 5 years?" }
]

const startInterview = () => {
  if (interviewSetup.value.role) {
    interviewQuestions.value = mockQuestions
    interviewStarted.value = true
    currentQuestion.value = 0
    currentAnswer.value = ''
    currentFeedback.value = ''
    interviewResults.value = null
  }
}

const submitAnswer = () => {
  if (!currentAnswer.value.trim()) return

  isProcessingAnswer.value = true
  
  setTimeout(() => {
    currentFeedback.value = "Good answer! Try to be more specific with examples and quantify your achievements when possible."
    isProcessingAnswer.value = false
    
    setTimeout(() => {
      if (currentQuestion.value < interviewQuestions.value.length - 1) {
        currentQuestion.value++
        currentAnswer.value = ''
        currentFeedback.value = ''
      } else {
        finishInterview()
      }
    }, 2000)
  }, 1500)
}

const finishInterview = () => {
  interviewResults.value = {
    overallScore: 78,
    communication: 8,
    technical: 7,
    confidence: 8,
    answers: interviewQuestions.value.map(q => ({
      question: q.question,
      feedback: "Your answer demonstrated good understanding. Consider providing more specific examples."
    }))
  }
  interviewStarted.value = false
}

const endInterview = () => {
  if (confirm('Are you sure you want to end the interview?')) {
    resetInterview()
  }
}

const resetInterview = () => {
  interviewStarted.value = false
  currentQuestion.value = 0
  currentAnswer.value = ''
  currentFeedback.value = ''
  interviewResults.value = null
}
</script>

<style scoped>
.section-content {
  flex: 1;
  overflow-y: auto;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  flex-shrink: 0;
}

.section-header h2 {
  color: #ffffff;
  font-size: 1.5rem;
}

.interview-setup {
  max-width: 600px;
  margin: 0 auto;
  width: 100%;
}

.interview-setup h3 {
  color: #ffffff;
  margin-bottom: 0.5rem;
  text-align: center;
}

.interview-setup p {
  color: rgba(255, 255, 255, 0.85);
  text-align: center;
  margin-bottom: 1.5rem;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-group.full-width {
  grid-column: 1 / -1;
}

.form-group label {
  display: block;
  color: #ffffff;
  margin-bottom: 0.5rem;
  font-weight: 600;
  font-size: 0.9rem;
}

.form-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.9);
  color: #2c3e50;
  font-family: inherit;
  font-size: 0.95rem;
  outline: none;
}

.form-input:focus {
  border-color: rgba(255, 255, 255, 0.6);
}

.btn-primary {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
  font-size: 0.95rem;
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  width: 100%;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(139, 92, 246, 0.4);
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-large {
  padding: 1rem 2rem;
  font-size: 1.05rem;
}

/* Interview */
.interview-progress {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  margin-bottom: 1.5rem;
  color: #ffffff;
  font-weight: 600;
}

.btn-end {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
  border: 2px solid #ef4444;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}

.interviewer-section {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.interviewer-avatar {
  font-size: 3rem;
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.question-bubble {
  background: rgba(255, 255, 255, 0.9);
  padding: 1.5rem;
  border-radius: 12px;
  flex: 1;
  color: #2c3e50;
}

.answer-section {
  margin-bottom: 2rem;
}

.answer-section label {
  display: block;
  color: #ffffff;
  margin-bottom: 0.5rem;
  font-weight: 600;
}

.answer-feedback {
  background: rgba(6, 182, 212, 0.2);
  border: 2px solid #06b6d4;
  padding: 1rem;
  border-radius: 8px;
  color: #ffffff;
}

.interview-results {
  max-width: 800px;
  margin: 0 auto;
}

.interview-results h3 {
  color: #ffffff;
  text-align: center;
  margin-bottom: 2rem;
  font-size: 1.5rem;
}

.results-overview {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.result-stat {
  background: rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
  border-radius: 12px;
  text-align: center;
}

.stat-label {
  display: block;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.stat-value {
  display: block;
  color: #ffffff;
  font-size: 1.8rem;
  font-weight: bold;
}

.results-details h4 {
  color: #ffffff;
  margin-bottom: 1rem;
}

.result-item {
  background: rgba(255, 255, 255, 0.1);
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
}

.result-item h5 {
  color: #ffffff;
  margin-bottom: 0.5rem;
  font-size: 0.95rem;
}

.answer-summary {
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.9rem;
  line-height: 1.5;
}

@media (max-width: 768px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>
