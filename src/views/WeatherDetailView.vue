<!-- src/views/WeatherDetailView.vue -->
<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import { useConfigStore } from '@/stores/config'
import WeatherIcon from '@/components/exercise/WeatherIcon.vue'
import UIIcon from '@/components/exercise/UIIcon.vue'

const route = useRoute()
const router = useRouter()
const configStore = useConfigStore()

const weatherInfo = ref(null)
const forecastList = ref([])
const isLoading = ref(true)
const errorMessage = ref('')

const cityCoordinates = {
  city_01: { name: '서울특별시', lat: 37.5665, lon: 126.978 },
  서울: { name: '서울특별시', lat: 37.5665, lon: 126.978 },
  서울특별시: { name: '서울특별시', lat: 37.5665, lon: 126.978 },

  city_03: { name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },
  'West-lafayette': { name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },

  city_tokyo: { name: '도쿄', lat: 35.6762, lon: 139.6503 },
  도쿄: { name: '도쿄', lat: 35.6762, lon: 139.6503 },
  Tokyo: { name: '도쿄', lat: 35.6762, lon: 139.6503 },

  city_paris: { name: '파리', lat: 48.8566, lon: 2.3522 },
  파리: { name: '파리', lat: 48.8566, lon: 2.3522 },
  Paris: { name: '파리', lat: 48.8566, lon: 2.3522 },

  city_moscow: { name: '모스크바', lat: 55.7558, lon: 37.6173 },
  모스크바: { name: '모스크바', lat: 55.7558, lon: 37.6173 },
  Moscow: { name: '모스크바', lat: 55.7558, lon: 37.6173 },
}

const userResolvedName = ref('')

const currentCityMeta = computed(() => {
  const param = route.params.cityId
  const query = route.query

  // 1. Check if name and coordinates exist in query parameters
  if (query.name && query.lat && query.lon) {
    return {
      name: query.name,
      lat: parseFloat(query.lat),
      lon: parseFloat(query.lon),
    }
  }

  // 2. Predefined city lookup
  if (cityCoordinates[param]) {
    return cityCoordinates[param]
  }

  // 3. Fallback for city_user_geo or raw param
  let fallbackName = param
  if (param === 'city_user_geo') {
    fallbackName = userResolvedName.value
      ? `내 위치 (${userResolvedName.value})`
      : '내 위치'
  }

  return {
    name: fallbackName || '알 수 없는 도시',
    lat: 37.5665,
    lon: 126.978,
  }
})

const formatCityTime = (timestamp, timezoneOffset = 0) => {
  if (!timestamp) return '-'
  const date = new Date((timestamp + timezoneOffset) * 1000)
  const hours = date.getUTCHours()
  const minutes = date.getUTCMinutes()
  const period = hours >= 12 ? '오후' : '오전'
  const displayHours = hours % 12 === 0 ? 12 : hours % 12
  const formattedMinutes = minutes < 10 ? `0${minutes}` : minutes
  return `${period} ${displayHours}:${formattedMinutes}`
}

const formatCityHour = (dt, timezoneOffset = 0) => {
  if (!dt) return '-'
  const date = new Date((dt + timezoneOffset) * 1000)
  const hours = date.getUTCHours()
  const period = hours >= 12 ? '오후' : '오전'
  const displayHours = hours % 12 === 0 ? 12 : hours % 12
  return `${period} ${displayHours}시`
}

const formatTemp = (val) => {
  if (val === undefined || val === null) return '-'
  const converted =
    configStore.unit === 'fahrenheit' ? Math.round((val * 9) / 5 + 32) : Math.round(val)
  return `${converted}${configStore.unitSymbol}`
}

