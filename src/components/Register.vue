<script setup>
import { ref } from 'vue'

const emit = defineEmits(['backToLogin', 'goToArticles'])

const avatarInputRef = ref(null)

const formData = ref({
  username: '',
  email: '',
  phoneNumber: '',
  password: '',
  confirmPassword: '',
  avatar: null,
  avatarPreview: null,
  bio: ''
})

const errors = ref({})
const isLoading = ref(false)

const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

const validatePhoneNumber = (phone) => {
  const phoneRegex = /^09\d{8}$/
  return phoneRegex.test(phone.replace(/\D/g, ''))
}

const validateForm = () => {
  errors.value = {}

  if (!formData.value.username.trim()) {
    errors.value.username = '請輸入使用者名稱'
  } else if (formData.value.username.length < 3) {
    errors.value.username = '使用者名稱至少需要 3 個字元'
  }

  if (!formData.value.email.trim()) {
    errors.value.email = '請輸入 Email'
  } else if (!validateEmail(formData.value.email)) {
    errors.value.email = 'Email 格式不正確'
  }

  if (!formData.value.phoneNumber.trim()) {
    errors.value.phoneNumber = '請輸入手機號碼'
  } else if (!validatePhoneNumber(formData.value.phoneNumber)) {
    errors.value.phoneNumber = '手機號碼格式不正確 (例: 0912345678)'
  }

  if (!formData.value.password) {
    errors.value.password = '請輸入密碼'
  } else if (formData.value.password.length < 6) {
    errors.value.password = '密碼至少需要 6 個字元'
  }

  if (!formData.value.confirmPassword) {
    errors.value.confirmPassword = '請確認密碼'
  } else if (formData.value.password !== formData.value.confirmPassword) {
    errors.value.confirmPassword = '密碼不一致'
  }

  return Object.keys(errors.value).length === 0
}

const handleAvatarChange = (event) => {
  const file = event.target.files[0]
  if (file) {
    formData.value.avatar = file
    const reader = new FileReader()
    reader.onload = (e) => {
      formData.value.avatarPreview = e.target.result
    }
    reader.readAsDataURL(file)
  }
}

const triggerAvatarInput = () => {
  avatarInputRef.value?.click()
}

const removeAvatar = () => {
  formData.value.avatar = null
  formData.value.avatarPreview = null
}

const handleRegister = async () => {
  if (!validateForm()) {
    return
  }

  isLoading.value = true

  try {
    // 將圖片轉換為 base64
    let coverImageBase64 = null
    if (formData.value.avatar) {
      coverImageBase64 = formData.value.avatarPreview
    }

    // 準備請求數據
    const requestData = {
      userName: formData.value.username,
      email: formData.value.email,
      phone: formData.value.phoneNumber,
      password: formData.value.password,
      coverImage: coverImageBase64,
      biography: formData.value.bio || ''
    }

    // 發送 API 請求
    const response = await fetch('http://localhost:8080/api/user/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(requestData)
    })

    const data = await response.json()

    if (response.ok) {
      alert(`註冊成功！`)
      // 重置表單
      formData.value = {
        username: '',
        email: '',
        phoneNumber: '',
        password: '',
        confirmPassword: '',
        avatar: null,
        avatarPreview: null,
        bio: ''
      }
      isLoading.value = false
      // 返回登入頁面
      emit('backToLogin')
    } else {
      alert(`註冊失敗: ${data.message || '請稍後重試'}`)
      isLoading.value = false
    }
  } catch (error) {
    console.error('註冊失敗:', error)
    alert(`註冊失敗: ${error.message}`)
    isLoading.value = false
  }
}

const handleKeydown = (event) => {
  if (event.key === 'Enter' && !event.target.tagName === 'TEXTAREA') {
    handleRegister()
  }
}
</script>

