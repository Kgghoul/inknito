<template>
  <div class="wellbeing-motivation">
    <div class="header">
      <h1>Wellbeing and Motivation</h1>
      <p class="subtitle">Your mental health and connections matter</p>
    </div>

    <!-- Main Content -->
    <div class="content-grid">
      <!-- Anonymous Chat Bot -->
      <div class="section anonymous-chat">
        <div class="section-header">
          <div class="section-title-group">
            <div class="section-icon-wrapper chat-icon-wrapper">
              <img :src="base + 'safe-shield-svgrepo-com.svg'" class="section-icon" alt="Chat" />
            </div>
            <div>
              <h2>Anonymous Support Chat</h2>
              <p class="section-tagline">Talk freely. Your identity is completely protected.</p>
            </div>
          </div>
          <div class="anonymous-badge">
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
              <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
            </svg>
            100% Anonymous
          </div>
        </div>

        <ChatWindow
          ref="chatWindowRef"
          :messages="chatMessages"
          :is-loading="isLoading"
          placeholder="Share what's on your mind... (completely anonymous)"
          welcome-title="Welcome to Your Safe Space"
          welcome-subtitle="This is a judgment-free zone. Share what's on your mind."
          welcome-icon="/safe-shield-svgrepo-com.svg"
          bot-label="Support Bot"
          bot-class="bot"
          @send="sendMessage"
        >
          <template #welcome-extra>
            <div class="chat-topics">
              <p class="topics-label">I can help with:</p>
              <div class="topics-grid">
                <button 
                  v-for="topic in topics" 
                  :key="topic"
                  class="topic-chip"
                  @click="sendMessage(`I need help with ${topic.toLowerCase()}`)"
                >
                  {{ topic }}
                </button>
              </div>
            </div>
          </template>
        </ChatWindow>
      </div>

      <!-- Friend Finder -->
      <div class="section friend-finder">
        <div class="section-header">
          <div class="section-title-group">
            <div class="section-icon-wrapper finder-icon-wrapper">
              <img :src="base + 'find-location-svgrepo-com.svg'" class="section-icon" alt="Friends" />
            </div>
            <div>
              <h2>Find Study Buddies</h2>
              <p class="section-tagline">Connect with students who share your goals.</p>
            </div>
          </div>
          <label class="toggle-switch">
            <input type="checkbox" v-model="friendFinderEnabled" @change="toggleFriendFinder">
            <span class="toggle-slider"></span>
          </label>
        </div>

        <div v-if="!friendFinderEnabled" class="disabled-state">
          <div class="disabled-content">
            <div class="disabled-icon-wrapper">
              <img :src="base + 'find-location-svgrepo-com.svg'" class="disabled-icon" alt="Disabled" />
            </div>
            <h3>Friend Finder is Off</h3>
            <p>Toggle the switch above to start connecting with other students</p>
          </div>
        </div>

        <div v-else class="friend-finder-content">
          <!-- Profile Setup -->
          <div v-if="!profileComplete" class="profile-setup">
            <div class="setup-header">
              <h3>Complete Your Profile</h3>
              <p>Tell us about yourself to find the best matches</p>
            </div>
            <div class="form-group">
              <label>
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"/><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"/></svg>
                University / College
              </label>
              <input 
                v-model="userProfile.university" 
                type="text" 
                placeholder="e.g., Harvard University"
                class="form-input"
              />
            </div>
            <div class="form-group">
              <label>
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>
                Your Interests
              </label>
              <input 
                v-model="userProfile.interests" 
                type="text" 
                placeholder="e.g., Computer Science, Music, Sports"
                class="form-input"
              />
            </div>
            <div class="form-group">
              <label>
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
                Study Goals
              </label>
              <textarea 
                v-model="userProfile.goals" 
                placeholder="What are you working towards?"
                rows="3"
                class="form-input"
              ></textarea>
            </div>
            <button @click="saveProfile" class="btn-primary">
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="20 6 9 17 4 12"/></svg>
              Save Profile
            </button>
          </div>

          <!-- Friend List -->
          <div v-else class="friends-list">
            <div class="filter-bar">
              <div class="search-wrapper">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="search-icon">
                  <circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>
                </svg>
                <input 
                  v-model="searchQuery"
                  type="text" 
                  placeholder="Search by interests or university..."
                  class="search-input"
                />
              </div>
            </div>

            <div class="friends-grid">
              <div 
                v-for="friend in filteredFriends" 
                :key="friend.id"
                class="friend-card"
              >
                <div class="friend-avatar">{{ friend.avatar }}</div>
                <h4>{{ friend.name }}</h4>
                <p class="friend-university">{{ friend.university }}</p>
                <div class="friend-interests">
                  <span 
                    v-for="(interest, idx) in friend.interests.slice(0, 3)" 
                    :key="idx"
                    class="interest-tag"
                  >
                    {{ interest }}
                  </span>
                </div>
                <button @click="connectFriend(friend)" class="btn-connect">
                  <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="8.5" cy="7" r="4"/><line x1="20" y1="8" x2="20" y2="14"/><line x1="23" y1="11" x2="17" y2="11"/></svg>
                  Connect
                </button>
              </div>
            </div>

            <div v-if="filteredFriends.length === 0" class="no-results">
              <p>No matching students found. Try different search terms!</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const base = import.meta.env.BASE_URL
