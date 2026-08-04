<!-- src/App.vue -->
<script setup>
import { RouterLink, RouterView } from 'vue-router'
import UnitToggler from '@/components/exercise/UnitToggler.vue'
</script>

<template>
  <!-- 🌟 전체 배경에 밝은 파스텔 블루 적용 -->
  <div class="app-container">
    <header class="app-header">
      <!-- ... 헤더 내용 동일 ... -->
      <div>
        <h2 style="color: #1f2937; margin: 0 0 5px 0; font-size: 18px">과제 5: 스토어 적용 🌟</h2>
        <nav>
          <RouterLink
            to="/"
            style="margin-right: 15px; color: #2563eb; font-weight: bold; text-decoration: none"
            >날씨 대시보드</RouterLink
          >
          |
          <RouterLink
            to="/about"
            style="margin-left: 15px; color: #2563eb; font-weight: bold; text-decoration: none"
            >서비스 소개</RouterLink
          >
        </nav>
      </div>
      <UnitToggler />
    </header>

    <RouterView />
  </div>
</template>

<style scoped>
/* 🌟 최상위 컨테이너 스타일 추가 */
.app-container {
  min-height: 100vh;
  background-color: #f0f9ff; /* 연한 파스텔 하늘색 */
  color: #1f2937; /* 진한 회색 글자 */
  padding-bottom: 30px;
}

/* 🌟 헤더 스타일 수정 (기존 어두운 배경 -> 흰색 배경) */
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 30px;
  background-color: #ffffff; /* 흰색 헤더 */
  border-bottom: 1px solid #e5e7eb; /* 연한 테두리 추가 */
  margin-bottom: 20px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}
</style>
