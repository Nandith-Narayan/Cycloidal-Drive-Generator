<script setup>
import DisplayDiskConstruction from './DisplayDiskConstruction.vue'
import DisplayDiskAnimation from './DisplayDiskAnimation.vue'
import InputPanel from './InputPanel.vue'
import { ref } from 'vue'
const params = ref({})
function updateParams(p) {
  params.value = { ...p }
}
</script>

<template>
  <div id="grid">
    <InputPanel @updateParams="updateParams" />
    <DisplayDiskConstruction :params="params" />
    <DisplayDiskAnimation :params="params" />
  </div>
</template>

<style scoped>
#grid {
  display: flex;
  flex-direction: flex;
  flex-wrap: wrap;
  justify-content: center;
  width: 100vw;
  gap: 0.5em;
  background-color: var(--bg-color-primary);
  text-color: var(--fg-color-primary);
}

.card {
  background-color: var(--bg-color-secondary);
  border-radius: 0.5em;
  max-width: 90vw;
  height: auto;
}
</style>
