<template>
  <div class="section-content">
    <div class="section-header">
      <div class="section-title-group">
        <div class="section-icon-wrapper">
          <img src="/event-svgrepo-com.svg" class="section-icon" alt="Events" />
        </div>
        <div>
          <h2>Professional Events</h2>
          <p class="section-tagline">AI-curated events based on your profile</p>
        </div>
      </div>
      <button v-if="eventsProfileComplete" @click="refreshEvents" class="btn-refresh" :disabled="isLoadingEvents">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" :class="{ spinning: isLoadingEvents }">
          <polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/>
          <path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/>
        </svg>
        Refresh
      </button>
    </div>

    <!-- User Profile for Events -->
    <div v-if="!eventsProfileComplete" class="profile-setup">
      <div class="setup-icon-wrapper">
        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>
        </svg>
      </div>
      <h3>Tell us about yourself</h3>
      <p class="setup-description">Help our AI find the best events tailored to your career goals</p>
      <div class="form-grid">
        <div class="form-group">
          <label>
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"/><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"/></svg>
            Specialization
          </label>
          <input 
            v-model="eventsProfile.specialization" 
            type="text" 
            placeholder="e.g., Computer Science, Marketing"
            class="form-input"
          />
        </div>
        <div class="form-group">
          <label>
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 20h9"/><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>
            Experience Level
          </label>
          <select v-model="eventsProfile.experience" class="form-input">
            <option value="">Select level</option>
            <option value="beginner">Beginner</option>
            <option value="intermediate">Intermediate</option>
            <option value="advanced">Advanced</option>
          </select>
        </div>
        <div class="form-group full-width">
          <label>
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>
            Interests
          </label>
          <input 
            v-model="eventsProfile.interests" 
            type="text" 
            placeholder="e.g., Web Development, Data Science, Networking"
            class="form-input"
          />
        </div>
      </div>
      <button @click="saveEventsProfile" class="btn-primary">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        Save & Find Events
      </button>
    </div>

    <!-- Events List -->
    <div v-else class="events-content">
      <div v-if="isLoadingEvents" class="loading-state">
        <div class="spinner"></div>
        <p>AI is analyzing events for you...</p>
      </div>

      <div v-else class="events-grid">
        <div 
          v-for="event in recommendedEvents" 
          :key="event.id"
          class="event-card"
        >
          <div class="event-header">
            <span class="event-type">{{ event.type }}</span>
            <div class="ai-match">
              <div class="match-circle">
                <svg viewBox="0 0 36 36" class="match-ring">
                  <path class="ring-bg" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                  <path class="ring-fill" :stroke-dasharray="`${event.match}, 100`" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                </svg>
                <span class="match-value">{{ event.match }}%</span>
              </div>
              <span class="match-label">Match</span>
            </div>
          </div>
          <h3>{{ event.name }}</h3>
          <p class="event-organizer">
            <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
            {{ event.organizer }}
          </p>
          <div class="event-details">
            <div class="detail-chip">
              <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
              {{ event.date }}
            </div>
            <div class="detail-chip">
              <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>
              {{ event.location }}
            </div>
          </div>
          <p class="event-description">{{ event.description }}</p>
          <div class="event-actions">
            <button @click="attendEvent(event)" class="btn-attend">
              <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="20 6 9 17 4 12"/></svg>
              Attended
            </button>
            <button @click="interestedEvent(event)" class="btn-interested">
              <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg>
              Save
            </button>
          </div>
        </div>
      </div>

      <div class="attended-events" v-if="attendedEvents.length > 0">
        <h3>
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
          Event History
        </h3>
        <div class="history-list">
          <div 
            v-for="event in attendedEvents" 
            :key="event.id"
            class="history-item"
          >
            <div class="history-info">
              <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="20 6 9 17 4 12"/></svg>
              <span>{{ event.name }}</span>
            </div>
            <span class="event-date">{{ event.attendedDate }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const eventsProfileComplete = ref(false)
const eventsProfile = ref({
  specialization: '',
  experience: '',
  interests: ''
})

const isLoadingEvents = ref(false)
const attendedEvents = ref([])

const mockEvents = [
  {
    id: 1,
    name: 'Tech Career Fair 2026',
    organizer: 'Tech Companies Alliance',
    type: 'Career Fair',
    date: 'March 15, 2026',
    location: 'Online',
    description: 'Connect with top tech companies hiring for internships and full-time positions.',
    match: 95
  },
  {
    id: 2,
    name: 'AI & Machine Learning Workshop',
    organizer: 'DataScience Hub',
    type: 'Workshop',
    date: 'March 20, 2026',
    location: 'San Francisco, CA',
    description: 'Hands-on workshop covering latest trends in AI and practical applications.',
    match: 88
  },
  {
    id: 3,
    name: 'Startup Pitch Competition',
    organizer: 'Entrepreneurship Network',
    type: 'Competition',
    date: 'April 5, 2026',
    location: 'Boston, MA',
    description: 'Showcase your startup idea and compete for funding and mentorship opportunities.',
    match: 75
  },
  {
    id: 4,
    name: 'Women in Tech Summit',
    organizer: 'WomenTech Global',
    type: 'Conference',
    date: 'April 12, 2026',
    location: 'New York, NY',
    description: 'Annual summit celebrating and empowering women in technology fields.',
    match: 82
  }
]

const recommendedEvents = ref([])

const saveEventsProfile = () => {
  if (eventsProfile.value.specialization && eventsProfile.value.experience) {
    eventsProfileComplete.value = true
    isLoadingEvents.value = true
    
    setTimeout(() => {
      recommendedEvents.value = mockEvents
      isLoadingEvents.value = false
    }, 2000)
  }
}

const refreshEvents = () => {
  isLoadingEvents.value = true
  setTimeout(() => {
    recommendedEvents.value = [...mockEvents].sort(() => Math.random() - 0.5)
    isLoadingEvents.value = false
  }, 1500)
}

const attendEvent = (event) => {
  attendedEvents.value.push({
    ...event,
    attendedDate: new Date().toLocaleDateString()
  })
  alert(`Marked "${event.name}" as attended!`)
}

const interestedEvent = (event) => {
  alert(`Saved "${event.name}" to your interested list!`)
}
</script>

<style scoped>
.section-content {
  flex: 1;
  overflow-y: auto;
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.14) 0%, rgba(255, 255, 255, 0.06) 100%);
  backdrop-filter: blur(24px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.06);
}

