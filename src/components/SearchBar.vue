<script setup>
defineProps({
  modelValue: String,
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <div
    style="
      margin-bottom: 20px;
      background: #ffffff;
      padding: 15px;
      border-radius: 8px;
      color: #333;
      border: 1px solid #ddd;
    "
  >
    <label style="font-weight: bold; color: #333">도시 검색 (한글 즉시 동기화)</label><br />
    <input
      type="text"
      :value="modelValue"
      @input="emit('update:modelValue', $event.target.value)"
      placeholder="검색할 도시 이름 입력"
      style="
        padding: 8px;
        width: 100%;
        margin-top: 5px;
        box-sizing: border-box;
        background: #fff;
        color: #333;
        border: 1px solid #ccc;
      "
    />
    <p style="margin-top: 5px; font-size: 14px; color: #555">
      검색 중인 도시: <strong>{{ modelValue || '전체 조회 중' }}</strong>
    </p>
  </div>
</template>

<style scoped></style>
