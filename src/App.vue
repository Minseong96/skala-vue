<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header style="text-align: center; padding: 20px; margin-bottom: 20px">
    <h2 style="color: #fff; margin-bottom: 10px">과제 4: 라우터 적용 🌟</h2>
    <nav>
      <RouterLink
        to="/"
        style="margin-right: 15px; color: #60a5fa; font-weight: bold; text-decoration: none"
        >날씨 대시보드</RouterLink
      >
      |
      <RouterLink
        to="/about"
        style="margin-left: 15px; color: #60a5fa; font-weight: bold; text-decoration: none"
        >서비스 소개</RouterLink
      >
    </nav>
  </header>

  <RouterView />
</template>

<style scoped></style>
