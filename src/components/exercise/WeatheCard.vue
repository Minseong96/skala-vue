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

    <!-- 조건부 뱃지 렌더링 -->
    <span
      v-if="city.status === '비'"
      style="
        background: #e0f2fe;
        color: #0369a1;
        padding: 4px 8px;
        border-radius: 4px;
        fontsize: 12px;
        fontweight: bold;
        display: inline-block;
      "
    >
      🌧️ 촉촉한 비
    </span>
    <span
      v-else-if="city.status === '구름'"
      style="
        background: #f1f5f9;
        color: #475569;
        padding: 4px 8px;
        border-radius: 4px;
        fontsize: 12px;
        fontweight: bold;
        display: inline-block;
      "
    >
      ☁️ 구름 많음
    </span>
    <span
      v-else-if="city.temp >= 28"
      style="
        background: #ffebee;
        color: #c62828;
        padding: 4px 8px;
        border-radius: 4px;
        fontsize: 12px;
        fontweight: bold;
        display: inline-block;
      "
    >
      🔥 쨍쨍한 무더위
    </span>
    <span
      v-else
      style="
        background: #fef3c7;
        color: #b45309;
        padding: 4px 8px;
        border-radius: 4px;
        fontsize: 12px;
        fontweight: bold;
        display: inline-block;
      "
    >
      ☀️ 맑고 쾌적함
    </span>

    <!-- 상세보기 버튼 -->
    <button
      @click.stop="emit('click-detail', city.name, city.status)"
      style="
        float: right;
        padding: 6px 12px;
        cursor: pointer;
        background: #ffffff;
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
