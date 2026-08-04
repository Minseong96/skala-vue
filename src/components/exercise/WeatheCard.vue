<script setup>
import { computed } from 'vue'
import { useConfigStore } from '@/stores/config'
import WeatherIcon from '@/components/exercise/WeatherIcon.vue'

const props = defineProps({
  city: Object,
})

const emit = defineEmits(['select-card', 'click-detail'])
const configStore = useConfigStore()

const displayTemp = computed(() => {
  const rawTemp = props.city.temp
  if (configStore.unit === 'fahrenheit') {
    return Math.round((rawTemp * 9) / 5 + 32)
  }
  return rawTemp
})
</script>

<template>
  <div class="weather-card" @click="emit('select-card', city.name)">
    <div class="card-top">
      <div>
        <h4 style="display: flex; align-items: center; gap: 6px;">
          <WeatherIcon :code="city.icon || '01d'" size="22" />
          <span>{{ city.name }}</span>
        </h4>
        <p class="muted">{{ city.status }}</p>
      </div>
      <div class="temp-pill">{{ displayTemp }}{{ configStore.unitSymbol }}</div>
    </div>

    <div class="card-bottom">
      <span class="chip">{{ city.status.includes('비') ? '촉촉한 비' : city.status.includes('구름') || city.status.includes('흐림') ? '부드러운 구름' : '맑고 선선한 공기' }}</span>
      <button @click.stop="emit('click-detail', city.id || city.name)">상세보기</button>
    </div>
  </div>
</template>

<style scoped>
.weather-card {
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
}

h4 {
  margin: 0 0 4px;
  font-size: 17px;
  font-weight: 700;
  color: #f8fafc;
}

.muted {
  margin: 0;
  color: #cbd5e1;
  font-size: 13px;
}

.temp-pill {
  background: rgba(255, 255, 255, 0.16);
  padding: 8px 10px;
  border-radius: 999px;
  font-weight: 700;
  color: #f8fafc;
  white-space: nowrap;
}

.card-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
}

.chip {
  background: rgba(56, 189, 248, 0.16);
  color: #bae6fd;
  padding: 6px 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 700;
}

button {
  border: none;
  border-radius: 999px;
  padding: 7px 10px;
  background: rgba(255, 255, 255, 0.16);
  color: #f8fafc;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
}
</style>
