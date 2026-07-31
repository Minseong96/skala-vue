<template>
  <div
    style="
      border: 1px solid #ddd;
      padding: 15px;
      margin-bottom: 15px;
      border-radius: 8px;
      background: #ffffff;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    "
  >
    <!-- 부모가 넣어주는 내용이 쏙 들어가는 자리 (Slot) -->
    <slot></slot>
  </div>
</template>

<style scoped>
div {
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}
div:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15) !important;
}
</style>
