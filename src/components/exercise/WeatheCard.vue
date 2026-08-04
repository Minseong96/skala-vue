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

const formatVal = (val) => {
  if (val === undefined || val === null) return '-'
  if (configStore.unit === 'fahrenheit') {
    return Math.round((val * 9) / 5 + 32)
  }
  return Math.round(val)
}

const displayTemp = computed(() => formatVal(props.city?.temp))
const displayMax = computed(() =>
  formatVal(props.city?.tempMax ?? (props.city?.temp !== undefined ? props.city.temp + 2 : null)),
)
const displayMin = computed(() =>
  formatVal(props.city?.tempMin ?? (props.city?.temp !== undefined ? props.city.temp - 3 : null)),
)

// Determine FX category for background sky atmosphere
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
  <div class="apple-weather-card" @click="emit('select-card', city)">
    <!-- 🌟 Dynamic Apple Weather Sky Atmosphere Overlay -->
    <div :class="['card-sky-fx', fxType]">
      <!-- Sunny Lens Flare & Light Rays -->
      <template v-if="fxType === 'sun'">
        <div class="sun-flare-main"></div>
        <div class="sun-light-ray ray-a"></div>
        <div class="sun-light-ray ray-b"></div>
      </template>

      <!-- Night Stars & Moon Glow -->
      <template v-else-if="fxType === 'night'">
        <div class="moon-glow-bg"></div>
        <div class="night-star s1">✦</div>
        <div class="night-star s2">✦</div>
        <div class="night-star s3">✦</div>
      </template>

      <!-- Rain Streaks -->
      <template v-else-if="fxType === 'rain' || fxType === 'storm'">
        <div class="rain-streak r1"></div>
        <div class="rain-streak r2"></div>
        <div class="rain-streak r3"></div>
        <div class="rain-streak r4"></div>
        <div class="rain-streak r5"></div>
      </template>

      <!-- Floating Cloud Layers -->
      <template v-else-if="fxType === 'cloud'">
        <div class="cloud-puff c1"></div>
        <div class="cloud-puff c2"></div>
        <div class="cloud-puff c3"></div>
      </template>
    </div>

    <!-- Apple Weather Card Content -->
    <div class="apple-card-content">
      <!-- Top Row: City Name + Subtitle (Left), Temp (Right) -->
      <div class="apple-card-header">
        <div class="city-info-group">
          <h3 class="city-title">
            <span>{{ city.name }}</span>
            <span v-if="city.isUserLoc" class="user-badge">내 위치</span>
          </h3>
          <p class="city-subtitle">
            {{ city.isUserLoc ? '현재 위치' : city.localTime || '현지 시각' }}
          </p>
        </div>
        <div class="city-temp-big">{{ displayTemp }}°</div>
      </div>

      <!-- Bottom Row: Condition (Left), High / Low Range (Right) -->
      <div class="apple-card-footer">
        <div class="condition-group">
          <WeatherIcon :code="city.icon || '01d'" size="22" />
          <span class="condition-text">{{ city.status }}</span>
        </div>
        <div class="range-group">
          <span class="range-text">최고: {{ displayMax }}° 최저: {{ displayMin }}°</span>
          <button
            class="detail-btn"
            @click.stop="emit('click-detail', city.id || city.name, city)"
          >
            상세보기 ›
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.apple-weather-card {
  position: relative;
  overflow: hidden;
  border-radius: 22px;
  cursor: pointer;
  user-select: none;
  padding: 4px 6px;
}

.apple-card-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: 96px;
}

/* Header */
.apple-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.city-info-group {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.city-title {
  margin: 0;
  font-size: 21px;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.02em;
  display: flex;
  align-items: center;
  gap: 6px;
  text-shadow: 0 1px 4px rgba(0, 0, 0, 0.4);
}

.user-badge {
  font-size: 10px;
  font-weight: 700;
  color: #7dd3fc;
  background: rgba(56, 189, 248, 0.25);
  border: 1px solid rgba(56, 189, 248, 0.4);
  padding: 2px 7px;
  border-radius: 999px;
}

.city-subtitle {
  margin: 0;
  font-size: 12px;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.85);
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.city-temp-big {
  font-size: 46px;
  font-weight: 300;
  color: #ffffff;
  line-height: 1;
  letter-spacing: -0.04em;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.4);
}

/* Footer */
.apple-card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 14px;
}

.condition-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.condition-text {
  font-size: 13px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.range-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.range-text {
  font-size: 12px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.85);
  letter-spacing: 0.01em;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.detail-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.28);
  color: #ffffff;
  font-size: 11px;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 999px;
  cursor: pointer;
  backdrop-filter: blur(10px);
  transition: all 0.2s ease;
}

