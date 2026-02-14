<template>
  <div class="profile-container">
    <!-- Header -->
    <div class="profile-header">
      <div class="header-bg"></div>
      <div class="header-content">
        <div class="avatar-container">
          <img src="/person-svgrepo-com.svg" alt="User Avatar" class="avatar" />
          <button class="edit-avatar-btn">
            <img src="/photo-9-svgrepo-com.svg" alt="Edit" class="icon-img" />
          </button>
        </div>
        <div class="user-info">
          <h1>{{ userProfile.name }}</h1>
          <p class="user-role">{{ userProfile.major }} Student • Class of {{ userProfile.year }}</p>
          <div class="level-badge">
            <span>🏆 Level 5 Scholar</span>
          </div>
        </div>
      </div>
    </div>

    <div class="profile-body">
      <!-- Sidebar Navigation -->
      <div class="profile-sidebar">
        <button 
          v-for="tab in tabs" 
          :key="tab.id"
          @click="activeTab = tab.id"
          :class="['sidebar-link', { active: activeTab === tab.id }]"
        >
          <img :src="tab.icon" class="sidebar-icon" :alt="tab.name" />
          {{ tab.name }}
        </button>
      </div>

      <!-- Main Content Area -->
      <div class="profile-content">
        
        <!-- Personal Info Tab -->
        <div v-if="activeTab === 'info'" class="content-section">
          <h2>
            <img src="/person-svgrepo-com.svg" class="section-icon" alt="" />
            Personal Information
          </h2>
          <div class="form-card">
            <div class="form-grid">
              <div class="form-group">
                <label>Full Name</label>
                <input v-model="userProfile.name" type="text" class="form-input" />
              </div>
              <div class="form-group">
                <label>Email</label>
                <input v-model="userProfile.email" type="email" class="form-input" />
              </div>
              <div class="form-group">
                <label>University</label>
                <input v-model="userProfile.university" type="text" class="form-input" />
              </div>
              <div class="form-group">
                <label>Major</label>
                <input v-model="userProfile.major" type="text" class="form-input" />
              </div>
              <div class="form-group">
                <label>Graduation Year</label>
                <input v-model="userProfile.year" type="number" class="form-input" />
              </div>
              <div class="form-group">
                <label>Student ID</label>
                <input v-model="userProfile.studentId" type="text" class="form-input" disabled />
              </div>
            </div>
            <div class="form-actions">
              <button class="btn-primary" @click="saveProfile">Save Changes</button>
            </div>
          </div>
        </div>

        <!-- Statistics Tab -->
        <div v-if="activeTab === 'stats'" class="content-section">
          <h2>
             <img src="/statistics-graph-stats-analytics-business-data-svgrepo-com.svg" class="section-icon" alt="" />
             Your Progress
          </h2>
          <div class="stats-grid">
            <div class="stat-card blue">
              <div class="stat-icon"><img src="/time-svgrepo-com.svg" alt="Study Hours" class="stat-svg" /></div>
              <div class="stat-value">124</div>
              <div class="stat-label">Study Hours</div>
            </div>
            <div class="stat-card purple">
              <div class="stat-icon"><img src="/target-arrow-svgrepo-com.svg" alt="Avg Grade" class="stat-svg" /></div>
              <div class="stat-value">85%</div>
              <div class="stat-label">Avg. Grade</div>
            </div>
            <div class="stat-card green">
              <div class="stat-icon"><img src="/event-svgrepo-com.svg" alt="Events" class="stat-svg" /></div>
              <div class="stat-value">12</div>
              <div class="stat-label">Events Attended</div>
            </div>
            <div class="stat-card orange">
              <div class="stat-icon"><img src="/fire-svgrepo-com.svg" alt="Streak" class="stat-svg" /></div>
              <div class="stat-value">5</div>
              <div class="stat-label">Current Streak</div>
            </div>
          </div>

          <div class="chart-section">
            <h3>Activity Overview</h3>
            <div class="mock-chart">
              <!-- Mock bars for visualization -->
              <div class="chart-bar" style="height: 60%"><span>Mon</span></div>
              <div class="chart-bar" style="height: 80%"><span>Tue</span></div>
              <div class="chart-bar" style="height: 45%"><span>Wed</span></div>
              <div class="chart-bar" style="height: 90%"><span>Thu</span></div>
              <div class="chart-bar" style="height: 75%"><span>Fri</span></div>
              <div class="chart-bar" style="height: 30%"><span>Sat</span></div>
              <div class="chart-bar" style="height: 50%"><span>Sun</span></div>
            </div>
          </div>
        </div>

        <!-- Settings Tab -->
        <div v-if="activeTab === 'settings'" class="content-section">
          <h2>
            <img src="/settings-1390-svgrepo-com.svg" class="section-icon" alt="" />
            Account Settings
          </h2>
          <div class="settings-group">
            <h3>Preferences</h3>
            <div class="setting-item">
              <div class="setting-info">
                <span class="setting-name">Dark Mode</span>
                <span class="setting-desc">Toggle application theme</span>
              </div>
              <label class="switch">
                <input type="checkbox" v-model="settings.darkMode">
                <span class="slider round"></span>
              </label>
            </div>
            <div class="setting-item">
              <div class="setting-info">
                <span class="setting-name">Email Notifications</span>
                <span class="setting-desc">Receive updates about events and grades</span>
              </div>
              <label class="switch">
                <input type="checkbox" v-model="settings.emailNotifs">
                <span class="slider round"></span>
              </label>
            </div>
          </div>

          <div class="settings-group">
            <h3>Privacy</h3>
            <div class="setting-item">
              <div class="setting-info">
                <span class="setting-name">Public Profile</span>
                <span class="setting-desc">Allow others to see your badges</span>
              </div>
              <label class="switch">
                <input type="checkbox" v-model="settings.publicProfile">
                <span class="slider round"></span>
              </label>
            </div>
          </div>
          
           <div class="form-actions">
              <button class="btn-danger">Sign Out</button>
            </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const activeTab = ref('info')