const fetchAllDetailData = async () => {
  isLoading.value = true
  errorMessage.value = ''
  const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY || '31df255e3ec6209feb7ed06f52a7a1f8'
  const city = currentCityMeta.value

  try {
    const [resWeather, resForecast] = await Promise.all([
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.lat}&lon=${city.lon}&appid=${API_KEY}&units=metric&lang=kr`,
      ),
      axios.get(
        `https://api.openweathermap.org/data/2.5/forecast?lat=${city.lat}&lon=${city.lon}&appid=${API_KEY}&units=metric&lang=kr`,
      ),
    ])

    weatherInfo.value = resWeather.data
    if (resWeather.data && resWeather.data.name) {
      userResolvedName.value = resWeather.data.name
    }
    const tz = resForecast.data.city.timezone
    forecastList.value = resForecast.data.list.slice(0, 8).map((item) => ({
      time: formatCityHour(item.dt, tz),
      temp: Math.round(item.main.temp),
      description: item.weather[0].description,
      icon: item.weather[0].icon,
    }))
  } catch (err) {
    console.error(err)
    errorMessage.value = '실시간 기상 관측 데이터를 불러오는데 실패했습니다.'
  } finally {
    isLoading.value = false
  }
}

onMounted(() => {
  fetchAllDetailData()
})
</script>

