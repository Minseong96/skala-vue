<script setup>
defineProps({
  modelValue: String,
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <div class="search-shell">
    <label class="search-label">도시 검색</label>
    <input
      type="text"
      :value="modelValue"
      @input="emit('update:modelValue', $event.target.value)"
      placeholder="서울, 부산, 대구 또는 도시 이름"
    />
    <p class="search-hint">검색 중인 도시: <strong>{{ modelValue || '전체 조회 중' }}</strong></p>
  </div>
</template>

<style scoped>
.search-shell {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.search-label {
  font-size: 12px;
  font-weight: 700;
  color: #cbd5e1;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}

input {
  width: 100%;
  padding: 12px 14px;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(15, 23, 42, 0.7);
  color: #f8fafc;
  outline: none;
  box-sizing: border-box;
}

input::placeholder {
  color: #94a3b8;
}

.search-hint {
  margin: 0;
  font-size: 13px;
  color: #94a3b8;
}

.search-hint strong {
  color: #f8fafc;
}
</style>