const tabs = [
  { id: 'info', name: 'Personal Info', icon: '/person-svgrepo-com.svg' },
  { id: 'stats', name: 'Statistics', icon: '/statistics-graph-stats-analytics-business-data-svgrepo-com.svg' },
  { id: 'settings', name: 'Settings', icon: '/settings-1390-svgrepo-com.svg' }
]

const userProfile = ref({
  name: 'Kassymkhanov Karim',
  email: '8767YUYF@psba.edu.sg',
  university: 'PSB Academy',
  major: 'InfoComm Technology',
  year: 2026,
  studentId: '8767YUYF'
})

const settings = ref({
  darkMode: true,
  emailNotifs: true,
  publicProfile: false
})

const saveProfile = () => {
  alert('Profile changes saved successfully!')
}
</script>

<style scoped>
.profile-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  overflow: hidden;
}

/* Header */
.profile-header {
  position: relative;
  background: rgba(0, 0, 0, 0.2);
  padding: 2rem;
  flex-shrink: 0;
}

.header-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg, #6d28d9 0%, #06b6d4 100%);
  opacity: 0.15;
  z-index: 0;
}

.header-content {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  gap: 2rem;
  max-width: 1000px;
  margin: 0 auto;
}

.avatar-container {
  position: relative;
  width: 110px;
  height: 110px;
  flex-shrink: 0;
}