<template>
  <div class="register-container">
    <div class="register-card">
      <div class="register-header">
        <h1>建立帳戶</h1>
        <p>加入我們的社群媒體平台</p>
      </div>

      <form @submit.prevent="handleRegister" class="register-form">
        <!-- 使用者名稱 -->
        <div class="form-group">
          <label for="username" class="form-label">使用者名稱</label>
          <input
            id="username"
            v-model="formData.username"
            type="text"
            placeholder="請輸入使用者名稱"
            class="form-input"
            :class="{ 'input-error': errors.username }"
            @keydown="handleKeydown"
          />
          <span v-if="errors.username" class="error-message">
            {{ errors.username }}
          </span>
        </div>

        <!-- Email -->
        <div class="form-group">
          <label for="email" class="form-label">Email</label>
          <input
            id="email"
            v-model="formData.email"
            type="email"
            placeholder="example@email.com"
            class="form-input"
            :class="{ 'input-error': errors.email }"
            @keydown="handleKeydown"
          />
          <span v-if="errors.email" class="error-message">
            {{ errors.email }}
          </span>
        </div>

        <!-- 手機號碼 -->
        <div class="form-group">
          <label for="phone" class="form-label">手機號碼</label>
          <input
            id="phone"
            v-model="formData.phoneNumber"
            type="tel"
            placeholder="09XX-XXXXXX"
            class="form-input"
            :class="{ 'input-error': errors.phoneNumber }"
            @keydown="handleKeydown"
            maxlength="10"
          />
          <span v-if="errors.phoneNumber" class="error-message">
            {{ errors.phoneNumber }}
          </span>
        </div>

        <!-- 密碼 -->
        <div class="form-group">
          <label for="password" class="form-label">密碼</label>
          <input
            id="password"
            v-model="formData.password"
            type="password"
            placeholder="請輸入密碼"
            class="form-input"
            :class="{ 'input-error': errors.password }"
            @keydown="handleKeydown"
          />
          <span v-if="errors.password" class="error-message">
            {{ errors.password }}
          </span>
        </div>

        <!-- 確認密碼 -->
        <div class="form-group">
          <label for="confirmPassword" class="form-label">確認密碼</label>
          <input
            id="confirmPassword"
            v-model="formData.confirmPassword"
            type="password"
            placeholder="請再次輸入密碼"
            class="form-input"
            :class="{ 'input-error': errors.confirmPassword }"
            @keydown="handleKeydown"
          />
          <span v-if="errors.confirmPassword" class="error-message">
            {{ errors.confirmPassword }}
          </span>
        </div>

        <!-- 封面照片 -->
        <div class="form-group">
          <label for="avatar" class="form-label">封面照片 <span class="optional">(選填)</span></label>
          <div class="avatar-upload-area">
            <template v-if="!formData.avatarPreview">
              <input
                ref="avatarInputRef"
                id="avatar"
                type="file"
                accept="image/*"
                class="avatar-input"
                @change="handleAvatarChange"
              />
              <div class="avatar-placeholder" @click="triggerAvatarInput">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <path d="M21 19V5a2 2 0 0 0-2-2H5a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2z" />
                  <circle cx="8.5" cy="9.5" r="1.5" />
                  <polyline points="21 15 16 10 5 21" />
                </svg>
                <p>點擊上傳照片</p>
              </div>
            </template>
            <template v-else>
              <div class="avatar-preview">
                <img :src="formData.avatarPreview" alt="avatar preview" />
                <button type="button" class="remove-avatar-btn" @click="removeAvatar">
                  移除
                </button>
              </div>
            </template>
          </div>
        </div>

        <!-- 自我介紹 -->
        <div class="form-group">
          <label for="bio" class="form-label">自我介紹 <span class="optional">(選填)</span></label>
          <textarea
            id="bio"
            v-model="formData.bio"
            placeholder="介紹一下你自己..."
            class="form-textarea"
            maxlength="200"
          />
          <span class="char-count">{{ formData.bio.length }}/200</span>
        </div>

        <!-- 註冊按鈕 -->
        <button
          type="submit"
          class="register-button"
          :disabled="isLoading"
          :class="{ 'button-loading': isLoading }"
        >
          {{ isLoading ? '註冊中...' : '完成註冊' }}
        </button>
      </form>

      <!-- 返回登入 -->
      <div class="register-footer">
        <p>
          已有帳戶？
          <button type="button" class="back-link" @click="emit('backToLogin')">
            返回登入
          </button>
          <span class="divider"> | </span>
          <button type="button" class="back-link" @click="emit('goToArticles')">
            回到文章瀏覽頁面
          </button>
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.register-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  inset: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica',
    'Arial', sans-serif;
  overflow-y: auto;
}

.register-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 450px;
  padding: 30px;
}

.register-header {
  text-align: center;
  margin-bottom: 20px;
}

.register-header h1 {
  font-size: 24px;
  color: #333;
  margin: 0 0 6px 0;
  font-weight: 700;
}

.register-header p {
  font-size: 14px;
  color: #666;
  margin: 0;
  font-weight: 500;
}

.register-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-label {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  display: block;
}

.optional {
  font-size: 12px;
  font-weight: 400;
  color: #999;
}

.form-input,
.form-textarea {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s ease;
  font-family: inherit;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-input.input-error,
.form-textarea.input-error {
  border-color: #ff4757;
}

.form-input.input-error:focus,
.form-textarea.input-error:focus {
  box-shadow: 0 0 0 3px rgba(255, 71, 87, 0.1);
}

.form-textarea {
  resize: vertical;
  min-height: 80px;
}

.char-count {
  font-size: 12px;
  color: #999;
  text-align: right;
}

.error-message {
  font-size: 12px;
  color: #ff4757;
  font-weight: 500;
}

/* 頭像上傳 */
.avatar-upload-area {
  position: relative;
}

.avatar-input {
  display: none;
}

.avatar-placeholder {
  border: 2px dashed #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #999;
}

.avatar-placeholder:hover {
  border-color: #667eea;
  background-color: rgba(102, 126, 234, 0.05);
  color: #667eea;
}

.avatar-placeholder svg {
  margin-bottom: 10px;
}

.avatar-placeholder p {
  margin: 0;
  font-size: 14px;
  font-weight: 500;
}

.avatar-preview {
  position: relative;
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
  background: #f5f5f5;
}

.avatar-preview img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  display: block;
}

.remove-avatar-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 8px 16px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.remove-avatar-btn:hover {
  background: rgba(0, 0, 0, 0.9);
}

/* 註冊按鈕 */
.register-button {
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-top: 6px;
}

.register-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
}

.register-button:active:not(:disabled) {
  transform: translateY(0);
}

.register-button:disabled,
.button-loading {
  opacity: 0.7;
  cursor: not-allowed;
}

/* 返回登入 */
.register-footer {
  text-align: center;
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid #e0e0e0;
}

.register-footer p {
  margin: 0;
  font-size: 14px;
  color: #666;
}

.back-link {
  background: none;
  border: none;
  color: #667eea;
  cursor: pointer;
  font-weight: 600;
  transition: color 0.3s ease;
  text-decoration: underline;
}

.back-link:hover {
  color: #764ba2;
}

.divider {
  color: #ccc;
}

/* 響應式設計 */
@media (max-width: 480px) {
  .register-card {
    padding: 20px 15px;
  }

  .register-header h1 {
    font-size: 20px;
  }

  .register-form {
    gap: 12px;
  }
}
</style>
