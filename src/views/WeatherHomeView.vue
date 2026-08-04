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
  { id: 'city_tokyo', name: '도쿄', lat: 35.6762, lon: 139.6503 },
  { id: 'city_paris', name: '파리', lat: 48.8566, lon: 2.3522 },
  { id: 'city_moscow', name: '모스크바', lat: 55.7558, lon: 37.6173 },
]

const weatherList = ref([])
const extraWeatherList = ref([])
const timelineWeatherList = ref([])
const selectedCity = ref(null)

const activeTab = ref('weather')
const searchQuery = ref('')
const selectedCityInfo = ref('도시 카드를 선택하시면 우측 패널에 상세 날씨 분석이 표시됩니다.')
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
  if (value === undefined || value === null) return '-'
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
      const forecast = forecastResponses[index].data
      const tz = forecast.city.timezone
      const hourly = forecast.list.slice(0, 8).map((item) => ({
        time: formatCityHour(item.dt, tz),
        temp: Math.round(item.main.temp),
        description: item.weather[0].description,
        icon: item.weather[0].icon,
      }))

      return {
        id: citiesMeta[index].id,
        name: citiesMeta[index].name,
        temp: Math.round(data.main.temp),
        tempMax: Math.round(data.main.temp_max),
        tempMin: Math.round(data.main.temp_min),
        feelsLike: Math.round(data.main.feels_like),
        humidity: data.main.humidity,
        pressure: data.main.pressure,
        windSpeed: data.wind?.speed || 0,
        windDeg: data.wind?.deg || 0,
        sunrise: formatCityTime(data.sys.sunrise, data.timezone),
        sunset: formatCityTime(data.sys.sunset, data.timezone),
        localTime: formatCityTime(data.dt, data.timezone),
        status: data.weather[0].description,
        icon: data.weather[0].icon,
        lat: citiesMeta[index].lat,
        lon: citiesMeta[index].lon,
        hourly,
      }
    })

    extraWeatherList.value = weatherList.value
    timelineWeatherList.value = weatherList.value

    if (!selectedCity.value && weatherList.value.length > 0) {
      selectedCity.value = weatherList.value[0]
      selectedCityInfo.value = `기본 선택 도시: [ ${weatherList.value[0].name} ]`
    }

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

        const hourly = forecastData.list.slice(0, 8).map((item) => ({
          time: formatCityHour(item.dt, tz),
          temp: Math.round(item.main.temp),
          description: item.weather[0].description,
          icon: item.weather[0].icon,
        }))

        const userGeoWeatherItem = {
          id: geoId,
          name: geoName,
          temp: Math.round(data.main.temp),
          tempMax: Math.round(data.main.temp_max),
          tempMin: Math.round(data.main.temp_min),
          feelsLike: Math.round(data.main.feels_like),
          humidity: data.main.humidity,
          pressure: data.main.pressure,
          windSpeed: data.wind?.speed || 0,
          windDeg: data.wind?.deg || 0,
          sunrise: formatCityTime(data.sys.sunrise, data.timezone),
          sunset: formatCityTime(data.sys.sunset, data.timezone),
          localTime: formatCityTime(data.dt, data.timezone),
          status: data.weather[0].description,
          icon: data.weather[0].icon,
          lat: latitude,
          lon: longitude,
          isUserLoc: true,
          hourly,
        }

        weatherList.value = [
          userGeoWeatherItem,
          ...weatherList.value.filter((item) => item.id !== geoId),
        ]
        extraWeatherList.value = weatherList.value
        timelineWeatherList.value = weatherList.value

        // Auto select user location as initial active side panel
        selectedCity.value = userGeoWeatherItem
        selectedCityInfo.value = `현재 위치 실시간 세팅: [ ${geoName} ]`
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

