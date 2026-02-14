<template>
  <div class="chat-container">
    <div class="messages-area" ref="messagesArea">
      <div v-if="messages.length === 0" class="welcome-message">
        <div class="welcome-icon-wrapper" v-if="welcomeIcon">
          <img :src="welcomeIcon" class="welcome-icon" :alt="welcomeTitle" />
        </div>
        <h3>{{ welcomeTitle }}</h3>
        <p class="welcome-sub">{{ welcomeSubtitle }}</p>
        <slot name="welcome-extra"></slot>
      </div>

      <div 
        v-for="(message, index) in messages" 
        :key="index"
        :class="['message', message.sender]"
      >
        <div class="avatar" v-if="message.sender !== 'user'">
          <img :src="base + 'teacher-svgrepo-com.svg'" class="avatar-icon" alt="AI" />
        </div>
        <div class="message-content">
          <span class="msg-label" v-if="message.sender === 'user'">You</span>
          <span class="msg-label" v-else>{{ botLabel }}</span>
          <p>{{ message.text }}</p>
        </div>
        <div class="avatar user-avatar" v-if="message.sender === 'user'">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
            <circle cx="12" cy="7" r="4"></circle>
          </svg>
        </div>
      </div>

      <div v-if="isLoading" :class="['message', botClass]">
        <div class="avatar">
          <img :src="base + 'teacher-svgrepo-com.svg'" class="avatar-icon" alt="AI" />
        </div>
        <div class="message-content">
          <span class="msg-label">{{ botLabel }}</span>
          <div class="typing-indicator">
            <span></span><span></span><span></span>
          </div>
        </div>
      </div>
    </div>

    <div class="input-area">
      <div class="input-wrapper">
        <textarea 
          v-model="inputText"
          @keydown.enter.prevent="handleSend"
          :placeholder="placeholder"
          rows="1"
          ref="textareaRef"
          @input="autoResize"
        ></textarea>
        <button @click="handleSend" class="btn-send" :disabled="!inputText.trim() || isLoading">
          <img :src="base + 'send-alt-1-svgrepo-com.svg'" class="send-icon" alt="Send" />
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const base = import.meta.env.BASE_URL

const props = defineProps({
  messages: {
    type: Array,
    required: true
  },
  isLoading: {
    type: Boolean,
    default: false
  },
  placeholder: {
    type: String,
    default: 'Type a message...'
  },
  welcomeTitle: {
    type: String,
    default: 'Welcome!'
  },
  welcomeSubtitle: {
    type: String,
    default: ''
  },
  welcomeIcon: {
    type: String,
    default: ''
  },
  botLabel: {
    type: String,
    default: 'AI'
  },
  botClass: {
    type: String,
    default: 'bot'
  }
})

const emit = defineEmits(['send'])
const inputText = ref('')
const messagesArea = ref(null)
const textareaRef = ref(null)

const handleSend = () => {
  if (!inputText.value.trim() || props.isLoading) return
  emit('send', inputText.value)
  inputText.value = ''
  if (textareaRef.value) {
    textareaRef.value.style.height = 'auto'
  }
}

const autoResize = () => {
  const el = textareaRef.value
  if (el) {
    el.style.height = 'auto'
    el.style.height = Math.min(el.scrollHeight, 120) + 'px'
  }
}

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesArea.value) {
      messagesArea.value.scrollTop = messagesArea.value.scrollHeight
    }
  })
}

defineExpose({ scrollToBottom })
</script>

<style scoped>
.chat-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  background: linear-gradient(180deg, rgba(255,255,255,0.05) 0%, rgba(255,255,255,0.12) 100%);
  border-radius: 16px;
  overflow: hidden;
}

