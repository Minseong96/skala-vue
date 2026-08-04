<!-- src/views/WeatherHomeView.vue -->
<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { useConfigStore } from '@/stores/config'
import BaseDashboardCard from '@/components/exercise/BaseDashboardCard.vue'
import SearchBar from '@/components/exercise/SearchBar.vue'
import WeatheCard from '@/components/exercise/WeatheCard.vue'
import WeatherIcon from '@/components/exercise/WeatherIcon.vue'
import UIIcon from '@/components/exercise/UIIcon.vue'

const router = useRouter()
const configStore = useConfigStore()

const citiesMeta = [
  { id: 'city_01', name: '서울특별시', lat: 37.5665, lon: 126.978 },
  { id: 'city_03', name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },
  { id: 'city_04', name: '부산광역시', lat: 35.1796, lon: 129.0756 },
  { id: 'city_05', name: '대구광역시', lat: 35.8714, lon: 128.6014 },
]

const weatherList = ref([])
const extraWeatherList = ref([])
const timelineWeatherList = ref([])
const activeTab = ref('weather')
const searchQuery = ref('')
const selectedCityInfo = ref('카드를 터치하거나 도시를 검색하여 정보를 확인하세요.')
const isLoading = ref(false)
const errorMessage = ref('')
const currentTime = ref('')
const isGeoLoading = ref(false)

const updateClock = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' })
}

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

const formatTemp = (value) => {
  const converted =
    configStore.unit === 'fahrenheit' ? Math.round((value * 9) / 5 + 32) : Math.round(value)
  return `${converted}${configStore.unitSymbol}`
}

const fetchAllData = async () => {
  isLoading.value = true
  const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY || '31df255e3ec6209feb7ed06f52a7a1f8'

  try {
    const weatherRequests = citiesMeta.map((city) =>
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.lat}&lon=${city.lon}&appid=${API_KEY}&units=metric&lang=kr`,
      ),
    )

    const forecastRequests = citiesMeta.map((city) =>
      axios.get(
        `https://api.openweathermap.org/data/2.5/forecast?lat=${city.lat}&lon=${city.lon}&appid=${API_KEY}&units=metric&lang=kr`,
      ),
    )

    const [weatherResponses, forecastResponses] = await Promise.all([
      Promise.all(weatherRequests),
      Promise.all(forecastRequests),
    ])

    weatherList.value = weatherResponses.map((res, index) => {
      const data = res.data
      return {
        id: citiesMeta[index].id,
        name: citiesMeta[index].name,
        temp: Math.round(data.main.temp),
        status: data.weather[0].description,
        icon: data.weather[0].icon,
      }
    })

    extraWeatherList.value = weatherResponses.map((res, index) => {
      const data = res.data
      return {
        id: citiesMeta[index].id,
        name: citiesMeta[index].name,
        temp: Math.round(data.main.temp),
        feelsLike: Math.round(data.main.feels_like),
        humidity: data.main.humidity,
        pressure: data.main.pressure,
        sunrise: formatCityTime(data.sys.sunrise, data.timezone),
        sunset: formatCityTime(data.sys.sunset, data.timezone),
        status: data.weather[0].description,
        icon: data.weather[0].icon,
      }
    })

    timelineWeatherList.value = forecastResponses.map((res, index) => {
      const data = res.data
      const timezone = data.city.timezone
      const hourly = data.list.slice(0, 6).map((item) => ({
        time: formatCityHour(item.dt, timezone),
        temp: Math.round(item.main.temp),
        description: item.weather[0].description,
        icon: item.weather[0].icon,
      }))
      return {
        id: citiesMeta[index].id,
        name: citiesMeta[index].name,
        hourly,
      }
    })

    updateClock()
  } catch (error) {
    console.error('데이터를 가져오지 못했습니다:', error)
    errorMessage.value = '기상 관측망과 통신하는 데 실패했습니다.'
  } finally {
    isLoading.value = false
  }
}

