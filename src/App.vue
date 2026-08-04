<!-- src/App.vue -->
<script setup>
import { ref } from 'vue'
import { RouterLink, RouterView, useRouter } from 'vue-router'
import UnitToggler from '@/components/exercise/UnitToggler.vue'
import UIIcon from '@/components/exercise/UIIcon.vue'
import WeatherIcon from '@/components/exercise/WeatherIcon.vue'

const router = useRouter()
const isAboutModalOpen = ref(false)

const openAboutModal = () => {
  isAboutModalOpen.value = true
}

const closeAboutModal = () => {
  isAboutModalOpen.value = false
}
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <div class="header-left">
        <h2 class="header-title" @click="router.push('/')">Weather Studio</h2>
        <nav class="header-nav">
          <RouterLink to="/" class="nav-link">날씨 대시보드</RouterLink>
          <span class="divider">/</span>
          <button class="nav-link-btn" @click="openAboutModal">서비스 소개</button>
        </nav>
      </div>
      <UnitToggler />
    </header>

    <main class="main-content">
      <RouterView />
    </main>

    <!-- 🌟 Apple Glassmorphic Service Intro Modal Dialog -->
    <Transition name="modal-fade">
      <div v-if="isAboutModalOpen" class="about-modal-backdrop" @click.self="closeAboutModal">
        <div class="about-modal-content">
          <button class="modal-close-btn" @click="closeAboutModal">✕</button>

          <div class="modal-badge">
            <UIIcon name="globe" size="14" color="#38bdf8" />
            <span>Weather Studio v2.0 · Platform Overview</span>
          </div>

          <h2 class="modal-title">서비스 소개 및 기술 스펙</h2>
          <p class="modal-subtitle">
            애플(Apple) iOS 날씨 앱의 감성과 최첨단 Web Frontend 기술을 결합한 실시간 기상 대시보드
            플랫폼입니다.
          </p>

          <div class="modal-features-grid">
            <div class="m-card">
              <WeatherIcon code="01d" size="32" />
              <h4>실시간 글로벌 관측망</h4>
              <p>OpenWeather API 기반 서울, 성남, 도쿄, 파리, 모스크바 24시간 예보 분석</p>
            </div>
            <div class="m-card">
              <UIIcon name="target" size="28" color="#7dd3fc" />
              <h4>Geolocation 위치 자동 탐지</h4>
              <p>접속 GPS 좌표를 감지하여 내 동네 날씨를 대시보드 및 사이드 패널 최상단 세팅</p>
            </div>
            <div class="m-card">
              <UIIcon name="location" size="28" color="#38bdf8" />
              <h4>구글 맵 (Google Maps) 연동</h4>
              <p>도시 선택 시 GPS 정밀 타일을 구글 맵 지도로 시각화하여 실시간 위치 표출</p>
            </div>
            <div class="m-card">
              <UIIcon name="sun-horizon" size="28" color="#fbbf24" />
              <h4>3D 패럴랙스 틸트 & FX</h4>
              <p>3D 마우스 틸트 효과와 햇빛/별빛/빗방울/구름 4가지 대기 이펙트 레이어 연동</p>
            </div>
          </div>

          <div class="modal-tech-stack">
            <span class="m-tech">Vue 3.5</span>
            <span class="m-tech">Vite 8</span>
            <span class="m-tech">Pinia</span>
            <span class="m-tech">Vue Router</span>
            <span class="m-tech">Google Maps API</span>
            <span class="m-tech">OpenWeather API</span>
          </div>

          <div class="modal-action-row">
            <button class="modal-primary-btn" @click="closeAboutModal">
              <span>대시보드로 돌아가기</span>
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background: linear-gradient(180deg, #1b365d 0%, #0d1b2a 50%, #050b14 100%);
  color: #f8fafc;
  font-family:
    -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Segoe UI', Roboto,
    sans-serif;
  -webkit-font-smoothing: antialiased;
}
</style>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(180deg, #1b365d 0%, #0d1b2a 50%, #050b14 100%);
  color: #f8fafc;
  display: flex;
  flex-direction: column;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 24px;
  background: rgba(2, 6, 23, 0.55);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.12);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.header-title {
  color: #f8fafc;
  margin: 0;
  font-size: 16px;
  font-weight: 800;
  letter-spacing: -0.02em;
  cursor: pointer;
  transition: color 0.2s ease;
}

