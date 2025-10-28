<template>
  <div class="admin-news">
    <div class="page-header">
      <div>
        <h2>📰 Quản lý Tin tức</h2>
        <p>Tạo và quản lý các tin tức</p>
      </div>
      <button @click="showCreateModal = true" class="btn-primary">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <line x1="12" y1="5" x2="12" y2="19"></line>
          <line x1="5" y1="12" x2="19" y2="12"></line>
        </svg>
        Thêm tin tức mới
      </button>
    </div>

    <!-- Search & Filter -->
    <div class="filter-section">
      <div class="search-box">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <circle cx="11" cy="11" r="8"></circle>
          <path d="m21 21-4.35-4.35"></path>
        </svg>
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Tìm kiếm tin tức..."
          @input="handleSearch"
        />
      </div>

      <select v-model="filterCategory" @change="handleFilter" class="filter-select">
        <option value="">Tất cả danh mục</option>
        <option v-for="cat in categories" :key="cat._id" :value="cat._id">
          {{ cat.name }}
        </option>
      </select>
    </div>

    <!-- News Table -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Đang tải...</p>
    </div>

    <div v-else-if="newsList.length === 0" class="empty-state">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="64"
        height="64"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="1"
      >
        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
        <polyline points="14 2 14 8 20 8"></polyline>
      </svg>
      <p>Chưa có tin tức nào</p>
      <button @click="showCreateModal = true" class="btn-secondary">Tạo tin tức đầu tiên</button>
    </div>

    <div v-else class="news-grid">
      <div v-for="news in newsList" :key="news._id" class="news-card">
        <div class="news-card-header">
          <span class="category-tag">{{ news.category?.name || 'N/A' }}</span>
          <div class="news-actions">
            <button @click="editNews(news)" class="btn-icon" title="Chỉnh sửa">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
              >
                <path d="M17 3a2.828 2.828 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"></path>
              </svg>
            </button>
            <button @click="confirmDelete(news)" class="btn-icon btn-danger" title="Xóa">
              🗑️
            </button>
          </div>
        </div>
        <h3 class="news-title">{{ news.title }}</h3>
        <p class="news-content">{{ truncateText(news.content, 120) }}</p>
        <div class="news-footer">
          <span class="author">👤 {{ news.author }}</span>
          <span class="date">📅 {{ formatDate(news.publishedDate) }}</span>
        </div>
      </div>
    </div>

    <!-- Create/Edit Modal -->
    <div v-if="showCreateModal || editingNews" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <div class="modal-header">
          <h3>{{ editingNews ? '✏️ Chỉnh sửa tin tức' : '➕ Thêm tin tức mới' }}</h3>
          <button @click="closeModal" class="btn-close">&times;</button>
        </div>

        <form @submit.prevent="handleSubmit" class="modal-body">
          <div v-if="error" class="error-message">{{ error }}</div>

          <div class="form-group">
            <label>Tiêu đề *</label>
            <input
              type="text"
              v-model="formData.title"
              placeholder="Nhập tiêu đề tin tức"
              required
            />
          </div>

          <div class="form-group">
            <label>Nội dung *</label>
            <textarea
              v-model="formData.content"
              placeholder="Nhập nội dung tin tức"
              rows="6"
              required
            ></textarea>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label>Tác giả *</label>
              <input type="text" v-model="formData.author" placeholder="Tên tác giả" required />
            </div>

            <div class="form-group">
              <label>Danh mục *</label>
              <select v-model="formData.category" required>
                <option value="">Chọn danh mục</option>
                <option v-for="cat in categories" :key="cat._id" :value="cat._id">
                  {{ cat.name }}
                </option>
              </select>
            </div>
          </div>

          <div class="modal-footer">
            <button type="button" @click="closeModal" class="btn-secondary">Hủy</button>
            <button type="submit" class="btn-primary" :disabled="submitting">
              {{ submitting ? 'Đang lưu...' : editingNews ? 'Cập nhật' : 'Tạo mới' }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div v-if="deletingNews" class="modal-overlay" @click.self="deletingNews = null">
      <div class="modal modal-small">
        <div class="modal-header">
          <h3>⚠️ Xác nhận xóa</h3>
        </div>
        <div class="modal-body">
          <p>
            Bạn có chắc chắn muốn xóa tin tức "<strong>{{ deletingNews.title }}</strong
            >"?
          </p>
          <p style="color: #f5576c; font-size: 14px">Hành động này không thể hoàn tác!</p>
        </div>
        <div class="modal-footer">
          <button @click="deletingNews = null" class="btn-secondary">Hủy</button>
          <button @click="handleDelete" class="btn-danger" :disabled="submitting">
            {{ submitting ? 'Đang xóa...' : 'Xóa' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { newsService } from '@/services/news.service'
import { categoryService } from '@/services/category.service'

const newsList = ref([])
const categories = ref([])
const loading = ref(true)
const showCreateModal = ref(false)
const editingNews = ref(null)
const deletingNews = ref(null)
const submitting = ref(false)
const error = ref('')
const searchQuery = ref('')
const filterCategory = ref('')
let searchTimeout = null

const formData = ref({
  title: '',
  content: '',
  author: '',
  category: '',
})

const loadNews = async () => {
  try {
    loading.value = true
    const params = {}
    if (filterCategory.value) params.category = filterCategory.value
    if (searchQuery.value) params.search = searchQuery.value

    const data = await newsService.getAllNews(params)
    newsList.value = data.news || data || []
  } finally {
    loading.value = false
  }
}

const loadCategories = async () => {
  try {
    categories.value = await categoryService.getAllCategories()
  } catch (err) {
    console.error('Error loading categories:', err)
  }
}

const handleSearch = () => {
  // Clear timeout cũ nếu có
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }
  
  // Tạo timeout mới - chỉ gọi API sau 500ms khi người dùng ngừng nhập
  searchTimeout = setTimeout(() => {
    loadNews()
  }, 500)
}

const handleFilter = () => {
  // Clear timeout để tránh gọi API không cần thiết khi đổi filter
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }
  loadNews()
}

const editNews = (news) => {
  editingNews.value = news
  formData.value = {
    title: news.title,
    content: news.content,
    author: news.author,
    category: news.category?._id || news.category,
  }
}

const confirmDelete = (news) => {
  deletingNews.value = news
}

const handleSubmit = async () => {
  error.value = ''
  submitting.value = true

  try {
    if (editingNews.value) {
      await newsService.updateNews(editingNews.value._id, formData.value)
    } else {
      await newsService.createNews(formData.value)
    }
    closeModal()
    loadNews()
  } catch (err) {
    error.value = err.response?.data?.message || 'Có lỗi xảy ra'
  } finally {
    submitting.value = false
  }
}

const handleDelete = async () => {
  submitting.value = true
  try {
    await newsService.deleteNews(deletingNews.value._id)
    deletingNews.value = null
    loadNews()
  } catch {
    alert('Có lỗi xảy ra khi xóa tin tức')
  } finally {
    submitting.value = false
  }
}

const closeModal = () => {
  showCreateModal.value = false
  editingNews.value = null
  formData.value = {
    title: '',
    content: '',
    author: '',
    category: '',
  }
  error.value = ''
}

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('vi-VN')
}