<template>
  <div class="ios-detail-wrapper">
    <div class="ios-detail-container">
      <!-- Top Navigation -->
      <header class="nav-bar">
        <button class="back-btn" @click="router.push('/')">
          <span class="arrow">‹</span> 실시간 대시보드
        </button>
        <div class="nav-right">
          <span class="live-pill">LIVE OBSERVATION</span>
          <button class="unit-toggle" @click="configStore.toggleUnit()">
            {{ configStore.unit === 'fahrenheit' ? '°F' : '°C' }}
          </button>
        </div>
      </header>

      <!-- Loading State -->
      <div v-if="isLoading" class="loading-state">
        <div class="ios-spinner"></div>
        <p>상세 날씨 관측 정보를 불러오는 중...</p>
      </div>

      <!-- Error State -->
      <div v-else-if="errorMessage" class="error-state">
        <p>⚠️ {{ errorMessage }}</p>
        <button class="retry-btn" @click="fetchAllDetailData">다시 시도</button>
      </div>

      <!-- Main Detail Content -->
      <div v-else-if="weatherInfo" class="detail-content">
        <!-- Hero Section (Apple Weather Header) -->
        <div class="hero-section">
          <div class="hero-icon-box">
            <WeatherIcon :code="weatherInfo.weather[0].icon" size="96" />
          </div>
          <h1 class="hero-city">{{ currentCityMeta.name }}</h1>
          <div class="hero-temp-large">{{ formatTemp(weatherInfo.main.temp) }}</div>
          <div class="hero-desc">{{ weatherInfo.weather[0].description }}</div>
          <div class="hero-minmax">
            체감 {{ formatTemp(weatherInfo.main.feels_like) }} · 최고
            {{ formatTemp(weatherInfo.main.temp_max) }} / 최저
            {{ formatTemp(weatherInfo.main.temp_min) }}
          </div>
        </div>

        <!-- 8-Slot Hourly Forecast Widget -->
        <div class="widget-card timeline-widget">
          <div class="widget-title">
            <span class="title-text"><UIIcon name="clock" size="16" /> 시간대별 예보</span>
            <span class="widget-subtitle">현지 기준 3시간 간격</span>
          </div>
          <div class="timeline-scroll">
            <div v-for="(item, idx) in forecastList" :key="idx" class="timeline-item">
              <span class="item-time">{{ item.time }}</span>
              <WeatherIcon :code="item.icon" size="36" />
              <span class="item-temp">{{ formatTemp(item.temp) }}</span>
            </div>
          </div>
        </div>

        <!-- 2x2 Grid Weather Widgets -->
        <div class="widgets-grid">
          <!-- Sunrise / Sunset -->
          <div class="widget-card">
            <div class="widget-header">
              <span class="widget-icon"><UIIcon name="sun-horizon" size="18" color="#fbbf24" /></span>
              <span class="widget-label">일출 및 일몰 (현지)</span>
            </div>
            <div class="widget-body grid-2col">
              <div>
                <div class="sub-label">일출 시각</div>
                <div class="big-val">
                  {{ formatCityTime(weatherInfo.sys.sunrise, weatherInfo.timezone) }}
                </div>
              </div>
              <div>
                <div class="sub-label">일몰 시각</div>
                <div class="big-val">
                  {{ formatCityTime(weatherInfo.sys.sunset, weatherInfo.timezone) }}
                </div>
              </div>
            </div>
          </div>

          <!-- Wind Widget -->
          <div class="widget-card">
            <div class="widget-header">
              <span class="widget-icon"><UIIcon name="wind" size="18" color="#38bdf8" /></span>
              <span class="widget-label">바람 관측</span>
            </div>
            <div class="widget-body grid-2col">
              <div>
                <div class="sub-label">현재 풍속</div>
                <div class="big-val">{{ weatherInfo.wind.speed }} <span class="unit">m/s</span></div>
              </div>
              <div>
                <div class="sub-label">풍향</div>
                <div class="big-val">{{ weatherInfo.wind.deg }}<span class="unit">°</span></div>
              </div>
            </div>
          </div>

          <!-- Humidity Widget -->
          <div class="widget-card">
            <div class="widget-header">
              <span class="widget-icon"><UIIcon name="droplet" size="18" color="#38bdf8" /></span>
              <span class="widget-label">습도 및 구름량</span>
            </div>
            <div class="widget-body grid-2col">
              <div>
                <div class="sub-label">대기 습도</div>
                <div class="big-val">{{ weatherInfo.main.humidity }}<span class="unit">%</span></div>
              </div>
              <div>
                <div class="sub-label">구름Cover</div>
                <div class="big-val">
                  {{ weatherInfo.clouds?.all || 0 }}<span class="unit">%</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Pressure & Visibility -->
          <div class="widget-card">
            <div class="widget-header">
              <span class="widget-icon"><UIIcon name="compass" size="18" color="#a855f7" /></span>
              <span class="widget-label">기압 및 가시거리</span>
            </div>
            <div class="widget-body grid-2col">
              <div>
                <div class="sub-label">해수면 기압</div>
                <div class="big-val">
                  {{ weatherInfo.main.pressure }}<span class="unit">hPa</span>
                </div>
              </div>
              <div>
                <div class="sub-label">가시거리</div>
                <div class="big-val">
                  {{ Math.round((weatherInfo.visibility || 10000) / 1000) }}<span class="unit">km</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Google Maps Location Widget in Detail View -->
        <div v-if="currentCityMeta.lat && currentCityMeta.lon" class="widget-card detail-map-widget">
          <div class="widget-header" style="justify-content: space-between;">
            <div style="display: flex; align-items: center; gap: 8px;">
              <span class="widget-icon"><UIIcon name="location" size="18" color="#38bdf8" /></span>
              <span class="widget-label">구글 맵 위치 지도 (Google Maps)</span>
            </div>
            <span class="map-coord-tag">{{ currentCityMeta.lat.toFixed(4) }}° N, {{ currentCityMeta.lon.toFixed(4) }}° E</span>
          </div>
          <div class="detail-map-container">
            <iframe
              :key="`${currentCityMeta.lat}-${currentCityMeta.lon}`"
              class="detail-google-map-iframe"
              :src="`https://maps.google.com/maps?q=${currentCityMeta.lat},${currentCityMeta.lon}&hl=ko&z=12&output=embed`"
              loading="lazy"
              allowfullscreen
            ></iframe>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ios-detail-wrapper {
  width: 100%;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  padding: 20px 16px 60px;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', sans-serif;
  color: #f8fafc;
}

.ios-detail-container {
  width: min(100%, 720px);
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* Nav bar */
.nav-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 18px;
  background: rgba(15, 23, 42, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 999px;
  backdrop-filter: blur(20px);
}

.back-btn {
  background: transparent;
  border: none;
  color: #38bdf8;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: opacity 0.2s;
}

.back-btn:hover {
  opacity: 0.8;
}

.arrow {
  font-size: 18px;
  line-height: 1;
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.live-pill {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.05em;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.15);
  padding: 4px 10px;
  border-radius: 999px;
  border: 1px solid rgba(56, 189, 248, 0.3);
}

.unit-toggle {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.18);
  color: #fff;
  font-weight: 700;
  font-size: 13px;
  padding: 4px 12px;
  border-radius: 999px;
  cursor: pointer;
  transition: background 0.2s;
}