// 🌟 HTML5 Geolocation API Auto Detection
const detectUserLocation = () => {
  if (!('geolocation' in navigator)) return
  isGeoLoading.value = true

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const { latitude, longitude } = position.coords
      const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY || '31df255e3ec6209feb7ed06f52a7a1f8'
      try {
        const [resWeather, resForecast] = await Promise.all([
          axios.get(
            `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${API_KEY}&units=metric&lang=kr`,
          ),
          axios.get(
            `https://api.openweathermap.org/data/2.5/forecast?lat=${latitude}&lon=${longitude}&appid=${API_KEY}&units=metric&lang=kr`,
          ),
        ])

        const data = resWeather.data
        const forecastData = resForecast.data
        const tz = forecastData.city.timezone
        const geoName = `내 위치 (${data.name || '현재 지점'})`
        const geoId = 'city_user_geo'

        const userGeoWeatherItem = {
          id: geoId,
          name: geoName,
          temp: Math.round(data.main.temp),
          status: data.weather[0].description,
          icon: data.weather[0].icon,
          isUserLoc: true,
        }

        weatherList.value = [
          userGeoWeatherItem,
          ...weatherList.value.filter((item) => item.id !== geoId),
        ]

        extraWeatherList.value = [
          {
            id: geoId,
            name: geoName,
            temp: Math.round(data.main.temp),
            feelsLike: Math.round(data.main.feels_like),
            humidity: data.main.humidity,
            pressure: data.main.pressure,
            sunrise: formatCityTime(data.sys.sunrise, data.timezone),
            sunset: formatCityTime(data.sys.sunset, data.timezone),
            status: data.weather[0].description,
            icon: data.weather[0].icon,
            isUserLoc: true,
          },
          ...extraWeatherList.value.filter((item) => item.id !== geoId),
        ]

        const hourly = forecastData.list.slice(0, 6).map((item) => ({
          time: formatCityHour(item.dt, tz),
          temp: Math.round(item.main.temp),
          description: item.weather[0].description,
          icon: item.weather[0].icon,
        }))

        timelineWeatherList.value = [
          {
            id: geoId,
            name: geoName,
            hourly,
            isUserLoc: true,
          },
          ...timelineWeatherList.value.filter((item) => item.id !== geoId),
        ]

        selectedCityInfo.value = `현재 위치 감지 완료: [ ${geoName} ]`
      } catch (err) {
        console.warn('Geolocation weather fetch failed:', err)
      } finally {
        isGeoLoading.value = false
      }
    },
    (err) => {
      console.info('Geolocation permission skipped or denied:', err.message)
      isGeoLoading.value = false
    },
    { timeout: 8000 },
  )
}

onMounted(async () => {
  await fetchAllData()
  detectUserLocation()
  const timer = window.setInterval(updateClock, 1000)
  onBeforeUnmount(() => clearInterval(timer))
})

