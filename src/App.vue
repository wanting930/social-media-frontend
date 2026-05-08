<script setup>
import { ref } from 'vue'
import Login from './components/Login.vue'
import Register from './components/Register.vue'
import Articles from './components/Articles.vue'

const currentPage = ref('articles')

const goToRegister = () => {
  currentPage.value = 'register'
}

const goToLogin = () => {
  currentPage.value = 'login'
}

const goToArticles = () => {
  currentPage.value = 'articles'
}
</script>

<template>
  <div id="app">
    <Login
      v-if="currentPage === 'login'"
      @goToRegister="goToRegister"
      @loginSuccess="goToArticles"
      @goToArticles="goToArticles"
    />
    <Register
      v-else-if="currentPage === 'register'"
      @backToLogin="goToLogin"
      @goToArticles="goToArticles"
    />
    <Articles v-else-if="currentPage === 'articles'" @goToLogin="goToLogin" />
  </div>
</template>

<style>
html,
body,
#app {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