/* Header */
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.25rem;
  flex-shrink: 0;
}

.section-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.section-icon-wrapper {
  width: 42px;
  height: 42px;
  border-radius: 12px;
  background: linear-gradient(135deg, rgba(6, 182, 212, 0.5), rgba(8, 145, 178, 0.6));
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border: 1px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 2px 12px rgba(6, 182, 212, 0.3);
}

.section-icon {
  width: 22px;
  height: 22px;
  filter: brightness(0) saturate(100%) invert(100%);
}

.section-header h2 {
  color: #ffffff;
  font-size: 1.2rem;
  font-weight: 700;
  margin: 0;
  line-height: 1.2;
}

.section-tagline {
  color: rgba(255, 255, 255, 0.65);
  font-size: 0.8rem;
  margin: 0.1rem 0 0;
}

.btn-refresh {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 1rem;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-refresh:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.22);
  transform: translateY(-1px);
}

.btn-refresh:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.spinning {
  animation: spin 1s linear infinite;
}

/* Profile Setup */
.profile-setup {
  max-width: 550px;
  margin: 1rem auto;
  width: 100%;
  text-align: center;
}

.setup-icon-wrapper {
  width: 64px;
  height: 64px;
  border-radius: 18px;
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.4), rgba(6, 182, 212, 0.35));
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1rem;
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 8px 32px rgba(139, 92, 246, 0.2);
}

.profile-setup h3 {
  color: #ffffff;
  margin-bottom: 0.3rem;
  font-size: 1.25rem;
  font-weight: 700;
}

.setup-description {
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 1.5rem;
  font-size: 0.9rem;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 0.85rem;
  margin-bottom: 1rem;
  text-align: left;
}

.form-group.full-width {
  grid-column: 1 / -1;
}

.form-group label {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 0.4rem;
  font-weight: 600;
  font-size: 0.85rem;
}

.form-group label svg {
  opacity: 0.7;
}

.form-input {
  width: 100%;
  padding: 0.7rem 0.85rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
  font-family: inherit;
  font-size: 0.9rem;
  outline: none;
  transition: all 0.3s;
  backdrop-filter: blur(8px);
}

