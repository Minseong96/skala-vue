<script setup>
import { computed } from 'vue'
import { useConfigStore } from '@/stores/config'

const props = defineProps({
  city: Object,
})

const emit = defineEmits(['select-card', 'click-detail'])
const configStore = useConfigStore()

// 🌟 핵심: Pinia 스토어의 단위 상태에 따라 온도를 실시간 계산 (Computed)
const displayTemp = computed(() => {
  const rawTemp = props.city.temp // 기본 원본 데이터는 섭씨 숫자
  if (configStore.unit === 'fahrenheit') {
    return Math.round((rawTemp * 9) / 5 + 32) // 화씨 변환 공식
  }
  return rawTemp // celsius일 때는 원본 그대로 반환
})

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
    <!-- 화씨/섭씨 단위와 기호를 Pinia에서 가져와 동적으로 표시 -->
    <p style="margin: 0 0 10px 0; color: #444">
      현재 기온: {{ displayTemp }}{{ configStore.unitSymbol }}
    </p>

    <div style="display: flex; justify-content: space-between; align-items: center">
      <span
        v-if="city.status === '비'"
        style="
          background: #e0f2fe;
          color: #0369a1;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 12px;
          font-weight: bold;
        "
        >촉촉한 비</span
      >
      <span
        v-else-if="city.status === '구름'"
        style="
          background: #f1f5f9;
          color: #475569;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 12px;
          font-weight: bold;
        "
        >구름 조금</span
      >
      <span
        v-else
        style="
          background: #fff5f5;
          color: #c53030;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 12px;
          font-weight: bold;
        "
        >땡볕한 무더위</span
      >

      <button
        @click.stop="emit('click-detail', city.name, city.status)"
        style="padding: 4px 10px; font-size: 12px; cursor: pointer"
      >
        상세보기
      </button>
    </div>
  </div>
</template>
