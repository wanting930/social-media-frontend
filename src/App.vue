<script setup>
import { ref } from 'vue'
import Login from './components/Login.vue'
import Register from './components/Register.vue'

const currentPage = ref('login')

const goToRegister = () => {
  currentPage.value = 'register'
}

const goToLogin = () => {
  currentPage.value = 'login'
}
</script>

<template>
  <div id="app">
    <Login v-if="currentPage === 'login'" @goToRegister="goToRegister" />
    <Register v-else-if="currentPage === 'register'" @backToLogin="goToLogin" />
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