import ChatWindow from './ChatWindow.vue'

const topics = ['Academic Stress', 'Anxiety', 'Relationships', 'Work-Life Balance', 'Motivation', 'General Support']

// Chat State
const chatMessages = ref([])
const isLoading = ref(false)
const chatWindowRef = ref(null)

// Friend Finder State
const friendFinderEnabled = ref(false)
const profileComplete = ref(false)
const searchQuery = ref('')
const userProfile = ref({
  university: '',
  interests: '',
  goals: ''
})

// Mock Friends Data
const friends = ref([
  { id: 1, name: 'Alex M.', avatar: '👨‍💻', university: 'MIT', interests: ['Computer Science', 'AI', 'Gaming'] },
  { id: 2, name: 'Sarah K.', avatar: '👩‍🔬', university: 'Stanford', interests: ['Biology', 'Research', 'Yoga'] },
  { id: 3, name: 'James L.', avatar: '👨‍🎓', university: 'Harvard', interests: ['Business', 'Entrepreneurship', 'Sports'] },
  { id: 4, name: 'Emily R.', avatar: '👩‍💼', university: 'MIT', interests: ['Engineering', 'Robotics', 'Music'] },
  { id: 5, name: 'Michael T.', avatar: '👨‍🏫', university: 'Yale', interests: ['Literature', 'Writing', 'Photography'] },
  { id: 6, name: 'Lisa W.', avatar: '👩‍🎨', university: 'Stanford', interests: ['Design', 'Art', 'Travel'] }
])

// Chat Functions
const sendMessage = (messageText) => {
  chatMessages.value.push({
    sender: 'user',
    text: messageText
  })

  isLoading.value = true

  setTimeout(() => {
    chatMessages.value.push({
      sender: 'bot',
      text: generateSupportResponse(messageText)
    })
    isLoading.value = false

    setTimeout(() => {
      chatWindowRef.value?.scrollToBottom()
    }, 100)
  }, 1500)
}

const generateSupportResponse = (question) => {
  const responses = [
    "I hear you, and your feelings are completely valid. It's okay to feel overwhelmed sometimes. What specifically is weighing on your mind right now?",
    "Thank you for sharing that with me. It takes courage to open up. Remember, it's okay to ask for help, and you're not alone in this. Have you tried any coping strategies that have worked for you in the past?",
    "That sounds really challenging. It's important to acknowledge what you're going through. Have you considered breaking down the problem into smaller, manageable steps?",
    "I understand how stressful that can be. Remember to be kind to yourself - you're doing better than you think. What's one small thing you could do today to feel a bit better?"
  ]
  return responses[Math.floor(Math.random() * responses.length)]
}