.form-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.form-input:focus {
  border-color: rgba(139, 92, 246, 0.6);
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.15);
}

select.form-input {
  cursor: pointer;
}

select.form-input option {
  background: #2c3e50;
  color: white;
}

.btn-primary {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  width: 100%;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 700;
  font-size: 0.95rem;
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.3);
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 24px rgba(139, 92, 246, 0.45);
}

/* Loading */
.loading-state {
  text-align: center;
  padding: 3rem;
  color: rgba(255, 255, 255, 0.85);
}

.spinner {
  width: 44px;
  height: 44px;
  border: 3px solid rgba(255, 255, 255, 0.15);
  border-top-color: #8b5cf6;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Events Grid */
.events-content {
  flex: 1;
  overflow-y: auto;
}

.events-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.event-card {
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0.05) 100%);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 16px;
  padding: 1.2rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  flex-direction: column;
}

.event-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
  border-color: rgba(255, 255, 255, 0.35);
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.22) 0%, rgba(255, 255, 255, 0.08) 100%);
}

.event-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.75rem;
}

.event-type {
  background: rgba(6, 182, 212, 0.15);
  color: #67e8f9;
  padding: 0.3rem 0.75rem;
  border-radius: 8px;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border: 1px solid rgba(6, 182, 212, 0.3);
}

/* Match circle */
.ai-match {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.15rem;
}

.match-circle {
  position: relative;
  width: 40px;
  height: 40px;
}

.match-ring {
  width: 40px;
  height: 40px;
  transform: rotate(-90deg);
}

.ring-bg {
  fill: none;
  stroke: rgba(255, 255, 255, 0.1);
  stroke-width: 3;
}

.ring-fill {
  fill: none;
  stroke: #8b5cf6;
  stroke-width: 3;
  stroke-linecap: round;
  transition: stroke-dasharray 0.6s ease;
}

.match-value {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 0.65rem;
  font-weight: 800;
  color: #ffffff;
}

.match-label {
  font-size: 0.6rem;
  color: rgba(255, 255, 255, 0.55);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.event-card h3 {
  color: #ffffff;
  margin-bottom: 0.25rem;
  font-size: 1.05rem;
  font-weight: 700;
  line-height: 1.3;
}

.event-organizer {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.82rem;
  margin-bottom: 0.75rem;
  display: flex;
  align-items: center;
  gap: 0.35rem;
}

.event-organizer svg {
  opacity: 0.6;
}

.event-details {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
}

.detail-chip {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.8rem;
  background: rgba(255, 255, 255, 0.08);
  padding: 0.3rem 0.65rem;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.12);
}

.detail-chip svg {
  opacity: 0.65;
  flex-shrink: 0;
}

.event-description {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.85rem;
  line-height: 1.5;
  margin-bottom: 1rem;
  flex: 1;
}

.event-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-attend, .btn-interested {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  padding: 0.55rem 0.75rem;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 700;
  font-size: 0.82rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-attend {
  background: linear-gradient(135deg, #8b5cf6, #6d28d9);
  color: white;
  box-shadow: 0 2px 10px rgba(139, 92, 246, 0.25);
}

.btn-attend:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(139, 92, 246, 0.4);
}

.btn-interested {
  background: rgba(255, 255, 255, 0.12);
  color: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.25);
}

.btn-interested:hover {
  background: rgba(255, 255, 255, 0.22);
  transform: translateY(-2px);
}

/* Event History */
.attended-events {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.12);
}

.attended-events h3 {
  color: #ffffff;
  margin-bottom: 0.75rem;
  font-size: 1rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.attended-events h3 svg {
  opacity: 0.7;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.history-item {
  background: rgba(255, 255, 255, 0.08);
  padding: 0.65rem 0.85rem;
  border-radius: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #ffffff;
  font-size: 0.85rem;
  border: 1px solid rgba(255, 255, 255, 0.08);
  transition: background 0.2s;
}

.history-item:hover {
  background: rgba(255, 255, 255, 0.12);
}

.history-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.history-info svg {
  color: #34d399;
  flex-shrink: 0;
}

.event-date {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.8rem;
}

@media (max-width: 768px) {
  .events-grid {
    grid-template-columns: 1fr;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-wrap: wrap;
    gap: 0.5rem;
  }
}
</style>
