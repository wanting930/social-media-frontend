<script setup>
import { ref, onMounted, computed, reactive } from 'vue'

const emit = defineEmits(['goToLogin'])

const authToken = ref(localStorage.getItem('authToken'))
const userInfo = ref(JSON.parse(localStorage.getItem('userInfo') || 'null'))
const isLoggedIn = computed(() => !!authToken.value)

const posts = ref([])
const isLoading = ref(true)
const error = ref(null)
const commentInputs = reactive({})
const showCreateModal = ref(false)
const newPost = reactive({
  content: '',
  imageFile: null,
  imagePreview: ''
})
const isPosting = ref(false)
const showEditModal = ref(false)
const editingPost = ref(null)
const editPostData = reactive({
  content: '',
  imageFile: null,
  imagePreview: ''
})
const isUpdating = ref(false)

const getComments = (post) => {
  return post.comments || []
}

const addComment = async (post) => {
    const text = (commentInputs[post.postId] || '').trim()
    if (!text) {
      return
    }

    if (!authToken.value) {
      alert('請先登入後再留言。')
      emit('goToLogin')
      return
    }

    try {
      const response = await fetch(`http://localhost:8080/api/post/${post.postId}/comment`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${authToken.value}`
        },
        body: JSON.stringify({ content: text })
      })

      const result = await response.json()

      if (!response.ok || result.rtnCode !== '0000') {
        throw new Error(result.rtnMsg || '留言失敗，請稍後重試')
      }

      post.comments = post.comments || []
      post.comments.push({
        text,
        author: userInfo.value?.userName || '匿名',
        commentId: result.data?.commentId || `${Date.now()}`,
        createdAt: new Date().toISOString()
      })

      commentInputs[post.postId] = ''
    } catch (err) {
      console.error('留言失敗:', err)
      alert(err.message || '留言失敗，請稍後重試')
    }
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
      posts.value = data.data.map((post) => ({
        ...post,
        comments: Array.isArray(post.comments) ? post.comments : []
      }))
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

const handleImageUpload = async (event) => {
  const file = event.target.files?.[0]
  if (!file) {
    newPost.imageFile = null
    newPost.imagePreview = ''
    return
  }

  newPost.imageFile = file
  newPost.imagePreview = await new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => resolve(reader.result)
    reader.onerror = reject
    reader.readAsDataURL(file)
  })
}

const removeImage = () => {
  newPost.imageFile = null
  newPost.imagePreview = ''
  // 清除文件輸入
  const fileInput = document.querySelector('.modal-file-input')
  if (fileInput) fileInput.value = ''
}

const editPost = (post) => {
  editingPost.value = post
  editPostData.content = post.content
  editPostData.imageFile = null
  editPostData.imagePreview = post.image || ''
  showEditModal.value = true
}

const handleEditImageUpload = async (event) => {
  const file = event.target.files?.[0]
  if (!file) {
    editPostData.imageFile = null
    editPostData.imagePreview = editingPost.value?.image || ''
    return
  }

  editPostData.imageFile = file
  editPostData.imagePreview = await new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => resolve(reader.result)
    reader.onerror = reject
    reader.readAsDataURL(file)
  })
}

const removeEditImage = () => {
  editPostData.imageFile = null
  editPostData.imagePreview = ''
  // 清除文件輸入
  const fileInput = document.querySelector('.edit-modal-file-input')
  if (fileInput) fileInput.value = ''
}

const resetEditModal = () => {
  editingPost.value = null
  editPostData.content = ''
  editPostData.imageFile = null
  editPostData.imagePreview = ''
}

const updatePost = async () => {
  const content = editPostData.content.trim()
  if (!content) {
    alert('請輸入文章內容')
    return
  }

  if (!editingPost.value || !authToken.value) {
    alert('編輯失敗，請重新登入')
    return
  }

  isUpdating.value = true
  try {
    const response = await fetch(`http://localhost:8080/api/post/${editingPost.value.postId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${authToken.value}`
      },
      body: JSON.stringify({
        content,
        image: editPostData.imagePreview || ''
      })
    })

    const result = await response.json()
    if (!response.ok || result.rtnCode !== '0000') {
      throw new Error(result.rtnMsg || '編輯失敗，請稍後重試')
    }

    // 更新成功後重新載入文章列表
    await fetchPosts()

    resetEditModal()
    showEditModal.value = false
  } catch (err) {
    console.error('編輯失敗:', err)
    alert(err.message || '編輯失敗，請稍後重試')
  } finally {
    isUpdating.value = false
  }
}

