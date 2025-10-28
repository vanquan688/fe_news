<template>
  <div class="home-page">
    <!-- Hero Section -->
    <section class="hero">
      <h1 class="hero-title">📰 Chào mừng đến News Portal</h1>
      <p class="hero-subtitle">Khám phá tin tức mới nhất và đáng tin cậy nhất</p>
    </section>

    <!-- Search Box -->
    <div class="search-section">
      <div class="search-box">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
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
          placeholder="Tìm kiếm tin tức theo tiêu đề..."
          @input="handleSearch"
        />
        <button v-if="searchQuery" @click="clearSearch" class="clear-btn">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="18"
            height="18"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
    </div>

    <!-- Categories Filter -->
    <div class="filter-bar">
      <button
        @click="selectCategory(null)"
        class="filter-btn"
        :class="{ active: !selectedCategory }"
      >
        Tất cả
      </button>
      <button
        v-for="cat in categories"
        :key="cat._id"
        @click="selectCategory(cat._id)"
        class="filter-btn"
        :class="{ active: selectedCategory === cat._id }"
      >
        {{ cat.name }}
      </button>
    </div>

    <!-- News Grid -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Đang tải tin tức...</p>
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
      <p>Không có tin tức nào</p>
    </div>

    <div v-else class="news-grid">
      <article
        v-for="news in newsList"
        :key="news._id"
        class="news-card"
        @click="viewNews(news._id)"
      >
        <div class="news-card-badge">
          <span class="category-badge">{{ news.category?.name || 'N/A' }}</span>
        </div>

        <h2 class="news-card-title">{{ news.title }}</h2>

        <p class="news-card-excerpt">
          {{ truncateText(news.content, 150) }}
        </p>

        <div class="news-card-footer">
          <div class="author-info">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
              <circle cx="12" cy="7" r="4"></circle>
            </svg>
            <span>{{ news.author }}</span>
          </div>
          <div class="date-info">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
              <line x1="16" y1="2" x2="16" y2="6"></line>
              <line x1="8" y1="2" x2="8" y2="6"></line>
              <line x1="3" y1="10" x2="21" y2="10"></line>
            </svg>
            <span>{{ formatDate(news.publishedDate) }}</span>
          </div>
        </div>

        <div class="news-card-action">
          <span class="read-more">Đọc thêm →</span>
        </div>
      </article>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { useRouter } from 'vue-router'
import { newsService } from '@/services/news.service'
import { categoryService } from '@/services/category.service'

const router = useRouter()
const newsList = ref([])
const categories = ref([])
const selectedCategory = ref(null)
const searchQuery = ref('')
const loading = ref(true)
let searchTimeout = null

const loadNews = async () => {
  try {
    loading.value = true
    const params = {}
    
    // Add search parameter
    if (searchQuery.value) {
      params.search = searchQuery.value
    }
    
    // Add category filter parameter
    if (selectedCategory.value) {
      params.category = selectedCategory.value
    }
    
    newsList.value = await newsService.getAllNews(params)
    console.log('🏠 Total news loaded:', newsList.value.length)
  } catch (error) {
    console.error('❌ Error loading news:', error)
    newsList.value = []
  } finally {
    loading.value = false
  }
}

const loadCategories = async () => {
  try {
    categories.value = await categoryService.getAllCategories()
    console.log('📂 Total categories loaded:', categories.value.length)
  } catch (error) {
    console.error('❌ Error loading categories:', error)
    categories.value = []
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

const clearSearch = () => {
  // Clear timeout để tránh gọi API không cần thiết
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }
  searchQuery.value = ''
  loadNews()
}

const selectCategory = (categoryId) => {
  selectedCategory.value = categoryId
  loadNews()
}

const viewNews = (id) => {
  router.push(`/news/${id}`)
}

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('vi-VN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
}

const truncateText = (text, length) => {
  if (!text) return ''
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
.home-page {
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Hero Section */
.hero {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px;
  padding: 60px 40px;
  text-align: center;
  margin-bottom: 40px;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.3);
  animation: slideDown 0.6s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-title {
  margin: 0 0 16px 0;
  font-size: 48px;
  color: white;
  font-weight: 700;
}

.hero-subtitle {
  margin: 0;
  font-size: 20px;
  color: rgba(255, 255, 255, 0.9);
}

/* Search Section */
.search-section {
  margin-bottom: 30px;
  animation: slideUp 0.6s ease-out;
}

.search-box {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 20px;
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 16px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.search-box:focus-within {
  border-color: #667eea;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.15);
}

.search-box svg {
  color: #667eea;
  flex-shrink: 0;
}

.search-box input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 16px;
  color: #333;
}

.search-box input::placeholder {
  color: #999;
}

.clear-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  background: #f5f7fa;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
  flex-shrink: 0;
}

.clear-btn:hover {
  background: #667eea;
  transform: scale(1.1);
}

.clear-btn:hover svg {
  color: white;
}

.clear-btn svg {
  color: #999;
  transition: color 0.3s ease;
}

/* Filter Bar */
.filter-bar {
  display: flex;
  gap: 12px;
  margin-bottom: 40px;
  flex-wrap: wrap;
  justify-content: center;
  animation: slideUp 0.7s ease-out;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.filter-btn {
  padding: 10px 24px;
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 25px;
  color: #666;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
}

.filter-btn:hover {
  border-color: #667eea;
  color: #667eea;
  transform: translateY(-2px);
}

.filter-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: transparent;
  color: white;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

/* Loading */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 20px;
  color: #666;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid #f0f0f0;
  border-top-color: #667eea;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin-bottom: 20px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 80px 20px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.empty-state svg {
  color: #ccc;
  margin-bottom: 20px;
}

.empty-state p {
  color: #666;
  font-size: 18px;
  margin: 0;
}

/* News Grid */
.news-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 30px;
  animation: fadeInUp 0.8s ease-out;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.news-card {
  background: white;
  border-radius: 16px;
  padding: 28px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.news-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.news-card:hover::before {
  transform: scaleX(1);
}

.news-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.12);
}

.news-card-badge {
  margin-bottom: 16px;
}

.category-badge {
  display: inline-block;
  padding: 6px 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.news-card-title {
  margin: 0 0 16px 0;
  font-size: 22px;
  color: #333;
  line-height: 1.4;
  font-weight: 700;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.news-card-excerpt {
  margin: 0 0 20px 0;
  color: #666;
  font-size: 15px;
  line-height: 1.7;
}

.news-card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16px;
  border-top: 1px solid #f0f0f0;
  margin-bottom: 16px;
}

.author-info,
.date-info {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #999;
  font-size: 13px;
}

.author-info svg,
.date-info svg {
  color: #667eea;
}

.news-card-action {
  display: flex;
  justify-content: flex-end;
}

.read-more {
  color: #667eea;
  font-weight: 600;
  font-size: 14px;
  transition: all 0.3s ease;
}

.news-card:hover .read-more {
  transform: translateX(4px);
}

@media (max-width: 768px) {
  .hero {
    padding: 40px 24px;
  }

  .hero-title {
    font-size: 32px;
  }

  .hero-subtitle {
    font-size: 16px;
  }

  .search-box {
    padding: 12px 16px;
  }

  .search-box input {
    font-size: 14px;
  }

  .filter-bar {
    gap: 8px;
  }

  .filter-btn {
    padding: 8px 16px;
    font-size: 13px;
  }

  .news-grid {
    grid-template-columns: 1fr;
  }
}
</style>
