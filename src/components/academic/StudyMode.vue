<template>
  <div class="study-mode">
    <ChatWindow
      ref="chatWindow"
      :messages="studyMessages"
      :is-loading="isLoading"
      placeholder="Ask a question about this book..."
      welcome-title="Welcome to Study Mode!"
      :welcome-subtitle="`Ask me anything about \u0022${book.customName || book.name}\u0022`"
      :welcome-icon="'/book-heart-love-svgrepo-com.svg'"
      bot-label="AI Tutor"
      bot-class="ai"
      @send="askQuestion"
    >
      <template #welcome-extra>
        <div class="suggestions">
          <button 
            v-for="(q, i) in quickQuestions" 
            :key="i" 
            class="suggestion-chip"
            @click="askQuestion(q)"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="12" r="10"></circle>
              <line x1="12" y1="8" x2="12" y2="12"></line>
              <line x1="12" y1="16" x2="12.01" y2="16"></line>
            </svg>
            {{ q }}
          </button>
        </div>
      </template>
    </ChatWindow>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import ChatWindow from '../ChatWindow.vue'

const props = defineProps({
  book: {
    type: Object,
    required: true
  }
})

const studyMessages = ref([])
const isLoading = ref(false)
const chatWindow = ref(null)

const quickQuestions = computed(() => [
  `Summarize "${props.book.customName || props.book.name}"`,
  'Explain the main concepts',
  'Give me key takeaways',
  'Quiz me on this material'
])

const askQuestion = (questionText) => {
  studyMessages.value.push({
    sender: 'user',
    text: questionText
  })

  isLoading.value = true

  setTimeout(() => {
    studyMessages.value.push({
      sender: 'ai',
      text: generateAIResponse(questionText)
    })
    isLoading.value = false

    setTimeout(() => {
      chatWindow.value?.scrollToBottom()
    }, 100)
  }, 1500)
}

const generateAIResponse = (question) => {
  const responses = [
    `Great question! Based on "${props.book.name}", let me explain: This concept is fundamental because it establishes the foundation for understanding the broader topic. The key points to remember are: 1) The relationship between the components, 2) How they interact in practical scenarios, and 3) The implications for real-world applications.`,
    `Excellent! In "${props.book.name}", this topic is covered in detail. The main idea is that we need to understand the underlying principles before we can apply them effectively. Think of it as building blocks - each concept builds on the previous one.`,
    `That's an important question about "${props.book.name}". To understand this better, consider the following: The author emphasizes that this principle works because of the interconnected nature of the subject matter. Let me break it down into simpler terms...`
  ]
  return responses[Math.floor(Math.random() * responses.length)]
}
</script>

<style scoped>
.study-mode {
  background: rgba(255, 255, 255, 0.12);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  overflow: hidden;
  backdrop-filter: blur(20px);
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
}

.suggestions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  justify-content: center;
  max-width: 550px;
  margin: 1rem auto 0;
}

.suggestion-chip {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 1rem;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.82rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(8px);
}

.suggestion-chip:hover {
  background: rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.5);
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  color: #ffffff;
}

.suggestion-chip svg {
  opacity: 0.7;
}

@media (max-width: 768px) {
  .suggestions {
    padding: 0 0.5rem;
  }
  
  .suggestion-chip {
    font-size: 0.78rem;
    padding: 0.4rem 0.8rem;
  }
}
</style>
