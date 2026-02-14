<template>
  <div class="book-management">
    <div class="upload-section">
      <h2>Upload Your Books</h2>
      <div class="upload-area" @drop.prevent="handleDrop" @dragover.prevent>
        <input 
          type="file" 
          ref="fileInput" 
          @change="handleFileUpload" 
          accept=".pdf,.txt,.doc,.docx"
          style="display: none"
        />
        <div class="upload-placeholder" @click="$refs.fileInput.click()">
          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
            <polyline points="17 8 12 3 7 8"></polyline>
            <line x1="12" y1="3" x2="12" y2="15"></line>
          </svg>
          <p>Click or drag files here to upload</p>
          <span class="file-types">Supported: PDF, TXT, DOC, DOCX</span>
        </div>
      </div>
    </div>

    <!-- Books Library -->
    <div class="books-library" v-if="books.length > 0">
      <h2>Your Library</h2>
      <div class="books-grid">
        <div 
          v-for="book in books" 
          :key="book.id" 
          class="book-card"
        >
          <img :src="base + 'book-heart-love-svgrepo-com.svg'" class="book-icon" alt="Book icon" />
          <div v-if="editingBookId === book.id" class="book-name-edit">
            <input 
              v-model="editingBookName" 
              @keydown.enter="saveBookName(book)"
              @blur="saveBookName(book)"
              class="book-name-input"
              autofocus
            />
          </div>
          <h3 v-else @dblclick="startEditingName(book)" class="book-title" :title="book.customName || book.name">
            {{ truncateBookName(book.customName || book.name) }}
          </h3>
          <p class="book-info">{{ book.pages }} pages • Uploaded {{ book.uploadDate }}</p>
          <div class="book-actions">
            <button @click="$emit('select-book', book)" class="btn-primary">
              Start Learning
            </button>
            <button @click="deleteBook(book.id)" class="btn-danger">
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="empty-library">
      <p>Your library is empty. Upload a book to get started!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const base = import.meta.env.BASE_URL

const emit = defineEmits(['select-book'])

const books = ref([])
const editingBookId = ref(null)
const editingBookName = ref('')

onMounted(() => {
  const savedBooks = localStorage.getItem('academicBooks')
  if (savedBooks) {
    books.value = JSON.parse(savedBooks)
  }
})

const handleFileUpload = (event) => {
  const files = event.target.files
  if (files.length > 0) {
    uploadBook(files[0])
  }
}

const handleDrop = (event) => {
  const files = event.dataTransfer.files
  if (files.length > 0) {
    uploadBook(files[0])
  }
}

const uploadBook = (file) => {
  const newBook = {
    id: Date.now(),
    name: file.name.replace(/\.[^/.]+$/, ''),
    pages: Math.floor(Math.random() * 300) + 50,
    uploadDate: new Date().toLocaleDateString(),
    file: file
  }
  
  books.value.push(newBook)
  saveBooks()
  alert(`✓ "${newBook.name}" uploaded successfully!`)
}

const deleteBook = (bookId) => {
  if (confirm('Are you sure you want to delete this book?')) {
    books.value = books.value.filter(b => b.id !== bookId)
    saveBooks()
  }
}

const saveBooks = () => {
  localStorage.setItem('academicBooks', JSON.stringify(books.value))
}

const startEditingName = (book) => {
  editingBookId.value = book.id
  editingBookName.value = book.customName || book.name
}

const saveBookName = (book) => {
  if (editingBookName.value.trim()) {
    book.customName = editingBookName.value.trim()
    saveBooks()
  }
  editingBookId.value = null
  editingBookName.value = ''
}

const truncateBookName = (name) => {
  const maxLength = 25
  if (name.length <= maxLength) return name
  
  const parts = []
  const separators = []
  let currentPart = ''
  
  for (let i = 0; i < name.length; i++) {
    const char = name[i]
    if (char === ' ' || char === '_') {
      if (currentPart) {
        parts.push(currentPart)
        separators.push(char)
        currentPart = ''
      }
    } else {
      currentPart += char
    }
  }
  if (currentPart) parts.push(currentPart)
  
  let truncated = ''
  for (let i = 0; i < parts.length; i++) {
    const separator = i > 0 ? separators[i - 1] : ''
    const testString = truncated + separator + parts[i]
    if (testString.length > maxLength) break
    truncated = testString
  }
  
  return truncated + '...'
}
</script>

<style scoped>
.book-management {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.upload-section {
  margin-bottom: 2rem;
  flex-shrink: 0;
}

.upload-section h2 {
  margin-bottom: 0.75rem;
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  font-size: 1.3rem;
}

.upload-area {
  border: 3px dashed rgba(255, 255, 255, 0.4);
  border-radius: 12px;
  padding: 2rem;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
}

.upload-area:hover {
  border-color: rgba(255, 255, 255, 0.7);
  background: rgba(255, 255, 255, 0.25);
  box-shadow: 0 8px 32px rgba(255, 255, 255, 0.1);
}

.upload-placeholder {
  color: rgba(255, 255, 255, 0.95);
}

.upload-placeholder svg {
  color: #ffffff;
  margin-bottom: 0.5rem;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
  width: 40px;
  height: 40px;
}

.upload-placeholder p {
  font-size: 1rem;
  margin-bottom: 0.3rem;
  color: #ffffff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.file-types {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.8);
}

.books-library {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  overflow: hidden;
}

.books-library h2 {
  margin-bottom: 1.5rem;
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  font-size: 1.3rem;
  flex-shrink: 0;
}

.books-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
  padding: 0.75rem 0.25rem 1.5rem 0.25rem;
  overflow-y: auto;
  align-content: start;
}

.book-card {
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  padding: 1rem;
  text-align: center;
  transition: all 0.3s;
  backdrop-filter: blur(10px);
}

.book-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.5);
  background: rgba(255, 255, 255, 0.3);
}

.book-icon {
  width: 60px;
  height: 60px;
  margin-bottom: 0.75rem;
  filter: brightness(0) saturate(100%) invert(100%) sepia(0%) saturate(0%) hue-rotate(0deg) brightness(100%) contrast(100%) drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
}

.book-title {
  color: #ffffff;
  margin-bottom: 0.3rem;
  font-size: 1rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: color 0.2s;
  word-break: break-word;
}

.book-title:hover {
  color: rgba(255, 255, 255, 0.8);
}

.book-name-edit {
  margin-bottom: 0.3rem;
}

.book-name-input {
  width: 100%;
  padding: 0.3rem 0.5rem;
  border: 2px solid rgba(255, 255, 255, 0.5);
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.9);
  color: #2c3e50;
  font-size: 1rem;
  font-weight: bold;
  text-align: center;
  outline: none;
}

.book-name-input:focus {
  border-color: #8b5cf6;
}

.book-info {
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
}

.book-actions {
  display: flex;
  gap: 0.5rem;
  flex-direction: column;
}

.empty-library {
  text-align: center;
  padding: 3rem;
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

button {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 600;
  transition: all 0.3s;
}

button:hover:not(:disabled) {
  transform: translateY(-2px);
}

.btn-primary {
  background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.4);
}

.btn-danger {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(239, 68, 68, 0.4);
}

@media (max-width: 768px) {
  .books-grid {
    grid-template-columns: 1fr;
  }
}
</style>
