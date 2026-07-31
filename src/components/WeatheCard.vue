<script setup>
defineProps({
  city: Object,
})

const emit = defineEmits(['select-card', 'click-detail'])

const getWeatherIcon = (status) => {
  if (status === '맑음') return '☀️'
  if (status === '비') return '🌧️'
  if (status === '구름') return '☁️'
  return '🌡️'
}
</script>

<template>
  <div @click="emit('select-card', city.name)" style="cursor: pointer">
    <h4 style="margin: 0 0 8px 0; color: #111; font-size: 18px">
      {{ getWeatherIcon(city.status) }} {{ city.name }} ({{ city.status }})
    </h4>
    <p style="margin: 0 0 10px 0; color: #444">현재 기온: {{ city.temp }}°C</p>

    <!-- 조건부 렌더링 라벨 -->
    <span
      v-if="city.temp >= 25"
      style="
        background: #ffebee;
        color: #c62828;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
        font-weight: bold;
      "
    >
      🔥 더움
    </span>
    <span
      v-else
      style="
        background: #e3f2fd;
        color: #1565c0;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
        font-weight: bold;
      "
    >
      ❄️ 선선함
    </span>

    <!-- 상세보기 버튼 (.stop으로 버블링 방지) -->
    <button
      @click.stop="emit('click-detail', city.name, city.status)"
      style="
        float: right;
        padding: 6px 12px;
        cursor: pointer;
        background: #f0f0f0;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-weight: bold;
        color: #333;
      "
    >
      상세보기
    </button>
    <div style="clear: both"></div>
  </div>
</template>

<style scoped></style>
