<template>
    <div class="fab-container" @mouseenter="expanded = true" @mouseleave="expanded = false">
        <button class="fab-main">+</button>
        <div v-if="expanded" class="fab-options">
            <button @click="emit('loadIfc')">Load IFC</button>
            <button @click="emit('loadFrag')">Load Frag</button>
            <button @click="emit('clearScene')">Clear</button>
            <button @click="$emit('downloadFrag')">Download</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const expanded = ref(false);
const emit = defineEmits(['loadIfc', 'loadFrag', 'clearScene', 'downloadFrag']);
</script>

<style scoped>
.fab-container {
    position: fixed;
    bottom: 20px;
    right: 20px;
    z-index: 1000;
}

.fab-main {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background-color: white;
    color: #002a3a;
    font-size: 24px;
    border: none;
    cursor: pointer;
}

.fab-options {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 10px;
  align-items: flex-end;
}

.fab-options button {
  background: white;
  color: #002a3a;
  padding: 6px 12px;
  border: 1px solid #00a3e0;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
}
</style>