// 🌟 Dynamic Atmosphere Theme Gradient based on Selected City
const currentThemeGradient = computed(() => {
  const target = selectedCity.value || weatherList.value[0]
  if (!target) return 'radial-gradient(ellipse at 50% 0%, #1e293b 0%, #0f172a 60%, #020617 100%)'

  const status = (target.status || '').toLowerCase()
  const icon = target.icon || ''

  if (icon.endsWith('n')) {
    return 'radial-gradient(ellipse at 50% 0%, rgba(30, 27, 75, 0.95) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  if (status.includes('비') || status.includes('소나기') || status.includes('뇌우')) {
    return 'radial-gradient(ellipse at 50% 0%, rgba(14, 116, 144, 0.85) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  if (status.includes('구름') || status.includes('흐림') || status.includes('안개')) {
    return 'radial-gradient(ellipse at 50% 0%, rgba(71, 85, 105, 0.85) 0%, rgba(15, 23, 42, 0.95) 55%, #020617 100%)'
  }
  return 'radial-gradient(ellipse at 50% 0%, rgba(217, 119, 6, 0.65) 0%, rgba(30, 27, 75, 0.85) 50%, #020617 100%)'
})

const handleSelectCard = (cityObj) => {
  if (cityObj && typeof cityObj === 'object') {
    selectedCity.value = cityObj
    selectedCityInfo.value = `선택된 도시 퀵 브리핑: [ ${cityObj.name} ]`
  }
}

const handleDetail = (cityId, cityObj) => {
  const target = cityObj || selectedCity.value
  if (target && (target.lat || target.isUserLoc)) {
    router.push({
      path: `/weather/${cityId}`,
      query: {
        name: target.name,
        lat: target.lat,
        lon: target.lon,
      },
    })
  } else {
    router.push(`/weather/${cityId}`)
  }
}

const getCardBorder = (temp) => {
  if (temp === undefined || temp === null) return 'rgba(255, 255, 255, 0.2)'
  if (temp >= 33) return 'rgba(244, 63, 94, 0.85)'
  if (temp >= 28) return 'rgba(249, 115, 22, 0.85)'
  if (temp >= 20) return 'rgba(56, 189, 248, 0.6)'
  if (temp >= 10) return 'rgba(99, 102, 241, 0.65)'
  return 'rgba(168, 85, 247, 0.75)'
}

const getCardBackground = (status = '', icon = '') => {
  const label = status.toLowerCase()
  if (icon.endsWith('n')) {
    return 'linear-gradient(135deg, rgba(27, 38, 59, 0.95) 0%, rgba(13, 21, 39, 0.95) 60%, rgba(6, 11, 20, 0.98) 100%)'
  }
  if (label.includes('비') || label.includes('소나기') || label.includes('뇌우')) {
    return 'linear-gradient(135deg, rgba(36, 48, 66, 0.95) 0%, rgba(22, 32, 46, 0.95) 60%, rgba(11, 17, 26, 0.98) 100%)'
  }
  if (label.includes('구름') || label.includes('흐림') || label.includes('안개')) {
    return 'linear-gradient(135deg, rgba(61, 74, 93, 0.95) 0%, rgba(40, 51, 66, 0.95) 60%, rgba(24, 34, 48, 0.98) 100%)'
  }
  return 'linear-gradient(135deg, rgba(79, 150, 216, 0.95) 0%, rgba(53, 126, 189, 0.95) 50%, rgba(30, 91, 166, 0.98) 100%)'
}
</script>

