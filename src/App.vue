<!-- src/App.vue -->
<script setup>
import { RouterLink, RouterView } from 'vue-router'
import UnitToggler from '@/components/exercise/UnitToggler.vue'
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <div class="header-left">
        <h2 class="header-title">Weather Studio</h2>
        <nav class="header-nav">
          <RouterLink to="/" class="nav-link">날씨 대시보드</RouterLink>
          <span class="divider">/</span>
          <RouterLink to="/about" class="nav-link">서비스 소개</RouterLink>
        </nav>
      </div>
      <UnitToggler />
    </header>

    <main class="main-content">
      <RouterView />
    </main>
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
  padding: 16px 24px;
  background: rgba(2, 6, 23, 0.36);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-title {
  color: #f8fafc;
  margin: 0 0 4px 0;
  font-size: 15px;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.header-nav {
  display: flex;
  align-items: center;
  gap: 8px;
}

.nav-link {
  color: #7dd3fc;
  font-weight: 500;
  font-size: 13px;
  text-decoration: none;
  transition: opacity 0.2s;
}

.nav-link:hover {
  opacity: 0.8;
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
</style>
