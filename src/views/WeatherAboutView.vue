<script setup>
import { useRouter } from 'vue-router'
const router = useRouter()
</script>

<template>
  <div
    style="
      padding: 20px;
      max-width: 600px;
      margin: 0 auto;
      background: #fff;
      border-radius: 8px;
      color: #333;
    "
  >
    <h2>서비스 소개</h2>
    <p style="margin-top: 10px; line-height: 1.6">
      본 앱은 Vue 3 및 Vue Router를 기반으로 제작된 실습용 기상 관측 대시보드 시스템입니다.
    </p>
    <button
      @click="router.push('/')"
      style="
        margin-top: 20px;
        padding: 10px 20px;
        background: #3b82f6;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      "
    >
      메인 대시보드로 이동
    </button>
  </div>
</template>
