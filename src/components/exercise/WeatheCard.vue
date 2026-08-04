<!-- src/components/exercise/WeatheCard.vue -->
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
  const rawTemp = props.city?.temp
  if (rawTemp === undefined || rawTemp === null) return '-'
  if (configStore.unit === 'fahrenheit') {
    return Math.round((rawTemp * 9) / 5 + 32)
  }
  return rawTemp
})

// Determine FX category
const fxType = computed(() => {
  const status = (props.city?.status || '').toLowerCase()
  const icon = props.city?.icon || ''

  if (icon.endsWith('n')) return 'night'
  if (status.includes('비') || status.includes('소나기')) return 'rain'
  if (status.includes('뇌우')) return 'storm'
  if (status.includes('구름') || status.includes('흐림') || status.includes('안개')) return 'cloud'
  return 'sun'
})
</script>

<template>
  <div class="weather-card" @click="emit('select-card', city.name)">
    <!-- 🌟 Dynamic Weather Atmosphere Background Overlay -->
    <div :class="['card-weather-fx', fxType]">
      <!-- Sun Flare FX -->
      <template v-if="fxType === 'sun'">
        <div class="sun-flare"></div>
        <div class="sun-ray ray-1"></div>
        <div class="sun-ray ray-2"></div>
      </template>

      <!-- Night Stars FX -->
      <template v-else-if="fxType === 'night'">
        <div class="star star-1">✦</div>
        <div class="star star-2">✦</div>
        <div class="star star-3">✦</div>
        <div class="star star-4">✦</div>
      </template>

      <!-- Rain FX -->
      <template v-else-if="fxType === 'rain' || fxType === 'storm'">
        <div class="rain-drop drop-1"></div>
        <div class="rain-drop drop-2"></div>
        <div class="rain-drop drop-3"></div>
        <div class="rain-drop drop-4"></div>
        <div class="rain-drop drop-5"></div>
      </template>

      <!-- Cloud FX -->
      <template v-else-if="fxType === 'cloud'">
        <div class="cloud-puff puff-1"></div>
        <div class="cloud-puff puff-2"></div>
      </template>
    </div>

    <!-- Main Card Content -->
    <div class="card-content">
      <div class="card-top">
        <div>
          <h4 style="display: flex; align-items: center; gap: 8px;">
            <WeatherIcon :code="city.icon || '01d'" size="26" />
            <span>{{ city.name }}</span>
            <span v-if="city.isUserLoc" class="user-loc-badge">내 위치</span>
          </h4>
          <p class="muted">{{ city.status }}</p>
        </div>
        <div class="temp-pill">{{ displayTemp }}{{ configStore.unitSymbol }}</div>
      </div>

      <div class="card-bottom">
        <span class="chip">
          {{
            city.status.includes('비')
              ? '🌧️ 촉촉한 비'
              : city.status.includes('구름') || city.status.includes('흐림')
                ? '☁️ 부드러운 구름'
                : '☀️ 맑고 선선한 공기'
          }}
        </span>
        <button @click.stop="emit('click-detail', city.id || city.name)">상세보기 ›</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.weather-card {
  position: relative;
  cursor: pointer;
  overflow: hidden;
  border-radius: 20px;
}

.card-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.user-loc-badge {
  font-size: 10px;
  font-weight: 700;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.2);
  border: 1px solid rgba(56, 189, 248, 0.4);
  padding: 2px 7px;
  border-radius: 999px;
  margin-left: 2px;
}

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
}

h4 {
  margin: 0 0 4px;
  font-size: 18px;
  font-weight: 800;
  color: #f8fafc;
}

.muted {
  margin: 0;
  color: #cbd5e1;
  font-size: 13px;
}

.temp-pill {
  background: rgba(255, 255, 255, 0.18);
  border: 1px solid rgba(255, 255, 255, 0.12);
  padding: 8px 12px;
  border-radius: 999px;
  font-weight: 800;
  font-size: 16px;
  color: #ffffff;
  white-space: nowrap;
  backdrop-filter: blur(12px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.card-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
}

.chip {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.15);
  color: #e0f2fe;
  padding: 6px 12px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 700;
  backdrop-filter: blur(10px);
}