const deletePost = async (post) => {
  if (!confirm('確定要刪除這篇文章嗎？此操作無法復原。')) {
    return
  }

  if (!authToken.value) {
    alert('請先登入')
    emit('goToLogin')
    return
  }

  try {
    const response = await fetch(`http://localhost:8080/api/post/${post.postId}`, {
      method: 'DELETE',
      headers: {
        Authorization: `Bearer ${authToken.value}`
      }
    })

    const result = await response.json()
    if (!response.ok || result.rtnCode !== '0000') {
      throw new Error(result.rtnMsg || '刪除失敗，請稍後重試')
    }

    // 刪除成功後重新載入文章列表
    await fetchPosts()
  } catch (err) {
    console.error('刪除失敗:', err)
    alert(err.message || '刪除失敗，請稍後重試')
  }
}

const resetCreateModal = () => {
  newPost.content = ''
  newPost.imageFile = null
  newPost.imagePreview = ''
}

const createPost = async () => {
  const content = newPost.content.trim()
  if (!content) {
    alert('請輸入文章內容')
    return
  }

  if (!authToken.value) {
    alert('請先登入才能新增文章')
    emit('goToLogin')
    return
  }

  isPosting.value = true
  try {
    const response = await fetch('http://localhost:8080/api/post', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${authToken.value}`
      },
      body: JSON.stringify({
        content,
        image: newPost.imagePreview || ''
      })
    })

    const result = await response.json()
    if (!response.ok || result.rtnCode !== '0000') {
      throw new Error(result.rtnMsg || '發文失敗，請稍後重試')
    }

    // 發文成功後重新載入文章列表
    await fetchPosts()

    resetCreateModal()
    showCreateModal.value = false
  } catch (err) {
    console.error('發文失敗:', err)
    alert(err.message || '發文失敗，請稍後重試')
  } finally {
    isPosting.value = false
  }
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
          <div class="header-actions">
            <button
              v-if="isLoggedIn"
              class="create-post-btn"
              type="button"
              @click="showCreateModal = true"
            >
              新增文章
            </button>
            <button class="login-btn" @click="isLoggedIn ? logout() : emit('goToLogin')">
              {{ isLoggedIn ? '登出' : '登入' }}
            </button>
          </div>
        </div>
      </div>

      <div v-if="showCreateModal" class="modal-overlay">
        <div class="modal-card">
          <div class="modal-header">
            <h2>新增文章</h2>
            <button type="button" class="modal-close" @click="showCreateModal = false">
              ×
            </button>
          </div>
          <div class="modal-body">
            <label class="modal-label">文章內容</label>
            <textarea
              v-model="newPost.content"
              placeholder="請輸入文章內容..."
              class="modal-textarea"
            ></textarea>
            <label class="modal-label">上傳圖片</label>
            <input
              type="file"
              accept="image/*"
              @change="handleImageUpload"
              class="modal-file-input"
            />
            <div v-if="newPost.imagePreview" class="image-preview">
              <img :src="newPost.imagePreview" alt="圖片預覽" />
              <button type="button" class="remove-image-btn" @click="removeImage">
                移除圖片
              </button>
            </div>
          </div>
          <div class="modal-actions">
            <button type="button" class="modal-cancel" @click="showCreateModal = false">
              取消
            </button>
            <button
              type="button"
              class="modal-submit"
              :disabled="isPosting || !newPost.content.trim()"
              @click="createPost"
            >
              {{ isPosting ? '發文中...' : '發表文章' }}
            </button>
          </div>
        </div>
      </div>

      <div v-if="showEditModal" class="modal-overlay">
        <div class="modal-card">
          <div class="modal-header">
            <h2>編輯文章</h2>
            <button type="button" class="modal-close" @click="showEditModal = false">
              ×
            </button>
          </div>
          <div class="modal-body">
            <label class="modal-label">文章內容</label>
            <textarea
              v-model="editPostData.content"
              placeholder="請輸入文章內容..."
              class="modal-textarea"
            ></textarea>
            <label class="modal-label">上傳圖片</label>
            <input
              type="file"
              accept="image/*"
              @change="handleEditImageUpload"
              class="modal-file-input edit-modal-file-input"
            />
            <div v-if="editPostData.imagePreview" class="image-preview">
              <img :src="editPostData.imagePreview" alt="圖片預覽" />
              <button type="button" class="remove-image-btn" @click="removeEditImage">
                移除圖片
              </button>
            </div>
          </div>
          <div class="modal-actions">
            <button type="button" class="modal-cancel" @click="showEditModal = false">
              取消
            </button>
            <button
              type="button"
              class="modal-submit"
              :disabled="isUpdating || !editPostData.content.trim()"
              @click="updatePost"
            >
              {{ isUpdating ? '更新中...' : '更新文章' }}
            </button>
          </div>
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
              <div class="post-actions">
                <div class="post-time">{{ formatDateTime(post.createdAt) }}</div>
                <button
                  v-if="post.userId === userInfo?.userId"
                  type="button"
                  class="edit-post-btn"
                  @click="editPost(post)"
                >
                  編輯
                </button>
                <button
                  v-if="post.userId === userInfo?.userId"
                  type="button"
                  class="delete-post-btn"
                  @click="deletePost(post)"
                >
                  刪除
                </button>
              </div>
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
                  @click="addComment(post)"
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
                <p v-if="getComments(post).length === 0" class="no-comments">
                  目前還沒有留言
                </p>
                <div
                  v-for="comment in getComments(post)"
                  :key="comment.commentId || comment.id"
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

.header-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

.create-post-btn {
  padding: 8px 16px;
  background: #4f7cff;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.create-post-btn:hover {
  background: #3d6ce0;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.48);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 50;
  padding: 20px;
}

.modal-card {
  width: 100%;
  max-width: 520px;
  background: white;
  border-radius: 20px;
  box-shadow: 0 24px 70px rgba(0, 0, 0, 0.18);
  padding: 28px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.modal-header h2 {
  margin: 0;
  font-size: 22px;
  color: #1f2b45;
}

.modal-close {
  width: 36px;
  height: 36px;
  border: none;
  background: #f3f4ff;
  color: #4d5afe;
  border-radius: 50%;
  font-size: 20px;
  cursor: pointer;
}

.modal-body {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.modal-label {
  font-size: 14px;
  color: #4f5d7a;
}

.modal-textarea,
.modal-file-input {
  width: 100%;
  padding: 14px 16px;
  border-radius: 12px;
  border: 1px solid #d1d7ee;
  font-size: 14px;
  font-family: inherit;
}

.modal-textarea {
  min-height: 140px;
  resize: vertical;
}

.image-preview {
  border-radius: 14px;
  overflow: hidden;
  background: #f7f8ff;
  padding: 8px;
  position: relative;
}

.image-preview img {
  width: 100%;
  display: block;
  object-fit: cover;
}

.remove-image-btn {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 6px 10px;
  background: rgba(255, 255, 255, 0.9);
  color: #ff4757;
  border: none;
  border-radius: 8px;
  font-size: 12px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
}

.remove-image-btn:hover {
  background: rgba(255, 255, 255, 1);
  transform: scale(1.05);
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 20px;
}

.modal-cancel,
.modal-submit {
  padding: 12px 20px;
  border: none;
  border-radius: 12px;
  font-weight: 600;
  cursor: pointer;
}

.modal-cancel {
  background: #eef1ff;
  color: #4f5d7a;
}

.modal-submit {
  background: #667eea;
  color: white;
}

.modal-submit:disabled {
  opacity: 0.65;
  cursor: not-allowed;
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

.post-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.edit-post-btn {
  padding: 4px 8px;
  background: #f0f4ff;
  color: #4f7cff;
  border: 1px solid #d1d7ee;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.edit-post-btn:hover {
  background: #e3e9ff;
}

.delete-post-btn {
  padding: 4px 8px;
  background: #fff0f0;
  color: #ff4757;
  border: 1px solid #ffd1d1;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.delete-post-btn:hover {
  background: #ffeaea;
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