<template>
  <div class="apple-weather-wrapper" :style="{ background: currentThemeGradient }">
    <div class="apple-weather-app">
      <!-- Top Navigation Header -->
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
          <h1 class="apple-title">날씨 대시보드</h1>
          <button class="apple-unit-pill" @click="configStore.toggleUnit()">
            {{ configStore.unit === 'fahrenheit' ? '°F' : '°C' }} ·
            {{ configStore.unit === 'fahrenheit' ? '화씨' : '섭씨' }}
          </button>
        </div>
        <p class="apple-subtitle">
          애플 웨더의 감성을 담은 데스크탑 스마트 반응형 사이드 패널 기상 대시보드입니다.
        </p>
      </header>

      <!-- Main Split Layout Container (Left Dashboard + Right Side Panel) -->
      <div class="apple-dashboard-layout">
        <!-- 👈 MAIN DASHBOARD AREA (LEFT) -->
        <main class="apple-main-content">
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
                  background: getCardBackground(city.status, city.icon),
                  borderColor: getCardBorder(city.temp),
                  boxShadow: selectedCity?.id === city.id
                    ? `0 0 0 2px #38bdf8, 0 0 24px ${getCardBorder(city.temp)}`
                    : city.isUserLoc ? `0 0 16px ${getCardBorder(city.temp)}` : undefined,
                }"
                :class="['apple-glass-card', { active: selectedCity?.id === city.id }]"
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
        </main>

        <!-- 👉 DESKTOP SIDE PANEL (RIGHT SIDEBAR) -->
        <aside class="apple-side-panel">
          <div v-if="selectedCity" class="side-panel-glass">
            <!-- Side Panel Hero Weather Section -->
            <div class="side-hero-card">
              <div class="side-hero-top">
                <WeatherIcon :code="selectedCity.icon || '01d'" size="56" />
                <span v-if="selectedCity.isUserLoc" class="side-user-tag">📍 내 위치</span>
              </div>
              <h2 class="side-city-title">{{ selectedCity.name }}</h2>
              <div class="side-big-temp">{{ formatTemp(selectedCity.temp) }}</div>
              <p class="side-status-desc">{{ selectedCity.status }}</p>
              <p class="side-feels-desc">
                체감 {{ formatTemp(selectedCity.feelsLike) }} · 최고 {{ formatTemp(selectedCity.tempMax) }} / 최저 {{ formatTemp(selectedCity.tempMin) }}
              </p>
            </div>

            <!-- Side Panel Hourly Forecast Widget -->
            <div v-if="selectedCity.hourly && selectedCity.hourly.length > 0" class="side-widget-box">
              <div class="side-widget-title">
                <UIIcon name="clock" size="14" color="#38bdf8" />
                <span>시간대별 예보 (현지)</span>
              </div>
              <div class="side-hourly-scroll">
                <div v-for="(h, idx) in selectedCity.hourly" :key="idx" class="side-hourly-item">
                  <span class="side-h-time">{{ h.time }}</span>
                  <WeatherIcon :code="h.icon" size="30" />
                  <span class="side-h-temp">{{ formatTemp(h.temp) }}</span>
                </div>
              </div>
            </div>

            <!-- Side Panel 2x2 Weather Quick Details Grid -->
            <div class="side-grid-widgets">
              <!-- Sunrise & Sunset -->
              <div class="side-mini-widget">
                <div class="side-w-header">
                  <UIIcon name="sun-horizon" size="14" color="#fbbf24" />
                  <span>일출·일몰</span>
                </div>
                <div class="side-w-val-row">
                  <div>
                    <span class="w-sub">일출</span>
                    <span class="w-val">{{ selectedCity.sunrise || '-' }}</span>
                  </div>
                  <div>
                    <span class="w-sub">일몰</span>
                    <span class="w-val">{{ selectedCity.sunset || '-' }}</span>
                  </div>
                </div>
              </div>

              <!-- Humidity & Pressure -->
              <div class="side-mini-widget">
                <div class="side-w-header">
                  <UIIcon name="droplet" size="14" color="#38bdf8" />
                  <span>습도·기압</span>
                </div>
                <div class="side-w-val-row">
                  <div>
                    <span class="w-sub">습도</span>
                    <span class="w-val">{{ selectedCity.humidity }}%</span>
                  </div>
                  <div>
                    <span class="w-sub">기압</span>
                    <span class="w-val">{{ selectedCity.pressure }}hPa</span>
                  </div>
                </div>
              </div>

              <!-- Wind Speed -->
              <div class="side-mini-widget">
                <div class="side-w-header">
                  <UIIcon name="wind" size="14" color="#38bdf8" />
                  <span>바람 관측</span>
                </div>
                <div class="side-w-val-row">
                  <div>
                    <span class="w-sub">풍속</span>
                    <span class="w-val">{{ selectedCity.windSpeed }}m/s</span>
                  </div>
                  <div>
                    <span class="w-sub">풍향</span>
                    <span class="w-val">{{ selectedCity.windDeg }}°</span>
                  </div>
                </div>
              </div>

              <!-- Temp Range -->
              <div class="side-mini-widget">
                <div class="side-w-header">
                  <UIIcon name="temp" size="14" color="#f97316" />
                  <span>기온 범위</span>
                </div>
                <div class="side-w-val-row">
                  <div>
                    <span class="w-sub">최고</span>
                    <span class="w-val">{{ formatTemp(selectedCity.tempMax) }}</span>
                  </div>
                  <div>
                    <span class="w-sub">최저</span>
                    <span class="w-val">{{ formatTemp(selectedCity.tempMin) }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Side Panel Google Map Location Widget -->
            <div v-if="selectedCity.lat && selectedCity.lon" class="side-map-widget">
              <div class="side-widget-title" style="margin-bottom: 8px;">
                <UIIcon name="location" size="14" color="#38bdf8" />
                <span>구글 맵 위치 (Google Maps)</span>
                <span class="map-coord-badge">{{ selectedCity.lat.toFixed(2) }}°, {{ selectedCity.lon.toFixed(2) }}°</span>
              </div>
              <div class="map-iframe-container">
                <iframe
                  :key="`${selectedCity.lat}-${selectedCity.lon}`"
                  class="google-map-iframe"
                  :src="`https://maps.google.com/maps?q=${selectedCity.lat},${selectedCity.lon}&hl=ko&z=11&output=embed`"
                  loading="lazy"
                  allowfullscreen
                  referrerpolicy="no-referrer-when-downgrade"
                ></iframe>
              </div>
            </div>

            <!-- Full Detail Page Navigation Button -->
            <button class="side-detail-btn" @click="handleDetail(selectedCity.id, selectedCity)">
              <span>전체 상세 분석 페이지 이동</span>
              <span class="arrow">›</span>
            </button>
          </div>

          <!-- Fallback when no city selected -->
          <div v-else class="side-panel-placeholder">
            <UIIcon name="globe" size="32" color="#94a3b8" />
            <p>대시보드에서 도시 카드를 클릭하면 퀵 상세 패널이 표시됩니다.</p>
          </div>
        </aside>
      </div>

      <!-- Bottom Status Bar -->
      <footer class="apple-bottom-bar">
        <span class="apple-dot"></span>
        <span class="apple-status-msg">{{ selectedCityInfo }}</span>
      </footer>
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
  width: min(100%, 1280px);
  min-height: 100vh;
  padding: 24px 20px 28px;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', sans-serif;
  color: #f8fafc;
}

