<script setup>
import { useConfigStore } from '@/stores/config'

const configStore = useConfigStore()
</script>

<template>
  <div style="display: flex; align-items: center; gap: 10px; font-size: 14px; color: #fff">
    <span>날씨단위: {{ configStore.unit === 'fahrenheit' ? '화씨(°F)' : '섭씨(°C)' }}</span>
    <button
      @click="configStore.toggleUnit()"
      style="
        padding: 4px 10px;
        background: #3b82f6;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      "
    >
      단위변경
    </button>
  </div>
</template>
