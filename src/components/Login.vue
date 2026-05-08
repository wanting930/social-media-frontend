<script setup>
import { ref } from 'vue'

const emit = defineEmits(['goToRegister', 'loginSuccess'])

const phoneNumber = ref('')
const password = ref('')
const errors = ref({})
const isLoading = ref(false)

const validatePhoneNumber = (phone) => {
  // 台灣手機號碼驗證 (09XX-XXXXXX)
  const phoneRegex = /^09\d{8}$/
  return phoneRegex.test(phone.replace(/\D/g, ''))
}

const validateForm = () => {
  errors.value = {}

  if (!phoneNumber.value.trim()) {
    errors.value.phoneNumber = '請輸入手機號碼'
  } else if (!validatePhoneNumber(phoneNumber.value)) {
    errors.value.phoneNumber = '手機號碼格式不正確 (例: 0912345678)'
  }

  if (!password.value) {
    errors.value.password = '請輸入密碼'
  } else if (password.value.length < 6) {
    errors.value.password = '密碼至少需要 6 個字元'
  }

  return Object.keys(errors.value).length === 0
}

const handleLogin = async () => {
  if (!validateForm()) {
    return
  }

  isLoading.value = true

  try {
    // 準備請求數據
    const requestData = {
      phone: phoneNumber.value,
      password: password.value
    }

    // 發送 API 請求
    const response = await fetch('http://localhost:8080/api/user/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(requestData)
    })

    const data = await response.json()

    if (response.ok) {
      // 清空表單
      phoneNumber.value = ''
      password.value = ''
      // 儲存 token 到 localStorage
      if (data.token) {
        localStorage.setItem('authToken', data.token)
      }
      isLoading.value = false
      emit('loginSuccess')
    } else {
      alert(`登入失敗: ${data.message || '手機號碼或密碼錯誤'}`)
      isLoading.value = false
    }
  } catch (error) {
    console.error('登入失敗:', error)
    alert(`登入失敗: ${error.message}`)
    isLoading.value = false
  }
}

const handleKeydown = (event) => {
  if (event.key === 'Enter') {
    handleLogin()
  }
}
</script>

<template>
  <div class="login-container">
    <div class="login-card">
      <div class="login-header">
        <h1>社群媒體平台</h1>
        <p>歡迎登入</p>
      </div>

      <form @submit.prevent="handleLogin" class="login-form">
        <!-- 手機號碼輸入 -->
        <div class="form-group">
          <label for="phone" class="form-label">手機號碼</label>
          <input
            id="phone"
            v-model="phoneNumber"
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

        <!-- 密碼輸入 -->
        <div class="form-group">
          <label for="password" class="form-label">密碼</label>
          <input
            id="password"
            v-model="password"
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

        <!-- 登入按鈕 -->
        <button
          type="submit"
          class="login-button"
          :disabled="isLoading"
          :class="{ 'button-loading': isLoading }"
        >
          {{ isLoading ? '登入中...' : '登入' }}
        </button>
      </form>

      <!-- 額外選項 -->
      <div class="login-footer">
        <a href="#" class="link" @click.prevent="emit('goToRegister')">註冊帳戶</a>
      </div>
    </div>
  </div>
</template>

<style scoped>
:global(html),
:global(body),
:global(#app) {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  inset: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica',
    'Arial', sans-serif;
}

.login-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 400px;
  padding: 40px;
}

.login-header {
  text-align: center;
  margin-bottom: 30px;
}

.login-header h1 {
  font-size: 28px;
  color: #333;
  margin: 0 0 8px 0;
  font-weight: 700;
}

.login-header p {
  font-size: 16px;
  color: #666;
  margin: 0;
  font-weight: 500;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-label {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  display: block;
}

.form-input {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s ease;
  font-family: inherit;
}

.form-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-input.input-error {
  border-color: #ff4757;
}

.form-input.input-error:focus {
  box-shadow: 0 0 0 3px rgba(255, 71, 87, 0.1);
}

.error-message {
  font-size: 12px;
  color: #ff4757;
  font-weight: 500;
}

.login-button {
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-top: 10px;
}

.login-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
}

.login-button:active:not(:disabled) {
  transform: translateY(0);
}

.login-button:disabled,
.button-loading {
  opacity: 0.7;
  cursor: not-allowed;
}

.login-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid #e0e0e0;
}

.link {
  font-size: 14px;
  color: #667eea;
  text-decoration: none;
  transition: color 0.3s ease;
  font-weight: 500;
}

.link:hover {
  color: #764ba2;
  text-decoration: underline;
}

.divider {
  color: #ccc;
}

/* 響應式設計 */
@media (max-width: 480px) {
  .login-card {
    padding: 30px 20px;
  }

  .login-header h1 {
    font-size: 24px;
  }
}
</style>