// Friend Finder Functions
const toggleFriendFinder = () => {
  if (friendFinderEnabled.value && !profileComplete.value) {
    // Keep it enabled to show profile setup
  }
}

const saveProfile = () => {
  if (userProfile.value.university && userProfile.value.interests) {
    profileComplete.value = true
  }
}

const filteredFriends = computed(() => {
  if (!searchQuery.value) return friends.value

  const query = searchQuery.value.toLowerCase()
  return friends.value.filter(friend => 
    friend.university.toLowerCase().includes(query) ||
    friend.interests.some(interest => interest.toLowerCase().includes(query))
  )
})

const connectFriend = (friend) => {
  alert(`Connection request sent to ${friend.name}!`)
}
</script>

<style scoped>
.wellbeing-motivation {
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
  margin-bottom: 1.25rem;
  flex-shrink: 0;
}

.header h1 {
  font-family: 'Showpop', sans-serif;
  font-size: 2.2rem;
  color: #ffffff;
  margin-bottom: 0.3rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.subtitle {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* Content Grid */
.content-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(480px, 1fr));
  gap: 1rem;
  flex: 1;
  overflow: hidden;
}

/* Section Cards */
.section {
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.14) 0%, rgba(255, 255, 255, 0.06) 100%);
  backdrop-filter: blur(24px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.06);
}

/* Section Header */
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
  flex-shrink: 0;
}

.section-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.section-icon-wrapper {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.chat-icon-wrapper {
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.5), rgba(109, 40, 217, 0.6));
  border: 1px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 2px 12px rgba(139, 92, 246, 0.3);
}

.finder-icon-wrapper {
  background: linear-gradient(135deg, rgba(6, 182, 212, 0.5), rgba(8, 145, 178, 0.6));
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
  font-size: 1.15rem;
  font-weight: 700;
  margin: 0;
  line-height: 1.2;
}

.section-tagline {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.8rem;
  margin: 0.15rem 0 0;
}

/* Anonymous Badge */
.anonymous-badge {
  background: rgba(16, 185, 129, 0.15);
  color: #34d399;
  padding: 0.35rem 0.75rem;
  border-radius: 20px;
  font-size: 0.78rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 0.35rem;
  border: 1px solid rgba(16, 185, 129, 0.35);
  white-space: nowrap;
  letter-spacing: 0.3px;
}

/* Chat Topics (welcome chips) */
.chat-topics {
  margin-top: 1rem;
  text-align: center;
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
}

.topics-label {
  font-size: 0.85rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.85);
  margin-bottom: 0.6rem;
}

.topics-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  justify-content: center;
}

.topic-chip {
  background: rgba(255, 255, 255, 0.12);
  color: rgba(255, 255, 255, 0.9);
  padding: 0.45rem 0.9rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
  border: 1px solid rgba(255, 255, 255, 0.25);
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(8px);
}

.topic-chip:hover {
  background: rgba(255, 255, 255, 0.28);
  border-color: rgba(255, 255, 255, 0.5);
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  color: #ffffff;
}

/* Toggle Switch */
.toggle-switch {
  position: relative;
  display: inline-block;
  width: 48px;
  height: 26px;
  flex-shrink: 0;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.15);
  transition: 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 26px;
  border: 1px solid rgba(255, 255, 255, 0.25);
}

.toggle-slider:before {
  position: absolute;
  content: "";
  height: 20px;
  width: 20px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  transition: 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 50%;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
}