.avatar {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  border: 3px solid transparent;
  background:
    linear-gradient(135deg, rgba(255,255,255,0.95), rgba(240,240,255,0.95)) padding-box,
    linear-gradient(135deg, #8b5cf6, #06b6d4) border-box;
  object-fit: cover;
  padding: 18px;
  box-shadow: 0 4px 20px rgba(139, 92, 246, 0.3), 0 0 40px rgba(6, 182, 212, 0.15);
}

.edit-avatar-btn {
  position: absolute;
  bottom: 2px;
  right: 2px;
  background: linear-gradient(135deg, #8b5cf6, #6d28d9);
  border: 2px solid rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  padding: 0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s;
}

.edit-avatar-btn:hover {
  transform: scale(1.1);
}

.icon-img {
  width: 16px;
  height: 16px;
  filter: brightness(0) invert(1);
}

.user-info h1 {
  color: white;
  margin: 0 0 0.5rem 0;
  font-size: 2rem;
}

.user-role {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.1rem;
  margin-bottom: 0.8rem;
}

.level-badge {
  display: inline-block;
  background: linear-gradient(135deg, #fbbf24 0%, #d97706 100%);
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  color: #fff;
  font-weight: bold;
  font-size: 0.9rem;
  box-shadow: 0 2px 10px rgba(251, 191, 36, 0.3);
}

/* Body */
.profile-body {
  flex: 1;
  display: flex;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  overflow: hidden;
}

/* Sidebar */
.profile-sidebar {
  width: 250px;
  padding: 2rem;
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.sidebar-link {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 1rem;
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  text-align: left;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.2s;
}

.sidebar-link:hover {
  background: rgba(255, 255, 255, 0.1);
  color: white;
}

.sidebar-link.active {
  background: rgba(109, 40, 217, 0.2);
  color: #a78bfa;
  font-weight: 600;
}

.sidebar-link.active .sidebar-icon {
  filter: brightness(0) saturate(100%) invert(21%) sepia(95%) saturate(3095%) hue-rotate(262deg) brightness(68%) contrast(97%); /* Same as active tab icon color */
  opacity: 1;
}

.sidebar-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) invert(1);
  opacity: 0.7;
}

/* Content */
.profile-content {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
}

.content-section {
  max-width: 800px;
  animation: fadeIn 0.3s ease-out;
}

.content-section h2 {
  color: white;
  margin-bottom: 1.5rem;
  font-size: 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.section-icon {
  width: 28px;
  height: 28px;
  filter: brightness(0) invert(1);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Forms */
.form-card {
  background: rgba(255, 255, 255, 0.05);
  padding: 2rem;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.form-group label {
  display: block;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
}

.form-input {
  width: 100%;
  padding: 0.8rem;
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: white;
  font-size: 1rem;
}

.form-input:focus {
  outline: none;
  border-color: #8b5cf6;
  background: rgba(0, 0, 0, 0.3);
}

.form-input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
}

.btn-primary {
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  border: none;
  padding: 0.8rem 2rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s;
}

.btn-primary:hover {
  transform: translateY(-2px);
}

.btn-danger {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
  border: 1px solid #ef4444;
  padding: 0.6rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-danger:hover {
  background: rgba(239, 68, 68, 0.4);
}

/* Stats */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: rgba(255, 255, 255, 0.05);
  padding: 1.5rem;
  border-radius: 12px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: transform 0.3s;
}

.stat-card:hover {
  transform: translateY(-5px);
}

.stat-card.blue { border-top: 4px solid #3b82f6; }
.stat-card.purple { border-top: 4px solid #8b5cf6; }
.stat-card.green { border-top: 4px solid #10b981; }
.stat-card.orange { border-top: 4px solid #f59e0b; }

.stat-icon {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.stat-svg {
  width: 36px;
  height: 36px;
  filter: brightness(0) invert(1);
  opacity: 0.9;
}

.stat-value {
  font-size: 2rem;
  font-weight: bold;
  color: white;
  margin-bottom: 0.2rem;
}

.stat-label {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.9rem;
}

.chart-section {
  background: rgba(255, 255, 255, 0.05);
  padding: 2rem;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.chart-section h3 {
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 1.5rem;
  font-size: 1.1rem;
}

.mock-chart {
  height: 200px;
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.chart-bar {
  width: 40px;
  background: rgba(139, 92, 246, 0.6);
  border-radius: 4px 4px 0 0;
  position: relative;
  transition: height 1s ease-out;
}

.chart-bar span {
  position: absolute;
  bottom: -25px;
  left: 50%;
  transform: translateX(-50%);
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.8rem;
}

/* Settings */
.settings-group {
  background: rgba(255, 255, 255, 0.05);
  padding: 1.5rem;
  border-radius: 12px;
  margin-bottom: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.settings-group h3 {
  color: #a78bfa;
  margin-bottom: 1rem;
  font-size: 1rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.setting-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.setting-item:last-child {
  border-bottom: none;
}

.setting-name {
  display: block;
  color: white;
  margin-bottom: 0.3rem;
  font-weight: 500;
}

.setting-desc {
  display: block;
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.9rem;
}

/* Switch */
.switch {
  position: relative;
  display: inline-block;
  width: 50px;
  height: 26px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.2);
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
}

input:checked + .slider {
  background-color: #8b5cf6;
}

input:focus + .slider {
  box-shadow: 0 0 1px #8b5cf6;
}

input:checked + .slider:before {
  transform: translateX(24px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}

/* Responsive */
@media (max-width: 768px) {
  .profile-body {
    flex-direction: column;
    overflow-y: auto;
  }

  .profile-sidebar {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    flex-direction: row;
    overflow-x: auto;
    padding: 1rem;
  }

  .sidebar-link {
    white-space: nowrap;
  }
}
</style>
