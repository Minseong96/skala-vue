<script setup>
import { RouterLink, RouterView } from 'vue-router'
import UnitToggler from '@/components/exercise/UnitToggler.vue'
</script>

<template>
  <header
    style="
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 30px;
      background: rgba(0, 0, 0, 0.2);
      margin-bottom: 20px;
      border-radius: 8px;
    "
  >
    <div>
      <h2 style="color: #fff; margin: 0 0 5px 0; font-size: 18px">과제 5: 스토어 적용 🌟</h2>
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
    </div>
    <!-- 우측에 단위 변경 버튼 배치 -->
    <UnitToggler />
  </header>

  <RouterView />
</template>

<style scoped></style>