.header-title:hover {
  color: #38bdf8;
}

.header-nav {
  display: flex;
  align-items: center;
  gap: 8px;
}

.nav-link,
.nav-link-btn {
  color: #94a3b8;
  font-weight: 600;
  font-size: 13px;
  text-decoration: none;
  padding: 5px 12px;
  border-radius: 999px;
  background: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
}

.nav-link:hover,
.nav-link-btn:hover {
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.12);
  border-color: rgba(56, 189, 248, 0.2);
}

.nav-link.router-link-exact-active {
  color: #ffffff;
  background: rgba(37, 99, 235, 0.65);
  border-color: rgba(56, 189, 248, 0.4);
  box-shadow: 0 2px 10px rgba(37, 99, 235, 0.35);
}

.divider {
  color: #475569;
  font-size: 12px;
}

.main-content {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  width: 100%;
}

/* 🌟 ABOUT SERVICE MODAL OVERLAY STYLES */
.about-modal-backdrop {
  position: fixed;
  inset: 0;
  z-index: 999;
  background: rgba(2, 6, 23, 0.75);
  backdrop-filter: blur(16px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.about-modal-content {
  position: relative;
  width: min(100%, 640px);
  background: rgba(15, 23, 42, 0.88);
  border: 1px solid rgba(255, 255, 255, 0.16);
  border-radius: 26px;
  padding: 32px 28px 28px;
  backdrop-filter: blur(30px);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5);
  color: #f8fafc;
}

.modal-close-btn {
  position: absolute;
  top: 18px;
  right: 20px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.18);
  color: #ffffff;
  font-size: 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.modal-close-btn:hover {
  background: rgba(244, 63, 94, 0.4);
  border-color: rgba(244, 63, 94, 0.6);
}

.modal-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: rgba(56, 189, 248, 0.15);
  border: 1px solid rgba(56, 189, 248, 0.3);
  color: #7dd3fc;
  font-size: 11px;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 999px;
  margin-bottom: 12px;
}

.modal-title {
  margin: 0 0 6px;
  font-size: 24px;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.02em;
}

.modal-subtitle {
  margin: 0 0 20px;
  font-size: 13.5px;
  color: #94a3b8;
  line-height: 1.5;
}

.modal-features-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-bottom: 20px;
}

@media (max-width: 520px) {
  .modal-features-grid {
    grid-template-columns: 1fr;
  }
}

.m-card {
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  padding: 14px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.m-card h4 {
  margin: 4px 0 2px;
  font-size: 14px;
  font-weight: 700;
  color: #ffffff;
}

.m-card p {
  margin: 0;
  font-size: 11.5px;
  color: #cbd5e1;
  line-height: 1.4;
}

.modal-tech-stack {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 24px;
}

.m-tech {
  background: rgba(56, 189, 248, 0.1);
  border: 1px solid rgba(56, 189, 248, 0.2);
  color: #93c5fd;
  font-size: 11px;
  font-weight: 600;
  padding: 3px 9px;
  border-radius: 999px;
}

.modal-action-row {
  display: flex;
  justify-content: flex-end;
}

.modal-primary-btn {
  width: 100%;
  padding: 12px;
  border-radius: 14px;
  border: 1px solid rgba(56, 189, 248, 0.4);
  background: linear-gradient(135deg, #2563eb 0%, #0e7490 100%);
  color: #ffffff;
  font-size: 13.5px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 14px rgba(37, 99, 235, 0.35);
  transition: all 0.2s ease;
}

.modal-primary-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px rgba(37, 99, 235, 0.5);
}

/* Modal Transition */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.25s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}
</style>
