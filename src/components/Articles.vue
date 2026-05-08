<script setup>
import { ref, onMounted, computed, reactive } from 'vue'

const emit = defineEmits(['goToLogin'])

const authToken = ref(localStorage.getItem('authToken'))
const userInfo = ref(JSON.parse(localStorage.getItem('userInfo') || 'null'))
const isLoggedIn = computed(() => !!authToken.value)

const posts = ref([])
const isLoading = ref(true)
const error = ref(null)
const comments = ref([])
const commentInputs = reactive({})

const getComments = (postId) => {
  return comments.value.filter((comment) => comment.postId === postId)
}

const addComment = (postId) => {
    const text = (commentInputs[postId] || '').trim()
    if (!text) {
      return
    }

    comments.value.push({
      id: `${postId}-${Date.now()}`,
      postId,
      author: userInfo.value?.userName || '匿名',
      text,
      createdAt: new Date().toISOString()
    })

    commentInputs[postId] = ''
}

const logout = () => {
  localStorage.removeItem('authToken')
  localStorage.removeItem('userInfo')
  authToken.value = null
  userInfo.value = null
}

const fetchPosts = async () => {
  try {
    isLoading.value = true
    error.value = null

    const response = await fetch('http://localhost:8080/api/posts')

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()

    if (data.rtnCode === '0000') {
      posts.value = data.data
    } else {
      throw new Error(data.rtnMsg || '取得文章失敗')
    }
  } catch (err) {
    console.error('取得文章失敗:', err)
    error.value = err.message
  } finally {
    isLoading.value = false
  }
}

const formatDateTime = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleString('zh-TW', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

onMounted(() => {
  fetchPosts()
})

const handleImageError = (event) => {
  event.target.style.display = 'none'
}
</script>

<template>
  <div class="articles-container">
    <div class="articles-card">
      <div class="articles-header">
        <div class="header-content">
          <div>
            <h1>瀏覽文章</h1>
            <p>歡迎回來！以下是最新文章列表。</p>
          </div>
          <button class="login-btn" @click="isLoggedIn ? logout() : emit('goToLogin')">
            {{ isLoggedIn ? '登出' : '登入' }}
          </button>
        </div>
      </div>

      <div class="articles-list">
        <!-- 載入中 -->
        <div v-if="isLoading" class="loading-state">
          <div class="loading-spinner"></div>
          <p>載入文章中...</p>
        </div>

        <!-- 錯誤狀態 -->
        <div v-else-if="error" class="error-state">
          <p>❌ {{ error }}</p>
          <button class="retry-btn" @click="fetchPosts">重試</button>
        </div>

        <!-- 文章列表 -->
        <div v-else-if="posts.length > 0" class="posts-grid">
          <article v-for="post in posts" :key="post.postId" class="article-item">
            <div class="article-header">
              <div class="author-info">
                <span class="author-name">{{ post.userName }}</span>
                <span class="post-id">#{{ post.postId }}</span>
              </div>
              <div class="post-time">{{ formatDateTime(post.createdAt) }}</div>
            </div>

            <div class="article-content">
              <p class="content-text">{{ post.content }}</p>

              <div v-if="post.image && post.image.startsWith('data:image')" class="content-image">
                <img :src="post.image" :alt="`文章 ${post.postId} 的圖片`" @error="handleImageError" />
              </div>
            </div>

            <div class="article-comments">
              <div v-if="isLoggedIn" class="comment-form">
                <textarea
                  v-model="commentInputs[post.postId]"
                  placeholder="寫下你的留言..."
                  class="comment-textarea"
                ></textarea>
                <button
                  type="button"
                  class="comment-submit"
                  :disabled="!(commentInputs[post.postId] || '').trim()"
                  @click="addComment(post.postId)"
                >
                  送出留言
                </button>
              </div>
              <div v-else class="comment-login-prompt">
                <p>登入後即可在此留言。</p>
                <button type="button" class="comment-login-btn" @click="emit('goToLogin')">
                  前往登入
                </button>
              </div>

              <div class="comment-list">
                <p v-if="getComments(post.postId).length === 0" class="no-comments">
                  目前還沒有留言
                </p>
                <div
                  v-for="comment in getComments(post.postId)"
                  :key="comment.id"
                  class="comment-item"
                >
                  <div class="comment-meta">
                    <span class="comment-author">{{ comment.author }}</span>
                    <span class="comment-time">{{ formatDateTime(comment.createdAt) }}</span>
                  </div>
                  <p class="comment-text">{{ comment.text }}</p>
                </div>
              </div>
            </div>
          </article>
        </div>

        <!-- 無文章 -->
        <div v-else class="empty-state">
          <p>目前沒有文章</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.articles-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  inset: 0;
  background: linear-gradient(135deg, #f5f7ff 0%, #dfe7fd 100%);
  padding: 20px;
}

.articles-card {
  width: 100%;
  max-width: 900px;
  background: white;
  border-radius: 18px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.12);
  padding: 36px;
}

