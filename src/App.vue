<script setup lang="ts">
import { ref } from 'vue';

import DiscreteInput from './components/DiscreteInput.vue';

const inputText = ref('');
</script>

<template>
  <img src="/vite.svg" class="logo" alt="Vite logo" />

  <DiscreteInput :mount-input-cell="6" v-model="inputText" />

  <input
    type="text"
    v-model="inputText"
    placeholder="普通输入框"
    style="margin-top: 30px"
  />
</template>

<style lang="scss">
div,
span,
p,
input {
  box-sizing: border-box;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;

  &:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
}
</style>