input:checked + .toggle-slider {
  background: linear-gradient(135deg, #10b981, #059669);
  border-color: rgba(16, 185, 129, 0.5);
  box-shadow: 0 0 12px rgba(16, 185, 129, 0.3);
}

input:checked + .toggle-slider:before {
  transform: translateX(22px);
}

/* Disabled State */
.disabled-state {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.disabled-content {
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
  padding: 2rem 1rem;
}

.disabled-icon-wrapper {
  width: 72px;
  height: 72px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px dashed rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1.25rem;
}

.disabled-icon {
  width: 36px;
  height: 36px;
  filter: brightness(0) saturate(100%) invert(100%);
  opacity: 0.35;
}

.disabled-content h3 {
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 0.5rem;
  font-size: 1.1rem;
}

.disabled-content p {
  font-size: 0.88rem;
}

/* Friend Finder Content */
.friend-finder-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* Profile Setup */
.profile-setup {
  flex: 1;
  overflow-y: auto;
  padding-right: 0.25rem;
}

.setup-header {
  margin-bottom: 1rem;
}

.setup-header h3 {
  color: #ffffff;
  font-size: 1.1rem;
  font-weight: 700;
  margin-bottom: 0.2rem;
}

.setup-header p {
  color: rgba(255, 255, 255, 0.65);
  font-size: 0.85rem;
}

.form-group {
  margin-bottom: 0.85rem;
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
  transition: all 0.3s;
  backdrop-filter: blur(8px);
}

.form-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.form-input:focus {
  outline: none;
  border-color: rgba(139, 92, 246, 0.6);
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.15);
}

textarea.form-input {
  resize: none;
}

.btn-primary {
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  padding: 0.7rem 1.5rem;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 700;
  font-size: 0.9rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  width: 100%;
  margin-top: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.3);
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 24px rgba(139, 92, 246, 0.45);
}

/* Friends List */
.friends-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.filter-bar {
  margin-bottom: 0.75rem;
  flex-shrink: 0;
}

.search-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.search-icon {
  position: absolute;
  left: 0.85rem;
  color: rgba(255, 255, 255, 0.5);
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 0.7rem 0.85rem 0.7rem 2.5rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
  font-family: inherit;
  font-size: 0.9rem;
  transition: all 0.3s;
  backdrop-filter: blur(8px);
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.search-input:focus {
  outline: none;
  border-color: rgba(6, 182, 212, 0.6);
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 0 0 3px rgba(6, 182, 212, 0.15);
}

.friends-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 0.75rem;
  overflow-y: auto;
  padding-bottom: 0.5rem;
}

.friend-card {
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0.06) 100%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 1rem;
  text-align: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.friend-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  border-color: rgba(255, 255, 255, 0.4);
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.22) 0%, rgba(255, 255, 255, 0.1) 100%);
}

.friend-avatar {
  font-size: 2.5rem;
  margin-bottom: 0.4rem;
  line-height: 1;
}

.friend-card h4 {
  color: #ffffff;
  margin-bottom: 0.15rem;
  font-size: 0.95rem;
  font-weight: 700;
}

.friend-university {
  color: rgba(255, 255, 255, 0.65);
  font-size: 0.78rem;
  margin-bottom: 0.5rem;
}

.friend-interests {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
  justify-content: center;
  margin-bottom: 0.6rem;
}

.interest-tag {
  background: rgba(255, 255, 255, 0.12);
  padding: 0.15rem 0.5rem;
  border-radius: 8px;
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.btn-connect {
  background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
  color: white;
  padding: 0.45rem 0.85rem;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 700;
  font-size: 0.8rem;
  width: 100%;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  box-shadow: 0 2px 10px rgba(6, 182, 212, 0.25);
}

.btn-connect:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 18px rgba(6, 182, 212, 0.4);
}

.no-results {
  text-align: center;
  padding: 2rem;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.9rem;
}

/* Responsive */
@media (max-width: 1200px) {
  .content-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .wellbeing-motivation {
    padding: 1rem;
  }

  .header h1 {
    font-size: 1.6rem;
  }

  .content-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .friends-grid {
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  }
}
</style>