.detail-btn:hover {
  background: rgba(255, 255, 255, 0.35);
  transform: translateX(2px);
}

/* 🌟 DYNAMIC ATMOSPHERE SKY OVERLAY */
.card-sky-fx {
  position: absolute;
  inset: -10px;
  z-index: 1;
  pointer-events: none;
  overflow: hidden;
}

/* ☀️ Sunny Flare */
.sun-flare-main {
  position: absolute;
  top: -30px;
  right: 60px;
  width: 140px;
  height: 140px;
  background: radial-gradient(
    circle,
    rgba(255, 245, 180, 0.5) 0%,
    rgba(255, 190, 80, 0.2) 50%,
    transparent 75%
  );
  border-radius: 50%;
  animation: sunPulse 4s ease-in-out infinite alternate;
}

.sun-light-ray {
  position: absolute;
  top: -20px;
  right: 80px;
  width: 160px;
  height: 2px;
  background: linear-gradient(90deg, rgba(255, 250, 200, 0.5), transparent);
  transform-origin: right center;
}
.ray-a {
  transform: rotate(30deg);
  animation: rayGlow 3s ease-in-out infinite alternate;
}
.ray-b {
  transform: rotate(50deg);
  animation: rayGlow 4.5s ease-in-out 0.5s infinite alternate;
}

@keyframes sunPulse {
  from {
    transform: scale(0.95);
    opacity: 0.8;
  }
  to {
    transform: scale(1.2);
    opacity: 1;
  }
}

@keyframes rayGlow {
  from {
    opacity: 0.3;
  }
  to {
    opacity: 0.8;
  }
}

/* 🌙 Night Moon Glow & Stars */
.moon-glow-bg {
  position: absolute;
  top: -20px;
  right: 40px;
  width: 120px;
  height: 120px;
  background: radial-gradient(circle, rgba(226, 232, 240, 0.3) 0%, transparent 65%);
  border-radius: 50%;
}

.night-star {
  position: absolute;
  color: #f8fafc;
  font-size: 10px;
}
.s1 {
  top: 20%;
  right: 25%;
  animation: starFlicker 2.2s infinite alternate;
}
.s2 {
  top: 40%;
  right: 40%;
  animation: starFlicker 3s 0.6s infinite alternate;
}
.s3 {
  top: 65%;
  right: 20%;
  animation: starFlicker 2.5s 1.2s infinite alternate;
}

@keyframes starFlicker {
  from {
    opacity: 0.2;
    transform: scale(0.8);
  }
  to {
    opacity: 0.9;
    transform: scale(1.2);
    filter: drop-shadow(0 0 3px #ffffff);
  }
}

/* 🌧️ Rain Streaks */
.rain-streak {
  position: absolute;
  width: 1px;
  height: 18px;
  background: linear-gradient(180deg, rgba(186, 230, 253, 0.65), transparent);
  transform: rotate(15deg);
  animation: rainStreak 1.1s linear infinite;
}
.r1 {
  top: -20px;
  right: 15%;
  animation-delay: 0s;
}
.r2 {
  top: -20px;
  right: 35%;
  animation-delay: 0.2s;
}
.r3 {
  top: -20px;
  right: 55%;
  animation-delay: 0.4s;
}
.r4 {
  top: -20px;
  right: 75%;
  animation-delay: 0.6s;
}
.r5 {
  top: -20px;
  right: 40%;
  animation-delay: 0.8s;
}

@keyframes rainStreak {
  0% {
    transform: translateY(0) rotate(15deg);
    opacity: 0;
  }
  30% {
    opacity: 0.8;
  }
  100% {
    transform: translateY(130px) rotate(15deg);
    opacity: 0;
  }
}

/* ☁️ Cloud Layers */
.cloud-puff {
  position: absolute;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.45) 0%,
    rgba(226, 232, 240, 0.22) 75%,
    transparent 100%
  );
  border-radius: 60px;
  filter: blur(4px);
  box-shadow: 0 4px 20px rgba(255, 255, 255, 0.25);
  pointer-events: none;
}

.c1 {
  width: 140px;
  height: 70px;
  top: 5px;
  right: -10px;
  animation: cloudMove 7s ease-in-out infinite alternate;
}

.c2 {
  width: 160px;
  height: 80px;
  bottom: -15px;
  right: 50px;
  animation: cloudMove 9.5s ease-in-out 1s infinite alternate-reverse;
}

.c3 {
  width: 110px;
  height: 55px;
  top: 30px;
  right: 120px;
  animation: cloudMove 6s ease-in-out 0.4s infinite alternate;
}

@keyframes cloudMove {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-22px);
  }
}
</style>
