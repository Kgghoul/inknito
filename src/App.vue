<script setup>
import { ref } from 'vue'
import AcademicSuccess from './components/AcademicSuccess.vue'
import WellbeingMotivation from './components/WellbeingMotivation.vue'
import CareerFutureReadiness from './components/CareerFutureReadiness.vue'
import Profile from './components/Profile.vue'

const currentSection = ref('academic')

const sections = [
  { id: 'academic', name: 'Academic Success', icon: '' },
  { id: 'wellbeing', name: 'Wellbeing and Motivation', icon: '' },
  { id: 'career', name: 'Career and Future Readiness', icon: '' },
  { id: 'profile', name: 'Profile', icon: '' }
]

const setSection = (sectionId) => {
  currentSection.value = sectionId
}
</script>

<template>
  <div id="app">
    <header class="main-header">
      <div class="header-content">
        <div class="logo">
          <span class="logo-text">Inknito</span>
        </div>
        <nav class="main-nav">
          <button
            v-for="section in sections"
            :key="section.id"
            @click="setSection(section.id)"
            :class="['nav-item', { active: currentSection === section.id }]"
          >
            <span class="nav-icon">{{ section.icon }}</span>
            <span class="nav-text">{{ section.name }}</span>
          </button>
        </nav>
      </div>
    </header>

    <main class="main-content">
      <AcademicSuccess v-if="currentSection === 'academic'" />
      <WellbeingMotivation v-else-if="currentSection === 'wellbeing'" />
      <CareerFutureReadiness v-else-if="currentSection === 'career'" />
      <Profile v-else-if="currentSection === 'profile'" />
      
      <div v-else class="coming-soon">
        <h2>{{ sections.find(s => s.id === currentSection)?.name }}</h2>
        <p>Coming Soon...</p>
      </div>
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

button, input, select, textarea {
  outline: none;
}

button:focus, input:focus, select:focus, textarea:focus {
  outline: none;
}

#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background: url('/academicsuccessbg.jpeg') center/cover no-repeat fixed;
  min-height: 100vh;
  height: 100vh;
  padding: 0;
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

#app::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.25);
  pointer-events: none;
  z-index: 0;
}

body {
  margin: 0;
  background: url('/academicsuccessbg.jpeg') center/cover no-repeat fixed;
}

/* Header */
.main-header {
  position: relative;
  z-index: 10;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  flex-shrink: 0;
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0.75rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 2rem;
}

.logo {
  flex-shrink: 0;
}

.logo-text {
  font-family: 'Showpop', sans-serif;
  font-size: 1.8rem;
  color: #ffffff;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.main-nav {
  display: flex;
  gap: 0.5rem;
  flex: 1;
  justify-content: center;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.2rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  color: #ffffff;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 0.95rem;
  font-weight: 600;
  white-space: nowrap;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(255, 255, 255, 0.2);
}

.nav-item.active {
  background: rgba(255, 255, 255, 0.9);
  color: #6d28d9;
  border-color: rgba(255, 255, 255, 0.9);
  box-shadow: 0 4px 20px rgba(255, 255, 255, 0.3);
}

.nav-item:focus {
  outline: none;
}

.nav-icon {
  font-size: 1.2rem;
}

.nav-text {
  display: none;
}

@media (min-width: 768px) {
  .nav-text {
    display: inline;
}
}

/* Main Content */
.main-content {
  flex: 1;
  overflow: hidden;
  position: relative;
  z-index: 1;
}

/* Coming Soon */
.coming-soon {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  text-align: center;
  color: #ffffff;
  padding: 2rem;
}

.coming-soon h2 {
  font-family: 'Showpop', sans-serif;
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.coming-soon p {
  font-size: 1.5rem;
  opacity: 0.9;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* Responsive */
@media (max-width: 768px) {
  .header-content {
    padding: 0.5rem 1rem;
    flex-direction: column;
    gap: 0.75rem;
  }
  
  .main-nav {
    width: 100%;
    justify-content: space-around;
  }
  
  .nav-item {
    padding: 0.5rem 0.75rem;
    font-size: 0.85rem;
  }
  
  .logo-text {
    font-size: 1.5rem;
  }
}
</style>