.apple-header {
  margin-bottom: 24px;
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
  font-size: 38px;
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

/* 🌟 DASHBOARD SPLIT GRID LAYOUT (Main + Side Panel) */
.apple-dashboard-layout {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

@media (min-width: 1024px) {
  .apple-dashboard-layout {
    display: grid;
    grid-template-columns: 1fr 390px;
    align-items: start;
  }
}

.apple-main-content {
  width: 100%;
  display: flex;
  flex-direction: column;
}

/* Tab Bar */
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

/* Extra & Timeline Cards */
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

/* 🌟 SIDEBAR QUICK DETAIL PANEL */
.apple-side-panel {
  width: 100%;
}

@media (min-width: 1024px) {
  .apple-side-panel {
    position: sticky;
    top: 24px;
    align-self: start;
  }
}

.side-panel-glass {
  background: rgba(15, 23, 42, 0.72);
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 26px;
  padding: 22px;
  backdrop-filter: blur(28px);
  box-shadow: 0 20px 40px rgba(2, 6, 23, 0.4);
  display: flex;
  flex-direction: column;
  gap: 18px;
}

/* Side Hero Section */
.side-hero-card {
  text-align: center;
  padding: 14px 10px 18px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.side-hero-top {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  margin-bottom: 6px;
}

.side-user-tag {
  position: absolute;
  top: 0;
  right: 10px;
  font-size: 10px;
  font-weight: 700;
  color: #7dd3fc;
  background: rgba(56, 189, 248, 0.2);
  border: 1px solid rgba(56, 189, 248, 0.35);
  padding: 2px 8px;
  border-radius: 999px;
}

.side-city-title {
  margin: 4px 0 2px;
  font-size: 24px;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.02em;
}

.side-big-temp {
  font-size: 52px;
  font-weight: 300;
  color: #ffffff;
  line-height: 1;
  margin: 6px 0;
  letter-spacing: -0.04em;
}

.side-status-desc {
  margin: 0 0 6px;
  font-size: 15px;
  font-weight: 600;
  color: #e2e8f0;
}

.side-feels-desc {
  margin: 0;
  font-size: 12px;
  color: #94a3b8;
}

/* Side Widget Box (Hourly) */
.side-widget-box {
  background: rgba(30, 41, 59, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  padding: 14px;
}

.side-widget-title {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  font-weight: 700;
  color: #93c5fd;
  margin-bottom: 12px;
}

.side-hourly-scroll {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 6px;
}

.side-hourly-scroll::-webkit-scrollbar {
  height: 4px;
}
.side-hourly-scroll::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 999px;
}

.side-hourly-item {
  flex: 0 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 8px;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.6);
  min-width: 58px;
}

.side-h-time {
  font-size: 10px;
  color: #cbd5e1;
  font-weight: 600;
}

.side-h-temp {
  font-size: 13px;
  font-weight: 700;
  color: #ffffff;
}

/* Side 2x2 Grid Widgets */
.side-grid-widgets {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.side-mini-widget {
  background: rgba(30, 41, 59, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  padding: 12px;
}

.side-w-header {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
  font-weight: 700;
  color: #cbd5e1;
  margin-bottom: 8px;
}

.side-w-val-row {
  display: flex;
  justify-content: space-between;
}

/* Google Map Location Widget */
.side-map-widget {
  background: rgba(30, 41, 59, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.map-coord-badge {
  font-size: 10px;
  font-weight: 600;
  color: #7dd3fc;
  background: rgba(56, 189, 248, 0.15);
  padding: 2px 7px;
  border-radius: 999px;
  margin-left: auto;
}

.map-iframe-container {
  width: 100%;
  height: 175px;
  border-radius: 14px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.12);
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.25);
}

.google-map-iframe {
  width: 100%;
  height: 100%;
  border: 0;
  filter: contrast(1.05) saturate(1.1);
}

.w-sub {
  display: block;
  font-size: 10px;
  color: #94a3b8;
}

.w-val {
  display: block;
  font-size: 13px;
  font-weight: 700;
  color: #ffffff;
}

/* Navigation CTA Button */
.side-detail-btn {
  width: 100%;
  padding: 12px 16px;
  border-radius: 16px;
  border: 1px solid rgba(56, 189, 248, 0.4);
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.8) 0%, rgba(14, 116, 144, 0.8) 100%);
  color: #ffffff;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 8px 20px rgba(37, 99, 235, 0.3);
  transition: all 0.25s ease;
}

.side-detail-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 24px rgba(37, 99, 235, 0.45);
  border-color: rgba(56, 189, 248, 0.8);
}

.side-detail-btn .arrow {
  font-size: 16px;
}

.side-panel-placeholder {
  background: rgba(15, 23, 42, 0.5);
  border: 1px dashed rgba(255, 255, 255, 0.15);
  border-radius: 24px;
  padding: 40px 20px;
  text-align: center;
  color: #94a3b8;
  font-size: 13px;
}

/* Bottom Bar */
.apple-bottom-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 24px;
  padding: 12px 16px;
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