.articles-header {
  margin-bottom: 24px;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.header-content h1 {
  margin: 0 0 10px 0;
  font-size: 32px;
  color: #222;
}

.header-content p {
  margin: 0;
  color: #555;
  font-size: 16px;
}

.login-btn {
  padding: 8px 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.login-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.articles-list {
  display: grid;
  gap: 18px;
}

/* 載入狀態 */
.loading-state {
  text-align: center;
  padding: 40px;
  color: #666;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 錯誤狀態 */
.error-state {
  text-align: center;
  padding: 40px;
  color: #ff4757;
}

.retry-btn {
  margin-top: 16px;
  padding: 8px 16px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
}

.retry-btn:hover {
  background: #5a67d8;
}

/* 空狀態 */
.empty-state {
  text-align: center;
  padding: 40px;
  color: #666;
}

/* 文章網格 */
.posts-grid {
  display: grid;
  gap: 18px;
}

/* 文章項目 */
.article-item {
  padding: 24px;
  background: #f7f9ff;
  border-radius: 14px;
  border: 1px solid #e3e9ff;
  transition: all 0.3s ease;
}

.article-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

/* 文章標題 */
.article-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid #e3e9ff;
}

.author-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.author-name {
  font-weight: 600;
  color: #1f2b45;
  font-size: 16px;
}

.post-id {
  font-size: 12px;
  color: #667eea;
  background: rgba(102, 126, 234, 0.1);
  padding: 2px 8px;
  border-radius: 12px;
  font-weight: 500;
}

.post-time {
  font-size: 12px;
  color: #888;
  font-weight: 500;
}

/* 文章內容 */
.article-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.content-text {
  margin: 0;
  color: #333;
  line-height: 1.6;
  font-size: 15px;
  white-space: pre-wrap;
}

.content-image {
  border-radius: 8px;
  overflow: hidden;
  background: #f5f5f5;
}

.content-image img {
  width: 100%;
  height: auto;
  max-height: 300px;
  object-fit: cover;
  display: block;
}

.article-comments {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #e3e9ff;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.comment-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.comment-textarea {
  width: 100%;
  min-height: 100px;
  padding: 12px 14px;
  border-radius: 10px;
  border: 1px solid #d1d7ee;
  resize: vertical;
  font-size: 14px;
  font-family: inherit;
}

.comment-submit,
.comment-login-btn {
  width: fit-content;
  padding: 10px 18px;
  border: none;
  border-radius: 10px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.comment-submit:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.comment-submit:hover:not(:disabled),
.comment-login-btn:hover {
  transform: translateY(-1px);
}

.comment-login-prompt {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
  color: #555;
}

.comment-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.no-comments {
  margin: 0;
  color: #777;
  font-size: 14px;
}

.comment-item {
  padding: 14px 16px;
  border-radius: 12px;
  background: #ffffff;
  border: 1px solid #e7ecff;
}

.comment-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
}

.comment-author {
  font-weight: 600;
  color: #1f2b45;
}

.comment-time {
  font-size: 12px;
  color: #888;
}

.comment-text {
  margin: 0;
  color: #333;
  line-height: 1.6;
  font-size: 14px;
}

@media (max-width: 640px) {
  .articles-card {
    padding: 24px;
  }

  .header-content {
    flex-direction: column;
    gap: 16px;
  }

  .articles-header h1 {
    font-size: 26px;
  }

  .login-btn {
    align-self: flex-end;
  }

  .article-item {
    padding: 16px;
  }

  .article-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .author-info {
    gap: 8px;
  }

  .content-image img {
    max-height: 200px;
  }
}
</style>
