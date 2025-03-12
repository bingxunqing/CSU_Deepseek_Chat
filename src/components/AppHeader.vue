<template>
  <div class="app-header">
    <div class="app-logo">
      <img src="../assets/logo.png" alt="Logo" />
    </div>
  </div>
</template>

<script setup></script>

<style scoped>
.app-header {
  height: 50px;
  background-color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 1px solid #e4e7ed;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
}

.app-logo {
  height: 100px;
  width: 170px;
}

.app-logo img {
  height: 100%;
  width: 100%;
  object-fit: contain;
}
</style>
