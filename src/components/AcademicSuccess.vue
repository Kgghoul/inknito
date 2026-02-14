<template>
  <div class="academic-success">
    <div class="header">
      <h1>Academic Success</h1>
      <p class="subtitle">AI-powered learning tools for better grades</p>
    </div>

    <!-- Book Library (shown when no book selected) -->
    <BookLibrary v-if="!selectedBook" @select-book="selectBook" />

    <!-- Learning Interface (shown when book is selected) -->
    <div v-else class="learning-interface">
      <div class="current-book-header">
        <button @click="selectedBook = null" class="btn-back">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="15 18 9 12 15 6"></polyline>
          </svg>
          Library
        </button>

        <div class="book-badge">
          <div class="badge-icon-wrapper">
            <img src="/book-heart-love-svgrepo-com.svg" class="badge-icon" alt="Book" />
          </div>
          <span class="badge-text">{{ truncateBookName(selectedBook.customName || selectedBook.name) }}</span>
        </div>

        <div class="mode-selector">
          <button 
            :class="['mode-btn', { active: mode === 'study' }]"
            @click="mode = 'study'"
          >
            <img src="/book-heart-love-svgrepo-com.svg" class="mode-svg-icon" alt="Study" />
            <span class="mode-label">Study</span>
          </button>
          <button 
            :class="['mode-btn', { active: mode === 'test' }]"
            @click="mode = 'test'"
          >
            <img src="/study-svgrepo-com.svg" class="mode-svg-icon" alt="Test" />
            <span class="mode-label">Test</span>
          </button>
        </div>
      </div>

      <transition name="mode-fade" mode="out-in">
        <StudyMode v-if="mode === 'study'" :book="selectedBook" key="study" />
        <TestingMode v-else :book="selectedBook" key="test" />
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import BookLibrary from './academic/BookLibrary.vue'
import StudyMode from './academic/StudyMode.vue'
import TestingMode from './academic/TestingMode.vue'

const selectedBook = ref(null)
const mode = ref('study')

const selectBook = (book) => {
  selectedBook.value = book
  mode.value = 'study'
}

const truncateBookName = (name) => {
  const maxLength = 30
  if (name.length <= maxLength) return name
  return name.substring(0, maxLength) + '...'
}
</script>

<style scoped>
.academic-success {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1.5rem;
  position: relative;
  z-index: 1;
  height: 100%;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.header {
  text-align: center;
  margin-bottom: 1.5rem;
  flex-shrink: 0;
}

.header h1 {
  font-family: 'Showpop', sans-serif;
  font-size: 2.5rem;
  color: #ffffff;
  margin-bottom: 0.3rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.subtitle {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* Learning Interface */
.learning-interface {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  gap: 0.75rem;
}

.current-book-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.6rem 0.75rem;
  background: linear-gradient(135deg, rgba(109, 40, 217, 0.35) 0%, rgba(6, 182, 212, 0.25) 100%);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(20px);
  flex-shrink: 0;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.1), inset 0 1px 0 rgba(255, 255, 255, 0.15);
}

/* Back Button */
.btn-back {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.5rem 0.9rem;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  flex-shrink: 0;
}

.btn-back:hover {
  background: rgba(255, 255, 255, 0.25);
  transform: translateX(-2px);
  color: #ffffff;
}

/* Book Badge */
.book-badge {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.4rem 1rem 0.4rem 0.4rem;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24px;
  box-shadow: 0 2px 12px rgba(109, 40, 217, 0.2);
  flex-shrink: 1;
  min-width: 0;
}

.badge-icon-wrapper {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: linear-gradient(135deg, #8b5cf6, #6d28d9);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.badge-icon {
  width: 18px;
  height: 18px;
  filter: brightness(0) saturate(100%) invert(100%);
}

.badge-text {
  font-weight: 700;
  font-size: 0.9rem;
  color: #4c1d95;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Mode Selector */
.mode-selector {
  display: flex;
  gap: 0.4rem;
  margin-left: auto;
  background: rgba(0, 0, 0, 0.15);
  border-radius: 12px;
  padding: 0.3rem;
  flex-shrink: 0;
}

.mode-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.55rem 1.2rem;
  border: none;
  background: transparent;
  color: rgba(255, 255, 255, 0.75);
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 0.9rem;
  font-weight: 600;
  position: relative;
}

.mode-btn:hover:not(.active) {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}

.mode-btn.active {
  background: rgba(255, 255, 255, 0.95);
  color: #6d28d9;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
}

.mode-btn.active .mode-svg-icon {
  filter: brightness(0) saturate(100%) invert(21%) sepia(95%) saturate(3095%) hue-rotate(262deg) brightness(68%) contrast(97%);
}

.mode-btn:focus {
  outline: none;
}

.mode-svg-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(100%);
  transition: filter 0.3s;
}

.mode-label {
  font-size: 0.9rem;
}

/* Transition */
.mode-fade-enter-active,
.mode-fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}
.mode-fade-enter-from {
  opacity: 0;
  transform: translateY(8px);
}
.mode-fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

@media (max-width: 768px) {
  .academic-success {
    padding: 1rem;
  }
  
  .header h1 {
    font-size: 2rem;
  }

  .current-book-header {
    flex-wrap: wrap;
    gap: 0.5rem;
  }
  
  .mode-selector {
    margin-left: 0;
    width: 100%;
    justify-content: center;
  }

  .book-badge {
    flex: 1;
    min-width: 0;
  }
}
</style>