.messages-area {
  flex: 1;
  overflow-y: auto;
  padding: 1.25rem;
  min-height: 0;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

/* Welcome Screen */
.welcome-message {
  text-align: center;
  padding: 2rem 1rem;
  color: rgba(255, 255, 255, 0.95);
  margin: auto 0;
}

.welcome-icon-wrapper {
  width: 64px;
  height: 64px;
  border-radius: 20px;
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.5), rgba(6, 182, 212, 0.4));
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 8px 32px rgba(139, 92, 246, 0.25);
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}

.welcome-icon {
  width: 32px;
  height: 32px;
  filter: brightness(0) saturate(100%) invert(100%);
}

.welcome-message h3 {
  color: #ffffff;
  margin-bottom: 0.4rem;
  font-size: 1.3rem;
  font-weight: 700;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.welcome-sub {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.95rem;
  margin-bottom: 0.5rem;
}

/* Messages */
.message {
  display: flex;
  align-items: flex-start;
  gap: 0.6rem;
  animation: msgIn 0.3s ease;
}

@keyframes msgIn {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.message.user {
  flex-direction: row;
  justify-content: flex-end;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 10px;
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.6), rgba(6, 182, 212, 0.5));
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.25);
}

.user-avatar {
  background: linear-gradient(135deg, #8b5cf6, #6d28d9);
}

.avatar-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(100%);
}

.send-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(100%);
}

.message-content {
  padding: 0.75rem 1rem;
  border-radius: 16px;
  max-width: 75%;
  font-size: 0.93rem;
  line-height: 1.55;
  position: relative;
}

.message.user .message-content {
  background: linear-gradient(135deg, #8b5cf6, #7c3aed);
  color: #ffffff;
  border-bottom-right-radius: 4px;
  box-shadow: 0 2px 12px rgba(139, 92, 246, 0.3);
}

.message.bot .message-content,
.message.ai .message-content {
  background: rgba(255, 255, 255, 0.18);
  border: 1px solid rgba(255, 255, 255, 0.25);
  color: #ffffff;
  border-bottom-left-radius: 4px;
  backdrop-filter: blur(8px);
}

.msg-label {
  display: block;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  opacity: 0.65;
  margin-bottom: 0.25rem;
}

.message-content p {
  margin: 0;
}

/* Typing */
.typing-indicator {
  display: flex;
  gap: 0.35rem;
  padding: 0.25rem 0;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 50%;
  animation: bounce 1.4s infinite;
}

.typing-indicator span:nth-child(2) { animation-delay: 0.2s; }
.typing-indicator span:nth-child(3) { animation-delay: 0.4s; }

/* Input Area */
.input-area {
  padding: 0.75rem 1rem;
  background: rgba(255, 255, 255, 0.08);
  border-top: 1px solid rgba(255, 255, 255, 0.12);
  flex-shrink: 0;
}

.input-wrapper {
  display: flex;
  align-items: flex-end;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 14px;
  padding: 0.4rem 0.4rem 0.4rem 1rem;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(139, 92, 246, 0.15);
  transition: border-color 0.3s, box-shadow 0.3s;
}

.input-wrapper:focus-within {
  border-color: rgba(139, 92, 246, 0.4);
  box-shadow: 0 2px 20px rgba(139, 92, 246, 0.12);
}

.input-wrapper textarea {
  flex: 1;
  padding: 0.5rem 0;
  border: none;
  resize: none;
  font-family: inherit;
  font-size: 0.93rem;
  background: transparent;
  color: #2c3e50;
  outline: none;
  line-height: 1.45;
  max-height: 120px;
}

.input-wrapper textarea::placeholder {
  color: #a0aec0;
}

.btn-send {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  flex-shrink: 0;
}

.btn-send:hover:not(:disabled) {
  transform: scale(1.05);
  box-shadow: 0 4px 16px rgba(109, 40, 217, 0.4);
}

.btn-send:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

@keyframes bounce {
  0%, 60%, 100% { transform: translateY(0); }
  30% { transform: translateY(-10px); }
}

@media (max-width: 768px) {
  .message-content {
    max-width: 85%;
  }
}
</style>