const truncateText = (text, length) => {
  return text.length > length ? text.substring(0, length) + '...' : text
}

onMounted(() => {
  loadNews()
  loadCategories()
})

onBeforeUnmount(() => {
  // Clear timeout để tránh memory leak
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }
})
</script>

<style scoped>
.admin-news {
  max-width: 1400px;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.page-header h2 {
  margin: 0 0 8px 0;
  font-size: 28px;
  color: #333;
}

.page-header p {
  margin: 0;
  color: #666;
  font-size: 14px;
}

.btn-primary {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-secondary {
  padding: 12px 24px;
  background: #f5f7fa;
  color: #333;
  border: none;
  border-radius: 10px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-secondary:hover {
  background: #e0e0e0;
}

.btn-danger {
  padding: 12px 24px;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-danger:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(245, 87, 108, 0.3);
}

.filter-section {
  display: flex;
  gap: 16px;
  margin-bottom: 24px;
}

.search-box {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  transition: border-color 0.3s ease;
}

.search-box:focus-within {
  border-color: #667eea;
}

.search-box svg {
  color: #999;
}

.search-box input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 14px;
}

.filter-select {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 14px;
  background: white;
  cursor: pointer;
  min-width: 200px;
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px;
  color: #666;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f0f0f0;
  border-top-color: #667eea;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.empty-state {
  text-align: center;
  padding: 60px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.empty-state svg {
  color: #ccc;
  margin-bottom: 16px;
}

.empty-state p {
  color: #666;
  margin: 16px 0 24px;
}

.news-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 24px;
}

.news-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  animation: fadeInUp 0.5s ease-out;
}

.news-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.news-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.category-tag {
  display: inline-block;
  padding: 4px 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.news-actions {
  display: flex;
  gap: 8px;
}

.btn-icon {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f7fa;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #666;
}

.btn-icon:hover {
  background: #e0e0e0;
  transform: scale(1.1);
  color: #333;
}

.btn-icon.btn-danger {
  color: #f5576c;
}

.btn-icon.btn-danger:hover {
  background: #ffe0e0;
  color: #dc3545;
}

.news-title {
  margin: 0 0 12px 0;
  font-size: 18px;
  color: #333;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.news-content {
  margin: 0 0 16px 0;
  color: #666;
  font-size: 14px;
  line-height: 1.6;
}

.news-footer {
  display: flex;
  justify-content: space-between;
  padding-top: 16px;
  border-top: 1px solid #f0f0f0;
  font-size: 13px;
  color: #999;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal {
  background: white;
  border-radius: 16px;
  width: 100%;
  max-width: 600px;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  animation: slideUpModal 0.3s ease-out;
}

.modal-small {
  max-width: 450px;
}

@keyframes slideUpModal {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24px;
  border-bottom: 1px solid #f0f0f0;
}

.modal-header h3 {
  margin: 0;
  font-size: 20px;
  color: #333;
}

.btn-close {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f7fa;
  border: none;
  border-radius: 8px;
  font-size: 24px;
  color: #666;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-close:hover {
  background: #e0e0e0;
  transform: rotate(90deg);
}

.modal-body {
  padding: 24px;
  overflow-y: auto;
}

.error-message {
  background: #fee;
  color: #c33;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 20px;
  font-size: 14px;
  border: 1px solid #fcc;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  font-weight: 600;
  color: #333;
  margin-bottom: 8px;
  font-size: 14px;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 14px;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  outline: none;
  border-color: #667eea;
}

.form-group textarea {
  resize: vertical;
  min-height: 100px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 20px 24px;
  border-top: 1px solid #f0f0f0;
  background: #f8f9fa;
}

@media (max-width: 768px) {
  .filter-section {
    flex-direction: column;
  }

  .news-grid {
    grid-template-columns: 1fr;
  }

  .form-row {
    grid-template-columns: 1fr;
  }

  .page-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }
}
</style>
