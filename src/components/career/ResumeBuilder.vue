<template>
  <div class="section-content resume-section-content">
    <div class="resume-header">
      <div class="header-content">
        <h2>Resume Builder</h2>
        <p>Create a professional resume in minutes</p>
      </div>
      <transition-group name="btn-fade" tag="div" class="header-actions">
         <button v-if="resumeData.sections.length > 0 || resumeData.name" key="analyze" @click="analyzeResume" class="btn-secondary">
          <img src="/analyze-svgrepo-com.svg" alt="Analyze" class="btn-icon" /> Analyze
        </button>
        <button v-if="resumeData.sections.length > 0 || resumeData.name" key="download" @click="downloadResume" class="btn-primary">
          <img src="/download-svgrepo-com.svg" alt="Download" class="btn-icon" /> Download PDF
        </button>
      </transition-group>
    </div>

    <div class="resume-workspace">
      <!-- Left Side: Editor -->
      <div class="resume-editor">
        <div class="editor-section">
          <h3>Personal Information</h3>
          <div class="form-grid">
            <div class="form-group full-width">
              <label>Full Name</label>
              <input v-model="resumeData.name" type="text" class="form-input" placeholder="e.g. Alex Morgan" />
            </div>
            <div class="form-group">
              <label>Email</label>
              <input v-model="resumeData.email" type="email" class="form-input" placeholder="alex@example.com" />
            </div>
            <div class="form-group">
              <label>Phone</label>
              <input v-model="resumeData.phone" type="tel" class="form-input" placeholder="+1 234 567 8900" />
            </div>
            <div class="form-group full-width">
              <label>LinkedIn / Portfolio</label>
              <input v-model="resumeData.linkedin" type="text" class="form-input" placeholder="linkedin.com/in/alexmorgan" />
            </div>
          </div>
        </div>

        <div class="editor-section">
          <div class="section-title-row">
            <h3>Professional Summary</h3>
            <button @click="getAISuggestion('summary')" class="btn-text-ai">
              ✨ AI Suggestion
            </button>
          </div>
          <textarea 
            v-model="resumeData.summary" 
            class="form-input" 
            rows="4"
            placeholder="Brief overview of your professional background, key achievements, and career goals..."
          ></textarea>
        </div>

        <div class="editor-section">
          <h3>Work Experience</h3>
          <div 
            v-for="(exp, index) in resumeData.experience" 
            :key="index"
            class="experience-card"
          >
            <div class="card-header">
              <h4>Experience {{ index + 1 }}</h4>
              <button @click="removeExperience(index)" class="btn-icon-remove" title="Remove">
                ×
              </button>
            </div>
            <div class="form-grid">
              <div class="form-group">
                <label>Job Title</label>
                <input v-model="exp.title" type="text" class="form-input" placeholder="e.g. Senior Developer" />
              </div>
              <div class="form-group">
                <label>Company</label>
                <input v-model="exp.company" type="text" class="form-input" placeholder="e.g. Tech Corp" />
              </div>
              <div class="form-group full-width">
                <label>Duration</label>
                <input v-model="exp.duration" type="text" class="form-input" placeholder="e.g. Jan 2020 - Present" />
              </div>
            </div>
            <div class="form-group mt-2">
              <label>Description</label>
              <textarea 
                v-model="exp.description" 
                class="form-input" 
                rows="3"
                placeholder="Describe your responsibilities and achievements..."
              ></textarea>
            </div>
          </div>
          <button @click="addExperience" class="btn-dashed full-width">
            + Add Experience
          </button>
        </div>

        <div class="editor-section">
          <h3>Skills</h3>
          <div class="skills-input-container">
            <input 
              v-model="skillInput" 
              @keydown.enter="addSkill"
              type="text" 
              class="form-input" 
              placeholder="Type a skill and press Enter"
            />
            <button @click="addSkill" class="btn-secondary small">Add</button>
          </div>
          <div class="skills-tags">
            <span 
              v-for="(skill, index) in resumeData.skills" 
              :key="index"
              class="skill-chip"
            >
              {{ skill }}
              <button @click="removeSkill(index)" class="remove-chip">×</button>
            </span>
          </div>
        </div>
        
         <!-- AI Feedback Overlay -->
        <div v-if="resumeFeedback" class="feedback-overlay">
          <div class="feedback-content">
             <div class="feedback-header">
               <h3>AI Resume Analysis</h3>
               <button @click="resumeFeedback = null" class="close-feedback">×</button>
             </div>
             <div class="feedback-body">
              <div class="score-container">
                <div class="score-ring" :style="{ '--score': resumeFeedback.score }">
                  <span class="score-number">{{ resumeFeedback.score }}</span>
                  <span class="score-label">Score</span>
                </div>
              </div>

              <div class="feedback-lists">
                <div class="feedback-list positive">
                  <h4>✅ Strengths</h4>
                  <ul>
                    <li v-for="(strength, index) in resumeFeedback.strengths" :key="index">{{ strength }}</li>
                  </ul>
                </div>
                <div class="feedback-list warning">
                   <h4>⚠️ Improvements</h4>
                  <ul>
                    <li v-for="(improvement, index) in resumeFeedback.improvements" :key="index">{{ improvement }}</li>
                  </ul>
                </div>
                 <div class="feedback-list info">
                   <h4>💡 Recommendations</h4>
                  <ul>
                    <li v-for="(rec, index) in resumeFeedback.recommendations" :key="index">{{ rec }}</li>
                  </ul>
                </div>
              </div>
             </div>
          </div>
        </div>
      </div>

      <!-- Right Side: Live Preview -->
      <div class="resume-preview-container">
        <div class="preview-header">
          <span>Live Preview</span>
        </div>
        <div class="resume-paper" id="resume-preview">
          <div class="paper-header">
            <h1 class="paper-name">{{ resumeData.name || 'Your Name' }}</h1>
            <div class="paper-contact">
              <span v-if="resumeData.email">{{ resumeData.email }}</span>
              <span v-if="resumeData.email && resumeData.phone"> • </span>
              <span v-if="resumeData.phone">{{ resumeData.phone }}</span>
              <span v-if="(resumeData.email || resumeData.phone) && resumeData.linkedin"> • </span>
              <span v-if="resumeData.linkedin">{{ resumeData.linkedin }}</span>
            </div>
          </div>
          
          <div class="paper-content">
            <div v-if="resumeData.summary" class="paper-section">
              <h2 class="paper-section-title">Professional Summary</h2>
              <p class="paper-text">{{ resumeData.summary }}</p>
            </div>

            <div v-if="resumeData.experience.length > 0" class="paper-section">
              <h2 class="paper-section-title">Experience</h2>
              <div v-for="(exp, index) in resumeData.experience" :key="index" class="paper-item">
                <div class="paper-item-header">
                  <h3 class="paper-item-title">{{ exp.title || 'Job Title' }}</h3>
                  <span class="paper-item-date">{{ exp.duration }}</span>
                </div>
                <div class="paper-item-subtitle">{{ exp.company || 'Company Name' }}</div>
                <p class="paper-text">{{ exp.description }}</p>
              </div>
            </div>

            <div v-if="resumeData.skills.length > 0" class="paper-section">
              <h2 class="paper-section-title">Skills</h2>
              <div class="paper-skills">
                <span v-for="(skill, index) in resumeData.skills" :key="index" class="paper-skill-item">
                  {{ skill }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const resumeData = ref({
  name: '',
  email: '',
  phone: '',
  linkedin: '',
  summary: '',
  experience: [],
  skills: [],
  sections: []
})

const skillInput = ref('')
const resumeFeedback = ref(null)

const addExperience = () => {
  resumeData.value.experience.push({
    title: '',
    company: '',
    duration: '',
    description: ''
  })
}

const removeExperience = (index) => {
  resumeData.value.experience.splice(index, 1)
}

const addSkill = () => {
  if (skillInput.value.trim()) {
    resumeData.value.skills.push(skillInput.value.trim())
    skillInput.value = ''
  }
}

const removeSkill = (index) => {
  resumeData.value.skills.splice(index, 1)
}

const getAISuggestion = (field) => {
  alert('AI suggestions feature coming soon!')
}

const analyzeResume = () => {
  resumeFeedback.value = {
    score: Math.floor(Math.random() * 20) + 70,
    strengths: [
      'Clear and concise professional summary',
      'Well-structured work experience section',
      'Good mix of technical and soft skills'
    ],
    improvements: [
      'Add more quantifiable achievements (metrics, percentages)',
      'Include relevant certifications or courses',
      'Expand on leadership experiences'
    ],
    recommendations: [
      'Consider adding a portfolio link or GitHub profile',
      'Tailor your summary to specific job roles',
      'Get 2-3 years of experience in cloud technologies for better opportunities'
    ]
  }
}

const downloadResume = () => {
  alert('Resume download feature coming soon!')
}
</script>

<style scoped>
.resume-section-content {
  display: flex;
  flex-direction: column;
  padding: 0;
  overflow: hidden;
  background: transparent;
  border: none;
  backdrop-filter: none;
}

.resume-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px 12px 0 0;
}

