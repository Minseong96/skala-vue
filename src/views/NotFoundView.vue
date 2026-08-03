<script setup>
import { useRouter } from 'vue-router'
const router = useRouter()
</script>

<template>
  <div
    style="
      padding: 40px;
      max-width: 600px;
      margin: 0 auto;
      text-align: center;
      background: #fff;
      border-radius: 8px;
      color: #333;
    "
  >
    <h1 style="font-size: 50px; margin-bottom: 10px">☀️❓</h1>
    <h2>페이지를 찾을 수 없습니다.</h2>
    <p style="color: #666; margin: 15px 0">
      요청하신 주소가 존재하지 않거나, 아직 개발되지 않았습니다.
    </p>
    <button
      @click="router.push('/')"
      style="
        padding: 10px 20px;
        background: #3b82f6;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      "
    >
      날씨 메인으로 이동
    </button>
  </div>
</template>