const filteredWeatherList = computed(() => {
  if (!searchQuery.value) return weatherList.value
  return weatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

const filteredExtraList = computed(() => {
  if (!searchQuery.value) return extraWeatherList.value
  return extraWeatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

const filteredTimelineList = computed(() => {
  if (!searchQuery.value) return timelineWeatherList.value
  return timelineWeatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

// 🌟 Dynamic Atmosphere Theme Gradient based on Top Weather Condition
const currentThemeGradient = computed(() => {
  const topItem = weatherList.value[0]
  if (!topItem) return 'radial-gradient(ellipse at 50% 0%, #1e293b 0%, #0f172a 60%, #020617 100%)'

  const status = (topItem.status || '').toLowerCase()
  const icon = topItem.icon || ''

  if (icon.endsWith('n')) {
    // Starry Deep Night Atmosphere
    return 'radial-gradient(ellipse at 50% 0%, rgba(30, 27, 75, 0.95) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  if (status.includes('비') || status.includes('소나기') || status.includes('뇌우')) {
    // Rain / Storm Cool Blue Atmosphere
    return 'radial-gradient(ellipse at 50% 0%, rgba(14, 116, 144, 0.85) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  if (status.includes('구름') || status.includes('흐림') || status.includes('안개')) {
    // Cool Slate Cloud Atmosphere
    return 'radial-gradient(ellipse at 50% 0%, rgba(71, 85, 105, 0.85) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  // Sunny Golden Atmosphere
  return 'radial-gradient(ellipse at 50% 0%, rgba(217, 119, 6, 0.65) 0%, rgba(30, 27, 75, 0.85) 50%, #020617 100%)'
})

const handleSelectCard = (cityName) => {
  selectedCityInfo.value = `현재 선택된 구역: [ ${cityName} ]`
}

const handleDetail = (cityId) => {
  router.push(`/weather/${cityId}`)
}

const getCardBorder = (temp) => {
  return temp >= 28 ? 'rgba(244, 63, 94, 0.6)' : 'rgba(255, 255, 255, 0.2)'
}

const getCardBackground = (status = '') => {
  const label = status.toLowerCase()
  if (label.includes('비') || label.includes('소나기')) {
    return 'linear-gradient(135deg, rgba(14, 116, 144, 0.38) 0%, rgba(15, 23, 42, 0.72) 100%)'
  }
  if (label.includes('구름') || label.includes('흐림') || label.includes('안개')) {
    return 'linear-gradient(135deg, rgba(71, 85, 105, 0.42) 0%, rgba(15, 23, 42, 0.72) 100%)'
  }
  return 'linear-gradient(135deg, rgba(217, 119, 6, 0.35) 0%, rgba(15, 23, 42, 0.72) 100%)'
}
</script>

<template>
  <div class="apple-weather-wrapper" :style="{ background: currentThemeGradient }">
    <div class="apple-weather-app">
      <header class="apple-header">
        <div class="apple-header-top">
          <div class="apple-header-badges">
            <span class="apple-location"><UIIcon name="location" size="13" color="#93c5fd" /> Global Hub</span>
            <span class="apple-pill">Live Weather</span>
            <button class="geo-btn" @click="detectUserLocation">
              <UIIcon name="target" size="13" color="#7dd3fc" />
              <span>{{ isGeoLoading ? '위치 감지 중...' : '내 위치 감지' }}</span>
            </button>
          </div>
          <span class="apple-time">{{ currentTime }}</span>
        </div>
        <div class="apple-title-row">
          <h1 class="apple-title">날씨</h1>
          <button class="apple-unit-pill" @click="configStore.toggleUnit()">
            {{ configStore.unit === 'fahrenheit' ? '°F' : '°C' }} ·
            {{ configStore.unit === 'fahrenheit' ? '화씨' : '섭씨' }}
          </button>
        </div>
        <p class="apple-subtitle">
          애플 웨더의 잔잔한 감성과 세련된 인터랙션을 담은 실시간 관측 대시보드입니다.
        </p>
      </header>

      <div class="menu-tab-bar">
        <button
          :class="['tab-btn', { active: activeTab === 'weather' }]"
          @click="activeTab = 'weather'"
        >
          <UIIcon name="globe" size="15" /> 실시간 날씨
        </button>
        <button
          :class="['tab-btn', { active: activeTab === 'extra' }]"
          @click="activeTab = 'extra'"
        >
          <UIIcon name="sun-horizon" size="15" /> 일출·일몰
        </button>
        <button
          :class="['tab-btn', { active: activeTab === 'timeline' }]"
          @click="activeTab = 'timeline'"
        >
          <UIIcon name="clock" size="15" /> 3시간 타임라인
        </button>
      </div>

      <div class="apple-search-box">
        <SearchBar v-model="searchQuery" />
      </div>

      <div v-if="isLoading" class="apple-state-view">
        <div class="apple-spinner"></div>
        <p>날씨 데이터를 불러오는 중...</p>
      </div>

      <div v-else-if="errorMessage" class="apple-state-view error">
        <p>⚠️ {{ errorMessage }}</p>
      </div>

      <template v-else-if="activeTab === 'weather'">
        <div v-if="filteredWeatherList.length === 0" class="apple-state-view">
          <p>검색 결과가 없습니다.</p>
        </div>

        <div class="apple-cards-stack">
          <BaseDashboardCard
            v-for="city in filteredWeatherList"
            :key="city.id"
            :style="{
              background: getCardBackground(city.status),
              borderColor: city.isUserLoc ? 'rgba(56, 189, 248, 0.8)' : getCardBorder(city.temp),
            }"
            class="apple-glass-card"
          >
            <WeatheCard :city="city" @select-card="handleSelectCard" @click-detail="handleDetail" />
          </BaseDashboardCard>
        </div>
      </template>

      <template v-else-if="activeTab === 'extra'">
        <div v-if="filteredExtraList.length === 0" class="apple-state-view">
          <p>검색 결과가 없습니다.</p>
        </div>

        <div class="apple-cards-stack">
          <div v-for="item in filteredExtraList" :key="item.id" class="apple-extra-card">
            <div class="extra-header">
              <div style="display: flex; align-items: center; gap: 8px;">
                <WeatherIcon :code="item.icon" size="24" />
                <h3>{{ item.name }}</h3>
              </div>
              <span class="extra-badge">{{ item.isUserLoc ? '내 위치 실시간 분석' : '일조 및 체감 분석' }}</span>
            </div>
            <div class="extra-body">
              <div class="extra-row">
                <span class="label"><UIIcon name="temp" size="14" color="#38bdf8" /> 현재 / 체감</span>
                <span class="value"
                  >{{ formatTemp(item.temp) }} (체감 {{ formatTemp(item.feelsLike) }})</span
                >
              </div>
              <div class="extra-row">
                <span class="label"><UIIcon name="sun-horizon" size="14" color="#fbbf24" /> 일출 시각 (현지)</span>
                <span class="value">{{ item.sunrise }}</span>
              </div>
              <div class="extra-row">
                <span class="label"><UIIcon name="sunset" size="14" color="#f97316" /> 일몰 시각 (현지)</span>
                <span class="value">{{ item.sunset }}</span>
              </div>
              <div class="extra-row">
                <span class="label"><UIIcon name="droplet" size="14" color="#38bdf8" /> 습도 / 기압</span>
                <span class="value">{{ item.humidity }}% / {{ item.pressure }}hPa</span>
              </div>
            </div>
          </div>
        </div>
      </template>

      <template v-else-if="activeTab === 'timeline'">
        <div v-if="filteredTimelineList.length === 0" class="apple-state-view">
          <p>검색 결과가 없습니다.</p>
        </div>

        <div class="apple-cards-stack">
          <div
            v-for="city in filteredTimelineList"
            :key="city.id"
            class="apple-timeline-container-card"
          >
            <div class="extra-header">
              <h3>{{ city.name }}</h3>
              <span class="extra-badge">{{ city.isUserLoc ? '내 위치 3시간 예보' : '3시간 단위 미래 예보' }}</span>
            </div>
            <div class="fit-tl-grid">
              <div v-for="(hour, idx) in city.hourly" :key="idx" class="fit-tl-box">
                <span class="fit-time">{{ hour.time }}</span>
                <WeatherIcon :code="hour.icon" size="38" />
                <span class="fit-temp">{{ formatTemp(hour.temp) }}</span>
                <span class="fit-desc">{{ hour.description }}</span>
              </div>
            </div>
          </div>
        </div>
      </template>

      <div class="apple-bottom-bar">
        <span class="apple-dot"></span>
        <span class="apple-status-msg">{{ selectedCityInfo }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.apple-weather-wrapper {
  width: 100%;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  padding: 24px 0 60px;
  transition: background 1.2s ease-in-out;
}

.apple-weather-app {
  width: min(100%, 760px);
  min-height: 100vh;
  padding: 24px 18px 28px;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', sans-serif;
  color: #f8fafc;
}

.apple-header {
  margin-bottom: 20px;
  padding: 0 4px;
}

.apple-header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.apple-header-badges {
  display: flex;
  align-items: center;
  gap: 8px;
}

.apple-location,
.apple-pill {
  display: inline-flex;
  align-items: center;
  border-radius: 999px;
  padding: 6px 10px;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.02em;
  background: rgba(15, 23, 42, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(18px);
}

.apple-location {
  color: #93c5fd;
}

.apple-pill {
  color: #bae6fd;
}

.geo-btn {
  background: rgba(56, 189, 248, 0.18);
  border: 1px solid rgba(56, 189, 248, 0.35);
  color: #7dd3fc;
  font-size: 11px;
  font-weight: 700;
  padding: 6px 12px;
  border-radius: 999px;
  cursor: pointer;
  transition: all 0.2s ease;
  backdrop-filter: blur(18px);
}

.geo-btn:hover {
  background: rgba(56, 189, 248, 0.35);
  color: #ffffff;
}

.apple-time {
  font-size: 13px;
  font-weight: 600;
  color: #e0f2fe;
  font-variant-numeric: tabular-nums;
}

.apple-title-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 8px;
}

.apple-title {
  font-size: 40px;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.04em;
  margin: 0;
  line-height: 1.1;
}

.apple-unit-pill {
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid rgba(255, 255, 255, 0.16);
  color: #38bdf8;
  font-weight: 700;
  font-size: 13px;
  padding: 8px 14px;
  border-radius: 999px;
  cursor: pointer;
  backdrop-filter: blur(18px);
  transition: background 0.2s ease;
}

.apple-unit-pill:hover {
  background: rgba(30, 41, 59, 0.85);
}

.apple-subtitle {
  font-size: 14px;
  color: #94a3b8;
  margin: 0;
}

.menu-tab-bar {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
}

.tab-btn {
  flex: 1;
  padding: 10px 14px;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(15, 23, 42, 0.5);
  color: #94a3b8;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  backdrop-filter: blur(16px);
  transition: all 0.25s ease;
}

.tab-btn.active {
  background: #2563eb;
  color: #ffffff;
  border-color: #3b82f6;
  box-shadow: 0 4px 14px rgba(37, 99, 235, 0.4);
}

.apple-search-box {
  margin-bottom: 24px;
}

.apple-cards-stack {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.apple-extra-card,
.apple-timeline-container-card {
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 24px;
  padding: 20px;
  backdrop-filter: blur(24px);
  box-shadow: 0 12px 28px rgba(2, 6, 23, 0.2);
}

.extra-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.extra-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 800;
}

.extra-badge {
  font-size: 11px;
  font-weight: 700;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.15);
  padding: 4px 10px;
  border-radius: 999px;
  border: 1px solid rgba(56, 189, 248, 0.25);
}

.extra-body {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.extra-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  border-radius: 12px;
  background: rgba(30, 41, 59, 0.4);
}

.label {
  font-size: 13px;
  color: #cbd5e1;
}

.value {
  font-size: 14px;
  font-weight: 700;
  color: #f8fafc;
}

.fit-tl-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 8px;
}

@media (max-width: 640px) {
  .fit-tl-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.fit-tl-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 12px 6px;
  border-radius: 16px;
  min-height: 120px;
  text-align: center;
  color: #f8fafc;
  background: rgba(15, 23, 42, 0.85);
  border: 1px solid rgba(148, 163, 184, 0.16);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.04);
}

.fit-time {
  font-size: 11px;
  color: #93c5fd;
  font-weight: 700;
  margin-bottom: 4px;
}

.fit-temp {
  font-size: 15px;
  font-weight: 800;
  margin-top: 4px;
}

.fit-desc {
  font-size: 11px;
  color: #cbd5e1;
  line-height: 1.3;
}

.apple-bottom-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 20px;
  padding: 12px 14px;
  border-radius: 999px;
  background: rgba(15, 23, 42, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #cbd5e1;
  font-size: 12px;
  backdrop-filter: blur(20px);
}

.apple-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #38bdf8;
  box-shadow: 0 0 0 4px rgba(56, 189, 248, 0.16);
}

.apple-state-view {
  text-align: center;
  padding: 40px;
  background: rgba(15, 23, 42, 0.5);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.apple-spinner {
  width: 28px;
  height: 28px;
  margin: 0 auto 12px;
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
</style>