.resume-header h2 {
  color: white;
  margin: 0;
  font-size: 1.5rem;
}

.resume-header p {
  color: rgba(255, 255, 255, 0.7);
  margin: 0.2rem 0 0 0;
  font-size: 0.9rem;
}

.header-actions {
  display: flex;
  gap: 1rem;
  transition: all 0.4s ease;
}

.resume-header .header-content {
  transition: all 0.4s ease;
}

.btn-fade-enter-active,
.btn-fade-leave-active {
  transition: all 0.4s ease;
}

.btn-fade-enter-from,
.btn-fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

.resume-workspace {
  display: flex;
  flex: 1;
  overflow: hidden;
  gap: 0;
  background: rgba(255, 255, 255, 0.05);
}

.resume-editor {
  flex: 1;
  padding: 1.5rem;
  overflow-y: auto;
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  min-width: 400px;
}

.editor-section {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.editor-section h3 {
  color: #fff;
  margin-bottom: 1.2rem;
  font-size: 1.1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.section-title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.2rem;
}

.section-title-row h3 {
  margin-bottom: 0;
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

.btn-primary, .btn-secondary {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
  font-size: 0.95rem;
}

.btn-primary {
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(139, 92, 246, 0.4);
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-icon {
  width: 18px;
  height: 18px;
  vertical-align: middle;
  margin-right: 0.5rem;
  filter: brightness(0) invert(1);
}

.btn-secondary .btn-icon {
  filter: brightness(0) saturate(100%) invert(21%) sepia(95%) saturate(3095%) hue-rotate(262deg) brightness(68%) contrast(97%);
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.8);
  color: #6d28d9;
}

.btn-secondary:hover {
  background: rgba(255, 255, 255, 0.95);
}

.experience-card {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.card-header h4 {
  color: rgba(255, 255, 255, 0.9);
  margin: 0;
  font-size: 0.95rem;
}

.btn-icon-remove {
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  font-size: 1.2rem;
  line-height: 1;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
}

.btn-icon-remove:hover {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
}

.btn-dashed {
  width: 100%;
  padding: 0.8rem;
  background: transparent;
  border: 2px dashed rgba(255, 255, 255, 0.2);
  color: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-dashed:hover {
  border-color: rgba(255, 255, 255, 0.5);
  background: rgba(255, 255, 255, 0.05);
  color: white;
}

.btn-text-ai {
  background: transparent;
  border: none;
  background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 600;
  cursor: pointer;
  font-size: 0.9rem;
}

.skills-input-container {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.skills-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.skill-chip {
  background: rgba(139, 92, 246, 0.15);
  border: 1px solid rgba(139, 92, 246, 0.3);
  color: #e9d5ff;
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.remove-chip {
  background: transparent;
  border: none;
  color: rgba(233, 213, 255, 0.6);
  cursor: pointer;
  font-size: 1.1rem;
  line-height: 1;
  padding: 0;
  display: flex;
  align-items: center;
}

.remove-chip:hover {
  color: white;
}

/* Preview */
.resume-preview-container {
  flex: 1;
  background: #2c2c2c;
  padding: 2rem;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.preview-header {
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 1rem;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.resume-paper {
  background: white;
  width: 100%;
  max-width: 210mm;
  min-height: 297mm;
  padding: 2.5rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  color: #333;
  font-family: 'Times New Roman', serif;
  font-size: 11pt;
  line-height: 1.4;
}

.paper-header {
  text-align: center;
  margin-bottom: 2rem;
  border-bottom: 2px solid #333;
  padding-bottom: 1rem;
}

.paper-name {
  font-family: 'Arial', sans-serif;
  font-size: 24pt;
  font-weight: bold;
  text-transform: uppercase;
  margin-bottom: 0.5rem;
  color: #000;
}

.paper-contact {
  font-size: 10pt;
  color: #555;
  font-family: 'Arial', sans-serif;
}

.paper-section {
  margin-bottom: 1.5rem;
}

.paper-section-title {
  font-family: 'Arial', sans-serif;
  font-size: 12pt;
  font-weight: bold;
  text-transform: uppercase;
  border-bottom: 1px solid #ccc;
  margin-bottom: 0.8rem;
  padding-bottom: 0.2rem;
  color: #000;
}

.paper-item {
  margin-bottom: 1rem;
}

.paper-item-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

.paper-item-title {
  font-weight: bold;
  font-size: 11pt;
  margin: 0;
  color: #000;
}

.paper-item-date {
  font-style: italic;
  font-size: 10pt;
}

.paper-item-subtitle {
  font-weight: bold;
  font-size: 10pt;
  color: #444;
  margin-bottom: 0.2rem;
}

.paper-text {
  margin: 0;
  text-align: justify;
}

.paper-skills {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.paper-skill-item {
  background: #eee;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-size: 10pt;
}

/* Feedback Overlay */
.feedback-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  padding: 2rem;
}

.feedback-content {
  background: #1e1e24;
  border: 1px solid rgba(255, 255, 255, 0.1);
  width: 100%;
  max-width: 600px;
  max-height: 80vh;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 50px rgba(0,0,0,0.5);
}

.feedback-header {
  padding: 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.feedback-header h3 {
  color: white;
  margin: 0;
}

.close-feedback {
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.5);
  font-size: 1.5rem;
  cursor: pointer;
}

.feedback-body {
  padding: 1.5rem;
  overflow-y: auto;
}

.score-container {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.score-ring {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: conic-gradient(#8b5cf6 calc(var(--score) * 1%), rgba(255,255,255,0.1) 0);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}

.score-ring::before {
  content: '';
  position: absolute;
  width: 100px;
  height: 100px;
  background: #1e1e24;
  border-radius: 50%;
}

.score-number {
  position: relative;
  font-size: 2.5rem;
  font-weight: bold;
  color: white;
  z-index: 1;
}

.score-label {
  position: relative;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
  z-index: 1;
}

.feedback-lists {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.feedback-list h4 {
  margin-bottom: 0.8rem;
  font-size: 1rem;
}

.feedback-list.positive h4 { color: #4ade80; }
.feedback-list.warning h4 { color: #fbbf24; }
.feedback-list.info h4 { color: #60a5fa; }

.feedback-list ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.feedback-list li {
  padding: 0.5rem;
  background: rgba(255, 255, 255, 0.03);
  margin-bottom: 0.5rem;
  border-radius: 6px;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
}

/* Responsive */
@media (max-width: 1024px) {
  .resume-workspace {
    flex-direction: column;
    overflow-y: auto;
  }

  .resume-editor {
    border-right: none;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    min-width: unset;
    max-height: 50vh;
  }

  .resume-preview-container {
    max-height: 50vh;
  }
}

@media (max-width: 768px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
  
  .resume-preview-container {
    display: none;
  }
}
</style>