button {
  border: none;
  border-radius: 999px;
  padding: 7px 14px;
  background: rgba(255, 255, 255, 0.18);
  color: #ffffff;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
  transition: all 0.2s ease;
  backdrop-filter: blur(10px);
}

button:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateX(2px);
}

/* 🌟 DYNAMIC ATMOSPHERE FX LAYER */
.card-weather-fx {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  overflow: hidden;
}

/* ☀️ Sun FX */
.sun-flare {
  position: absolute;
  top: -25px;
  right: -25px;
  width: 130px;
  height: 130px;
  background: radial-gradient(
    circle,
    rgba(253, 224, 71, 0.5) 0%,
    rgba(249, 115, 22, 0.18) 50%,
    transparent 70%
  );
  border-radius: 50%;
  animation: sunGlow 4s ease-in-out infinite alternate;
}

.sun-ray {
  position: absolute;
  top: -10px;
  right: 10px;
  width: 140px;
  height: 2px;
  background: linear-gradient(90deg, rgba(253, 224, 71, 0.4), transparent);
  transform-origin: right center;
}

.ray-1 {
  transform: rotate(25deg);
  animation: rayPulse 3.5s ease-in-out infinite alternate;
}
.ray-2 {
  transform: rotate(45deg);
  animation: rayPulse 4.5s ease-in-out 0.5s infinite alternate;
}

@keyframes sunGlow {
  from {
    transform: scale(0.9);
    opacity: 0.7;
  }
  to {
    transform: scale(1.25);
    opacity: 1;
  }
}

@keyframes rayPulse {
  from {
    opacity: 0.3;
  }
  to {
    opacity: 0.8;
  }
}

/* 🌙 Night FX */
.star {
  position: absolute;
  color: #f8fafc;
  font-size: 11px;
  opacity: 0.6;
}

.star-1 {
  top: 15%;
  right: 20%;
  animation: starTwinkle 2s infinite alternate;
}
.star-2 {
  top: 45%;
  right: 35%;
  animation: starTwinkle 2.8s 0.5s infinite alternate;
}
.star-3 {
  top: 70%;
  right: 15%;
  animation: starTwinkle 2.2s 1s infinite alternate;
}
.star-4 {
  top: 25%;
  right: 50%;
  animation: starTwinkle 3.2s 1.5s infinite alternate;
}

@keyframes starTwinkle {
  from {
    opacity: 0.2;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1.3);
    filter: drop-shadow(0 0 4px #ffffff);
  }
}

/* 🌧️ Rain FX */
.rain-drop {
  position: absolute;
  width: 1.5px;
  height: 14px;
  background: linear-gradient(180deg, rgba(56, 189, 248, 0.7), rgba(255, 255, 255, 0.2));
  transform: rotate(15deg);
  animation: cardRain 1.2s linear infinite;
}

.drop-1 {
  top: -20px;
  right: 15%;
  animation-delay: 0s;
}
.drop-2 {
  top: -20px;
  right: 35%;
  animation-delay: 0.25s;
}
.drop-3 {
  top: -20px;
  right: 55%;
  animation-delay: 0.5s;
}
.drop-4 {
  top: -20px;
  right: 75%;
  animation-delay: 0.75s;
}
.drop-5 {
  top: -20px;
  right: 25%;
  animation-delay: 1s;
}

@keyframes cardRain {
  0% {
    transform: translateY(0) rotate(15deg);
    opacity: 0;
  }
  30% {
    opacity: 0.8;
  }
  100% {
    transform: translateY(120px) rotate(15deg);
    opacity: 0;
  }
}

/* ☁️ Cloud FX */
.cloud-puff {
  position: absolute;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 50%;
  filter: blur(8px);
}

.puff-1 {
  width: 90px;
  height: 60px;
  top: 10px;
  right: -10px;
  animation: cloudDrift 6s ease-in-out infinite alternate;
}

.puff-2 {
  width: 110px;
  height: 70px;
  bottom: -20px;
  right: 40px;
  animation: cloudDrift 8s ease-in-out 1s infinite alternate-reverse;
}

@keyframes cloudDrift {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-15px);
  }
}
</style>
