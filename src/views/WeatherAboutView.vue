<!-- src/views/WeatherAboutView.vue -->
<script setup>
import { useRouter } from 'vue-router'
import { useConfigStore } from '@/stores/config'
import UIIcon from '@/components/exercise/UIIcon.vue'
import WeatherIcon from '@/components/exercise/WeatherIcon.vue'

const router = useRouter()
const configStore = useConfigStore()
</script>

<template>
  <div class="about-wrapper">
    <div class="about-container">
      <!-- Top Navigation & Hero Banner -->
      <header class="about-header">
        <div class="header-badge">
          <UIIcon name="globe" size="14" color="#38bdf8" />
          <span>Weather Studio v2.0 · Platform Overview</span>
        </div>
        <h1 class="about-title">서비스 소개 및 기술 스펙</h1>
        <p class="about-subtitle">
          애플(Apple) iOS 날씨 앱의 고품격 감성과 최첨단 Web Frontend 아키텍처를 결합한 차세대 실시간 기상 대시보드 플랫폼입니다.
        </p>
      </header>

      <!-- Main Feature Grid (2x2) -->
      <div class="features-grid">
        <!-- Feature 1 -->
        <div class="feature-card">
          <div class="card-icon-box">
            <WeatherIcon code="01d" size="36" />
          </div>
          <h3>실시간 글로벌 기상 관측망</h3>
          <p>
            OpenWeather API와의 실시간 연동을 통해 서울, 성남, 도쿄, 파리, 모스크바 등 글로벌 주요 도시의 기온, 습도, 대기압, 풍속 및 24시간 타임라인 예보를 분석합니다.
          </p>
        </div>

        <!-- Feature 2 -->
        <div class="feature-card">
          <div class="card-icon-box">
            <UIIcon name="target" size="32" color="#7dd3fc" />
          </div>
          <h3>HTML5 Geolocation 자동 감지</h3>
          <p>
            사용자의 접속 GPS 위·경도 좌표를 실시간으로 자동 감지하여 사용자가 위치한 현재 지역의 날씨를 대시보드 최상단 및 퀵 브리핑 패널에 최우선 세팅합니다.
          </p>
        </div>

        <!-- Feature 3 -->
        <div class="feature-card">
          <div class="card-icon-box">
            <UIIcon name="location" size="32" color="#38bdf8" />
          </div>
          <h3>구글 맵 (Google Maps) 위치 연동</h3>
          <p>
            도시 선택 시 해당 지역의 GPS 정밀 위·경도 타일을 구글 맵 인터랙티브 지도로 시각화하여 대시보드 및 상세 페이지에서 즉시 지형 위치를 확인할 수 있습니다.
          </p>
        </div>

        <!-- Feature 4 -->
        <div class="feature-card">
          <div class="card-icon-box">
            <UIIcon name="sun-horizon" size="32" color="#fbbf24" />
          </div>
          <h3>3D 마우스 틸트 & 실시간 대기 FX</h3>
          <p>
            3D 시차(Parallax) 마우스 틸트 인터랙션과 함께 맑음(햇빛 플레어), 야간(별빛), 비(빗방울), 흐림(구름 표류) 4가지 고화질 3D 대기 레이어가 동작합니다.
          </p>
        </div>
      </div>

      <!-- Tech Stack Badges Section -->
      <div class="tech-section">
        <h4 class="tech-title">Core Technology Stack</h4>
        <div class="tech-badges">
          <span class="t-badge">Vue 3.5 (Composition API)</span>
          <span class="t-badge">Vite 8.2</span>
          <span class="t-badge">Pinia State Management</span>
          <span class="t-badge">Vue Router 5</span>
          <span class="t-badge">Axios HTTP Client</span>
          <span class="t-badge">OpenWeatherMap API</span>
          <span class="t-badge">Google Maps API</span>
          <span class="t-badge">Apple SF Symbols SVG</span>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="action-row">
        <button class="primary-btn" @click="router.push('/')">
          <UIIcon name="globe" size="16" />
          <span>메인 날씨 대시보드로 이동</span>
          <span class="arrow">›</span>
        </button>

        <button class="secondary-btn" @click="configStore.toggleUnit()">
          <span>섭씨 / 화씨 단위 전환 ({{ configStore.unitSymbol }})</span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.about-wrapper {
  width: 100%;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  padding: 30px 16px 60px;
  background: radial-gradient(ellipse at 50% 0%, #1e293b 0%, #0f172a 60%, #020617 100%);
  color: #f8fafc;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', sans-serif;
}

.about-container {
  width: min(100%, 860px);
  display: flex;
  flex-direction: column;
  gap: 28px;
}

/* Header */
.about-header {
  text-align: center;
  padding: 10px 0 10px;
}

.header-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.12);
  padding: 6px 14px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
  color: #93c5fd;
  backdrop-filter: blur(16px);
  margin-bottom: 12px;
}

.about-title {
  font-size: 36px;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.03em;
  margin: 0 0 10px;
}

.about-subtitle {
  font-size: 15px;
  color: #94a3b8;
  max-width: 640px;
  margin: 0 auto;
  line-height: 1.6;
}

/* Feature Grid */
.features-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

@media (max-width: 640px) {
  .features-grid {
    grid-template-columns: 1fr;
  }
}

.feature-card {
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 22px;
  padding: 22px;
  backdrop-filter: blur(24px);
  box-shadow: 0 12px 28px rgba(2, 6, 23, 0.25);
  display: flex;
  flex-direction: column;
  gap: 10px;
  transition: transform 0.25s ease, border-color 0.25s ease;
}

.feature-card:hover {
  transform: translateY(-3px);
  border-color: rgba(56, 189, 248, 0.4);
}

.card-icon-box {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  background: rgba(30, 41, 59, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 4px;
}

.feature-card h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 700;
  color: #ffffff;
}

.feature-card p {
  margin: 0;
  font-size: 13px;
  color: #cbd5e1;
  line-height: 1.5;
}

/* Tech Stack */
.tech-section {
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 20px;
  text-align: center;
  backdrop-filter: blur(20px);
}

.tech-title {
  margin: 0 0 14px;
  font-size: 13px;
  font-weight: 700;
  color: #38bdf8;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.tech-badges {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
}

.t-badge {
  background: rgba(30, 41, 59, 0.7);
  border: 1px solid rgba(148, 163, 184, 0.2);
  color: #e2e8f0;
  font-size: 12px;
  font-weight: 600;
  padding: 5px 12px;
  border-radius: 999px;
}

/* Actions */
.action-row {
  display: flex;
  gap: 12px;
  justify-content: center;
}

@media (max-width: 640px) {
  .action-row {
    flex-direction: column;
  }
}

.primary-btn {
  padding: 14px 24px;
  border-radius: 16px;
  border: 1px solid rgba(56, 189, 248, 0.4);
  background: linear-gradient(135deg, #2563eb 0%, #0e7490 100%);
  color: #ffffff;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 8px 20px rgba(37, 99, 235, 0.35);
  transition: all 0.25s ease;
}

.primary-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 26px rgba(37, 99, 235, 0.5);
}

.secondary-btn {
  padding: 14px 20px;
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(15, 23, 42, 0.65);
  color: #38bdf8;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  backdrop-filter: blur(16px);
  transition: all 0.2s ease;
}

.secondary-btn:hover {
  background: rgba(30, 41, 59, 0.85);
  color: #ffffff;
}
</style>