.unit-toggle:hover {
  background: rgba(255, 255, 255, 0.22);
}

/* Loading & Error */
.loading-state,
.error-state {
  text-align: center;
  padding: 60px 20px;
  background: rgba(15, 23, 42, 0.5);
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(16px);
}

.ios-spinner {
  width: 32px;
  height: 32px;
  margin: 0 auto 16px;
  border: 3px solid rgba(255, 255, 255, 0.15);
  border-top-color: #38bdf8;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.retry-btn {
  margin-top: 12px;
  padding: 8px 18px;
  background: #38bdf8;
  color: #0f172a;
  border: none;
  border-radius: 999px;
  font-weight: 700;
  cursor: pointer;
}

/* Hero Section */
.hero-section {
  text-align: center;
  padding: 32px 20px 28px;
  background: linear-gradient(180deg, rgba(30, 41, 59, 0.65) 0%, rgba(15, 23, 42, 0.8) 100%);
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 28px;
  backdrop-filter: blur(24px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.25);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.hero-icon-box {
  margin-bottom: 8px;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.3));
}

.hero-city {
  font-size: 32px;
  font-weight: 800;
  letter-spacing: -0.02em;
  margin: 0 0 4px;
}

.hero-temp-large {
  font-size: 64px;
  font-weight: 800;
  letter-spacing: -0.04em;
  line-height: 1;
  margin: 6px 0 8px;
}

.hero-desc {
  font-size: 17px;
  font-weight: 600;
  color: #93c5fd;
  margin-bottom: 8px;
}

.hero-minmax {
  font-size: 13px;
  font-weight: 500;
  color: #cbd5e1;
}

/* Widgets Base */
.widget-card {
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 22px;
  padding: 18px 20px;
  backdrop-filter: blur(20px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

/* Timeline Widget */
.timeline-widget {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.widget-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title-text {
  font-size: 14px;
  font-weight: 700;
  color: #f8fafc;
}

.widget-subtitle {
  font-size: 12px;
  color: #94a3b8;
}

.timeline-scroll {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 4px;
}

.timeline-scroll::-webkit-scrollbar {
  height: 4px;
}
.timeline-scroll::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4px;
}

.timeline-item {
  min-width: 68px;
  padding: 12px 8px;
  background: rgba(30, 41, 59, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  flex-shrink: 0;
}

.item-time {
  font-size: 11px;
  color: #93c5fd;
  font-weight: 600;
}

.item-temp {
  font-size: 14px;
  font-weight: 700;
}

/* Widgets Grid */
.widgets-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

@media (max-width: 580px) {
  .widgets-grid {
    grid-template-columns: 1fr;
  }
}

.widget-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.widget-icon {
  font-size: 16px;
}

.widget-label {
  font-size: 13px;
  font-weight: 700;
  color: #94a3b8;

}

.widget-body {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.grid-2col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

/* Google Maps Widget in Detail View */
.detail-map-widget {
  margin-top: 16px;
}

.map-coord-tag {
  font-size: 11px;
  font-weight: 600;
  color: #7dd3fc;
  background: rgba(56, 189, 248, 0.15);
  border: 1px solid rgba(56, 189, 248, 0.3);
  padding: 3px 9px;
  border-radius: 999px;
}

.detail-map-container {
  width: 100%;
  height: 240px;
  border-radius: 16px;
  overflow: hidden;
  margin-top: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
}

.detail-google-map-iframe {
  width: 100%;
  height: 100%;
  border: 0;
  filter: contrast(1.05) saturate(1.1);
}

.sub-label {
  font-size: 12px;
  color: #cbd5e1;
  margin-bottom: 2px;
}

.big-val {
  font-size: 20px;
  font-weight: 800;
  color: #f8fafc;
}

.unit {
  font-size: 13px;
  font-weight: 600;
  color: #93c5fd;
  margin-left: 2px;
}
</style>
