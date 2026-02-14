<template>
  <div class="career-readiness">
    <div class="header">
      <h1>Career and Future Readiness</h1>
      <p class="subtitle">Build your professional future with confidence</p>
    </div>

    <!-- Main Tabs -->
    <div class="tabs-container">
      <button 
        v-for="tab in tabs" 
        :key="tab.id"
        @click="activeTab = tab.id"
        :class="['tab-btn', { active: activeTab === tab.id }]"
      >
        <img :src="tab.icon" class="tab-icon" :alt="tab.name" />
        {{ tab.name }}
      </button>
    </div>

    <!-- Tab Content -->
    <EventsFinder v-if="activeTab === 'events'" />
    <ResumeBuilder v-else-if="activeTab === 'resume'" />
    <InterviewPractice v-else-if="activeTab === 'interview'" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import EventsFinder from './career/EventsFinder.vue'
import ResumeBuilder from './career/ResumeBuilder.vue'
import InterviewPractice from './career/InterviewPractice.vue'

const tabs = [
  { id: 'events', name: 'Professional Events', icon: '/find-location-svgrepo-com.svg' },
  { id: 'resume', name: 'Resume Builder', icon: '/clipboard-research-investigate-study-svgrepo-com.svg' },
  { id: 'interview', name: 'Interview Practice', icon: '/study-svgrepo-com.svg' }
]

const activeTab = ref('events')
</script>

<style scoped>
.career-readiness {
  max-width: 1400px;
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
  font-size: 2rem;
  color: #ffffff;
  margin-bottom: 0.3rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.subtitle {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.tabs-container {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  flex-shrink: 0;
  justify-content: center;
}

.tab-btn {
  padding: 0.75rem 1.5rem;
  background: rgba(255, 255, 255, 0.15);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  color: #ffffff;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 0.95rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.tab-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(100%) sepia(0%) saturate(0%) hue-rotate(0deg) brightness(100%) contrast(100%);
}

.tab-btn:hover {
  background: rgba(255, 255, 255, 0.25);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
}

.tab-btn.active {
  background: rgba(255, 255, 255, 0.9);
  color: #6d28d9;
  border-color: rgba(255, 255, 255, 0.9);
}

.tab-btn.active .tab-icon {
  filter: brightness(0) saturate(100%) invert(21%) sepia(95%) saturate(3095%) hue-rotate(262deg) brightness(68%) contrast(97%);
}

@media (max-width: 1024px) {
  .tabs-container {
    flex-wrap: wrap;
  }

  .tab-btn {
    font-size: 0.85rem;
    padding: 0.6rem 1rem;
  }
}

@media (max-width: 768px) {
  .career-readiness {
    padding: 1rem;
  }

  .header h1 {
    font-size: 1.5rem;
  }
}
</style>
