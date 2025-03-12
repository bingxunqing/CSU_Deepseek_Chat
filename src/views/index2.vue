<template>
  <div class="app-container">
    <Header />
    <div class="content-wrapper">
      <Sidebar />
      <!-- <ChatHeader /> -->
    </div>
  </div>
</template>

<script setup>
// import { he } from 'element-plus/es/locales.mjs';
import Header from '../components/AppHeader.vue'
import Sidebar from '../components/Sidebar.vue'
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
  background-color: #f0f2f5;
}

.content-wrapper {
  display: flex;
  flex: 1;
  overflow: hidden;
}
</style>
